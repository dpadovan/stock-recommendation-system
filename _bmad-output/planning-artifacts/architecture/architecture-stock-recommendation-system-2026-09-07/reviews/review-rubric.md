# Review Rubric — ARCHITECTURE-SPINE.md (Sistema de Recomendação de Ações)

**Veredito geral:** a spine está sólida nas fronteiras de módulo, dados e stack (AD-1 a AD-11 são, em geral, específicos e aplicáveis), mas deixa em silêncio o envelope operacional inteiro e não fecha o modelo de dados que o FR-8 (Must-have, cálculo de IR) precisa para registrar vendas — dois furos que deixam divergência real para a fase de build.

---

## Critical

### C1 — Envelope operacional/ambiental inteiro em silêncio
**Seção:** Stack / Consistency Conventions / Deferred (ausência transversal)
**Critério do checklist:** "every dimension the altitude owns is decided, deferred, or an open question — a whole dimension left silent is a finding, especially the operational/environmental envelope (deployment & environments, infra/provider strategy, operations)."

A AD-6 decide *onde* hospedar e *como* fazer deploy (Render, git-push, Cron nativo), mas nenhuma AD, convenção ou item em Deferred decide:
- **Backup/DR do Postgres** — dado financeiro sensível (posições, cálculo de IR) sem qualquer menção a política de backup/restore do banco gerenciado do Render.
- **Observabilidade/alerting sobre a falha do job diário (AD-5)** — o NFR do PRD exige que "indisponibilidade do provedor de dados degrade a experiência de forma comunicada, não silenciosa" para o *usuário*, mas nada instrumenta o *operador* (dev solo) para saber que o `update_market_data` falhou. Sem isso, o job pode falhar silenciosamente por dias e o painel continuar servindo dado obsoleto sem que ninguém saiba operacionalmente.
- **Ambientes** — nenhuma menção a dev local/homologação/produção, nem à ausência deliberada de um staging gate antes do `git push` para produção (AD-6 implica deploy contínuo direto a produção, mas isso não é dito como decisão consciente).

Isso não é uma lacuna de detalhe de build: é exatamente a categoria que o checklist marca como "especially" — e para uma fintech greenfield isso é um risco real de dado perdido/corrompido sem trilha de recuperação.

**Fix concreto:** adicionar uma AD (ou seção "Operações") cobrindo no mínimo: (a) política de backup do Postgres gerenciado do Render (frequência, retenção, teste de restore) — aceitar o padrão do Render é uma decisão válida, mas precisa ser dita; (b) mecanismo de alerta de falha do job diário (ex.: Render Cron failure notification, ou o management command gravar um heartbeat que outra rotina verifica); (c) uma frase explícita confirmando "sem ambiente de staging no v1, deploy direto a produção" como trade-off consciente (no padrão já usado em AD-8), ou definir um ambiente de homologação mínimo.

### C2 — FR-8 (isenção de R$20k / DARF) não tem modelo de dado para "venda"
**AD:** AD-11 — Propriedade do registro de posição
**Critério do checklist:** "if a spec drove it, it covers that spec's capabilities" + "every AD's Rule is enforceable and actually prevents its stated divergence."

O PRD (FR-8, consequência 2) exige: *"a isenção de R$20.000/mês é calculada apenas sobre o total de **vendas registradas** dentro do próprio sistema (via FR-13)"*. Mas FR-13 ("Registro de posição"), tanto no texto do FR quanto no `Rule` da AD-11, só cobre a **entrada** (compra): "referência à recomendação, preço de entrada, data de entrada e quantidade". Não existe, em nenhuma AD, convenção ou no Structural Seed, um model, campo ou responsabilidade de módulo para registrar uma **venda** (data, preço, quantidade vendida) — que é exatamente o dado necessário para somar "vendas registradas no mês" e testar o limite de isenção de R$20k, e para saber se DARF é devido.

Isso é precisamente o tipo de divergência que a AD-11 foi escrita para prevenir do lado da entrada ("Prevents: recommendations ou tax criarem seu próprio modelo paralelo para posição do usuário") — mas o lado da saída ficou inteiramente aberto. Sem uma AD resolvendo isso, um implementador do `tax` pode inventar um model `Sale` próprio, outro pode adicionar campos de saída na própria `Position` de `portfolio` — divergindo exatamente como AD-11 pretendia evitar.

**Fix concreto:** estender a AD-11 (ou criar uma AD-12) definindo explicitamente: quem é dono do registro de venda/saída (provavelmente `portfolio`, para manter consistência com "portfolio é o único dono do registro que liga usuário/recomendação/dados de entrada"), quais campos mínimos tem (data da venda, preço de venda, quantidade vendida, referência à `Position` de origem), e que `tax` lê esse registro pela interface pública de `portfolio` — mesmo padrão já usado para a entrada.

---

## High

### H1 — Mínimo de segurança de senha delegado ao arquitetura pelo PRD e nunca decidido
**AD:** AD-10 — Autenticação
**Critério do checklist:** "the spine fixes the real divergence points for the level below and misses none."

