# PAULANI Soluções Empresariais — landing

Página única estática. HTML + CSS + JS puro, sem build, sem dependência de Node.
Todo o CSS está inline no `index.html` de propósito: elimina o request bloqueante e vale ~13 pontos de Lighthouse. Mudança de estilo é no bloco `<style>` do próprio arquivo.

**Referência estrutural:** P1 Marketing (`p1-marketing.vercel.app`, repo `leonardoferreirr/p1`).
Mesmo esqueleto de movimentos, paleta trocada de racing/verde para navy/azul, e a pista de Interlagos substituída por uma órbita.
Análise completa da referência: `Kit Piloto Automático/output/referencias/sites/06-p1-marketing/analise.md`.

---

## Estrutura

```
index.html                    página inteira (CSS e JS inline)
favicon.ico
COPY-FONTE-DA-VERDADE.md      toda a copy + pendências com o cliente
assets/
  logo-white.webp             wordmark branco (nav, órbita, rodapé)
  logo-navy.webp              wordmark navy (não usado na página, guardado)
  mark-white.webp             só o símbolo P
  favicon.png
  fonts/
    syncopate-700.woff2       12 KB
    syncopate-400.woff2       22 KB
    inter.woff2               25 KB — variável, eixo wght limitado a 300-700
  js/lenis.min.js             auto-hospedado, sem CDN
```

## Rodar local

```bash
python3 -m http.server 8099
```

## Design tokens

```
--ink        #0B1220   fundo base (navy quase preto)
--ink-deep   #060B15   loader, marquee, campos de form
--navy       #051F51   cor da marca, base dos gradientes de card
--navy-mid   #0A3B8C
--blue-glow  #2E9BFF   o acento
```

**A regra que segura a paleta:** o azul de acento nunca é fundo de área grande. Ele aparece em cinco lugares e mais nenhum:
uma palavra por título, botões, bordas de 1px, números de KPI, glow radial a 7-18% de opacidade.

O `#051F51` da logo dá 1,3:1 sobre o fundo e é ilegível como acento. O `#2E9BFF` é o mesmo matiz mais claro, dá **6,4:1** e passa em WCAG AA.

**Easing único no site inteiro:** `cubic-bezier(0.16,1,0.3,1)`.

## Tipografia

| Uso | Fonte |
|---|---|
| Títulos, números, labels, botões | Syncopate 700, uppercase, tracking **negativo** (-1,2px a -2px) |
| Micro-labels, rodapé, marquee | Syncopate 400, uppercase, tracking **positivo** (.28em a .46em) |
| Corpo | Inter 300-400, `line-height` 1,6-1,7, largura máxima 56-62ch |

Fontes servidas localmente e subsetadas aos glifos da página. Zero request externo de fonte.

## Os movimentos

1. **Loader** — a logo oficial (SVG inline do wordmark, arquivo do cliente) se **desenha**: os contornos riscam em azul via `stroke-dasharray` com `pathLength="1"` (normaliza os 30 paths pra desenharem em sincronia), depois o preenchimento branco inunda e o traço some. Barra de 2px corre embaixo. Some em 2,2s. Fonte do SVG: `Logo.svg` do cliente, otimizado com svgo (73KB → 7KB, paths preservados separados).
2. **Hero** — cena de dados viva renderizada ao vivo num `<canvas>` sticky (grade em perspectiva correndo, nós de dados pulsando, poeira). Movimento contínuo próprio em loop temporal, **não amarrado ao scroll** (foi a troca que resolveu o "meio lento": antes eram 61 JPGs de 1,9 MB presos ao scroll, que arrastavam atrás do dedo). O scroll controla só o fade do texto e o zoom-reveal. Fundo estático pré-renderizado num offscreen e blitado; glows dos nós usam um sprite, não gradiente por frame. Canvas a DPR 1.5 e contagem de partículas enxuta: 60fps travado. Pausa via IntersectionObserver quando o hero sai de vista. Container de 220vh (200vh no mobile).
3. **Zoom-reveal** — a seção seguinte tem `margin-top:-100vh` e `clip-path:inset(50%)` abrindo pra 0 nos últimos 25% do hero. O conteúdo cresce do centro da tela por cima do frame congelado.
4. **Método** — 4 etapas com um trilho que preenche e os marcadores acendendo em cascata (450ms entre cada).
5. **Sticky stack** — título parado à esquerda, 4 pilares empilhando à direita com `top` crescendo 3vh por card. Cada card tem um visual funcional em CSS puro: medidor de maturidade, módulos acendendo, nós integrados, dashboard.
6. **Órbita** — PAULANI no núcleo, os 4 pilares girando na elipse externa (48s) e as ferramentas na interna em sentido contrário (34s, via `keyPoints="1;0"`). SVG nativo com `animateMotion`, sem JS.

