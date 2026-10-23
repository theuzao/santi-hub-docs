# Origem das imagens

| Arquivo | Origem | O que comprova |
| --- | --- | --- |
| `site-inicial-2026-10-23.png` | Captura da página pública `https://santiestudio.com.br` em 23/10/2026, navegador Chromium via Playwright, viewport de 1440 × 900 | Aparência da página inicial naquela data |
| `planos-2026-10-23.png` | Captura do elemento `#planos` na mesma página e data, Chromium via Playwright | Planos e preços **exibidos** naquela data; não comprova compra ou pagamento |
| `pescaria-simulador.png` | Cópia de `live-jogos/capturas/pescaria.png` no commit privado `7e65efb42702b5918d499a85a58c9751de997948` | Aparência do jogo no simulador; não comprova transmissão real |

As capturas do site foram feitas após carregar a página e aguardar a saída da tela de carregamento. O procedimento usado foi `page.goto('https://santiestudio.com.br')`, espera de 8 segundos, `page.screenshot()` para a página inicial e `page.locator('#planos').screenshot()` para a seção de planos. A captura do jogo foi copiada sem edição. As imagens não trazem dados de clientes.

Os valores que aparecem na imagem dos planos são um registro visual do site em 23/10/2026. Para valores atuais, abra a [seção de planos](https://santiestudio.com.br/#planos).
