# Adversarial Review — PRD Sistema de Recomendação de Ações

**Documento revisado:** `prd.md` (status: draft, atualizado 2026-09-05)
**Data da revisão:** 2026-09-05
**Postura:** adversarial — o objetivo é encontrar onde este PRD está errado ou perigosamente incompleto, não validar o que já está bom.

---

## CRÍTICO

### C1. O enquadramento "educacional" provavelmente não protege o produto de ser classificado como consultoria/análise de valores mobiliários regulada pela CVM

**Localização:** §4.7 (FR-9), §5 (Não-Objetivos), §2.2, Glossário ("Perfil de risco")

**Cenário de falha concreto:**
O produto (a) coleta perfil de risco e horizonte declarados pelo usuário (FR-2) — isso é, na prática, uma coleta de suitability; (b) usa esse perfil para filtrar e emitir **uma recomendação de compra de uma ação específica** compatível com aquele perfil (FR-5, critério testável explícito: "A recomendação é compatível com o perfil de risco e horizonte declarados"); (c) associa essa recomendação à conta identificada do usuário e monitora continuamente (FR-10). Isso é exatamente o fato gerador de duas atividades reguladas pela CVM:

- **Resolução CVM 20/2021** define "consultoria de valores mobiliários" como a prestação de recomendação personalizada de investimento, considerando o perfil do cliente. É precisamente o que FR-2 + FR-5 fazem juntos. Consultores de valores mobiliários precisam de registro na CVM (pessoa física com certificação CEA/CFP ou pessoa jurídica com diretor responsável certificado), dever fiduciário e guarda de registros.
- **Resolução CVM 19/2021** regula "análise de valores mobiliários" — quem emite relatório com recomendação de compra/venda sobre um valor mobiliário específico precisa ser analista certificado (CNPI/APIMEC) ou entidade registrada, com responsável técnico.

O PRD trata isso como resolvido só porque a "voz da CVM" foi ouvida no brainstorming e o produto "se posiciona como educacional" (§4.7) e evita linguagem imperativa ("os dados indicam" em vez de "compre") (FR-9). Isso é insuficiente: a CVM analisa **substância, não rótulo ou verbo escolhido**. Um mecanismo que faz suitability + recomendação individualizada e contínua de um ativo específico não deixa de ser consultoria regulada por trocar "compre" por "os dados indicam". Esse é um padrão que a CVM já fiscalizou em outras fintechs de "educação financeira" que, na prática, operavam como consultoria não registrada.

**Risco de negócio:** notificação/multa da CVM, obrigação de descontinuar o produto ou reestruturá-lo sob pressão, e (mais grave) o produto sendo lançado sobre uma premissa jurídica que não se sustenta — todo o trabalho de UX/engenharia em cima de FR-5/FR-6/FR-7 pode precisar ser refeito.

**Correção sugerida:** Antes de escrever uma linha de código, buscar parecer jurídico especializado em regulação CVM para fintechs. Alternativas de produto a avaliar: (a) remover a personalização por perfil da recomendação de um ativo específico (ex: mostrar a mesma lista/screening para todos e deixar o usuário filtrar) — isso pode tirar o produto do escopo de "consultoria personalizada"; (b) registrar-se como consultor de valores mobiliários (custo/tempo real, mas viabiliza o modelo atual); (c) fazer parceria com uma entidade já registrada. O PRD precisa de uma seção explícita de enquadramento regulatório com decisão tomada, não uma frase de disclaimer.

---

### C2. FR-8 (cálculo de IR) está incorreto em dois pontos estruturais — a promessa central do produto ("cálculo correto do IR") não se sustenta como especificada

**Localização:** §4.6, FR-8, Glossário ("Isenção fiscal"), §5 (justificativa de exclusão de day trade)

**Cenário de falha concreto 1 — isenção de R$20.000/mês não é calculável pelo sistema:**
A isenção de IR sobre ganho de capital em vendas de ações no mercado à vista é apurada sobre o **total de vendas (não o lucro) de TODAS as ações vendidas por aquele CPF naquele mês-calendário**, somando todas as corretoras e todos os ativos — não é "R$20.000 por operação" nem "por ação recomendada". O produto não se integra a corretoras nem a Open Finance (§5, item explícito de não-escopo) e nunca soube de nenhuma outra venda do usuário. Portanto o sistema **estruturalmente não tem como saber** se o total de vendas do mês do usuário (nesta e em outras posições, nesta e em outras corretoras) ultrapassa R$20.000 — logo não pode calcular corretamente se a isenção se aplica. Apresentar "lucro líquido" considerando essa isenção como se fosse determinável a partir dos dados que o produto tem é uma conta que parece precisa mas é, na prática, um chute — o oposto do que a Visão promete ("cálculo correto do Imposto de Renda").

