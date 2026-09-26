---
title: Sistema de Recomendação de Ações para Investidores Leigos
status: final
created: 2026-09-05
updated: 2026-09-07
name: Painel Único
description: Painel único de recomendação de ações para investidores leigos — "recibo bancário caloroso" com termômetro de status; credibilidade financeira sem frieza, ansiedade reduzida por padrão.
colors:
  # --- Base (light) — extraída literalmente de direction-merged-recibo-termometro.html ---
  bg: '#FFF8F0'
  card: '#FFFFFF'
  border: '#F0DFC8'
  surface-soft: '#FBF3E7'
  ink: '#3A2E22'
  ink-muted: '#8A7A68'
  overlay: 'rgba(58,46,34,.6)'

  # --- Terracota (acento primário / marca / CTA) ---
  terracotta: '#E0703B'
  terracotta-soft: '#FCE7D8'
  ink-on-terracotta-soft: '#8A4A26'
  # [ASSUMPTION] tom mais escuro, não presente no mockup — necessário porque texto
  # branco sobre --terracotta pura não atinge AA em tamanhos normais (ver Cores).
  terracotta-strong: '#B8562A'

  # --- Musgo (status "favorável") ---
  moss: '#5B8266'
  moss-light: '#7FAF8C'
  moss-soft: '#E7F0E9'
  ink-on-moss-soft: '#3F5F4C'

  # --- Âmbar (status "atenção") ---
  amber: '#D9A441'
  # [ASSUMPTION] --amber-soft e --amber-ink não existem no mockup (só o estado
  # "favorável" foi prototipado). Derivados por analogia ao par terracotta/moss.
  amber-soft: '#FBEDD3'
  amber-ink: '#8A6420'

  # --- Vermelho (status "alerta" / concentração) ---
  red: '#C1503F'
  red-soft: '#FBE4DD'
  # [ASSUMPTION] variante mais escura para texto sobre red-soft (ver Cores — o par
  # red/red-soft do mockup, usado em .warn-box, fica abaixo de AA).
  red-strong: '#9B3F2C'

  # ============================================================
  # [ASSUMPTION] Modo escuro — não existe mockup dark; paleta abaixo é derivada
  # preservando as mesmas famílias de matiz (terracota/musgo/âmbar/vermelho) e
  # validada por contraste (ver Cores). Precisa de confirmação visual humana.
  # ============================================================
  bg-dark: '#211A14'
  card-dark: '#2A2119'
  border-dark: '#3D3226'
  surface-soft-dark: '#332A20'
  ink-dark: '#F5EDE2'
  ink-muted-dark: '#B8A890'
  overlay-dark: 'rgba(0,0,0,.65)'

  terracotta-dark: '#F0916A'
  terracotta-soft-dark: '#3A2418'

  moss-dark: '#8FBFA0'
  moss-soft-dark: '#223026'

  amber-dark: '#E8BE72'
  amber-soft-dark: '#3A2E18'

  red-dark: '#E4897A'
  red-soft-dark: '#3A211C'
typography:
  # Pilha de sistema (extraída do mockup) para toda a UI/texto corrido.
  brand:
    fontFamily: '-apple-system, BlinkMacSystemFont, "Segoe UI", Helvetica, Arial, sans-serif'
    fontSize: 19px
    fontWeight: '800'
  heading:
    fontFamily: '-apple-system, BlinkMacSystemFont, "Segoe UI", Helvetica, Arial, sans-serif'
    fontSize: 20px
    fontWeight: '800'
    lineHeight: '1.2'
  label:
    fontFamily: '-apple-system, BlinkMacSystemFont, "Segoe UI", Helvetica, Arial, sans-serif'
    fontSize: 11px
    fontWeight: '700'
    letterSpacing: 0.07em
  body:
    fontFamily: '-apple-system, BlinkMacSystemFont, "Segoe UI", Helvetica, Arial, sans-serif'
    fontSize: 14px
    fontWeight: '400'
    lineHeight: '1.65'
  meta:
    fontFamily: '-apple-system, BlinkMacSystemFont, "Segoe UI", Helvetica, Arial, sans-serif'
    fontSize: 12.5px
    fontWeight: '400'
    lineHeight: '1.6'
  caption:
    fontFamily: '-apple-system, BlinkMacSystemFont, "Segoe UI", Helvetica, Arial, sans-serif'
    fontSize: 10.5px
    fontWeight: '400'
  action:
    fontFamily: '-apple-system, BlinkMacSystemFont, "Segoe UI", Helvetica, Arial, sans-serif'
    fontSize: 13.5px
    fontWeight: '700'
  # Monoespaçada — reservada a valores numéricos tabulares (moeda, %, indicadores).
  figure-lg:
    fontFamily: 'ui-monospace, SFMono-Regular, Menlo, monospace'
    fontSize: 16.5px
    fontWeight: '700'
  figure-sm:
    fontFamily: 'ui-monospace, SFMono-Regular, Menlo, monospace'
    fontSize: 12.5px
    fontWeight: '700'
