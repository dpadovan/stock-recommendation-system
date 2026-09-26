---
title: Sistema de Recomendação de Ações para Investidores Leigos
status: final
created: 2026-08-30
updated: 2026-09-05
---

> **`[NOTE FOR PM]` Riscos aceitos conscientemente:** este rascunho passou por revisão de qualidade e revisão adversarial (`review-rubric.md`, `review-adversarial.md`, nesta mesma pasta), que levantaram 3 achados críticos. Os três foram discutidos e decididos pelo Daniel — ver §8, itens 6, 7 e 8: (6) risco de classificação regulatória CVM aceito conscientemente, sem parecer jurídico antes de codar; (7) limitações do cálculo de IR comunicadas com transparência ao usuário, aceitas para o v1; (8) ação avulsa mantida para todos os perfis, mas sem ser tratada como "adequada" a conservador. Nenhum é mais um bloqueador de engenharia.

# PRD: Sistema de Recomendação de Ações para Investidores Leigos
*Título de trabalho — confirmar.*

## 0. Propósito do Documento

Este PRD orienta a construção do MVP web de um sistema que recomenda ações individuais da bolsa brasileira para investidores leigos, combinando análise técnica e fundamentalista. É destinado ao próprio Daniel (PM/fundador) e a quem vier a trabalhar em UX, arquitetura e implementação. Este documento constrói sobre uma sessão de brainstorming anterior (`_bmad-output/brainstorming/brainstorm-recomendacao-acoes-leigos-2026-08-25/brainstorm-intent.md`) e uma discussão multi-perspectiva (party-mode) que gerou decisões adicionais de escopo — ambos referenciados abaixo, não duplicados aqui. Vocabulário fica ancorado no Glossário (§3); features agrupam Requisitos Funcionais (FRs) numerados globalmente; suposições ficam marcadas inline como `[ASSUMPTION]` e indexadas em §9.

## 1. Visão

Investidores leigos brasileiros deixam dinheiro parado na poupança não por falta de informação, mas por uma mistura de ansiedade, culpa e medo de julgamento — o verdadeiro concorrente deste produto é a inércia, não a falta de uma ferramenta de análise. Este sistema existe para dar a essas pessoas confiança para agir: ele analisa ações da bolsa combinando dados fundamentalistas (balanços e indicadores de até 5 anos) e técnicos (gráficos de até 5 anos), e entrega uma recomendação única, explicada em linguagem simples, com uma trilha de transparência completa sobre os dados que a sustentam.

O diferencial não é o motor de análise em si — análise técnica e fundamentalista já existem em qualquer corretora. É a **camada de redução de ansiedade**: um painel único e calmo, linguagem visual simples (semáforo/termômetro em vez de números crus), monitoramento contínuo silencioso, e um enquadramento explicitamente educacional que substitui o "cunhado que entende de bolsa" por uma explicação rastreável que a pessoa pode usar para justificar sua própria decisão.

O MVP é **web, sem app mobile**, com cadastro simples (e-mail/senha), foco em perfil conservador/longo prazo, recomendação de **ação avulsa** (não carteira diversificada) com aviso de concentração, e cálculo correto do Imposto de Renda sobre ganho de capital.

## 2. Usuário-Alvo

### 2.1 Jobs To Be Done

- **Emocional:** "Quero confiança para decidir, para não sentir medo de perder dinheiro nem vergonha de não entender."
- **Social:** "Quero uma justificativa que eu possa defender perante família/amigos, para não depender só do palpite do cunhado que 'entende de bolsa'."
- **Funcional:** "Quero saber em que ação investir, quanto devo esperar de retorno e quanto vou pagar de imposto, sem precisar aprender a interpretar balanço ou gráfico sozinha."

### 2.2 Não-Usuários (v1)

- Day traders e investidores de perfil arrojado que buscam operações de curtíssimo prazo — fora de escopo (ver §6.2).
- Investidores que já têm carteira diversificada e querem gestão/rebalanceamento multi-ativo — v1 recomenda ação avulsa, não carteira (ver §8, Open Question).
- Quem busca consultoria de investimento regulamentada e personalizada — o produto é educacional/informativo, não consultoria (ver §4.7).

### 2.3 Jornadas-Chave do Usuário

- **UJ-1. Marta cria sua conta e define seu perfil.**
  - **Persona + contexto:** Marta, leiga em investimentos, tem dinheiro parado na poupança há anos por medo e falta de conhecimento. Chegou ao site por indicação de alguém.
  - **Estado de entrada:** não autenticada, acessando pelo navegador (desktop ou mobile web).
  - **Caminho:** Acessa a página inicial → cria conta com e-mail e senha → responde poucas perguntas objetivas (perfil de risco: conservador/moderado/arrojado; horizonte de investimento: curto/médio/longo prazo) → informa opcionalmente quanto tem hoje parado na poupança.
  - **Clímax:** ao concluir, cai direto no Painel Único (FR-3) — nenhuma etapa intermediária, nenhum formulário longo.
  - **Resolução:** Marta está no painel, pronta para ver o UJ-2.
  - **Edge case:** se ela pular o valor da poupança, a tela de comparação (FR-4) usa um valor de exemplo ilustrativo e a convida a inserir o valor real depois.

