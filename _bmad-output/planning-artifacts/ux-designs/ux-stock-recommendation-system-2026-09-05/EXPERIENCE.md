---
title: Sistema de Recomendação de Ações para Investidores Leigos
status: final
created: 2026-09-05
updated: 2026-09-07
sources:
  - _bmad-output/planning-artifacts/prds/prd-stock-recommendation-system-2026-08-30/prd.md
---

# Sistema de Recomendação de Ações para Investidores Leigos — Experience Spine

> Single-surface web responsivo (desktop + mobile web), sem app nativo no v1. Persona de referência: Marta. Paridade total entre temas light/dark — este documento descreve comportamento; `DESIGN.md` (em elaboração em paralelo, a partir de `.working/direction-merged-recibo-termometro.html`) descreve tokens visuais e componentes. `[NOTE FOR UX]` os nomes de token usados abaixo (`{colors.*}`, `{rounded.*}`) são descritivos, inferidos do CSS do mockup aprovado — reconciliar contra a nomenclatura final do `DESIGN.md` assim que publicado.

## Foundation

- **Superfície:** web responsivo, desktop + mobile web, sem app mobile nativo no v1 `[ASSUMPTION confirmado no PRD §6.2]`. Não existem duas experiências divergentes desktop/mobile — é o mesmo Painel Único em ambos os form factors, com o mesmo padrão de expansão inline (accordion nativo, modal) em vez de páginas dedicadas.
- **Sem UI system de terceiros.** Nada de shadcn/MUI/Bootstrap — todo componente é custom, construído a partir da direção "Recibo Caloroso" (cards brancos arredondados, sombra suave, números tabulares monoespaçados como cupom fiscal, paleta terracota/musgo) já validada no mockup mesclado.
- **Dark mode obrigatório desde o v1.** Todo Component Pattern e State Pattern descrito neste documento precisa funcionar de forma idêntica em light e dark — **só os tokens de cor trocam, nunca o comportamento, a hierarquia ou a sequência de interação.** Isso vale em especial para o termômetro de status (a cor do preenchimento não pode ser o único portador de significado — ver Accessibility Floor) e para o gate de concentração (o mesmo fluxo de foco/teclado em ambos os temas).
- **Direção de marca:** "extrato bancário limpo, mas mais quente" — credibilidade financeira sem ser frio/estéril. Isso se traduz em comportamento, não só em estilo: densidade média (nem denso feito planilha, nem vazio feito app de bem-estar), números sempre alinhados e tabulares, hierarquia de "recibo" (rótulo, sub-rótulo discreto, valor à direita).
- **`DESIGN.md`** é a referência de identidade visual (paleta, tipografia, espaçamento, sombra, os componentes visuais do termômetro e do card-recibo). Este documento (`EXPERIENCE.md`) é a espinha de experiência: arquitetura de informação, tom, comportamento de componente, estados, teclado. Em caso de conflito de leitura, esta spine vence sobre a composição visual do mockup — o mockup é referência de forma, não de contrato de comportamento.
- **Persona de referência:** Marta — leiga em investimentos, dinheiro parado na poupança há anos por medo e vergonha de não entender, não por falta de informação. Toda decisão de tom e sequência neste documento é calibrada para ela, não para um usuário avançado de mercado.
- **Vocabulário oficial** (Glossário do PRD, §3) a manter idêntico em toda a UI: Investidor leigo, Perfil de risco, Horizonte de investimento, Ação avulsa, Aviso de concentração, Trilha de transparência, Painel Único, IR, Ganho de capital, DARF, CDI. Não sinonimizar esses termos em copy.

## Information Architecture

O produto é deliberadamente **single-surface para o seu valor central**: o Painel Único é uma tela só, com tudo dentro (perda por inércia, recomendação, aviso de concentração, trilha de transparência, projeção/IR, status, alertas, registro de posição), expandido inline via accordion/modal — nunca uma página nova por seção (FR-3). Existem, no entanto, superfícies próprias para autenticação e configuração inicial de perfil, que o usuário só atravessa uma vez (ou raramente) antes de chegar ao painel.