rounded:
  sm: 8px
  md: 12px
  lg: 16px
  xl: 20px
  full: 9999px
spacing:
  '1': 4px
  '2': 8px
  '3': 12px
  '4': 16px
  '5': 20px
  '6': 24px
  '7': 32px
  card-gap: 18px
  card-padding: 22px 24px
  chip-padding: 7px 14px
  gate-padding: 26px 26px 24px
  content-margin: 32px
components:
  card:
    background: '{colors.card}'
    border: '1px solid {colors.border}'
    radius: '{rounded.lg}'
    padding: '{spacing.card-padding}'
    shadow: '0 8px 20px -14px rgba(58,46,34,.4)'
  receipt-row:
    divider: '1px dashed {colors.border}'
    label: '{typography.body}'
    sublabel: '{typography.meta}'
    value: '{typography.figure-lg}'
    value-diff-color: '{colors.terracotta}'
  disclaimer-pill:
    background: '{colors.terracotta-soft}'
    foreground: '{colors.ink-on-terracotta-soft}'
    radius: '{rounded.sm}'
    typography: '{typography.meta}'
  thermometer:
    wrap-background: '{colors.moss-soft}'
    wrap-radius: '{rounded.xl}'
    tube-background: '{colors.card}'
    tube-radius: '{rounded.md}'
    tube-shadow-inset: 'inset 0 2px 5px rgba(58,46,34,.1)'
    bulb-radius: '{rounded.full}'
    status-chip-background: '{colors.card}'
    status-chip-radius: '{rounded.xl}'
    state-favorable: '{colors.moss}'
    state-attention: '{colors.amber-ink}'
    state-alert: '{colors.red}'
  confirm-chip:
    background: '{colors.moss-soft}'
    foreground: '{colors.ink-on-moss-soft}'
    radius: '{rounded.full}'
    padding: '{spacing.chip-padding}'
  transparency-accordion:
    trigger-color: '{colors.terracotta}'
    panel-background: '{colors.surface-soft}'
    panel-radius: '{rounded.md}'
    row-divider: '1px dashed {colors.border}'
    row-value: '{typography.figure-sm}'
  concentration-gate-modal:
    overlay: '{colors.overlay}'
    card-background: '{colors.bg}'
    card-radius: '{rounded.xl}'
    card-padding: '{spacing.gate-padding}'
    card-shadow: '0 20px 50px rgba(0,0,0,.35)'
    warning-background: '{colors.red-soft}'
    warning-foreground: '{colors.red-strong}'
  button-primary:
    background: '{colors.terracotta-strong}'
    foreground: '#FFFFFF'
    radius: '{rounded.md}'
    typography: '{typography.action}'
  button-ghost:
    background: '{colors.card}'
    border: '1px solid {colors.border}'
    foreground: '{colors.ink-muted}'
    radius: '{rounded.md}'
    typography: '{typography.action}'
  # [ASSUMPTION] não extraído do mockup (que só cobria o Painel Único) — derivado
  # por consistência para Cadastro, Configuração de Perfil, Registro de Posição
  # e o campo de "total" do gate de concentração (FR-6).
  input-field:
    background: '{colors.card}'
    border: '1px solid {colors.border}'
    border-focus: '1px solid {colors.terracotta}'
    radius: '{rounded.md}'
    label-typography: '{typography.label}'
    text-typography: '{typography.body}'
    numeric-typography: '{typography.figure-sm}'
---

## Brand & Style