- **UJ-2. Marta enfrenta o custo de não ter feito nada — e recebe uma recomendação.**
  - **Persona + contexto:** mesma Marta, agora dentro do Painel Único pela primeira vez.
  - **Estado de entrada:** autenticada, perfil já preenchido (UJ-1).
  - **Caminho:** o painel abre mostrando, em destaque, quanto o valor informado por ela já deixou de render comparado à poupança e, logo abaixo, ao CDI (FR-4) — com o disclaimer educacional embutido no texto, não em rodapé (FR-9). Na sequência, vê **uma** recomendação de ação, compatível com seu perfil conservador/longo prazo, com explicação resumida do porquê (FR-5). Antes de qualquer ação, o sistema pede que ela confirme qual percentual do total pretende alocar ali, com um aviso claro de concentração (FR-6).
  - **Clímax:** ela expande a recomendação e vê a trilha de transparência — quais indicadores fundamentalistas e quais padrões gráficos pesaram (FR-7) — e a projeção de retorno médio (curto/médio/longo prazo) já líquida de IR estimado (FR-8).
  - **Resolução:** Marta sai da sessão sem ter comprado nada ainda, mas entendendo exatamente por que aquela ação foi sugerida e quanto ela pagaria de imposto se decidisse agir — a decisão continua sendo dela.
  - **Edge case:** se os dados do provedor financeiro estiverem indisponíveis para o ativo, o painel comunica isso explicitamente em vez de mostrar uma recomendação incompleta ou desatualizada.

- **UJ-3. Marta volta duas semanas depois.**
  - **Persona + contexto:** já tem uma recomendação registrada; não é notificada com frequência.
  - **Estado de entrada:** autenticada, retornando ao painel.
  - **Caminho:** login → painel mostra o status atualizado da recomendação anterior em linguagem visual simples (semáforo/termômetro, FR-11) — sem pedir nenhuma ação se estiver tudo dentro do esperado.
  - **Clímax:** o silêncio é o produto funcionando — nenhum alerta, nenhuma pressão para agir.
  - **Resolução:** ela fecha o painel tranquila; só recebe um alerta ativo se algo relevante mudou (FR-10).

## 3. Glossário

- **Investidor leigo** — usuário-alvo do produto: sem formação ou experiência prévia em mercado financeiro.
- **Perfil de risco** — classificação declarada pelo usuário: conservador, moderado ou arrojado. Determina quais recomendações o Motor de Recomendação (FR-5) pode sugerir.
- **Horizonte de investimento** — janela declarada pelo usuário: curto, médio ou longo prazo.
- **Análise fundamentalista** — análise de balanços e indicadores financeiros de uma empresa (até 5 anos de histórico).
- **Análise técnica** — análise de padrões em gráficos de preço de uma ação (até 5 anos de histórico).
- **Ação avulsa** — recomendação de uma única ação, em oposição a uma carteira diversificada de múltiplos ativos. Escopo do v1 (ver §8).
- **Aviso de concentração** — confirmação obrigatória, antes de qualquer recomendação, do percentual do total que o usuário pretende alocar naquele ativo.
- **Trilha de transparência** — exposição explícita de quais dados e indicadores (fundamentalistas e técnicos) fundamentaram uma recomendação específica.
- **IR (Imposto de Renda)** — tributo federal incidente sobre o ganho de capital (ver abaixo) na venda de ações.
- **Ganho de capital** — lucro obtido na venda de uma ação, base de cálculo do Imposto de Renda (IR).
- **DARF** — Documento de Arrecadação de Receitas Federais, usado para recolher o IR sobre ganho de capital.
- **Day trade / Swing trade** — operações de compra e venda no mesmo dia (day trade) vs. em dias diferentes (swing trade); alíquotas de IR distintas (20% e 15% respectivamente). Day trade está fora do escopo do v1 (ver §6.2).
- **Isenção fiscal (vendas)** — isenção de IR sobre ganho de capital para vendas de até R$20.000/mês no mercado à vista de ações.
- **CDI** — Certificado de Depósito Interbancário; taxa de referência usada como benchmark de rentabilidade de renda fixa e como segundo ponto de comparação na Tela "Quanto Você Perdeu Parado" (FR-4), depois da poupança.
- **JCP (Juros sobre Capital Próprio)** — forma de remuneração ao acionista, tributada a 15% na fonte (diferente de dividendos, isentos de IR na pessoa física); fora do escopo da projeção de retorno no v1 (ver FR-8).
- **Painel Único** — a interface central do produto: uma única tela onde o usuário acessa perda por inércia, recomendação, trilha de transparência e projeção.