| Surface | Reached from | Purpose | Jornada |
|---|---|---|---|
| Cadastro (Signup) | CTA na página inicial | Criar conta com e-mail/senha; aceite explícito e específico do uso de dados financeiros e de perfil de risco — não um checkbox genérico de termos de uso (FR-1, FR-12) | UJ-1 |
| Login | Retorno direto / link "já tenho conta" no Cadastro | Autenticar usuário existente (FR-1) | UJ-3 |
| Recuperação de senha | Login → "esqueci minha senha" | Reset de senha por e-mail (FR-1) | suporte a UJ-3 |
| Configuração de Perfil | Automático, uma única vez, logo após o Cadastro bem-sucedido — sem tela intermediária entre os dois | Perfil de risco (conservador/moderado/arrojado), horizonte de investimento (curto/médio/longo) e valor de poupança opcional (FR-2) | UJ-1 |
| **Painel Único** (Dashboard) — [mock](./mockups/painel-unico.html) | Direto após Configuração de Perfil (primeira vez) / Login (retornos seguintes) | Tela central única: perda por inércia (FR-4), recomendação (FR-5), gate de concentração (FR-6), trilha de transparência (FR-7), projeção de retorno e IR (FR-8), disclaimer embutido (FR-9), alerta de mudança relevante (FR-10), indicador visual de status (FR-11), registro de posição (FR-13), guia rápido na primeira visita. Tudo inline, zero navegação entre páginas. | UJ-1 (chegada), UJ-2, UJ-3 |
| Edição de Perfil | Painel Único → menu/configurações | Reabrir e editar perfil de risco e horizonte já declarados (FR-2) | — |
| Gestão de Conta e Consentimento | Painel Único → menu/configurações | Rever o consentimento de uso de dados e solicitar exclusão da conta e dos dados associados (FR-12) | — |

Regras de sequência que fazem parte da IA (não só do layout, mas do próprio contrato de produto):

- Dentro do Painel Único, a ordem é fixa: (1) perda por inércia, com disclaimer embutido → (2) recomendação resumida → (3) gate obrigatório de percentual/concentração → (4) trilha de transparência + projeção liberadas. A comparação de perda **nunca aparece isolada**: por design, ela sempre emenda direto na recomendação, porque mostrar "quanto você perdeu" sem uma ação imediata na sequência reforçaria a culpa/ansiedade que o produto existe para reduzir.
- O gate de concentração (FR-6) é um **bloqueio real de conteúdo**, não um banner dispensável: a trilha de transparência e a projeção de retorno/IR não renderizam até o percentual ser confirmado.
- Nenhuma dessas seções tem URL própria nem exige "voltar" para o painel — expansão é sempre inline (accordion nativo, modal).

## Voice and Tone

Microcopy comportamental. Identidade de marca (paleta, tipografia) vive em `DESIGN.md`; aqui vai a regra de linguagem que nenhum componente pode violar.

| Do | Don't |
|---|---|
| "Os dados indicam TAEE11 como compatível com o seu perfil conservador e horizonte de longo prazo — mesmo assim, uma ação avulsa concentra o risco em uma única empresa; nenhum perfil de risco elimina esse risco por definição." | "Compre TAEE11 agora." / qualquer imperativo de compra. **Também proibido:** citar "compatível com o perfil conservador" sem a ressalva de concentração na mesma frase/parágrafo — as duas cláusulas são uma unidade e nunca aparecem separadas (PRD §8, item 8; ver princípio abaixo). |
| "Isso não é uma recomendação de compra — a decisão é sempre sua." tecido na mesma frase/parágrafo da recomendação | Disclaimer isolado em rodapé, tooltip separado ou modal à parte |
| "R$ 990,00 dessa diferença ficaram parados junto com o resto do dinheiro. Aqui está uma ação para o seu perfil." — perda seguida imediatamente de ação | Mostrar a perda sozinha, sem a recomendação na sequência (reforça culpa em vez de resolver) |
| "Nenhuma mudança relevante desde sua última visita." | "🎉 Sua ação está indo bem, continue acompanhando!" / qualquer tom de celebração ou pressão para agir |
| Termo técnico sempre definido inline ou linkado ao Glossário na primeira ocorrência na tela | Jargão técnico cru sem explicação ("ROE", "swing trade" soltos) |
| "Como teria se comportado desde a recomendação" quando não há registro de posição | Apresentar número simulado com a mesma formulação de um resultado real do usuário |