O Painel Único existe para uma única pessoa: a Marta, leiga em investimentos, com dinheiro parado na poupança há anos por medo e vergonha de errar. O diferencial deste produto não é o motor de análise por trás da recomendação — é a **camada visual de redução de ansiedade** que fica entre esse motor e a Marta. Tudo neste documento serve a essa camada.

A direção de marca aprovada é "extrato bancário limpo, mas mais quente": credibilidade financeira sem frieza ou esterilidade. Na prática, isso significa cartões brancos arredondados sobre um fundo creme quente, sombras macias em vez de bordas duras, números sempre em fonte monoespaçada tabular — como um cupom fiscal ou um extrato impresso — e uma paleta terracota/musgo em vez do azul-corporativo genérico de fintech. É o registro de um "recibo caloroso" entregue na mão, não o de uma planilha.

A única peça que rompe esse registro de cartão é o indicador de status: um termômetro vertical literal (tubo + bulbo), não um semáforo de pontinhos. Mas mesmo essa peça foi redesenhada dentro do vocabulário quente do recibo — cápsula arredondada, sombra macia colorida no bulbo, selo de status carimbado — em vez do tubo de contorno duro e editorial que existia numa direção anterior descartada. Não existe, neste produto, aresta viva (ver Shapes).

Três regras de tom vêm diretamente do PRD e não são negociáveis em nenhuma tela nova que vier a ser desenhada sobre esta base:

1. **Calmo por padrão.** O painel não compete por atenção. Silêncio (nenhum alerta, nenhuma recomendação nova) é o estado esperado e correto, não um vazio a preencher.
2. **Status visual antes do número cru.** Termômetro/selo é sempre a primeira camada; o dado fundamentalista/técnico completo só aparece ao expandir a trilha de transparência.
3. **Nunca imperativo.** Copy usa "os dados indicam", nunca "compre". O disclaimer educacional é tecido no meio do texto da própria recomendação — nunca um rodapé, nunca um modal à parte. O aviso de concentração soa como cuidado genuíno, não como termo de uso.

## Colors

A paleta é deliberadamente estreita: dois neutros quentes (`{colors.bg}` / `{colors.ink}`), um acento de marca (`{colors.terracotta}`) e três matizes de status (`{colors.moss}`, `{colors.amber}`, `{colors.red}`) que existem *apenas* para alimentar o termômetro e os selos de recomendação — nunca para decoração.

