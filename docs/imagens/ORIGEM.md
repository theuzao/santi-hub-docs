# Origem das imagens

As duas capturas vieram do [site público](https://santiestudio.com.br) em 23/10/2026. Foram feitas com Chromium via Playwright, sem edição posterior.

| Arquivo | Captura | O que mostra |
| --- | --- | --- |
| `loading-2026-10-23.png` | `page.screenshot()` durante a animação de abertura, com viewport de 1440 × 900 | Tela de loading publicada naquela data |
| `jogos-2026-10-23.png` | `page.locator('#jogos').screenshot()` após o carregamento da página | Seção de jogos publicada naquela data |

O procedimento abriu `https://santiestudio.com.br` com `page.goto(..., { waitUntil: 'domcontentloaded' })`. A captura do loading foi feita após 1,8 segundo; a seção de jogos foi capturada depois de aguardar a saída do loading. Os estados dos cards podem mudar; para conferir o catálogo atual, abra o [site](https://santiestudio.com.br/#jogos).