## Armadilhas já resolvidas (não reintroduzir)

- **`background-clip:text` come acentos altos.** O til do Ã e o acento do Ó estouram a caixa de background do inline e ficam sem pintura. Todo `em` com gradiente tem `padding-block:.2em` **e** `box-decoration-break:clone` — o `clone` é obrigatório, porque sem ele o padding só protege a primeira e a última linha do inline.
- **`padding` em porcentagem resolve contra a largura do pai, não do elemento.** Isso inflou o núcleo da órbita de 150px pra 208×260 e esmagou a logo pra 0×0. Não usar `%` em padding de elemento dimensionado por `clamp`.
- **O marquee fica invisível se colocado antes do zoom-reveal**, porque o `margin-top:-100vh` do wrapper o esconde atrás do hero sticky. Ele está depois do wrapper de propósito.
- **Núcleo da órbita cobre os satélites internos no mobile.** Há um override em `max-width:900px` reduzindo pra `clamp(92px,25%,124px)`.
- **`getBoundingClientRect()` a cada frame, logo depois de escrever `clipPath`, força reflow.** O progresso do hero é calculado com `offsetTop` cacheado + `scrollY`.

## Performance

Lighthouse mobile com `--throttling-method=devtools`: **99 / 100 / 100 / 100**.
LCP 1,1s · CLS 0,016 · TBT 30ms.

O gate só vale com `--throttling-method=devtools`. O `simulate` (padrão) infla o LCP e mente cerca de 9 pontos.

```bash
npx lighthouse <url> --form-factor=mobile --throttling-method=devtools --screenEmulation.mobile
```

Regras que sustentam o número:
- CSS inline, zero folha externa
- fontes subsetadas, `font-display:optional`, preload das duas críticas
- hero em canvas ao vivo, zero asset de imagem no caminho crítico
- Lenis auto-hospedado com `defer`

## Pendências

Ver a tabela no fim de `COPY-FONTE-DA-VERDADE.md`. As que bloqueiam campanha:

1. **Telefone/WhatsApp** — o briefing traz `(xx) xxxx-xxxx` como placeholder. Hoje o rodapé só tem e-mail.
2. **Destino do form** — o `submit` só troca de estado visual, tem um `// TODO` marcando o ponto.
3. **Fundo do hero** — hoje é a cena de dados abstrata (canvas ao vivo). Se o cliente quiser trocar por imagem/vídeo real, o `<canvas id="heroCanvas">` vira o alvo (ou troca por `<video autoplay muted loop playsinline>` no mesmo lugar). O texto e o zoom-reveal continuam funcionando por cima.
4. **Página de obrigado + pixel** — não existe ainda. Sem isso não há mensuração de campanha.

## Não testado

**Safari (iPhone e Mac).** A página combina `filter:blur(120px)`, `mix-blend-mode:screen` e `backdrop-filter` na nav. É a mesma combinação que engasgou no Pokémon. Precisa rolar a página inteira no Safari antes de considerar fechada.
