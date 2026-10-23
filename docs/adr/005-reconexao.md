# ADR 005: Reconexão sem fila de reprodução

**Estado:** decisão observada no código. **Fonte:** `live-jogos/server/src/conector/tiktok.ts`, `server/src/ws.ts`, `game/src/core/rede.ts`.

## Contexto

A live, o servidor local e a fonte de navegador podem perder conexão durante uma rodada.

## Decisão

O conector da live agenda novas tentativas com espera crescente de 5 s a 60 s. O cliente WebSocket tenta a cada 2 s. Ao conectar, recebe o placar geral atual. O jogo continua seu cronômetro local enquanto o socket está fechado. O simulador pode continuar gerando eventos no servidor.

## Alternativas consideradas

Uma fila durável com sequência de eventos e retomada após último ID confirmado permitiria reprodução. Pausar a rodada automaticamente também seria possível. O código não registra comparação formal dessas alternativas.

## Consequências

A reconexão recupera o canal e o placar geral, mas não reproduz ações perdidas. Resultado enviado offline é descartado. A integração com live real ainda não foi validada, e o conector depende de uma biblioteca não oficial, incompatível com a exigência futura de coleta apenas por API oficial.
