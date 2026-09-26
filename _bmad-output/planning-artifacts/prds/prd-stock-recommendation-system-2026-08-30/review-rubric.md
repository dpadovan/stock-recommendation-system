# PRD Quality Review — Sistema de Recomendação de Ações para Investidores Leigos

## Overall verdict
This is a well-earned PRD: it has a real thesis ("o verdadeiro concorrente deste produto é a inércia"), traces every major cut (day trade, carteira diversificada, sinalização de venda) to a named trade-off rather than silence, and keeps a genuinely unresolved internal disagreement (Winston/Mary vs. John/Sally on ação avulsa, §8 item 4) visible instead of papering over it. The main risks are downstream, not strategic: two FRs (FR-6, FR-10) key their acceptance test on thresholds still to be defined, one FR consequence (FR-7) points to a cross-reference that doesn't resolve, and the LGPD/privacy concern — real given the product handles perfil de risco and financial values — is acknowledged but carries no interim stance at all, not even a placeholder consent requirement.

## Decision-readiness — strong
Trade-offs are named with what was given up, not just what was chosen. §6.2 states plainly that ação avulsa was "tomada para viabilizar o v1" while Winston and Mary "sinalizaram risco de reputação se um único ativo recomendado cair muito" — the dissent is preserved, not smoothed. §5 explains that cutting day trade is what "torna o cálculo de IR (FR-8) tratável," naming the complexity that was avoided rather than asserting the cut was costless. Open Question #4 goes further than most PRDs dare: it states the ação-avulsa-vs-carteira debate "não foi encerrado por consenso, apenas por decisão de escopo do Daniel" — an open question that is actually open, with the losing side's argument still recorded.

`[NOTE FOR PM]` callouts land on real tensions (§6.2, revisiting sinalização de saída and carteira diversificada) rather than safe checkpoints.

### Findings
- **medium** LGPD open question carries no interim position (§8 item 3) — See Scope honesty for full finding; noted here because it is the one place a decision-maker approving this PRD for build would need to see a stance, even provisional, before greenlighting registration/data-collection work (FR-1, FR-2).

## Substance over theater — strong
No persona theater: a single protagonist (Marta) carries all three JTBDs (§2.1) and all three UJs, and the JTBDs visibly drive features rather than sitting decoratively — the "social" JTBD ("uma justificativa que eu possa defender perante família/amigos") is the direct rationale for the trilha de transparência (§4.5) and the disclaimer framing (§4.7), not a generic aspiration.

No innovation theater: the Vision (§1) explicitly disclaims novelty in the analysis engine — "O diferencial não é o motor de análise em si — análise técnica e fundamentalista já existem em qualquer corretora" — and locates the actual bet in the "camada de redução de ansiedade." That is a falsifiable, specific claim, not boilerplate differentiation.

No Vision theater: the Vision names the actual competitor ("poupança," "cunhado que entende de bolsa") and the actual mechanism (semáforo/termômetro, monitoramento silencioso) — it could not be swapped into an unrelated fintech PRD without rewriting.

No NFR theater either, but only because there is essentially no NFR content to be theater — see Done-ness clarity.

## Strategic coherence — strong
The thesis (inertia, not lack of a tool, is the enemy) is stated once in §1 and then visibly organizes scope: the Painel Único is framed as "a manifestação direta da 'camada de redução de ansiedade'" (§4.2), FR-9's disclaimer literally exists to reduce "medo de julgamento" (§4.7), and FR-10/FR-11 exist to keep the panel calm by default (§4.8–4.9). This is not a backlog with headings — every Should-priority feature ties back to the anxiety-reduction bet.

Counter-metrics are genuinely counter, not decorative: SM-C1 explicitly says a rising rate of new recommendations "sugeriria o produto empurrando decisões, o oposto da tese" (§7) — a real check against the thesis being quietly abandoned for engagement.