- **`{colors.bg}` (`#FFF8F0`)** é o fundo do painel inteiro: um creme quente, não um branco de formulário. **`{colors.card}` (`#FFFFFF`)** é o fundo de todo cartão de conteúdo — o contraste entre os dois é o que dá a sensação de "papel sobre mesa", não de tela. **`{colors.surface-soft}` (`#FBF3E7`)** é um terceiro tom, ainda mais quente que o cartão, usado para blocos de texto explicativo dentro de um cartão (o "porquê" da recomendação, os boxes da trilha de transparência) — sinaliza "isto é leitura de apoio", não a informação principal do cartão.
- **`{colors.ink}` (`#3A2E22`)** é o texto primário — um marrom-escuro quente, nunca preto puro. **`{colors.ink-muted}` (`#8A7A68`)** é o texto secundário (sublabels, legendas, disclaimers de apoio). **Contraste medido: `ink-muted` sobre `bg`/`card` fica em ~3,9–4,1:1 — abaixo do mínimo AA de 4,5:1 para texto normal.** `[ASSUMPTION]` Use `ink-muted` apenas em rótulos curtos ≥14px em negrito ou texto claramente decorativo; para qualquer texto secundário com conteúdo essencial de leitura (ex.: a sublinha de `receipt-row`, o parágrafo "não é uma recomendação de compra"), considere escurecer para algo como `#7A6A5A` (≈4,9:1) até um teste de contraste real confirmar a leitura em tela.
- **`{colors.terracotta}` (`#E0703B`)** é o único acento de marca: usado no wordmark, no gatilho do acordeão de transparência e nos valores em destaque (ex.: a diferença de rendimento perdido). **Como texto, fica em ~3,0–3,2:1 sobre `bg`/`card` — abaixo de AA para texto normal**, então reserve terracota-como-texto para tamanhos grandes (≥18px) ou combine com outro sinal (sublinhado, ícone `+`/`－`) que não dependa só da cor para indicar interatividade. Para preenchimento de botão com texto branco, use `{colors.terracotta-strong}` (`#B8562A`, ~4,8:1) em vez da terracota pura — a terracota pura com texto branco mede ~3,2:1 e não passa em AA. `terracotta-soft` é o fundo do disclaimer inline; o par `terracotta-soft` / `ink-on-terracotta-soft` mede ~5,7:1, seguro.
- **`{colors.moss}` (`#5B8266`)** é o status "favorável" — a única cor além da terracota que aparece fora de um contexto de aviso. Usado no preenchimento do termômetro e no selo de status. Como texto sobre `card`, mede ~4,35:1 — levemente abaixo de 4,5:1; como é sempre um rótulo curto em caixa alta e negrito, o risco prático é baixo, mas vale escurecer ligeiramente (~`#547A5F`, ~4,9:1) se um teste de contraste formal for feito depois do v1. `moss-light` existe *só* para o topo do degradê do termômetro, nunca como cor de texto ou fundo isolado.
- **`{colors.amber}` (`#D9A441`)** é o status "atenção" — ainda não prototipado no mockup, só declarado. **Não use `amber` como cor de texto: mede ~2,25:1 sobre branco, muito abaixo de AA mesmo para texto grande.** `[ASSUMPTION]` Use `amber` apenas em preenchimentos/ícones do termômetro; para o rótulo de texto do selo "Atenção", use `{colors.amber-ink}` (`#8A6420`, ~5,3:1). `amber-soft` (fundo suave, análogo a `terracotta-soft`/`moss-soft`) é derivado por padrão, não extraído do mockup.
- **`{colors.red}` (`#C1503F`)** é o status "alerta" e a cor do aviso de concentração — a única cor com carga emocional de "pare e pense". Como texto sobre `card`/`bg`, mede ~4,4–4,7:1 (no limite, mas aceitável). **Sobre `red-soft` (como no box de aviso de concentração), cai para ~3,8:1 — abaixo de AA.** `[ASSUMPTION]` Nesse contexto específico, use `{colors.red-strong}` (`#9B3F2C`, ~5,5:1) como cor do texto, mantendo `red` reservado para ícones/realces pontuais dentro do mesmo box.
- **`{colors.overlay}`** é o véu por trás do modal de aviso de concentração — a única vez que o painel escurece a tela inteira, porque é o único gate bloqueante do produto.

Evite: gradientes decorativos fora do termômetro e mais de um acento de marca.

### Modo escuro `[ASSUMPTION]`

Não existe mockup dark — os tokens `*-dark` no frontmatter são derivados, preservando as mesmas famílias de matiz (terracota, musgo, âmbar, vermelho) sobre um fundo marrom-escuro quente (`{colors.bg-dark}`, `#211A14`) em vez de um cinza neutro genérico, para manter a mesma identidade "recibo", só que à noite. Todos os pares abaixo foram validados por cálculo de contraste, mas **precisam de confirmação visual humana antes de ir para produção**:

- `ink-dark` sobre `bg-dark`/`card-dark`: ~14,8:1 / ~13,6:1 — folga generosa.
- `ink-muted-dark` sobre `bg-dark`/`card-dark`: ~7,4:1 / ~6,8:1 — ao contrário do modo claro, o texto secundário escuro passa em AA com folga, porque o fundo é muito mais escuro que o texto (no modo claro os dois tons ficavam mais próximos em luminância).
- `terracotta-dark`, `moss-dark`, `amber-dark`, `red-dark` sobre seus respectivos `*-soft-dark`: todos entre ~5,8:1 e ~7,6:1. Curiosamente, nenhum precisou de uma variante "ink"/"strong" equivalente às do modo claro — o problema do `amber` e do `red-soft` no modo claro vem da proximidade de luminância com fundos muito claros (branco/creme), que não existe no modo escuro.
- Botão primário no escuro: preenchimento `terracotta-dark` com texto na cor `bg-dark` (não branco) — ~7,3:1.

## Typography

Uma única família — a pilha de sistema (`{typography.body.fontFamily}`) — cobre todo o texto de interface: títulos, rótulos, corpo, legendas. Não há uma segunda família decorativa; a variação de peso e caixa alta já carrega a hierarquia. Isso é deliberado: um investidor leigo não deveria sentir que está lendo uma peça de marketing.