**Cenário de falha concreto 2 — a lógica "uma ação por vez evita compensação de prejuízos" é factualmente errada:**
§5 justifica cortar day trade dizendo que isso torna FR-8 tratável "evitando... compensação de prejuízos entre modalidades", e FR-8 assume `[ASSUMPTION — usuário não terá múltiplas operações simultâneas no v1, já que a recomendação é de uma ação por vez]`. Isso confunde "simultâneo" com "ao longo do tempo". A legislação de IR sobre ganho de capital em swing trade **apura o resultado mensalmente, líquido de prejuízos anteriores compensáveis** (Lei 11.033/2004 e regulamentação da RFB) — inclusive entre ativos diferentes e entre meses diferentes (prejuízo é compensável indefinidamente contra ganhos futuros de mesma natureza). A UJ-3 do próprio PRD ("Marta volta duas semanas depois... recomendação anterior") deixa explícito que o produto pretende operar em ciclos sequenciais ao longo do tempo — ou seja, ao longo de meses um usuário real terá múltiplas operações de compra/venda (uma de cada vez, mas várias no total), com potencial de prejuízo em uma e ganho em outra, exigindo compensação. Excluir day trade não elimina essa necessidade; ela existe inteiramente dentro do próprio regime de swing trade que o v1 cobre.

**Risco de negócio:** o produto entrega ao usuário leigo — que por definição não tem como conferir a conta — um número de "quanto vai pagar de imposto" que pode estar sistematicamente errado, seja por desconhecer vendas concorrentes do mês, seja por ignorar prejuízos compensáveis de operações anteriores. Isso não é um detalhe técnico: é dano financeiro direto ao usuário (declarar/recolher IR errado gera multa e juros da Receita) e ao mesmo tempo é o diferencial mais citado do produto (§1: "cálculo correto do Imposto de Renda sobre ganho de capital").

**Correção sugerida:** (a) Reformular FR-8 para deixar explícito, na tela, que a isenção de R$20.000 depende do total de vendas do usuário em ações no mês **em todas as corretoras**, e que o sistema só pode estimar considerando esta operação isoladamente — apresentar dois cenários (com e sem isenção) em vez de uma única "estimativa" que parece definitiva; (b) reconhecer explicitamente que compensação de prejuízos entre meses É necessária mesmo em swing-trade-only e decidir conscientemente se isso entra no v1 ou é uma limitação comunicada com clareza ao usuário (hoje o PRD nega que o problema exista, em vez de escopar a ausência dele deliberadamente).

---

### C3. Contradição de suitability: recomendar uma ação individual concentrada é inadequado ao próprio perfil-alvo declarado do produto ("conservador/longo prazo")

**Localização:** §1 (Visão), §2.1, FR-5, FR-6

**Cenário de falha concreto:**
A Visão (§1) e as personas (Marta, UJ-1/UJ-2) definem o público-alvo do v1 como "perfil conservador/longo prazo". Em qualquer framework de suitability (inclusive o da própria CVM, Resolução CVM 30 sobre suitability, e as práticas ANBIMA) uma posição concentrada em **uma única ação** carrega risco idiossincrático (risco específico da empresa) que nenhuma metodologia de perfil considera adequado a um perfil conservador — independentemente de qual ação específica seja escolhida ou de quão "sólidos" sejam seus fundamentos. Perfil conservador é tradicionalmente associado a renda fixa/diversificação, não a stock picking de ativo único. FR-5 tenta mitigar isso dizendo "perfil conservador não recebe recomendação tipicamente associada a alta volatilidade" — mas isso trata volatilidade da ação escolhida como o único fator de risco, ignorando que **concentração em ativo único é, ela mesma, o fator de risco dominante**, não a volatilidade histórica do papel. FR-6 (aviso de concentração) reconhece o problema ao pedir confirmação de percentual alocado — mas depois de já ter entregue "a" recomendação como se fosse apropriada para aquele perfil. O próprio §8, item 4, registra que Winston e Mary já sinalizaram esse risco reputacional em party-mode e que o debate não foi encerrado por consenso, só por decisão de escopo do Daniel — ou seja, o time já sabe que isso é discutível e decidiu seguir mesmo assim, sem uma resposta de produto para o problema central: o que a Marta conservadora faz com o restante do dinheiro, e por que uma ação avulsa é "compatível" com o perfil dela.