### Findings
- **medium** SM-2 (30-day retention) sits in tension with the "silence is the product" thesis (§6, UJ-3: "o silêncio é o produto funcionando... nenhuma pressão para agir") — a product designed to not demand attention may not naturally produce return visits, and optimizing SM-2 upward could pull the product back toward the notification-heavy pattern §1 and UJ-3 are explicitly rejecting. *Fix:* either pair SM-2 with a counter-metric on notification/alert frequency (distinct from SM-C1, which covers new recommendations, not check-ins), or reframe SM-2 around passive engagement (e.g., panel opened without an active alert) so it doesn't reward re-introducing pressure.

## Done-ness clarity — adequate
Most FRs carry testable consequences with real bounds: FR-8's tax handling cites the exact R$20.000/mês isenção and the 15% swing-trade alíquota (§4.6), FR-4 specifies display order and disclaimer placement, FR-9 gives a concrete linguistic test ("não usa linguagem imperativa do tipo 'compre esta ação'"). This is the strongest part of the document.

Two FRs, however, key their pass/fail condition to thresholds the PRD defers, and one criterion is left vague without even being flagged as an assumption.

### Findings
- **medium** FR-6's "aviso de concentração excessiva" and FR-10's "evento relevante" have no defined trigger threshold (§4.4 FR-6: "`[ASSUMPTION: limiar de "excessivo" a definir]`"; §4.8 FR-10: "`[ASSUMPTION: limiar exato de "relevante" a definir]`"). Both are honestly flagged and carried into Open Question #5, so this isn't a scope-honesty problem — but as written, neither FR is currently story-ready: an engineer cannot write an acceptance test for "when does the concentration warning fire" or "when does the alert fire" without the threshold. *Fix:* keep the thresholds open if genuinely undecided, but mark FR-6 and FR-10 as blocked-on-threshold explicitly so story creation doesn't silently invent a number.
- **medium** FR-5's compatibility rule is unflagged but just as underspecified as the two assumptions above: "perfil conservador não recebe recomendação tipicamente associada a alta volatilidade" (§4.4) has no definition of what makes a recommendation "tipicamente associada a alta volatilidade," and no mention of how "horizonte de investimento" (curto/médio/longo) differentiates the recommendation at all — the FR states compatibility is required but not what drives it. *Fix:* either add an `[ASSUMPTION]` tag pointing to the same deferred-threshold treatment as FR-6/FR-10, or state the actual selection rule (even provisionally) so this isn't the one unflagged vague FR in an otherwise disciplined document.
- **low** No non-functional requirements section exists (performance, uptime, data-freshness, security bounds). FR-1's only nod to this is "senha abaixo de um mínimo de segurança definido na arquitetura" (§4.1), fully deferred. For a product whose core value proposition depends on data recency (5-year fundamentalist/technical series, FR-5/FR-7) and that stores financial profile data, the total absence of even placeholder bounds (e.g., how stale can price data be before the recommendation is considered unreliable — relevant given UJ-2's edge case about unavailable provider data) is a gap architecture will have to fill blind. *Fix:* add a short NFR stub with `[ASSUMPTION]` tags rather than leaving the section absent.