A exceção é o valor numérico. Todo número que representa dinheiro, percentual ou indicador (`{typography.figure-lg}`, `{typography.figure-sm}`) é composto em `ui-monospace, SFMono-Regular, Menlo, monospace`, sempre com largura tabular. Isso é um dispositivo de tom deliberado, não incidental: números alinhados em coluna monoespaçada são o que faz o painel parecer um extrato/recibo em vez de um dashboard — e ajuda a Marta a comparar "R$ 1.080,00" com "R$ 2.070,00" de relance, sem precisar recalcular nada de cabeça.

Ramp de papéis:

- `{typography.brand}` — wordmark, único lugar com peso 800 fora de um valor numérico.
- `{typography.heading}` — ticker da ação (ex. "TAEE11"), título do modal de concentração.
- `{typography.label}` — rótulos de seção em caixa alta (`Quanto seu dinheiro deixou de render`, `Status`, cabeçalhos da trilha de transparência).
- `{typography.body}` — texto principal de leitura (linha do item na comparação, explicação "os dados indicam").
- `{typography.meta}` — texto de apoio (sublinha, disclaimer não-imperativo, linhas da trilha).
- `{typography.caption}` — menor nível, só para atribuição de fonte de dado (ex. "Fonte: dados públicos B3 · ref. ago/2026").
- `{typography.action}` — gatilhos de acordeão e texto de botão.

Ver §Do's and Don'ts para a regra de uso de jargão técnico em `{typography.body}`/`{typography.meta}`.

## Layout & Spacing

O Painel Único é uma **tela única, sem navegação entre páginas** (FR-3). Toda expansão de conteúdo acontece inline — acordeão (`transparency-accordion`) ou modal bloqueante (`concentration-gate-modal`) — nunca uma rota nova. Isso não é só uma preferência de layout: é a materialização de "silêncio como produto" — não existe "próxima página" para empurrar a Marta adiante.

Escala de espaçamento: `{spacing.1}` a `{spacing.7}` (4/8/12/16/20/24/32px) cobre a maior parte dos ritmos internos de componente; alguns valores de padding foram mantidos como tokens nomeados e literais (`{spacing.card-padding}`, `{spacing.chip-padding}`, `{spacing.gate-padding}`) porque o mockup não segue uma grade estritamente geométrica — é um espaçamento afinado visualmente, não derivado de fórmula. `{spacing.card-gap}` (18px) é o ritmo vertical entre os cartões do painel; `{spacing.content-margin}` (32px) é a margem horizontal do conteúdo em desktop.

Ordem de conteúdo dentro do painel é uma regra de produto, não só de layout, e não deve ser reordenada em nenhuma tela futura: (1) comparação de perda por inércia com disclaimer embutido → (2) recomendação única + termômetro → (3) trilha de transparência expansível. A comparação de perda nunca aparece isolada — sempre emenda direto na recomendação, porque mostrar "quanto você perdeu" sem uma saída imediata reforça culpa em vez de resolvê-la.

`[ASSUMPTION]` O mockup só mostra o layout desktop (`reco-grid`: texto + termômetro lado a lado em `1fr auto`). Como o produto é web responsivo (desktop + mobile web, sem app nativo), o comportamento mobile precisa ser confirmado visualmente, mas a regra proposta é: coluna única sempre; `reco-grid` empilha (texto em cima, termômetro centralizado abaixo) abaixo de ~640px; `trilha-grid` (hoje duas colunas) empilha em uma coluna; margem de conteúdo cai de `{spacing.content-margin}` (32px) para algo entre `{spacing.4}` e `{spacing.5}` (16–20px) em telas estreitas.

## Elevation & Depth

A profundidade no Painel Único é sempre suave e nunca usada para hierarquia de navegação — só para separar "isto é um objeto físico sobre a mesa" do fundo. Cinco níveis, todos extraídos do mockup:

