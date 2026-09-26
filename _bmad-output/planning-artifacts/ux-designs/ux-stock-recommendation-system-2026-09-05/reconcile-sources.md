---
title: Reconciliação — PRD/Brainstorm vs. Spine de UX
created: 2026-09-07
---

# Reconciliação de Fontes

**Fontes (upstream):**
1. `_bmad-output/planning-artifacts/prds/prd-stock-recommendation-system-2026-08-30/prd.md`
2. `_bmad-output/brainstorming/brainstorm-recomendacao-acoes-leigos-2026-08-25/brainstorm-intent.md`

**Saídas (spine de UX avaliado):**
3. `_bmad-output/planning-artifacts/ux-designs/ux-stock-recommendation-system-2026-09-05/DESIGN.md`
4. `_bmad-output/planning-artifacts/ux-designs/ux-stock-recommendation-system-2026-09-05/EXPERIENCE.md`

## Achados

- **FR-8 (Projeção de retorno e IR) — o requisito mais crítico do PRD em termos de correção fiscal ficou sem tradução em UI/copy nas spines.**
  O que a fonte disse: FR-8 (destacado em negrito no PRD, ligado ao item `[DECIDIDO]` §8.7) exige dois comportamentos concretos e testáveis: (a) mostrar **dois cenários lado a lado** para a isenção de R$20.000/mês — "se esta for sua única venda de ações no mês" vs. "se você já vendeu outras ações este mês, a isenção pode não se aplicar" — em vez de uma estimativa única apresentada como definitiva; (b) informar explicitamente que prejuízos de operações anteriores (dentro ou fora do sistema) não são considerados no cálculo, e **recomendar consulta a um contador antes de declarar**.
  O que falta nas spines: nem `EXPERIENCE.md` nem `DESIGN.md` têm um Component Pattern dedicado para a projeção/IR (ao contrário de FR-4, FR-5, FR-6, FR-7, FR-11, FR-13, que têm cada um sua linha própria em "Component Patterns"/"Components"). FR-8 aparece só como item de lista na Information Architecture ("projeção de retorno e IR (FR-8)"), sem nenhuma menção ao padrão de dois cenários lado a lado nem à recomendação de consultar um contador em "Voice and Tone" ou em qualquer outra seção. Um engenheiro/designer que trabalhasse só a partir das spines não saberia que esse comportamento de UI existe.
  Severidade: **corrigir agora** — é exatamente um dos três itens `[DECIDIDO]` do §8 que a PM pediu para verificar com precisão, e é o FR com a redação mais detalhada/corrigida do PRD (a própria revisão adversarial já havia pego uma premissa errada aqui).

- **FR-6 (Aviso de concentração) — a definição de "total" (base do percentual) não tem mecanismo de captura nas spines.**
  O que a fonte disse: FR-6 define "total" como "o valor que o usuário diz pretender ter investido em renda variável no total (autoinformado **nesta etapa**)" — ou seja, o próprio gate de concentração precisa coletar esse valor absoluto, que não é a poupança (FR-2) nem patrimônio real (sem Open Finance). É a base sobre a qual o percentual do slider é calculado.
  O que falta nas spines: o "Gate de concentração" em ambos os documentos descreve apenas "pergunta de percentual, slider, caixa de aviso" — não há campo, passo ou padrão de estado para o usuário informar o valor absoluto do "total". Sem esse dado, o slider de percentual não tem contra o que ser calculado. `EXPERIENCE.md` usa o exemplo "Percentual confirmado: 12% do total" no Chip de confirmação sem nunca explicar de onde vem o número do "total".
  Severidade: **corrigir agora** — é um requisito de interação concreto e testável (FR-6) sem o qual a feature simplesmente não funciona como especificada; não é um detalhe de estilo.

- **FR-5 / §8 item 8 — tensão não resolvida entre "compatível" permitido com ressalva (FR-5) e "nunca tratado como compatível" (§8 item 8) foi resolvida silenciosamente a favor da leitura mais permissiva.**
  O que a fonte disse: FR-5 diz que a recomendação "nunca é chamada de 'adequada' ou 'compatível' com o perfil conservador **de forma irrestrita**" (permite "compatível" se acompanhado de ressalva). Já a decisão de risco aceito em §8, item 8 — um dos três itens `[DECIDIDO]` citados no aviso do topo do PRD — é mais categórica: "o produto nunca a trata como 'adequada'/'compatível' com o perfil conservador". O PRD tem, portanto, uma tensão interna não resolvida entre os dois trechos.
  O que está diluído/contraditório na spine: `EXPERIENCE.md`, em "Voice and Tone", lista como exemplo aprovado de copy ("Do"): *"Os dados indicam TAEE11 como compatível com o seu perfil conservador e horizonte de longo prazo."* — usando "compatível" isoladamente, em uma linha de tabela separada da ressalva de concentração (que só aparece em outra linha da mesma tabela). A spine adota a leitura mais permissiva de FR-5 sem sinalizar a tensão com a redação mais estrita do §8 item 8, e sem garantir estruturalmente (ex.: via um único bloco de copy, não duas linhas de tabela) que "compatível" nunca apareça sem a ressalva grudada na mesma frase.
  Severidade: **corrigir agora** — é precisamente a nuance que a PM pediu para checar ("nunca chamar ação avulsa de 'adequada' a um perfil conservador"); o risco é um copywriter/dev usar a linha 1 da tabela isoladamente, já que ela existe como exemplo autônomo de "Do".

- **Núcleo emocional do brainstorm (ansiedade, culpa, medo de julgamento/vergonha, inércia) — preservado, não abstraído.** *(não é gap — registrado por transparência do processo de verificação)*
  Verificado: `DESIGN.md` e `EXPERIENCE.md` usam as palavras "ansiedade", "culpa", "vergonha", "medo" e "inércia" diretamente (não as trocam por linguagem genérica tipo "calm UX"), e reproduzem explicitamente o raciocínio de FR-4 ("mostrar 'quanto você perdeu' sem uma ação imediata reforça culpa em vez de resolvê-la") e o JTBD social do §2.1 ("substituir o cunhado que entende de bolsa" — citado quase verbatim em "Voice and Tone"/EXPERIENCE.md). O termo "permissão" do brainstorm ("dando permissão simples e justificada para agir") também reaparece em EXPERIENCE.md ("Segurança e permissão, não venda"), indicando que a spine consultou o brainstorm diretamente, não só o PRD. Nenhum gap encontrado neste eixo.

## Não encontrado como gap (verificado e coberto)

FR-1, FR-2, FR-3, FR-4 (exceto o ponto acima), FR-7, FR-9, FR-10, FR-11, FR-12, FR-13 têm cobertura fiel nas duas spines, incluindo nuances específicas como a ordem poupança-antes-do-CDI (FR-4), o estado "ainda não suportado neste perfil" (FR-2/Questão em Aberto #9), o limiar de frescor de dados de 1 dia útil (NFR), a rotulagem dado real vs. simulado (FR-13) e as três contra-métricas do §7 como guard-rails explícitos de design.