Princípios de tom, não negociáveis:

- **Nunca imperativo.** Toda recomendação usa formulações como "os dados indicam" ou equivalente — nunca "compre", "venda", "aja agora".
- **Disclaimer sempre tecido no texto**, nunca em rodapé, tooltip ou modal separado — aparece na primeira visita ao painel e em todo ponto de recomendação (FR-9).
- **A nuance do gate de concentração é literal, não suavizada:** o texto precisa declarar, sem meio-termo, que uma ação avulsa não é "conservadora por definição" mesmo quando compatível com um perfil conservador — essa é uma decisão de produto deliberada (PRD §8, item 8), não um detalhe de copy a amaciar.
- **"Compatível com o perfil" e "concentra o risco" nunca se separam.** Qualquer tela, componente ou copy nova que mencione a compatibilidade de uma recomendação com o perfil de risco do usuário carrega a ressalva de concentração na mesma frase ou parágrafo — nunca em telas diferentes, nunca como um disclaimer genérico à parte. Isso vale para FR-5 (recomendação) tanto quanto para o gate de concentração (FR-6); são a mesma regra aplicada duas vezes.
- **Segurança e permissão, não venda.** O trabalho emocional do produto é substituir o "cunhado que entende de bolsa" por uma explicação rastreável — a voz é a de alguém que mostra os dados e devolve a decisão, nunca a de quem empurra uma ação.

## Component Patterns

Comportamental. Especificação visual (cor, sombra, raio exato) vive em `DESIGN.md.Components`.

| Component | Use | Behavioral rules |
|---|---|---|
| Card recibo | Toda seção do Painel Único (perda por inércia, recomendação, trilha) | Um card por seção lógica; título em uppercase discreto no topo; nunca mistura duas seções no mesmo card. Fundo `{colors.card}`, borda `{colors.border}`, raio `{rounded.lg}` — mesma estrutura em light/dark, só os tokens trocam. |
| Linha de recibo | Listas de valor dentro de um card (ex: poupança vs. CDI, indicadores da trilha) | Rótulo à esquerda (+ sub-rótulo discreto opcional), valor tabular monoespaçado à direita; divisor tracejado entre linhas, nenhum divisor após a última. Valor em destaque usa `{colors.terracotta}` quando representa a diferença/oportunidade perdida. |
| Termômetro (indicador de status) | Recomendação ativa (FR-11) | Termômetro vertical literal (tubo + bulbo), não semáforo de pontinhos. Preenchimento mapeia o status (favorável/atenção/alerta) por altura e cor — **sempre acompanhado de um selo de texto** ("Favorável", "Atenção", "Alerta"), nunca só cor. Mesmo comportamento em light/dark, cores redefinidas via token. |
| Trilha de transparência (accordion) | Expansão da recomendação (FR-7) | Implementado com `<details>`/`<summary>` nativos — sem JS de terceiros, preserva foco/teclado de graça. Fechado por padrão. Um nível só, sem accordion aninhado. Ao abrir, mostra dois grupos (fundamentalistas / padrões técnicos) com valor, fonte e data de referência — nunca só o nome do indicador. |
| Gate de concentração (modal bloqueante) | Antes de liberar trilha (FR-7) e projeção (FR-8) | Modal obrigatório, visualmente distinto de um checkbox de termos. Dois campos, não um: (1) campo numérico em R$ para o "total" que a Marta diz pretender ter investido em renda variável — a base autoinformada do cálculo, distinta do valor de poupança (FR-2) e de qualquer patrimônio real, já que o produto não tem Open Finance; (2) controle de percentual (slider) que expressa quanto dessa recomendação específica cai dentro desse total. Na primeira vez, os dois campos aparecem juntos; em gates seguintes, o campo de total vem pré-preenchido com o último valor informado, editável. Mais texto de aviso explícito + duas ações ("ajustar percentual" / "confirmar e ver detalhes"). Conteúdo completo da recomendação só renderiza após confirmação — ver Interaction Primitives e Accessibility Floor para o comportamento de foco/teclado. |
| Guia rápido (tour inline) | Primeira visita ao Painel Único (FR-3) | Destaques inline sobre as seções reais do painel, não uma tela separada nem vídeo. `[ASSUMPTION]` formato exato (tooltip sequencial com "próximo"/"pular") fica em aberto no PRD (`[NOTE FOR PM]` explícito) — esta é a leitura de UX default, a validar. Não reaparece em visitas seguintes; revisitável sob demanda a partir das configurações. |
| Registro de posição (form/modal leve) | Acionado a partir do card de recomendação (FR-13) | Três campos: data, preço, quantidade. Ao salvar, todo número de performance ligado àquela recomendação passa a se referenciar ao preço de entrada real — ver "Rotulagem: Dado Real vs. Simulado" abaixo. |
| Projeção de retorno e IR | Dentro da trilha de transparência expandida, junto da projeção curto/médio/longo (FR-8) | **Nunca um número único de isenção.** Sempre dois cenários lado a lado: "se esta for sua única venda de ações no mês" (com a isenção de R$20.000/mês aplicada) vs. "se você já vendeu outras ações este mês" (isenção pode não se aplicar) — o sistema não sabe qual é real, porque não tem visibilidade de vendas fora da plataforma (§5/§8 item 7 do PRD). Abaixo dos dois cenários, uma frase fixa recomendando consulta a um contador antes de declarar — não é um rodapé legal, é parte do mesmo bloco de texto dos cenários. |
| Chip de confirmação | Após o gate de concentração ser resolvido | Pill compacto mostrando o percentual confirmado ("Percentual confirmado: 12% do total"), fundo `{colors.moss-soft}` — reforço visual discreto, não um novo alerta. |
| Pílula de disclaimer | Dentro do card de perda por inércia e junto de toda recomendação | Bloco de texto integrado ao card (não modal, não rodapé), fundo `{colors.terracotta-soft}`, carrega a frase de "rentabilidade passada não garante rentabilidade futura" e o enquadramento educacional. |

