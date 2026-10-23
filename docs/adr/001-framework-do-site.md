# ADR 001: Next.js para site, conta e API

**Estado:** decisão observada no código. **Fonte:** `santi-hub/package.json`, `app/`, `lib/`.

## Contexto

O produto precisa apresentar planos, receber o retorno do checkout, autenticar clientes e expor rotas de conta e webhook.

## Decisão

O site usa Next.js com componentes React e rotas de servidor no mesmo repositório. Regras de preço e acesso ficam no servidor; a página não determina o valor cobrado.

## Alternativas consideradas

Uma interface HTML/React separada de uma API Node.js ou páginas estáticas com serviços externos poderiam cumprir partes desse trabalho. Os repositórios não registram comparação formal entre essas alternativas; elas são possibilidades para explicar a escolha observada, não uma decisão histórica atribuída ao autor.

## Consequências

Página e API compartilham tipos e implantação. A falha do processo do site afeta a conta e o checkout. O jogo em tempo real continua em aplicação local separada, e a ligação entre as duas partes depende do launcher ainda não criado.
