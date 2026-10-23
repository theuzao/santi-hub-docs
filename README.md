# santi.hub

<div align="center">
  <strong>Jogos interativos para lives do TikTok</strong><br>
  O público participa com comentários, curtidas e presentes.<br><br>
  <a href="https://santiestudio.com.br"><strong>Acessar o site</strong></a> ·
  <a href="https://santiestudio.com.br/#jogos">Ver os jogos</a> ·
  <a href="https://santiestudio.com.br/#planos">Ver os planos</a>
</div>

<br>

Este repositório público apresenta o santi.hub por meio de **capturas e documentação**. O código do site e o código dos jogos ficam em repositórios privados separados. O site reúne planos, cobrança e conta; o servidor local recebe interações da live e as envia ao jogo exibido no OBS ou LIVE Studio.

## Telas do produto

**Abertura do site.** Captura da animação de loading publicada em 23/10/2026.

<a href="https://santiestudio.com.br"><img src="docs/imagens/loading-2026-10-23.png" alt="Tela de loading do site com a marca SANTI ESTÚDIO em branco" width="780"></a>

**Catálogo de jogos.** A página mostra Corrida como disponível e identifica os outros jogos como próximos lançamentos.

<a href="https://santiestudio.com.br/#jogos"><img src="docs/imagens/jogos-2026-10-23.png" alt="Seção de jogos do site com Corrida disponível e outros jogos em breve" width="780"></a>

