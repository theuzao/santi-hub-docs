# Arquitetura do santi.hub

O produto tem **um lado de conta e pagamento** e **um lado de jogo em tempo real**. O primeiro administra quem teria acesso; o segundo recebe interações da live e desenha a partida. O launcher que ligaria os dois lados ainda não foi criado. Os diagramas abaixo separam fluxo implementado de integração pendente.

**Fonte da leitura:** `santi-hub@97ee1f200461bc575f524df66ef363cc9c9e2e8d` e `live-jogos@7e65efb42702b5918d499a85a58c9751de997948`, repositórios privados que o autor identificou como partes do projeto. Linha contínua indica caminho presente no código; linha pontilhada indica ligação ainda sem implementação ponta a ponta.

## 1. Quem interage com o sistema — C4 nível 1

```mermaid
flowchart TB
  Streamer[Pessoa que transmite]
  Publico[Público da live]
  Santi[santi.hub: conta, cobrança e jogos]
  TikTok[TikTok Live]
  Stripe[Stripe]
  Email[Serviço de e-mail]
  OBS[OBS ou LIVE Studio]

  Streamer -->|escolhe plano, entra na conta| Santi
  Publico -->|comenta, curte, envia presentes| TikTok
  TikTok -->|eventos da live: conector não oficial| Santi
  Santi -->|abre checkout e consulta assinatura| Stripe
  Stripe -->|confirma pagamento por webhook| Santi
  Santi -->|envia código de entrada| Email
  Santi -->|tela do jogo capturada| OBS
  OBS -->|vídeo da transmissão| TikTok
  TikTok -->|vídeo| Publico
```

A linha TikTok → santi.hub existe no conector, mas o repositório de jogos declara que ainda não foi testada numa live real. O site publicado foi informado pelo autor; este estudo não mediu operação em produção.

## 2. Onde cada parte roda — C4 nível 2

```mermaid
flowchart TB
  subgraph Web[Site e conta]
    Site[Next.js: páginas e rotas de API]
    DB[(Postgres: clientes, licenças, sessões)]
    Site -->|SQL| DB
  end

  subgraph Local[Máquina que transmite]
    Connector[Conector TikTok ou simulador]
    Server[Node.js: regras, times e placar]
    File[(dados/estado.json)]
    Game[Phaser no navegador: rodada e desenho]
    Connector -->|EventoLive| Server
    Server -->|snapshot agendado| File
    Server <-->|WebSocket JSON| Game
  end

  Stripe[Stripe] <-->|Checkout, consulta e webhook HTTPS| Site
  Email[E-mail] <-->|código de entrada| Site
  TikTok[TikTok Live] -->|biblioteca não oficial; sem validação em live real| Connector
  Game -->|fonte de navegador| OBS[OBS ou LIVE Studio]
  Site -. API e ticket assinado .-> Launcher[Launcher Windows: ainda não criado]
  Launcher -. iniciaria os jogos após conferir o acesso .-> Server
```

Os vínculos pontilhados representam a integração prevista. A API do site emite um ticket que um launcher futuro consumiria; esse cliente iniciaria os jogos após verificar o acesso. Como o launcher não existe e o servidor de jogos não confere licença, os dois blocos devem ser explicados separadamente numa entrevista.

## 3. Um comentário que vira jogada

Exemplo concreto: uma pessoa comenta **“Rabino”**. O simulador permite percorrer o mesmo pipeline sem entrar numa live.

<img src="imagens/pescaria-simulador.png" alt="Captura da Pescaria no simulador, mostrando times, placar e instruções na tela" width="250">

*Tela do jogo capturada no simulador. [Origem da imagem](imagens/ORIGEM.md).*

```mermaid
sequenceDiagram
  autonumber
  actor Pessoa as Pessoa no público
  participant TikTok as Live TikTok ou simulador
  participant Conector as Conector
  participant Regras as Pipeline e placar
  participant WS as Servidor WebSocket
  participant Tela as Jogo no navegador
  participant OBS as OBS

  Pessoa->>TikTok: comenta "Rabino"
  TikTok->>Conector: evento de comentário
  Conector->>Regras: EventoLive {tipo, user, texto}
  Regras->>Regras: interpreta time e aplica limite de 1 jogada por pessoa por segundo
  Regras->>WS: Acao {tipo: entrarTime, time: rabino, user}
  WS->>Tela: JSON {t: acao, acao} por WebSocket
  Tela->>Tela: rotacao entrega a ação ao minijogo ativo
  Tela-->>OBS: quadro atualizado da fonte de navegador
```