## Scope honesty — adequate
§5 (Não-Objetivos) does real work — every exclusion carries a reason, not just a label (day trade is cut because it's what "torna o cálculo de IR tratável," not just "out of scope"). The Assumptions Index (§9) round-trips cleanly against inline tags with one exception (see Mechanical notes). `[NOTE FOR PM]` callouts land on genuinely deferred decisions (§6.2).

### Findings
- **high** LGPD/privacy handling is named as "uma preocupação real" (§8 item 3) but the PRD takes no interim position whatsoever — not even a placeholder FR for consent capture, data retention default, or a ToS/privacy-policy acceptance step at signup. FR-1/FR-2 (cadastro e perfil) collect email, password, perfil de risco, and optionally a poupança balance with no mention of consent. Compliance items are usually fine to finish in architecture, but for a product processing sensitive financial data under Brazilian law, shipping an MVP with literally zero consent-related requirement in the PRD that governs FR-1/FR-2 is a real gap in what "done" means for those FRs, not just a detail deferred to a later phase. *Fix:* add a minimal FR (even `[ASSUMPTION]`-tagged) requiring consent/ToS acceptance at signup, and let architecture resolve the mechanics (encryption, retention period) as already planned.

## Downstream usability — strong
Glossary (§3) is used consistently — "Painel Único," "ação avulsa," "trilha de transparência," "aviso de concentração" all appear with stable casing and meaning everywhere they're used. FR/UJ/SM IDs are contiguous and unique (FR-1–FR-11, UJ-1–UJ-3, SM-1–SM-4 + SM-C1/C2). Every UJ has a named, contextualized protagonist (Marta) — no floating UJs.

### Findings
- **medium** Broken cross-reference inside a testable consequence: FR-7's consequence text says each indicator includes "a fonte do dado (ver §4.9 addendum de provedor)" (§4.5, line ~164) — but §4.9 is "Linguagem Visual Simplificada" (FR-11, semáforo/termômetro), not a data-provider addendum. The intended target is almost certainly Open Question #2 (§8) or the future `addendum.md`, not §4.9. As written, a reader following this reference to verify FR-7's "fonte do dado" requirement lands on the wrong section. *Fix:* repoint to "§8, Open Question #2 / addendum.md" or wherever the provider decision will actually live.

## Shape fit — strong
This is a consumer product with a meaningful UX bet (anxiety reduction as the differentiator), and the PRD is shaped accordingly: a single named protagonist (Marta) carries all three UJs with inline context, which is exactly where UJ density should concentrate for this product type — it is neither over-formalized (no persona-per-feature sprawl) nor under-formalized (a consumer product with zero UJs would have been a red flag; this has three, each load-bearing). The document is explicitly chain-top (§0: "orienta a construção do MVP... a quem vier a trabalhar em UX, arquitetura e implementação"), so the Downstream usability bar applies in full, and the PRD largely clears it.

One shape mismatch worth naming: this product sits adjacent to regulated investment advice and handles sensitive financial data, which normally calls for constraint traceability (compliance requirements mapped to FRs) alongside the UJ-driven consumer shape. FR-9 and §4.7 handle the "not regulated advice" constraint well; the LGPD/data-protection constraint does not get the same treatment (see Scope honesty finding above) — a genuine gap in an otherwise well-fitted shape.

## Mechanical notes
- **Assumptions Index section mismatch:** §9 lists "§4.6 (FR-6) — Limiar inicial de aviso de concentração..." but FR-6 is defined in §4.4 (Motor de Recomendação), not §4.6 (Cálculo de Rendimento Líquido e Imposto de Renda, which houses FR-8). The inline `[ASSUMPTION]` tag itself is correctly placed in §4.4; only the index's section pointer is wrong. Low-severity since the FR number still makes it findable, but worth a one-line fix.
- **FR-7 cross-reference** ("§4.9 addendum de provedor") does not resolve to relevant content — see Downstream usability finding above.
- **"IR" abbreviation** is used extensively (Glossário, FR-8, FR-9, §5, §8) but is never given its own Glossário line — it's only implicitly defined via the "DARF" and "Ganho de capital" entries assuming the reader already expands "Imposto de Renda (IR)" from §1. Low severity, easy fix: add a one-line Glossário entry for IR.
- Assumptions Index otherwise round-trips cleanly: all other inline `[ASSUMPTION]` tags (§4.1, §4.4, FR-8/§4.6, FR-10/§4.8, §5, §7) are indexed, and all index entries have a matching inline tag.
- No unresolved `[NOTE FOR PM]` orphans — both instances (§4.2, §6.2×2) are self-contained and don't require a missing companion section.

## Finding counts
- Critical: 0
- High: 1
- Medium: 5
- Low: 2
