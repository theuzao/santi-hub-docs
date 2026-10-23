# ADR 002 — Estado do campeonato no servidor e rodada no navegador

**Estado:** decisão observada no código. **Fonte:** `live-jogos/server/src/estado/placar.ts`, `server/src/index.ts`, `game/src/core/rotacao.ts` no commit `7e65efb42702b5918d499a85a58c9751de997948`.

## Contexto

O jogo precisa manter times e pontos ao trocar minijogos, atualizar a tela a cada ação e sobreviver parcialmente a um reinício do servidor.

## Decisão

O servidor mantém membros, pontos, presenteadores e contagem de rodadas em memória e salva um snapshot em `dados/estado.json`. O navegador mantém cronômetro, fase e estado do minijogo ativo. O servidor manda o placar geral ao novo cliente WebSocket.

## Alternativas consideradas

Persistir cada evento num banco, manter também a rodada no servidor ou usar uma fila de eventos permitiria recuperação mais completa. Não há registro de avaliação formal dessas alternativas no repositório.

## Consequências

O caminho de uma ação até a cena é curto. Uma queda antes da gravação pode perder mudanças recentes; o arquivo local não serve para múltiplas instâncias sem coordenação. Recarregar a tela reinicia a rodada; o placar geral volta do servidor, mas as ações perdidas não são reproduzidas.