O PRD é explícito duas vezes em empurrar essa decisão para a arquitetura: FR-1 diz "senha abaixo de um mínimo de segurança **definido na arquitetura**"; a seção 10 (NFRs) reforça "demais detalhes (algoritmo, criptografia de perfil financeiro) ficam para a arquitetura". A AD-10 resolve o *mecanismo* (sessão nativa do Django, hashing embutido) mas não fixa nenhum número/política de mínimo de senha (comprimento mínimo, complexidade). Como é uma decisão que o próprio PRD pede à arquitetura e a spine não a toma nem a lista em Deferred, dois implementadores do formulário de cadastro (FR-1) podem escolher validadores de senha diferentes.

**Fix concreto:** adicionar à AD-10 (ou convenção) uma linha fixando a política mínima — ex.: "usa os `AUTH_PASSWORD_VALIDATORS` padrão do Django (mínimo 8 caracteres, não similar a dados do usuário, não em lista de senhas comuns), sem validador customizado adicional no v1" — ou, se a intenção é revisitar, mover explicitamente para Deferred com essa justificativa.

---

## Medium

### M1 — Um ou dois jobs agendados? AD-5/AD-6 e o Structural Seed não concordam
**AD:** AD-5 — Trabalho em background / AD-6 — Hospedagem
**Critério do checklist:** "every AD's Rule is enforceable and actually prevents its stated divergence."

AD-5 vincula (`Binds`) tanto `market_data` (ingestão diária) quanto `recommendations` (recálculo de status/alerta, FR-10) ao mesmo mecanismo de "management command + agendador externo". Mas o Structural Seed só mostra **um** management command (`market_data/management/commands/update_market_data.py`), nenhum equivalente em `recommendations`; e a AD-6 fala d"o job diário" no singular ("o job diário (AD-5) usa o recurso nativo de Cron Jobs do Render"). Não fica decidido se é um único Cron Job/command fazendo ingestão + recálculo de alerta em sequência, ou dois Cron Jobs/commands independentes — decisão que afeta acoplamento entre `market_data` e `recommendations` e pode ser resolvida de formas diferentes por implementadores diferentes.

**Fix concreto:** adicionar ao Structural Seed um segundo command (ex.: `recommendations/management/commands/check_alerts.py`) e uma frase na AD-5/AD-6 dizendo explicitamente se são um ou dois recursos de Cron Job no Render.

### M2 — Dado de benchmark (CDI/poupança) do FR-4 não tem dono claro dentro da camada anti-corrupção
**AD:** AD-4 — Camada anti-corrupção do provedor de dados de mercado
**Critério do checklist:** "every AD's Rule is enforceable and actually prevents its stated divergence."

AD-4 escopa a camada anti-corrupção para "cotações e fundamentos". O FR-4 (comparação com poupança e CDI) precisa de uma série de taxa de referência (CDI) que não é, estritamente, cotação nem fundamento de uma empresa — é um dado macro, tipicamente de outra fonte (ex. Bacen). Não fica dito se essa fonte também deve passar pelo cliente único em `market_data` (estendendo o escopo da AD-4) ou se pode ser buscada por outro módulo diretamente, o que abriria exatamente o acoplamento direto a fonte externa que a AD-4 existe para prevenir.

**Fix concreto:** uma frase na AD-4 estendendo explicitamente o escopo do anti-corruption layer para "cotações, fundamentos e taxas de referência (CDI/poupança)", ou uma nota em Deferred se a fonte ainda não foi escolhida.

### M3 — Exclusão de conta (FR-12) não tem regra de propagação entre módulos
**Seção:** Consistency Conventions (ausência) / AD-3
**Critério do checklist:** "the spine fixes the real divergence points for the level below and misses none."

FR-12 exige que um usuário autenticado possa solicitar exclusão da conta "e os dados associados". Como AD-3 estabelece que cada módulo é dono dos seus próprios dados (`portfolio.Position`, dados de `tax`, etc., todos referenciando `accounts` por FK), a exclusão cruza fronteiras de módulo — mas nenhuma AD ou convenção define como isso se propaga (hard delete em cascata vs. anonimização, ordem de exclusão respeitando o grafo de dependência de AD-3, prazo de retenção). Implementadores de módulos diferentes podem divergir nesse comportamento.

**Fix concreto:** acrescentar uma linha na tabela de Consistency Conventions (ou uma AD dedicada a dados/LGPD) definindo a estratégia de exclusão (ex.: hard delete em cascata via `on_delete=CASCADE`/rotina dedicada percorrendo o grafo de AD-3 de cima para baixo) e se há período de retenção antes da exclusão efetiva.

---

## Low

### L1 — "Frescor de dados" (NFR) não tem dono explícito da checagem de staleness
**Seção:** NFR do PRD (frescor de dados) vs. AD-4/AD-5 (ausência)

O NFR do PRD exige que nenhuma recomendação se baseie em cotação com mais de 1 dia útil de atraso, e que o painel comunique explicitamente quando o provedor está indisponível/desatualizado (edge case de UJ-2). Nenhuma AD atribui a responsabilidade dessa checagem a um módulo específico (candidato natural: `market_data` expor um timestamp de "última atualização bem-sucedida" que `recommendations`/`dashboard` consultam antes de renderizar). Sem essa regra, `recommendations` e `dashboard` podem implementar critérios de staleness diferentes.

**Fix concreto:** uma frase na AD-4 ou AD-5 atribuindo a `market_data` a responsabilidade de expor o timestamp da última atualização bem-sucedida, e definindo que qualquer consumidor deve checar esse timestamp antes de tratar o dado como atual.