## 4. Features

### 4.1 Cadastro e Perfil do Investidor

**Descrição:** Cadastro simples via e-mail e senha `[ASSUMPTION: sem login social/OAuth na v1]`. Após criar a conta, o usuário define perfil de risco e horizonte de investimento em poucas perguntas objetivas, e opcionalmente informa quanto tem hoje parado na poupança. Realiza UJ-1.

**Requisitos Funcionais:**

#### FR-1: Cadastro via e-mail e senha
Um visitante pode criar uma conta usando e-mail e senha. Realiza UJ-1.

**Consequências (testáveis):**
- Conta não é criada com e-mail já cadastrado ou senha abaixo de um mínimo de segurança definido na arquitetura.
- Após cadastro bem-sucedido, o usuário é autenticado automaticamente e segue direto para FR-2.
- Existe um fluxo de recuperação de senha por e-mail. `[ASSUMPTION: autenticação de dois fatores (MFA) fica fora do v1 — decisão consciente, dado que a conta guarda perfil de risco e valores financeiros, não omissão]`.

#### FR-12: Consentimento e direitos sobre dados pessoais
Um visitante só conclui o cadastro (FR-1) após aceitar explicitamente o uso de seus dados financeiros e de perfil de risco para gerar recomendações. Um usuário autenticado pode solicitar a exclusão da própria conta. Realiza UJ-1.

**Consequências (testáveis):**
- Cadastro exibe e exige aceite de um texto específico sobre uso de dados financeiros (não um checkbox genérico de "termos de uso").
- Usuário autenticado encontra, a partir do Painel Único, uma opção para excluir a conta e os dados associados.
- Criptografia em repouso, período de retenção exato e mecanismos de portabilidade ficam para a fase de arquitetura — este FR cobre apenas a existência do consentimento e do direito de exclusão como requisito de produto, não a implementação técnica.

#### FR-2: Definição de perfil de risco e horizonte de investimento
Um usuário autenticado pode declarar seu perfil de risco (conservador/moderado/arrojado) e horizonte de investimento (curto/médio/longo prazo), e opcionalmente o valor parado na poupança. Realiza UJ-1.

**Consequências (testáveis):**
- Perfil e horizonte ficam associados à conta e persistem entre sessões.
- Usuário pode editar essas informações depois, a partir do Painel Único.
- `[ASSUMPTION: o formulário coleta todos os perfis (conservador/moderado/arrojado) e horizontes (curto/médio/longo) para não travar o modelo de dados, mas o Motor de Recomendação (FR-5) no v1 só está calibrado para perfil conservador + horizonte longo prazo — consistente com a Visão (§1). Um usuário que declarar moderado, arrojado, curto ou médio prazo vê "ainda não suportado neste perfil" em vez de uma recomendação mal calibrada. Ampliar cobertura é Questão em Aberto #9]`.

### 4.2 Painel Único (Dashboard)

**Descrição:** Interface central do produto — uma única tela onde o usuário acessa tudo: comparação de perda por inércia, recomendação atual, trilha de transparência, projeção de retorno e guia rápido de uso. Realiza UJ-1, UJ-2, UJ-3. É a manifestação direta da "camada de redução de ansiedade" descrita na Visão (§1): a interface deve se manter calma por padrão, e só chamar atenção quando algo relevante de fato mudou.

**Requisitos Funcionais:**

#### FR-3: Painel único consolidado
Um usuário autenticado pode ver, em uma única tela, sem navegação entre páginas separadas: a comparação de perda por inércia (FR-4), a recomendação atual (FR-5, FR-6), a trilha de transparência (FR-7), a projeção de retorno e impostos (FR-8) e um guia rápido de uso.

**Consequências (testáveis):**
- Nenhuma dessas informações exige sair do painel para outra página — expansão/detalhe acontece inline (ex: acordeões, modais).
- Na primeira visita, um guia rápido (tour inline) apresenta as seções do painel; não reaparece em visitas seguintes salvo se o usuário pedir para revê-lo.

**Notas:** `[NOTE FOR PM]` o formato exato do guia rápido (tour com destaques na tela vs. vídeo curto) é uma decisão de UX, não travada aqui.

### 4.3 Tela "Quanto Você Perdeu Parado"

**Descrição:** Primeiro conteúdo que o usuário vê no Painel Único após o cadastro: quanto o valor informado como parado na poupança deixou de render, comparado primeiro à poupança e depois ao CDI, no período. Usa as mesmas séries históricas já exigidas pelo Motor de Recomendação (FR-5) e pelo Cálculo de Rendimento Líquido e Imposto de Renda (FR-8) — nenhuma fonte de dado nova. Carrega o disclaimer educacional (FR-9) embutido no próprio texto. Realiza UJ-2. Decisão originada em discussão de party-mode com Mary (Analista), Sally (UX), Winston (Arquiteto) e Amelia (Dev).