**Risco de negócio:** se a ação recomendada cair de forma expressiva (e ativos individuais o fazem, mesmo "sólidos" — vide qualquer blue-chip da B3 em crise setorial ou idiossincrática), o dano de reputação recai exatamente sobre a promessa central do produto ("confiança para decidir com segurança"). Pior: como o perfil é autodeclarado como conservador, um evento de perda relevante aqui é também o cenário de maior risco de reclamação regulatória (CVM) e de mídia ("app disse que era seguro para investidor conservador e a pessoa perdeu dinheiro").

**Correção sugerida:** decisão explícita de produto — ou (a) o v1 não deveria oferecer stock-picking de ativo único a perfil conservador (recomendar apenas para moderado/arrojado, ou recomendar fundos/ETFs para conservador — o que reabre a Questão em Aberto #4), ou (b) se a decisão de ação avulsa para todos os perfis for mantida, isso precisa vir acompanhado de um enquadramento textual muito mais forte do que hoje, deixando claro que uma ação avulsa NUNCA é, por definição, um investimento conservador, e tratando isso como uma limitação assumida do v1 e não como algo "compatível" com o perfil.

---

## ALTO

### H1. Não existe FR de registro de posição/compra — isso quebra a base de FR-8, FR-10 e FR-11

**Localização:** FR-6, FR-8, FR-10, FR-11, UJ-3

**Cenário de falha concreto:** UJ-3 descreve o painel mostrando "o status atualizado da recomendação anterior" (FR-11) e alertando sobre "mudança relevante" (FR-10). FR-8 fala em "lucro líquido" da operação. Nenhum FR do documento captura **se, quando e a que preço o usuário efetivamente comprou** a ação recomendada — FR-6 só captura a intenção de alocação percentual, não uma confirmação de compra real com preço/data/quantidade. Sem isso, existem duas leituras possíveis e o PRD não escolhe nenhuma: (1) o "status" e o "lucro líquido" são calculados sobre a ação em abstrato (preço de quando a recomendação foi feita), o que **não é o retorno real do usuário** se ele comprou em outro momento/preço, tornando a promessa de "cálculo correto" genérica e potencialmente enganosa; ou (2) existe uma etapa de confirmação de compra que simplesmente não foi especificada como FR, e toda a lógica de FR-8/FR-10/FR-11 está subespecificada até isso ser resolvido.

**Correção sugerida:** adicionar um FR explícito de "registro de posição" (o usuário confirma que comprou, quando, a que preço e quantidade) antes que FR-10/FR-11 façam sentido como "status personalizado", e deixar claro em FR-8 se a projeção de lucro líquido usa o preço da recomendação ou um preço de entrada real informado pelo usuário.

---

### H2. LGPD é reconhecido como risco real e integralmente empurrado para "fase de arquitetura" sem nenhum FR placeholder — para um produto que agora tem contas e dados financeiros sensíveis de pessoas identificadas

**Localização:** §8, item 3

**Cenário de falha concreto:** o produto passou a ter cadastro (FR-1), perfil de risco, horizonte, e valor de patrimônio parado na poupança (FR-2) — todos vinculados a uma conta com e-mail. Isso é dado financeiro pessoal sob LGPD, tratado com base em consentimento e finalidade específica, com direitos do titular (acesso, correção, exclusão, portabilidade) que precisam existir desde o v1, não como afterthought de arquitetura. O PRD reconhece isso em uma linha ("reconhecido como uma preocupação real... mas resolução deferida") e não define nenhum FR relacionado a consentimento no cadastro, exclusão de conta, ou retenção de dados. Isso é diferente de "detalhe técnico de implementação" (ex: qual algoritmo de hash de senha) — consentimento e direito de exclusão são requisitos de produto, testáveis, e sua ausência é uma lacuna do PRD, não da arquitetura.

**Correção sugerida:** adicionar pelo menos FRs mínimos de v1: consentimento explícito no cadastro sobre uso de dados financeiros/perfil de risco; capacidade de o usuário excluir a conta e ver seus dados apagados; política de retenção declarada. Isso não precisa ser extenso, mas precisa existir como requisito, não como nota de rodapé em "questões em aberto".

---

### H3. Risco de provedor de dados tratado como decisão técnica trivial, quando é uma dependência de negócio potencialmente bloqueante/cara

**Localização:** §8, item 2

**Cenário de falha concreto:** o produto promete 5 anos de histórico técnico e fundamentalista para ações da B3, atualizado continuamente (FR-10 monitora "mudança relevante" nos indicadores). O PRD cita "Status Invest, dados da CVM" como exemplos, mas: (a) Status Invest é um site de consumo, não uma API comercial com contrato/SLA — usá-lo via scraping para um produto comercial que cobra e opera continuamente é um risco de ToS e de continuidade (o provedor pode bloquear o acesso a qualquer momento, derrubando o produto); (b) cotações em tempo real/quase real de bolsa têm **custo de licenciamento de redistribuição junto à B3** (contratos de vendor de market data) que não é mencionado em nenhum lugar; (c) montar 5 anos de dados fundamentalistas limpos para centenas de tickers a partir de fontes públicas da CVM (ITR/DFP) é um projeto de engenharia de dados não trivial, não uma escolha de "qual API usar". Tratar isso como uma linha em "questões em aberto" subestima o quanto essa decisão pode determinar se o MVP é viável dentro do orçamento/prazo esperado.

**Correção sugerida:** investigar e precificar pelo menos uma opção de provedor de dados comercial licenciado antes de comprometer o roadmap do MVP a este escopo de features; tratar isso como um risco de viabilidade do projeto, não como um detalhe de arquitetura a ser resolvido depois.

---

### H4. FR-2 coleta perfis (moderado/arrojado) e horizontes (curto/médio prazo) que o resto do PRD nunca trata — escopo real mais estreito do que o formulário sugere

**Localização:** §1 (Visão), FR-2, FR-5

**Cenário de falha concreto:** a Visão declara foco explícito em "perfil conservador/longo prazo" (§1) e todas as personas/jornadas (UJ-1 a UJ-3) são sobre a Marta, conservadora. Porém FR-2 oferece ao usuário as opções moderado e arrojado, e curto/médio prazo, como se fossem igualmente suportadas. Nenhuma outra parte do documento (motor de recomendação, critérios de aceite, métricas) trata do que acontece para esses outros perfis — a lógica de recomendação para perfil arrojado/curto prazo nunca é descrita nem validada com uma jornada própria. Isso é uma lacuna silenciosa de escopo: ou o produto coleta dados que não usa de verdade (a recomendação pode ficar mal calibrada ou ser tratada como "modo não suportado" sem que isso esteja dito em lugar nenhum), ou o escopo do motor de recomendação é maior do que o resto do PRD assume — nos dois casos, há inconsistência entre o que o formulário promete capturar e o que o produto de fato entrega.

**Correção sugerida:** decidir e declarar explicitamente se v1 atende só conservador/longo-prazo (e então restringir as opções do formulário de FR-2 a isso, ou avisar que outros perfis "ainda não são suportados"), ou se atende todos os perfis (e então escrever pelo menos critérios de aceite mínimos para moderado/arrojado).

---

## MÉDIO

### M1. Critério de aceite de FR-5 ("recomendação compatível com o perfil") não é de fato testável como escrito

**Localização:** FR-5, "Consequências (testáveis)"

O critério diz "a recomendação é compatível com o perfil de risco e horizonte declarados (ex: perfil conservador não recebe recomendação tipicamente associada a alta volatilidade)" — isso não é testável sem uma tabela/regra explícita de quais indicadores fundamentalistas/padrões técnicos qualificam uma ação para cada perfil. Hoje é um exemplo ilustrativo, não uma regra verificável por QA. **Correção:** definir (mesmo que na fase seguinte de arquitetura/dados) uma matriz mínima de regras de elegibilidade por perfil, e referenciar essa matriz como o critério de aceite real.

### M2. "Percentual do total" em FR-6 não define o que é "total"

**Localização:** FR-6, Glossário ("Aviso de concentração")

O aviso de concentração pede que o usuário informe "qual percentual do seu total pretende alocar" — mas "total" nunca é definido: é o valor informado como poupança parada (FR-2)? É o patrimônio total do usuário (que o produto não conhece, já que não há integração com Open Finance)? É o valor disponível para investir agora? Como essa é a peça central da proteção contra concentração excessiva — descrita como "requisito não-negociável" — a ambiguidade do denominador enfraquece a própria salvaguarda que o FR tenta implementar. **Correção:** definir explicitamente a base de cálculo do percentual (provavelmente: valor total que o usuário planeja ter investido em renda variável, autoinformado, distinto do valor da poupança).

### M3. FR-10 e FR-11 são "Should" mas aparecem misturados como itens flat de "Dentro do Escopo" do MVP, sem distinção de prioridade

**Localização:** §4.8, §4.9 (marcados *Should*), §6.1

§6.1 lista FR-10 e FR-11 lado a lado com os FRs "Must" (FR-1 a FR-9) sob o mesmo título "Dentro do Escopo", sem sinalizar que são de prioridade inferior. FR-10 em particular exige infraestrutura não trivial (monitoramento contínuo/job de background para detectar "mudança relevante" em preço/indicadores) — é o tipo de item que sob pressão de prazo é cortado primeiro, mas o PRD não deixa claro que isso é aceitável nem qual seria o comportamento do produto sem ele (o Painel Único, tal como descrito em FR-3, referencia explicitamente "a recomendação atual" e não incorpora um plano B claro caso FR-10/FR-11 sejam cortados). **Correção:** marcar explicitamente em §6.1 quais itens são Must vs Should dentro do MVP, e declarar o comportamento mínimo aceitável do Painel Único se Should for cortado.

### M4. Nenhum requisito de segurança de conta (recuperação de senha, MFA, bloqueio por tentativas) apesar de a conta guardar dados financeiros sensíveis

**Localização:** §4.1, FR-1

FR-1 cobre apenas criação de conta com e-mail/senha e um "mínimo de segurança" de senha a definir na arquitetura. Não há FR de recuperação de senha (fluxo básico esperado em qualquer produto com login), nem menção a limitação de tentativas de login/lockout, nem a necessidade (ou não-necessidade justificada) de segundo fator, apesar de a conta conter perfil de risco e valores financeiros — dado sensível o suficiente para justificar pelo menos uma decisão consciente sobre MFA, não silêncio total. **Correção:** adicionar FR mínimo de recuperação de senha (indispensável para v1 funcionar de verdade) e registrar como assumption a decisão sobre MFA.

---

## BAIXO

### L1. "DARF mensal" (FR-8) é uma formulação um pouco imprecisa

**Localização:** FR-8

O DARF de ganho de capital não é uma obrigação mensal fixa e recorrente — é devido apenas nos meses em que houve venda com ganho tributável líquido de isenção/prejuízo compensável, até o último dia útil do mês seguinte. A redação atual ("necessidade de DARF mensal quando aplicável") já tem o "quando aplicável" que mitiga isso, mas vale reforçar no texto do produto para não passar a impressão de uma obrigação recorrente automática.

### L2. Tratamento tributário de dividendos/JCP não é mencionado na "projeção de retorno" (FR-8)

**Localização:** FR-8

Não fica claro se "projeção de retorno médio" inclui apenas variação de preço ou também proventos (dividendos, isentos de IR na pessoa física, vs. JCP, tributados a 15% na fonte). Se a intenção é só ganho de capital na venda, está implícito mas não explícito; se inclui proventos, falta o tratamento fiscal diferenciado. Vale uma frase explícita definindo o escopo da projeção.

### L3. PRD não distingue CVM (vinculante) de ANBIMA (autorregulação por adesão) — risco de o time mirar no órgão errado

**Localização:** §4.7, Glossário

O documento (e o brief de revisão que motivou esta análise) trata CVM e ANBIMA como preocupações do mesmo tipo. Só a CVM tem poder regulatório vinculante sobre a atividade aqui descrita (consultoria/análise de valores mobiliários); ANBIMA é autorregulação de instituições associadas/certificação de profissionais e só vincula o produto se ele buscar essa certificação ou se associar a uma instituição membro. Vale deixar isso explícito no PRD para não gerar falsa sensação de conformidade ("seguimos os códigos ANBIMA") quando o requisito legal real vem da CVM.

---

## Resumo

| Severidade | Qtde |
|---|---|
| Crítico | 3 |
| Alto | 4 |
| Médio | 4 |
| Baixo | 3 |
| **Total** | **14** |

Os três achados críticos (C1–C3) são interdependentes: a classificação regulatória (C1) e a contradição de suitability (C3) atacam a premissa central do produto — que uma recomendação personalizada e individualizada de ação avulsa, rotulada "educacional", é segura de lançar como está — e o erro de cálculo de IR (C2) ataca o diferencial mais citado do produto no próprio documento. Recomenda-se resolver C1 (parecer jurídico + decisão de enquadramento) antes de qualquer investimento adicional em engenharia, já que ele pode invalidar a arquitetura de FR-5/FR-6/FR-7 como um todo.