**Arquivos que sustentam as setas:** `server/src/conector/tiktok.ts` ou `simulador.ts` → `server/src/pipeline.ts` → `server/src/ws.ts` → `game/src/core/rede.ts` → `game/src/core/rotacao.ts` → cena do minijogo. O tipo comum de mensagem fica em `shared/protocolo.ts`.

Curtidas são atribuídas ao último time comentado pela pessoa. Presentes também; quando ela ainda não escolheu time, o presente aparece como aviso e não age no jogo. Para presentes em combo, `ContadorCombo` calcula o aumento do `repeatCount` cumulativo antes de criar a ação. Dois presentes independentes recebidos no mesmo milissegundo são processados em sequência no processo Node.js: o limite de um segundo vale para **comentários que viram jogada**, não para presentes. O código não oferece ordenação global nem deduplicação durável de todas as entregas da live.

## 4. O fechamento de uma rodada

```mermaid
sequenceDiagram
  autonumber
  participant Tela as Rotação e minijogo no navegador
  participant WS as Servidor WebSocket
  participant Placar as Placar no Node.js
  participant Arquivo as estado.json

  Tela->>Tela: timer termina ou jogo encerra antes
  Tela->>WS: {t: resultado, pontuação por time}
  WS->>Placar: valida formato e registra pontos
  Placar->>Placar: agenda atualização para 300 ms depois
  Placar->>Arquivo: grava estado geral
  Placar-->>WS: snapshot do campeonato
  WS-->>Tela: {t: placar, placar}
  Tela->>Tela: transição e próximo minijogo
```

O servidor ignora um segundo resultado recebido em menos de 20 segundos. Essa janela tenta evitar dupla contagem quando OBS e aba de teste estão abertos, mas não identifica uma rodada por ID; pode ignorar uma segunda rodada legítima muito próxima ou aceitar duplicação tardia.

## 5. Quem guarda cada estado

```mermaid
flowchart LR
  Browser[Navegador: jogo ativo, timer, pontuação da rodada]
  Server[Node.js: times, pontos gerais, ranking, combos e limite de comentários]
  Disk[(estado.json: times, pontos gerais, ranking, número de rodadas)]
  DB[(Postgres: clientes, licenças, login e eventos Stripe)]

  Server -->|grava snapshot agendado| Disk
  Server -->|envia placar ao conectar| Browser
  Browser -->|envia resultado da rodada| Server
```

| Onde | O que acontece numa queda |
| --- | --- |
| Navegador | O jogo segue seu cronômetro durante perda do WebSocket; recarregar a página reinicia a rodada. O resultado enviado offline é perdido. |
| Processo Node.js | Times e placar geral são carregados de `estado.json` após reinício, se o arquivo estiver legível. Mapas de combos e limite de comentários não são restaurados. |
| Arquivo local | A gravação é agendada para 300 ms após a primeira mudança pendente; uma queda antes da gravação pode perder mudanças. `ultimaRodada` não é gravada. |
| Postgres | Mantém conta, licença, sessões, códigos de login e webhooks Stripe; não guarda ações ou rodadas dos jogos. |

## 6. Quando uma conexão cai

```mermaid
flowchart TB
  QuedaLive[Conector perde a live] --> TentativaLive[Tenta reconectar: espera de 5 s até 60 s]
  TentativaLive --> VoltaLive[Se voltar: recebe novos eventos]
  QuedaWS[WebSocket entre servidor e tela cai] --> TelaSegue[Tela segue a rodada local]
  TelaSegue --> TentativaWS[Cliente tenta reconectar após 2 s]
  TentativaWS --> Snapshot[Servidor envia o placar geral atual]
```

Não existe fila durável nem reenvio de ações perdidas. O simulador permanece disponível se a conexão com TikTok cair. Reconectar o WebSocket recupera o placar geral, mas não reconstrói a rodada da tela.

## Evidência e pendências

Os testes do servidor cobrem combo, regras de comentário, pontos e pipeline com simulador (`server/src/regras.test.ts`). O repositório `live-jogos` declara que ainda não houve validação numa live real. O conector usa `tiktok-live-connector`, biblioteca não oficial; por isso essa coleta não pode ser apresentada como uso de API oficial. Latência ponta a ponta, perda em reconexão e carga suportada: **[A MEDIR]**.