**Requisitos Funcionais:**

#### FR-4: Comparação de rendimento perdido (poupança e CDI)
Um usuário com valor de poupança informado (FR-2) vê, no Painel Único, o rendimento que esse valor teria tido na poupança e no CDI no mesmo período, comparado a ele ter ficado parado. Realiza UJ-2.

**Consequências (testáveis):**
- Poupança aparece antes do CDI na ordem de exibição.
- O texto inclui, na mesma tela (não em rodapé ou link separado), a frase de que rentabilidade passada não constitui garantia de rentabilidade futura, integrada ao disclaimer educacional (FR-9).
- Se o usuário não informou valor de poupança, a tela usa um valor ilustrativo e convida a inserir o valor real.
- A comparação nunca aparece isolada: a mesma tela emenda direto na recomendação (FR-5), por design — mostrar "quanto você perdeu" sem uma ação imediata na sequência reforçaria a culpa/ansiedade que o produto existe para reduzir (§1), em vez de resolvê-la.

### 4.4 Motor de Recomendação

**Descrição:** Combina análise fundamentalista (balanços e indicadores de até 5 anos) e análise técnica (gráficos de até 5 anos) para gerar **uma recomendação de compra** por vez, compatível com o perfil de risco e horizonte do usuário. `[ASSUMPTION: v1 cobre apenas ações listadas na B3 — não FIIs, ETFs, BDRs ou renda fixa]`. Realiza UJ-2.

**Requisitos Funcionais:**

#### FR-5: Recomendação combinada com explicação
Um usuário autenticado com perfil e horizonte definidos recebe uma recomendação de compra de uma ação, junto com uma explicação em linguagem simples de por que ela foi sugerida (quais indicadores fundamentalistas e quais padrões técnicos pesaram). Realiza UJ-2.

**Consequências (testáveis):**
- A ação sugerida usa critérios de elegibilidade ajustados ao perfil declarado (ex: perfil conservador não recebe recomendação tipicamente associada a alta volatilidade), mas **nunca é chamada de "adequada" ou "compatível" com o perfil conservador de forma irrestrita** — decisão consciente do Daniel (§8, item 8) de manter ação avulsa para todos os perfis sem fingir que ela elimina o risco de concentração que o perfil conservador tradicionalmente evita. `[ASSUMPTION: a regra exata de elegibilidade por perfil — quais indicadores/padrões qualificam uma ação para "conservador" dentro dessa ressalva — não está definida neste PRD; fica como critério a especificar com arquitetura/dados antes de virar história de engenharia, junto com os limiares de FR-6/FR-10 (§8, Questão em Aberto #5)]`.
- **Concentração em ativo único é, por definição, um fator de risco que nenhum perfil de risco elimina — inclusive o conservador.** A recomendação nunca é apresentada sem o aviso de concentração (FR-6) e sem o texto reconhecer explicitamente que uma ação avulsa não é, por si só, um investimento conservador (ver §8, item 8).
- A explicação usa linguagem sem jargão técnico não explicado — qualquer termo técnico usado é definido inline ou vinculado ao Glossário.
- Sistema não recomenda mais de uma ação por vez no v1.

**Fora de Escopo:**
- Sinalização de venda/saída e recomendação para day trade — fora de escopo (ver §6.2).

#### FR-6: Aviso de concentração
Antes de exibir os detalhes completos de uma recomendação, o sistema solicita que o usuário informe qual percentual do seu total pretende alocar naquele ativo, e exibe um aviso explícito quando esse percentual sugere concentração excessiva `[ASSUMPTION: limiar de "excessivo" a definir — sugestão inicial acima de 20-30% em um único ativo. Bloqueado até definição — não gerar história de engenharia sem esse número confirmado]`. Realiza UJ-2.

**Consequências (testáveis):**
- "Total" é definido como o valor que o usuário diz pretender ter investido em renda variável no total (autoinformado nesta etapa), não o valor de poupança (FR-2) nem um patrimônio total real — o produto não tem visibilidade de patrimônio fora do que o usuário digita, já que não há integração com Open Finance (§6.2).
- Recomendação completa (FR-5, FR-7, FR-8) só é exibida após essa confirmação.
- Aviso de concentração é visualmente distinto (não é um checkbox discreto de termos de uso) e o texto afirma explicitamente que uma ação avulsa não é, por definição, um investimento conservador — não apenas alerta sobre percentual.

#### FR-13: Registro de posição
Um usuário pode confirmar que comprou a ação recomendada, informando data, preço e quantidade. Sem esse registro, o sistema não tenta calcular status (FR-11), alerta (FR-10) ou lucro líquido (FR-8) "personalizados" — apenas exibe a recomendação original e sua evolução de mercado em abstrato, deixando claro que não reflete uma posição real do usuário. Realiza UJ-2, UJ-3.