## State Patterns

| State | Surface | Treatment |
|---|---|---|
| Carregamento frio | Painel Único | Skeleton nos cards na mesma estrutura final (título + linhas), resolve-se quando os dados chegam. Sem spinner central genérico. |
| Poupança não informada | Card de perda por inércia | Usa valor ilustrativo e convida a inserir o valor real ali mesmo — nunca bloqueia a tela nem esconde a seção (edge case UJ-1). |
| Perfil/horizonte não calibrado (moderado, arrojado, curto ou médio prazo) | Card de recomendação | Mensagem clara: "ainda não suportado neste perfil" no lugar da recomendação — nunca uma recomendação mal calibrada apresentada como se fosse válida (PRD §4.1, Questão em Aberto #9). |
| Dados do provedor indisponíveis ou desatualizados (> 1 dia útil) | Card de recomendação | Comunica isso explicitamente em texto simples, em vez de mostrar recomendação incompleta ou com cotação obsoleta (edge case UJ-2, NFR de frescor de dados). |
| Gate de concentração pendente | Painel Único | Recomendação resumida visível; trilha de transparência e projeção permanecem ocultas/bloqueadas até a confirmação do percentual. |
| Sem registro de posição | Card de recomendação, projeção, status | Todo número tratado como simulação — ver "Rotulagem: Dado Real vs. Simulado". |
| Silêncio (nada relevante mudou) | Painel Único, retorno (UJ-3) | Nenhum alerta, nenhum badge, nenhuma notificação — o painel mostra o status atual e para por aí. Este é o estado default esperado, não uma ausência a "preencher" com conteúdo. |
| Alerta ativo | Painel Único | Um aviso visível e único dentro do painel (nunca notificação push/e-mail obrigatória no v1) quando um evento relevante ocorre — nunca mais que um alerta por evento, para não contradizer a contra-métrica de frequência de alertas. |
| Exclusão de conta solicitada | Gestão de Conta | Passo de confirmação explícito antes de efetivar; após confirmar, sessão encerra e acesso ao Painel Único deixa de existir. |

## Interaction Primitives

- **Clique/toque para expandir.** A trilha de transparência abre e fecha com `<summary>` nativo — clique/toque ou Enter/Espaço quando focado.
- **Campo numérico + slider no gate de concentração.** O campo de total (R$) é um input numérico padrão, editável por teclado como qualquer formulário. O percentual usa slider — arrastável com mouse/toque, também operável via setas do teclado quando focado (ver Accessibility Floor). Os dois ficam sincronizados: mudar o total recalcula o percentual da recomendação atual, e vice-versa.
- **Sem atalhos de teclado globais.** Este não é um produto de power-user como uma ferramenta de produtividade — a interação primária é clique/toque, um slider e formulários curtos. Não introduzir paleta de comandos, atalhos vim-style ou gestos customizados.
- **Modais no máximo um nível.** O gate de concentração é o único modal bloqueante do v1; nunca abrir um modal sobre outro modal.
- **Banido:** scroll infinito, notificações push, badges/streaks/gamificação de qualquer tipo, contadores em tempo real ou cotação "piscando" para gerar urgência, carrosséis, autoplay de qualquer mídia, qualquer CTA com linguagem imperativa de compra/venda.

## Accessibility Floor

Piso básico para o v1 — WCAG AA de contraste e navegação por teclado funcional, **sem laudo formal de acessibilidade** (decisão explícita registrada no memlog). "Piso básico" aqui significa o comportamento concreto abaixo, não uma aspiração vaga:

- **Contraste AA** em todo par texto/ícone-fundo, em light **e** em dark — incluindo o selo de texto do termômetro sobre seu preenchimento e os textos dentro da pílula de disclaimer/aviso de concentração. Como os tokens dark ainda estão sendo derivados em paralelo (`DESIGN.md`), a verificação de contraste precisa ser repetida no tema dark, não assumida a partir do light.
- **Status nunca só por cor.** O termômetro sempre é acompanhado do selo de texto ("Favorável"/"Atenção"/"Alerta") — um usuário com daltonismo precisa conseguir ler o status sem depender da cor do preenchimento.
- **Ordem de foco = ordem de leitura** em todo o Painel Único: perda por inércia → recomendação → termômetro → trilha de transparência → projeção. `Tab`/`Shift+Tab` percorre essa sequência sem saltos.
- **Accordion (trilha de transparência) mantém a semântica nativa do `<details>`/`<summary>`:** operável via Enter/Espaço quando o `<summary>` está focado, sem JavaScript customizado que quebre esse comportamento padrão do HTML.
- **Gate de concentração (modal) é totalmente operável por teclado:**
  - Ao abrir, o foco move automaticamente para o primeiro elemento interativo dentro do modal (slider ou botão "ajustar percentual").
  - O foco fica preso dentro do modal enquanto ele estiver aberto — `Tab` cicla só entre os elementos do modal, nunca escapa para o conteúdo atrás dele.
  - `Esc` fecha o modal **sem confirmar** — o gate permanece fechado e a trilha/projeção continuam bloqueadas. Não existe atalho de teclado que "pule" o gate, porque ele é um checkpoint obrigatório de produto, não um banner dispensável.
  - O slider de percentual é ajustável via setas do teclado quando focado, com o valor numérico visível atualizado a cada mudança.
- **Foco visível em todo elemento interativo** — summary, botões, campo de slider, campos de formulário (cadastro, perfil, registro de posição) — usando um anel de foco visível a contraste AA contra `{colors.bg}`/`{colors.card}`, em ambos os temas.
- **Alvos de toque** com tamanho mínimo confortável (na ordem de 44px) em botões, thumb do slider e nos controles de abrir/fechar do `<summary>` — mobile web é superfície de primeira classe, não um modo reduzido.
- **Formulários** (cadastro, configuração de perfil, registro de posição) têm labels associados programaticamente a cada campo; mensagens de erro aparecem inline junto ao campo, nunca só por cor de borda.
- `[NOTE FOR UX]` o PRD não continha nenhum conteúdo de acessibilidade antes deste documento — este piso é a primeira definição do produto nesse eixo. Antes do lançamento público, vale um teste manual real de navegação por teclado e leitor de tela sobre o Painel Único e o gate de concentração especificamente (não só uma checagem de contraste), já que são os dois pontos de maior risco comportamental (bloqueio de conteúdo + accordion).

## Responsive & Platform

| Breakpoint | Behavior |
|---|---|
| Desktop (largura ampla, referência ~960px do mockup aprovado) | Painel Único em coluna única de cards; dentro do card de recomendação, texto e termômetro lado a lado (grid texto + coluna do termômetro). |
| Mobile web (viewport estreito) | Mesma pilha de cards, mesma ordem; o par texto+termômetro empilha verticalmente (texto acima, termômetro abaixo, centralizado) — o termômetro nunca é cortado nem escondido. |

- V1 é web responsivo apenas — desktop e mobile web, sem app nativo (PRD §6.2). Mobile web não é uma versão reduzida do desktop: é a mesma informação, a mesma sequência (perda → recomendação → gate → trilha/projeção), só reorganizada em uma coluna.
- `[ASSUMPTION]` breakpoint exato não está travado no PRD nem no memlog de UX; a referência acima vem da largura do próprio mockup aprovado — confirmar o valor final com `DESIGN.md`/engenharia.

## Rotulagem: Dado Real vs. Simulado

Padrão transversal que atravessa FR-8, FR-10 e FR-11 e merece uma regra própria porque é fácil de esquecer em qualquer tela nova: **sem um registro de posição (FR-13), nenhum número de performance pode ser apresentado como se fosse o resultado real de Marta.**

- Sem registro de posição: todo número (status do termômetro, lucro líquido estimado, alerta de mudança) é rotulado como simulação — "como teria se comportado desde a recomendação".
- Com registro de posição: os mesmos números passam a se referenciar ao preço de entrada real informado.
- A distinção é carregada por **texto**, não só por estilo visual — para não depender de cor (ver Accessibility Floor) e para não desaparecer em dark mode.
- Nenhum estado "real" e nenhum estado "simulado" pode compartilhar a mesma frase — a diferença precisa ser legível mesmo para quem só lê o número rapidamente.

## Guard-rails de Produto (Contra-métricas)

O PRD define três contra-métricas (§7) que funcionam como checklist de revisão para qualquer padrão de UI novo neste produto — e estão aqui porque design é onde elas são violadas ou respeitadas na prática:

- **SM-C1 (frequência de novas recomendações):** nenhum padrão de UI deve incentivar Marta a pedir/ver novas recomendações com mais frequência — nada de "atualizar" chamativo, nada de contador de recomendações disponíveis.
- **SM-C2 (percepção de consultoria não solicitada):** qualquer copy ou componente que soe como um conselheiro pressionando uma decisão (em vez de dados + decisão do usuário) está fora — reforça a regra de Voice and Tone acima.
- **SM-C3 (frequência de alertas):** nenhum componente deve aumentar a frequência de alertas como atalho para parecer "mais ativo" ou melhorar retenção — o estado silencioso (ver State Patterns) é o resultado esperado na maioria das visitas, não uma falha a corrigir.

Qualquer novo componente ou estado proposto depois deste documento deve ser checado contra essas três linhas antes de entrar no Painel Único.

## Inspiration & Anti-patterns

- **Lifted from extrato bancário / cupom fiscal:** números tabulares monoespaçados, linhas com divisor tracejado, densidade média "de recibo" — ancora credibilidade financeira sem cair em estética fria de terminal de corretora.
- **Lifted from termômetro editorial (mesclado com o registro do recibo):** peça central de status vertical e literal (tubo + bulbo), preferida ao semáforo de pontinhos original porque comunica intensidade/magnitude, não só um entre três estados discretos — redesenhada com cápsula arredondada, sombra macia e selo carimbado em vez do tubo de contorno duro do jornal econômico.
- **Lifted from native HTML semantics:** `<details>`/`<summary>` nativos para a trilha de transparência — sem biblioteca de accordion, ganha foco/teclado corretos de graça e reduz superfície de bug em um componente que também é ponto de acessibilidade.
- **Rejected — semáforo de pontinhos como peça principal de status:** descartado a favor do termômetro vertical na decisão de direção visual (memlog) — mantido apenas como referência histórica, não como padrão ativo.
- **Rejected — gamificação e urgência artificial** (itens já listados em "Banido", Interaction Primitives): contradiz diretamente as contra-métricas SM-C1/SM-C3 e a tese de "o silêncio é o produto funcionando" (UJ-3); incompatível com o enquadramento educacional (FR-9) e com a redução de ansiedade que é a proposta central do produto (§1 do PRD).
- **Rejected — chat conversacional ativo / "explica pro cunhado" compartilhável / formato em áudio:** todos itens Could do brainstorm original, deferidos para pós-MVP (PRD §6.2) — não fazem parte desta spine v1.

## Key Flows

As três jornadas abaixo são as jornadas-chave nomeadas no PRD (§2.3) — reformatadas no formato deste documento, sem reinventar conteúdo. IDs e nome da protagonista mantidos verbatim.

### UJ-1 — Marta cria sua conta e define seu perfil

*Marta, leiga em investimentos, dinheiro parado na poupança há anos por medo e falta de conhecimento. Chegou ao site por indicação de alguém. Estado de entrada: não autenticada, navegador desktop ou mobile web.*

1. Marta acessa a página inicial.
2. Cria conta com e-mail e senha (Cadastro) e aceita explicitamente o texto específico sobre uso de dados financeiros e de perfil de risco (FR-12) — não um checkbox genérico.
3. Após o cadastro, é autenticada automaticamente e segue direto para a Configuração de Perfil, sem tela intermediária.
4. Responde poucas perguntas objetivas: perfil de risco (conservador/moderado/arrojado) e horizonte de investimento (curto/médio/longo prazo).
5. Informa, opcionalmente, quanto tem hoje parado na poupança.
6. **Clímax:** ao concluir, cai direto no Painel Único — nenhuma etapa intermediária, nenhum formulário longo.
7. **Resolução:** Marta está no painel, pronta para o UJ-2.

Edge case: se ela pular o valor da poupança, a tela de comparação (FR-4) usa um valor ilustrativo e a convida a inserir o valor real depois.

### UJ-2 — Marta enfrenta o custo de não ter feito nada — e recebe uma recomendação

*Mesma Marta, agora dentro do Painel Único pela primeira vez. Estado de entrada: autenticada, perfil já preenchido (UJ-1).*

1. O painel abre mostrando, em destaque, quanto o valor informado por ela já deixou de render comparado à poupança e, logo abaixo, ao CDI (FR-4) — com o disclaimer educacional embutido no texto, não em rodapé (FR-9).
2. Na sequência, sem sair da tela, vê uma recomendação de ação compatível com seu perfil conservador/longo prazo, com explicação resumida do porquê (FR-5).
3. Antes de qualquer detalhe completo, o sistema pede que ela confirme qual percentual do total pretende alocar ali, com um aviso claro de concentração (FR-6) — o texto afirma explicitamente que uma ação avulsa não é, por definição, um investimento conservador, mesmo compatível com o perfil dela.
4. Ela confirma o percentual; o gate libera a recomendação completa.
5. **Clímax:** ela expande a trilha de transparência — quais indicadores fundamentalistas e quais padrões gráficos pesaram (FR-7) — e vê a projeção de retorno médio (curto/médio/longo prazo) já líquida de IR estimado (FR-8).
6. **Resolução:** Marta sai da sessão sem ter comprado nada ainda, mas entendendo exatamente por que aquela ação foi sugerida e quanto pagaria de imposto se decidisse agir — a decisão continua sendo dela.

Edge case: se os dados do provedor financeiro estiverem indisponíveis para o ativo, o painel comunica isso explicitamente em vez de mostrar uma recomendação incompleta ou desatualizada.

### UJ-3 — Marta volta duas semanas depois

*Já tem uma recomendação registrada; não é notificada com frequência. Estado de entrada: autenticada, retornando ao painel.*

1. Marta faz login.
2. O painel mostra o status atualizado da recomendação anterior em linguagem visual simples (termômetro, FR-11) — sem pedir nenhuma ação, porque está tudo dentro do esperado.
3. **Clímax:** o silêncio é o produto funcionando — nenhum alerta, nenhuma pressão para agir.
4. **Resolução:** ela fecha o painel tranquila; só receberia um alerta ativo se algo relevante tivesse mudado (FR-10).

Alternativa: se algo relevante de fato mudou desde a última visita, um único alerta visível aparece no painel (nunca uma notificação externa obrigatória no v1) — sem exagerar a frequência, para não contradizer a contra-métrica SM-C3.