- **Cartão de conteúdo** — `0 8px 20px -14px rgba(58,46,34,.4)`. É o nível padrão de todo `card`; a sombra é curta e difusa, nunca dura.
- **Selo/chip** — `0 4px 10px -6px rgba(58,46,34,.3)`, mais discreta, usada no selo de status do termômetro.
- **Realce colorido** — o bulbo do termômetro usa uma sombra tingida com a própria cor do estado (ex. `0 6px 14px -4px rgba(91,130,102,.55)` no estado favorável) em vez de uma sombra neutra — é o único lugar do sistema onde a sombra carrega cor, porque o bulbo é o ponto focal do indicador.
- **Sombra interna** — `inset 0 2px 5px rgba(58,46,34,.1)` no tubo do termômetro, para dar sensação de profundidade física ao "vidro" do tubo.
- **Modal de concentração** — `0 20px 50px rgba(0,0,0,.35)`, a única sombra do sistema em preto puro (não tingida de `ink`) e a mais pesada. É intencional: este é o único gate bloqueante do produto, e a elevação mais dramática do sistema deve coincidir com o único momento de fricção real.

`[ASSUMPTION]` Sombra escura sobre fundo escuro tende a "sumir" visualmente. No modo escuro, mantenha as mesmas formas de sombra, mas dependa mais de `{colors.border-dark}` como divisor de superfície do que da sombra em si para cartões comuns; reserve uma sombra ainda visível (preto puro, opacidade um pouco maior) só para o modal — o mesmo raciocínio de "elevação dramática só na fricção real" se aplica com ainda mais força no escuro.

## Shapes

Não existe aresta viva em nenhum componente do Painel Único — é a assinatura visual mais consistente do "recibo caloroso". Escala:

- `{rounded.sm}` (8px) — campos pequenos e o disclaimer inline (`disclaimer-pill`).
- `{rounded.md}` (12px) — blocos de conteúdo secundário (`reco-why`, boxes da trilha, tubo do termômetro), botões.
- `{rounded.lg}` (16px) — cartões principais do painel.
- `{rounded.xl}` (20px) — contêineres grandes: o cartão do modal de concentração, o invólucro do termômetro (`therm-wrap`), chips carimbados (selo de status, saudação "Olá, Marta").
- `{rounded.full}` — círculos verdadeiros (bulbo do termômetro, thumb do slider) e pílulas de chip (`confirm-chip`).

A lógica: quanto mais "objeto tocável" o elemento parece (bulbo, chip carimbado), mais arredondado ele fica; quanto mais "campo de dado" (linha da trilha, tubo), mais contido — mas nunca reto. Imagens e ícones, se algum dia forem introduzidos, devem seguir o raio do contêiner que os envolve.

## Components

