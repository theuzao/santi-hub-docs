# ADR 003: Deduplicação por tipo de evento

**Estado:** decisão observada no código. **Fonte:** `live-jogos/server/src/conector/combo.ts`, `server/src/index.ts`; `santi-hub/lib/licencas.ts`, `db/migracoes/001_licencas.sql` nos commits citados no README.

## Contexto

Combos de presentes repetem uma contagem crescente; duas telas podem enviar o mesmo resultado; webhooks de pagamento podem ser reenviados. Esses casos exigem respostas distintas.

## Decisão

O combo usa uma chave de usuário, presente e grupo e emite só `repeatCount` novo. Resultados de rodada são ignorados se chegarem menos de 20 segundos após outro. Webhooks Stripe usam `eventos_stripe.id` único na mesma transação que altera a licença; a sessão de checkout também tem restrição de unicidade.

## Alternativas consideradas

Um ID estável por evento de live e por rodada permitiria deduplicação persistente; uma fila transacional poderia guardar eventos antes do processamento. O código atual não dispõe desses identificadores para todas as ações.

## Consequências

Reenvio do mesmo webhook não duplica a licença. A deduplicação dos jogos é limitada: o mapa de combo some no reinício e a janela de 20 segundos pode descartar resultados legítimos próximos ou aceitar duplicação tardia. Dois presentes independentes recebidos no mesmo milissegundo seguem em sequência no processo e não são descartados pelo limite de comentários.