**Consequências (testáveis):**
- FR-8, FR-10 e FR-11 só calculam "status" ou "lucro líquido" referenciados ao preço de entrada real quando um registro de posição (FR-13) existe para aquela recomendação.
- Sem registro de posição, o painel rotula claramente a evolução exibida como "como teria se comportado desde a recomendação", não como o resultado real do usuário.

### 4.5 Trilha de Transparência

**Descrição:** Para cada recomendação, expõe exatamente quais indicadores fundamentalistas e quais padrões gráficos pesaram na decisão — requisito não-negociável levantado na sessão de brainstorming (voz do "trader cético" em role-playing). Realiza UJ-2.

**Requisitos Funcionais:**

#### FR-7: Exposição da trilha de dados
Um usuário pode expandir qualquer recomendação e ver a lista específica de indicadores fundamentalistas e padrões técnicos que a fundamentaram, com os valores usados (não apenas os nomes dos indicadores).

**Consequências (testáveis):**
- Cada indicador exibido inclui a fonte do dado (provedor a definir — ver §8, Questão em Aberto #2 / futuro `addendum.md`) e a data de referência.
- Trilha é acessível a partir do mesmo Painel Único, sem navegação para outra página.

`[NOTE FOR PM]` FR-7 cobre a recomendação atual, não um histórico persistente de recomendações passadas e os dados que as fundamentaram. Isso conecta com o risco aceito em §8, item 6: um consultor de valores mobiliários registrado tem obrigação de guarda de registros; o produto está conscientemente fora desse regime, mas manter histórico ajudaria tanto a defender decisões passadas quanto a eventual mudança de enquadramento futuro. Não travado como FR agora — candidato a v2 se o item 6 for revisitado.

### 4.6 Cálculo de Rendimento Líquido e Imposto de Renda

**Descrição:** Projeção de retorno médio (curto/médio/longo prazo) e cálculo do lucro líquido considerando corretamente as regras de IR sobre ganho de capital — requisito não-negociável levantado pela voz do "contador/Receita" na sessão de brainstorming. Realiza UJ-2.

**Requisitos Funcionais:**

#### FR-8: Projeção de retorno e IR
Um usuário vê, para a recomendação atual, uma projeção média de retorno para curto, médio e longo prazo, e uma estimativa de lucro líquido considerando a alíquota de swing trade (15%, único regime coberto no v1 — ver §6.2) e, quando aplicável, a necessidade de DARF.

**Consequências (testáveis):**
- "Lucro líquido" nunca é apresentado como preço de venda menos preço de compra sem considerar a alíquota aplicável.
- **A isenção de R$20.000/mês é calculada apenas sobre o total de vendas registradas dentro do próprio sistema (via FR-13) naquele mês — o texto declara explicitamente que essa é uma estimativa isolada, pois o sistema não tem visibilidade de vendas do usuário em outras corretoras ou ativos fora da plataforma (sem integração Open Finance, §6.2). O produto mostra dois cenários lado a lado ("se esta for sua única venda de ações no mês" vs. "se você já vendeu outras ações este mês, a isenção pode não se aplicar") em vez de uma única estimativa apresentada como definitiva.**
- Projeções são claramente rotuladas como estimativas, não garantias (ligado ao disclaimer FR-9).
- **Compensação de prejuízos entre operações anteriores fica fora do v1 como limitação assumida, não porque "uma ação por vez" a elimine — a legislação exige compensação de prejuízos entre meses e ativos diferentes dentro do próprio regime de swing trade, independentemente de quantas ações o usuário opera por vez (corrigido de uma premissa anterior incorreta; ver §6.2). O texto do produto informa explicitamente que prejuízos de operações anteriores (dentro ou fora do sistema) não são considerados no cálculo, e recomenda consulta a um contador antes de declarar.**
- Escopo da projeção de retorno cobre variação de preço; tratamento fiscal de proventos (dividendos, isentos na pessoa física; JCP, tributados a 15% na fonte) fica fora do v1 `[ASSUMPTION]`.

### 4.7 Disclaimer Educacional / Compliance

**Descrição:** O produto se posiciona explicitamente como educacional/informativo, não como consultoria de investimentos regulamentada — exigência levantada pela voz da CVM na sessão de brainstorming. Esse mesmo enquadramento também reduz o medo de julgamento do usuário (ele decide, o sistema não decide por ele). Realiza UJ-2.

**Requisitos Funcionais:**

#### FR-9: Disclaimer educacional embutido
O texto de toda recomendação e da tela de comparação (FR-4) comunica, de forma integrada (não em rodapé ou modal separado), que o sistema tem caráter educacional/informativo e que a decisão final é do usuário.

**Consequências (testáveis):**
- Nenhuma recomendação usa linguagem imperativa do tipo "compre esta ação" — usa formulações como "os dados indicam" ou equivalente.
- Disclaimer aparece na primeira visita ao Painel Único e em todo ponto de recomendação (FR-5).

**`[NOTE FOR PM]`** Este disclaimer reduz o medo de julgamento do usuário (§1), mas **não é, sozinho, garantia de que o produto escapa da classificação de consultoria/análise de valores mobiliários regulada pela CVM** — a CVM avalia substância (coleta de perfil + recomendação individualizada e contínua de um ativo específico), não o rótulo escolhido. Risco aceito conscientemente pelo Daniel para o v1 — ver §8, item 6. Revisitar antes do lançamento público.

### 4.8 Monitoramento Contínuo e Alertas *(Should)*

**Descrição:** Acompanha a recomendação já dada e avisa o usuário apenas quando algo relevante muda — mantendo o painel silencioso quando não há nada de ação. Realiza UJ-3.

**Requisitos Funcionais:**

#### FR-10: Alerta de mudança relevante
Um usuário com uma recomendação ativa recebe um alerta visível no Painel Único quando um evento relevante ocorre (ex: mudança significativa no preço ou nos indicadores usados na recomendação original) `[ASSUMPTION: limiar exato de "relevante" a definir em conjunto com arquitetura/dados]`.

**Consequências (testáveis):**
- Sem eventos relevantes, o painel não gera nenhum alerta ou notificação.

### 4.9 Linguagem Visual Simplificada *(Should)*

**Descrição:** Usa metáforas visuais (semáforo, termômetro) para comunicar o status de uma recomendação, reduzindo a dependência de números e gráficos técnicos como linguagem principal. Realiza UJ-3.

**Requisitos Funcionais:**

#### FR-11: Indicador visual de status
O status de qualquer recomendação é comunicado por padrão através de um indicador visual simples (ex: semáforo verde/amarelo/vermelho), com os dados numéricos completos disponíveis apenas ao expandir (FR-7).

**Consequências (testáveis):**
- Nenhuma tela do Painel Único exibe números técnicos crus como primeira camada de informação sem o equivalente visual simplificado ao lado.

## 5. Não-Objetivos (Explícitos)

- O produto **não** executa ordens de compra/venda nem se integra a corretoras — é uma ferramenta de recomendação e informação, não de corretagem.
- O produto **não** se apresenta como consultoria de investimentos regulamentada — é educacional/informativo (FR-9), com o risco de classificação regulatória tratado conscientemente em §8, item 6.

*As demais exclusões (day trade, sinalização de venda/saída, carteira diversificada, Open Finance, app mobile) estão em §6.2, cada uma já com o porquê e o prazo.*

## 6. Escopo do MVP

### 6.1 Dentro do Escopo

**Must (o produto não faz sentido sem):**
- Cadastro via e-mail/senha, consentimento e exclusão de conta (FR-1, FR-2, FR-12).
- Painel Único consolidado (FR-3).
- Tela "quanto você perdeu parado" (FR-4).
- Recomendação combinada técnica + fundamentalista, ação avulsa, uma por vez (FR-5).
- Aviso de concentração (FR-6).
- Registro de posição (FR-13).
- Trilha de transparência (FR-7).
- Cálculo de rendimento líquido e IR — regime swing trade (FR-8).
- Disclaimer educacional embutido (FR-9).

**Should (valiosos, mas cortáveis sob pressão de prazo sem invalidar o produto):**
- Monitoramento contínuo com alertas (FR-10).
- Linguagem visual simplificada (FR-11).

**Comportamento mínimo se os Should forem cortados:** o Painel Único (FR-3) continua funcional mostrando a recomendação e seu status em texto simples, sem indicador visual tipo semáforo/termômetro (FR-11) e sem alertas ativos (FR-10) — o usuário precisa abrir o painel para ver mudanças, em vez de ser avisado. Isso enfraquece a tese de "painel calmo" do §1, mas não quebra as demais features.

### 6.2 Fora do Escopo do MVP

- App mobile nativo — deferido para uma versão futura, quando o modelo web estiver validado; v1 é web responsivo (desktop + mobile web) `[ASSUMPTION]`.
- Day trade — fora de escopo permanente no formato atual do produto. Cortar day trade evita uma segunda alíquota de IR (20%) e compensação de prejuízos *entre* modalidades — mas não elimina a compensação *dentro* do próprio regime de swing trade, que segue sendo uma limitação conhecida e não resolvida do v1 (ver FR-8, §8 item 7).
- Sinalização de venda/saída — deferido; v1 cobre só entrada. `[NOTE FOR PM]` revisitar assim que houver usuários com posições abertas reais.
- Carteira diversificada multi-ativo — deferido; ação avulsa foi mantida para todos os perfis (inclusive conservador) por decisão consciente do Daniel, não porque seja de fato adequada a um perfil conservador em qualquer framework de suitability — o produto nunca a trata como "compatível" com esse perfil (ver §8, item 8). Winston e Mary (party-mode) sinalizaram esse mesmo risco de reputação/suitability. `[NOTE FOR PM]` revisitar com prioridade caso o feedback de usuários confirme o risco.
- Integração com Open Finance — deferido, risco de superfície de compliance/segurança desproporcional para o v1 (consenso de party-mode).
- Modo "explica pro cunhado" (relatório compartilhável), chat conversacional ativo, formato em áudio — todos itens Could do brainstorm original, deferidos para depois do MVP. `[NOTE FOR PM]` o brainstorm original tratava "explica pro cunhado" e a trilha de transparência (FR-7) como a mesma ideia vista por ângulos diferentes — no v1, só o ângulo inline (FR-7) entra; a versão exportável/compartilhável é a extensão natural de FR-7, não uma feature à parte, e é o primeiro candidato a promover de Could para Must se o "job de demitir o cunhado" (§2.1) não estiver sendo satisfeito só com a trilha inline.

## 7. Métricas de Sucesso

**Primárias**
- **SM-1**: Taxa de ativação — % de contas criadas que chegam a ver uma recomendação completa (pós aviso de concentração). Valida FR-1, FR-2, FR-5, FR-6. `[ASSUMPTION: meta inicial de 60%, a validar]`.
- **SM-2**: Retenção de 30 dias — % de usuários que retornam ao Painel Único em até 30 dias após a primeira recomendação. Valida FR-3, FR-10.

**Secundárias**
- **SM-3**: Tempo até a primeira recomendação completa, desde o início do cadastro. Valida FR-1 a FR-6.
- **SM-4**: % de usuários que expandem a trilha de transparência (FR-7) — proxy de confiança/engajamento com a explicação, não só com a recomendação em si.

**Contra-métricas (não otimizar)**
- **SM-C1**: Frequência de novas recomendações por usuário — não deve ser otimizada para cima. Um número alto sugeriria que o produto está empurrando decisões, o oposto da tese de reduzir ansiedade (contrabalança SM-1 e SM-2).
- **SM-C2**: Reclamações ou denúncias percebendo o produto como consultoria não solicitada — deve se manter em zero; é o sinal mais direto de falha do disclaimer educacional (FR-9).
- **SM-C3**: Frequência de alertas/notificações ativas (FR-10) por usuário — não deve ser otimizada para cima como forma indireta de melhorar SM-2 (retenção). O aumento de SM-2 por causa de alertas mais frequentes, em vez de retorno espontâneo, contradiz a tese de "o silêncio é o produto funcionando" (UJ-3, §1) — contrabalança especificamente SM-2, de forma distinta de SM-C1.

## 8. Questões em Aberto

1. **Modelo de monetização** (gratuito, assinatura, outro) — ainda não decidido; não bloqueia o desenvolvimento do MVP, mas deve ser resolvido antes do lançamento público.
2. **Provedor de dados fundamentalistas e de cotações** — decisão técnica de qual API usar (ex: Status Invest, dados da CVM); nome exato e viabilidade de cotações históricas de 5 anos ficam para `addendum.md` / fase de arquitetura. `[NOTE FOR PM]` revisão adversarial (`review-adversarial.md`, achado H3) sinaliza que isso pode não ser uma decisão técnica trivial: sites de consumo (ex: Status Invest) não são APIs comerciais com contrato/SLA, e cotações de bolsa costumam ter custo de licenciamento de redistribuição junto à B3. Tratar como risco de viabilidade do MVP, não só como escolha de fornecedor, antes de comprometer o roadmap.
3. **LGPD e privacidade dos dados financeiros do usuário** — consentimento e exclusão de conta agora têm um FR mínimo (FR-12); criptografia em repouso, período de retenção e portabilidade continuam deferidos para a fase de arquitetura.
4. **Carteira diversificada vs. ação avulsa** — decisão tomada para viabilizar o v1 (ação avulsa), mas o debate de party-mode (Winston/Mary vs. John/Sally) não foi encerrado por consenso, apenas por decisão de escopo do Daniel. Ver item 8 abaixo — essa tensão tem uma leitura regulatória, não só reputacional.
5. **Limiares exatos** de "concentração excessiva" (FR-6) e "mudança relevante" (FR-10) — não travados neste PRD, a definir com dados reais ou em conjunto com UX/arquitetura. Nenhuma história de engenharia para FR-6 ou FR-10 deve ser criada sem esse número.
6. **`[DECIDIDO — risco aceito]` Classificação regulatória junto à CVM.** Revisão adversarial (achado C1) apontou que a combinação de FR-2 (coleta de perfil de risco/suitability) com FR-5 (recomendação individualizada e contínua de uma ação específica) pode se enquadrar como "consultoria de valores mobiliários" (Res. CVM 20/2021) ou "análise de valores mobiliários" (Res. CVM 19/2021) — atividades que exigem registro/certificação. O disclaimer educacional (FR-9) não resolve isso sozinho, pois a CVM avalia substância, não rótulo. **Decisão do Daniel:** aceitar esse risco conscientemente e seguir construindo sem parecer jurídico prévio. `[NOTE FOR PM]` revisitar antes do lançamento público (não antes de codar) — o risco de negócio (notificação/multa/obrigação de reestruturar) permanece real e não foi mitigado, só aceito.
7. **`[DECIDIDO — aceitável como está]` Cálculo de IR (FR-8) tem duas limitações estruturais, não um limiar a ajustar.** Revisão adversarial (achado C2): (a) a isenção de R$20.000/mês só pode ser calculada sobre vendas dentro do próprio sistema — o produto não sabe se o usuário vendeu ações em outras corretoras no mesmo mês; (b) compensação de prejuízos entre meses/ativos é exigida pela legislação mesmo dentro do regime swing-trade-only do v1, e não é eliminada por recomendar uma ação por vez. **Decisão do Daniel:** a comunicação transparente dessas duas limitações ao usuário (já refletida em FR-8) é aceitável para o v1 — compensação de prejuízos entre operações fica para uma versão futura, sem mudança de escopo agora.
8. **`[DECIDIDO — mantido com ressalva]` Contradição de suitability: ação avulsa concentrada vs. perfil "conservador".** Revisão adversarial (achado C3): nenhum framework de suitability (incluindo o da própria CVM) trata uma posição concentrada em ativo único como adequada a um perfil conservador, independentemente da ação escolhida — o risco idiossincrático é estrutural ao formato, não à ação. **Decisão do Daniel:** manter ação avulsa para todos os perfis (não restringir a moderado/arrojado), mas o produto nunca a trata como "adequada"/"compatível" com o perfil conservador — é comunicada como uma limitação assumida do v1 (ver FR-5, FR-6). Conecta com o item 4 (debate ação avulsa vs. carteira, ainda registrado como tensão não fechada por consenso da equipe).
9. **Ampliar cobertura de perfil.** FR-2 coleta perfil moderado/arrojado e horizonte curto/médio prazo, mas o Motor de Recomendação (FR-5) no v1 só está calibrado para conservador/longo prazo (ver `[ASSUMPTION]` em §4.1). Definir se e quando estender a cobertura, e como validar as regras de elegibilidade para os demais perfis.

## 9. Índice de Suposições

- §4.1 (FR-1) — Cadastro apenas via e-mail/senha, sem login social/OAuth na v1; MFA fora do v1 por decisão consciente.
- §4.1 (FR-2) — Motor de Recomendação (FR-5) só calibrado para perfil conservador + horizonte longo prazo no v1; demais combinações exibem "ainda não suportado" (ver Questão em Aberto #9).
- §4.4 (FR-5) — v1 cobre apenas ações listadas na B3 (não FIIs, ETFs, BDRs, renda fixa); regra exata de elegibilidade por perfil ainda não especificada (ver Questão em Aberto #5).
- §4.4 (FR-6) — Limiar inicial de aviso de concentração sugerido acima de 20-30% em um único ativo, a validar (ver Questão em Aberto #5).
- §4.6 (FR-8) — Tratamento fiscal de proventos (dividendos/JCP) fora do escopo da projeção de retorno no v1.
- §4.8 (FR-10) — Limiar de "evento relevante" para alerta não definido; depende de arquitetura/dados disponíveis (ver Questão em Aberto #5).
- §6.2 — Painel é web responsivo (desktop + mobile web), sem app nativo no v1.
- §7 — Metas numéricas de SM-1 são estimativas iniciais, não validadas com dados reais.

## 10. Requisitos Não-Funcionais (Iniciais)

*Seção adicionada após revisão de qualidade — o v1 depende de dados recentes (FR-5, FR-7) e guarda dados financeiros sensíveis (FR-1, FR-2, FR-12), então mesmo limites provisórios são melhores do que nenhum. Detalhamento completo é trabalho de arquitetura.*

- **Frescor de dados:** `[ASSUMPTION]` uma recomendação não deve se basear em cotação com mais de 1 dia útil de atraso; se o provedor de dados estiver indisponível ou desatualizado além disso, o Painel Único comunica isso explicitamente em vez de mostrar uma recomendação potencialmente obsoleta (liga com o edge case de UJ-2).
- **Segurança de dados em repouso:** `[ASSUMPTION]` senha nunca armazenada em texto plano (hash com salt); demais detalhes (algoritmo, criptografia de perfil financeiro) ficam para a arquitetura.
- **Disponibilidade:** `[ASSUMPTION]` sem meta de uptime formal no v1 (não é um produto financeiro transacional/tempo-real); indisponibilidade do provedor de dados degrada a experiência de forma comunicada, não silenciosa.