- **Card** — Contêiner base de toda seção do painel. `{colors.card}` sobre `{colors.bg}`, borda de 1px em `{colors.border}`, `{rounded.lg}`, sombra de nível "cartão". Título de seção sempre em `{typography.label}`.
- **Receipt row** — Padrão da tela "quanto você perdeu parado": rótulo + sublabel à esquerda, valor monoespaçado tabular à direita, divisor tracejado (`1px dashed {colors.border}`) entre linhas, sem divisor na última. O valor em destaque (a diferença/perda) usa `{colors.terracotta}`.
- **Disclaimer pill** — Bloco de texto embutido (nunca rodapé, nunca modal), `{colors.terracotta-soft}` de fundo, `{colors.ink-on-terracotta-soft}` de texto, `{rounded.sm}`. Aparece na primeira visita ao painel e em todo ponto de recomendação (FR-9).
- **Reco card** — Grade de duas colunas (texto + termômetro) em desktop, empilhada em mobile `[ASSUMPTION]`. Ticker em `{typography.heading}`, explicação do "porquê" sobre `{colors.surface-soft}`, texto não-imperativo em `{typography.meta}`/`{colors.ink-muted}`, e o `confirm-chip` confirmando o percentual já alocado.
- **Thermometer** — O componente central do sistema. Anatomia: rótulo (`Status`, `{typography.label}`) → tubo em cápsula com marcações de nível → preenchimento em degradê (`{colors.moss}` → `{colors.moss-light}`, ou a cor equivalente de âmbar/vermelho conforme o estado) → bulbo circular com sombra tingida → selo de texto carimbado. Três estados, sempre cor + texto juntos, nunca só cor: **Favorável** (`{colors.moss}`), **Atenção** (`{colors.amber}` no preenchimento, `{colors.amber-ink}` no texto do selo), **Alerta** (`{colors.red}`). É a camada que substitui o número cru como primeira informação (FR-11) — os valores completos só aparecem ao expandir a trilha de transparência.
- **Confirm chip** — Pílula pequena (`{rounded.full}`) confirmando uma escolha já feita pelo usuário (ex. percentual alocado), fundo `{colors.moss-soft}`, texto `{colors.ink-on-moss-soft}`, com um ponto colorido de 7px à esquerda do texto.
- **Transparency accordion** — `<details>/<summary>` nativo estilizado: gatilho em `{colors.terracotta}` com marcador "+/－" textual (nunca ícone de seta genérico), painel em `{colors.surface-soft}`, duas colunas (fundamentalista / técnico) que empilham em uma só em mobile `[ASSUMPTION]`. Cada linha de indicador mostra valor monoespaçado + fonte/data de referência em `{typography.caption}` — nunca só o nome do indicador sem o valor.
- **Concentration gate modal** — O único modal do sistema e o único gate bloqueante. Véu `{colors.overlay}` sobre o painel desfocado; cartão em `{colors.bg}` (não `{colors.card}` — ligeiramente mais quente que um cartão comum, para não parecer "mais um card"), `{rounded.xl}`, sombra de modal. Contém, nesta ordem: campo numérico de "total" (ver **Input field** abaixo — `[ASSUMPTION]` não estava no mockup original, adicionado após reconciliação com FR-6), slider de percentual, caixa de aviso (`{colors.red-soft}` de fundo / `{colors.red-strong}` de texto — ver nota de contraste em Cores) afirmando explicitamente que ação avulsa não é, por definição, investimento conservador, e dois botões (`button-ghost` para ajustar, `button-primary` para confirmar). Recomendação completa (FR-5/FR-7/FR-8) só é liberada depois desta confirmação.
- **Input field** `[ASSUMPTION]` — Não existia no mockup original (que só cobria o Painel Único); estilo derivado por consistência com os demais campos de superfície para servir também Cadastro, Configuração de Perfil e Registro de Posição. Fundo `{colors.card}`, borda 1px `{colors.border}` (`{colors.terracotta}` no foco), `{rounded.md}`, texto em `{typography.body}` — valores numéricos (R$, %, quantidade) usam `{typography.figure-sm}` monoespaçada, consistente com o resto do sistema. Rótulo sempre visível acima do campo (`{typography.label}`), nunca só placeholder.
- **Button** — Primário: `{colors.terracotta-strong}` de fundo, texto branco, `{rounded.md}` (nunca a terracota pura com texto branco — ver nota de contraste). Ghost: `{colors.card}` com borda `{colors.border}` e texto `{colors.ink-muted}`. Nenhum outro peso de botão existe no v1 (sem "danger button" dedicado — o próprio texto e a caixa de aviso já carregam essa carga).

## Do's and Don'ts

| Do | Don't |
|---|---|
| Termômetro/selo (cor + texto) como primeira camada de status | Mostrar indicador técnico cru (%, índice) como primeira informação sem o equivalente visual ao lado |
| "Os dados indicam...", disclaimer tecido no meio do texto da recomendação | Linguagem imperativa ("compre", "invista agora"); disclaimer em rodapé ou modal separado |
| Uma única recomendação por vez, sempre com trilha de transparência acessível | Lista/feed de múltiplas ações sugeridas, ou recomendação sem justificativa expansível |
| Números monetários/percentuais em `{typography.figure-lg}`/`{typography.figure-sm}` (monoespaçada tabular) | Números de valor em fonte proporcional (perde o efeito "recibo/extrato") |
| Cantos suaves em todo componente (`{rounded.sm}` a `{rounded.full}`) | Qualquer aresta viva ou canto reto |
| `amber`/`red` como preenchimento; `amber-ink`/`red-strong` como texto quando o fundo for claro ou `*-soft` | Usar `amber` ou `red` puro como cor de texto sobre fundo claro (contraste insuficiente — ver Cores) |
| Silêncio (nenhum alerta, nenhuma recomendação nova) como estado padrão e correto | Forçar um "estado vazio" que pareça erro, ou aumentar frequência de alertas para parecer mais "ativo" |
| Gate de concentração como modal bloqueante de verdade, com aviso que soa como cuidado | Tratar o aviso de concentração como banner dispensável ou aviso legal frio |
| Termo técnico (CDI, DARF, ação avulsa) definido inline ou linkado ao glossário ao aparecer em corpo/texto de apoio | Assumir que termo técnico é óbvio para a audiência leiga |
