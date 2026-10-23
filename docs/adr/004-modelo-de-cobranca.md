# ADR 004: Checkout hospedado e licença derivada de webhook

**Estado:** decisão observada no código. **Fonte:** `santi-hub/app/api/checkout/route.ts`, `app/api/stripe/webhook/route.ts`, `lib/licencas.ts`, `lib/conta.ts`.

## Contexto

O produto oferece compra única e assinaturas. Precisa reconhecer pagamento assíncrono, renovação, atraso, cancelamento e reembolso sem confiar no retorno do navegador.

## Decisão

O servidor monta a sessão do Stripe Checkout e calcula preço e modo. O webhook assinado altera licenças no Postgres dentro de transação. O acesso é derivado do estado da licença; assinaturas atrasadas recebem a carência codificada. Um ticket assinado foi implementado na API para uso futuro pelo launcher.

## Alternativas consideradas

Liberação no redirecionamento de sucesso ou valor de preço enviado pelo cliente seriam fluxos mais curtos, mas não dariam uma confirmação confiável de pagamento. Não há registro de comparação comercial entre provedores no repositório.

## Consequências

O retorno ao site pode anteceder a criação da licença. Falha de webhook deixa o acesso pendente até nova tentativa ou intervenção. A liberação dentro do jogo ainda não foi comprovada, pois o launcher e a verificação do ticket no jogo não existem no código analisado.