![Next.js](https://img.shields.io/badge/Next.js-000000?style=for-the-badge&logo=nextdotjs&logoColor=white)
![TypeScript](https://img.shields.io/badge/TypeScript-3178C6?style=for-the-badge&logo=typescript&logoColor=white)
![Node.js](https://img.shields.io/badge/Node.js-339933?style=for-the-badge&logo=nodedotjs&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-4169E1?style=for-the-badge&logo=postgresql&logoColor=white)
![Stripe](https://img.shields.io/badge/Stripe-635BFF?style=for-the-badge&logo=stripe&logoColor=white)

## Índice

- [Sobre o projeto](#sobre-o-projeto)
- [Funcionalidades](#funcionalidades)
- [Arquitetura](#arquitetura)
- [Tecnologias](#tecnologias)
- [Estado atual](#estado-atual)
- [Documentação](#documentação)
- [Autor e licença](#autor-e-licença)

## Sobre o projeto

Quem transmite escolhe um plano no site e pode consultar sua conta. A confirmação de pagamento chega por webhook e grava uma licença no Postgres. A API já calcula o acesso e emite um ticket assinado, mas o launcher Windows que ligaria esse acesso à execução do jogo ainda não foi criado.

No jogo, comentar o nome de um time conta como jogada. Curtidas e presentes ajudam o último time escolhido. O servidor interpreta os eventos e envia ações por WebSocket ao navegador capturado pelo OBS. O simulador permite testar esse fluxo sem uma live.

## Funcionalidades

| Área | O que o código faz |
| --- | --- |
| Planos e checkout | O site calcula o preço no servidor e abre uma sessão Stripe Checkout. |
| Conta e licença | O webhook assinado grava a licença; a API calcula o acesso disponível e pode emitir um ticket assinado. |
| Interação | O servidor transforma comentários, curtidas e presentes em ações de jogo. O simulador percorre esse mesmo caminho. |
| Tela da partida | O navegador recebe ações por WebSocket, atualiza o minijogo e envia o resultado da rodada ao servidor. |
| Reconexão | O conector e o cliente WebSocket tentam restabelecer a conexão; ações perdidas durante a queda não são reproduzidas. |

## Arquitetura

### Componentes e integrações

```mermaid
flowchart TB
  subgraph Conta[Site, cobrança e conta]
    Pessoa[Pessoa que transmite] --> Site[Site Next.js]
    Site --> Stripe[Stripe Checkout]
    Stripe -->|webhook| Site
    Site --> Banco[(Postgres: clientes e licenças)]
  end
  subgraph Partida[Jogo na máquina que transmite]
    Publico[Público] --> TikTok[TikTok]
    TikTok --> Servidor[Servidor Node.js]
    Servidor -->|WebSocket| Jogo[Jogo no navegador]
    Jogo --> OBS[OBS ou LIVE Studio]
  end
  Banco -. launcher pendente .-> Servidor
```

A ligação pontilhada ainda não funciona de ponta a ponta. O launcher não existe. O conector de TikTok usa uma biblioteca não oficial, e o repositório dos jogos não registra validação numa live real.

### Interação do público até a tela

```mermaid
flowchart TB
  Evento[Comentário, curtida ou presente] --> Conector[Conector ou simulador]
  Conector --> Regras[Regras e placar no servidor]
  Regras -->|ação em JSON por WebSocket| Tela[Jogo no navegador]
  Tela --> OBS[Tela capturada no OBS]
  Tela -->|resultado da rodada| Regras
```

O servidor aplica as regras e envia a ação ao minijogo ativo. Quando a rodada termina, o navegador devolve o resultado e o servidor atualiza o placar geral.

### Compra até a licença

```mermaid
sequenceDiagram
  actor Pessoa as Pessoa que transmite
  participant Site as Site Next.js
  participant Stripe
  participant Banco as Postgres
  Pessoa->>Site: escolhe um plano
  Site->>Stripe: cria sessão de checkout
  Stripe-->>Pessoa: apresenta checkout
  Stripe->>Site: envia webhook assinado
  Site->>Banco: registra evento e licença na mesma transação
  Banco-->>Site: confirma gravação
  Site-->>Stripe: HTTP 200
```

A página de retorno do checkout não libera o acesso. Se o webhook falhar durante a gravação, a rota responde 500 e a transação é desfeita. O [fluxo de pagamento](docs/fluxo-pagamento.md) detalha reenvio, idempotência e casos de pagamento pendente.

### Onde o estado fica

```mermaid
flowchart TB
  Navegador[Navegador: jogo ativo, timer e rodada]
  Servidor[Node.js: times, pontos gerais e ranking]
  Arquivo[(estado.json: snapshot do placar geral)]
  Site[Site e API: conta e licenças]
  Banco[(Postgres: dados da conta)]
  Navegador <-->|ações e resultados| Servidor
  Servidor -->|grava snapshot| Arquivo
  Site -->|SQL| Banco
```

Na reconexão do WebSocket, a tela recebe o placar geral atual. As ações perdidas não são reenviadas, e uma recarga da página reinicia a rodada local. Veja os limites no [documento de arquitetura](docs/arquitetura.md).

## Tecnologias

| Parte | Tecnologia | Papel |
| --- | --- | --- |
| Site e API | Next.js, React e TypeScript | Páginas, conta, checkout e webhook |
| Dados da conta | PostgreSQL | Clientes, licenças, sessões e eventos Stripe |
| Pagamento | Stripe | Checkout, assinatura e avisos de pagamento |
| Servidor do jogo | Node.js e TypeScript | Interpretação dos eventos, placar e WebSocket |
| Tela do jogo | Phaser e TypeScript | Rodadas, interface e ações visíveis na transmissão |

## Estado atual

| Parte | Estado |
| --- | --- |
| Página pública | Acessível em [santiestudio.com.br](https://santiestudio.com.br) em 23/10/2026; teste de acesso: `curl -IL --max-time 15 https://santiestudio.com.br` |
| Servidor e tela dos jogos | Testados com o simulador; live real não validada na documentação do projeto |
| Launcher Windows | Ainda não criado |
| Liberação do jogo por licença | Ainda não implementada de ponta a ponta |
| Coleta de eventos TikTok | Conector atual usa `tiktok-live-connector`, biblioteca não oficial |

Tráfego, receita, latência e disponibilidade: **[A MEDIR]**. As capturas mostram telas reais, mas não comprovam pagamento concluído nem uso numa live real.

## Documentação

- [Arquitetura](docs/arquitetura.md): componentes, caminho de um evento, estados e reconexão.
- [Modelo de dados](docs/modelo-de-dados.md): tabelas do site e estado dos jogos fora do banco.
- [Fluxo de pagamento](docs/fluxo-pagamento.md): checkout, webhook, licença e falhas.
- [Decisões arquiteturais](docs/adr/): escolhas e consequências observadas no código.

## Autor e licença

[Matheus Guilherme](https://github.com/theuzao). A documentação está sob a [licença MIT](LICENSE).
