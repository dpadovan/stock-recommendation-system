---
stepsCompleted: [1, 2, 3, 4]
inputDocuments:
  - _bmad-output/planning-artifacts/prds/prd-stock-recommendation-system-2026-08-30/prd.md
  - _bmad-output/planning-artifacts/ux-designs/ux-stock-recommendation-system-2026-09-05/DESIGN.md
  - _bmad-output/planning-artifacts/ux-designs/ux-stock-recommendation-system-2026-09-05/EXPERIENCE.md
  - _bmad-output/planning-artifacts/architecture/architecture-stock-recommendation-system-2026-09-07/ARCHITECTURE-SPINE.md
---

# Sistema de Recomendação de Ações para Investidores Leigos - Epic Breakdown

## Overview

Este documento decompõe o PRD, a UX (DESIGN.md/EXPERIENCE.md) e a Arquitetura (ARCHITECTURE-SPINE.md) do sistema em épicos e stories implementáveis.

## Requirements Inventory

### Functional Requirements

FR1: Um visitante pode criar uma conta usando e-mail e senha (Cadastro).
FR2: Um usuário autenticado pode declarar seu perfil de risco (conservador/moderado/arrojado), horizonte de investimento (curto/médio/longo prazo) e, opcionalmente, o valor parado na poupança.
FR3: Um usuário autenticado pode ver, em uma única tela (Painel Único), sem navegação entre páginas separadas, a comparação de perda por inércia, a recomendação atual, a trilha de transparência, a projeção de retorno/impostos e um guia rápido de uso.
FR4: Um usuário com valor de poupança informado vê, no Painel Único, o rendimento que esse valor teria tido na poupança e no CDI no mesmo período, comparado a ele ter ficado parado — sempre emendando direto na recomendação (FR5), nunca isolada.
FR5: Um usuário autenticado com perfil e horizonte definidos recebe uma recomendação de compra de uma ação, com explicação em linguagem simples do porquê — nunca chamada de "adequada"/"compatível" com o perfil conservador sem a ressalva de concentração na mesma frase.
FR6: Antes de exibir os detalhes completos de uma recomendação, o sistema exige que o usuário informe o valor total (R$) que pretende ter investido em renda variável e qual percentual desse total pretende alocar naquele ativo, exibindo aviso de concentração excessiva.
FR7: Um usuário pode expandir qualquer recomendação e ver a trilha de transparência — indicadores fundamentalistas e padrões técnicos específicos, com valores, fonte e data de referência.
FR8: Um usuário vê, para a recomendação atual, uma projeção média de retorno (curto/médio/longo prazo) e uma estimativa de lucro líquido, com dois cenários lado a lado para a isenção de R$20.000/mês e recomendação de consulta a um contador.
FR9: O texto de toda recomendação e da tela de comparação de perda comunica, de forma integrada (nunca em rodapé/modal separado), que o sistema tem caráter educacional/informativo.
FR10: Um usuário com uma recomendação ativa recebe um alerta visível no Painel Único quando um evento relevante ocorre (mudança significativa em preço/indicadores) — silêncio é o estado padrão esperado.
FR11: O status de qualquer recomendação é comunicado por padrão por um indicador visual (termômetro), nunca só por cor — sempre com selo de texto.
FR12: Um visitante só conclui o cadastro após aceitar explicitamente o uso de dados financeiros/perfil de risco; um usuário autenticado pode solicitar exclusão da conta e dos dados associados.
FR13: Um usuário pode confirmar que comprou a ação recomendada (data, preço, quantidade) e, futuramente, registrar uma venda — sem esses registros, todo número de performance é rotulado como simulação.

### NonFunctional Requirements

NFR1: Nenhuma recomendação se baseia em cotação com mais de 1 dia útil de atraso; se o provedor de dados estiver indisponível/desatualizado, o painel comunica isso explicitamente em vez de mostrar dado obsoleto silenciosamente.
NFR2: Senha nunca armazenada em texto plano (hash com salt) — ver AD-10 (validadores padrão do Django) para a política mínima.
NFR3: Sem meta de uptime formal no v1 — indisponibilidade do provedor de dados degrada a experiência de forma comunicada ao usuário, não silenciosa.
NFR4: Todo valor monetário/percentual usa `Decimal`/`NUMERIC`, nunca `float`, normalizado com `ROUND_HALF_UP` no ponto de cálculo (AD-7).
NFR5: Proteção de dados sensíveis em repouso/trânsito no v1 é o piso do provedor de hospedagem (criptografia em repouso + TLS) — sem criptografia adicional de campo (AD-8); logs nunca contêm PII/dado financeiro identificável em texto livre.
NFR6: Contraste WCAG AA em todo par texto/ícone-fundo, em light e dark; navegação por teclado funcional (foco visível, ordem de leitura, focus-trap no modal de concentração); sem laudo formal de acessibilidade no v1.

### Additional Requirements

- **Sem starter/scaffold de terceiros**: projeto Django greenfield puro (`django-admin startproject`), sem template de boilerplate externo — relevante para a Epic 1 / Story 1 (setup do projeto).
- **Monólito modular**: um único projeto Django, apps internos como módulos de domínio — `accounts`, `market_data`, `recommendations`, `portfolio`, `tax`, `dashboard` — comunicação só via `services.py`/`selectors.py` do app dono (AD-1, AD-3).
- **Grafo de dependência fixo**: `market_data` não depende de nenhum módulo interno; `dashboard` pode depender de todos; demais seguem o grafo de AD-3 (diagrama mermaid na spine). `tax` nunca importa `market_data`/`recommendations` diretamente — orquestração via `dashboard`.
- **PostgreSQL único banco** (AD-2); qualquer cache no v1 é tabela Postgres ou `LocMemCache`, nunca Redis/Memcached gerenciado.
- **Camada anti-corrupção do provedor de dados** (AD-4): só `market_data` importa o cliente do provedor externo (cotações, fundamentos, e também séries de referência macroeconômica CDI/poupança para FR4); provedor exato ainda não escolhido (Deferred).
- **Job diário orquestrado**: um único management command orquestrador (`run_daily_pipeline`) chama ingestão de `market_data` e, só em caso de sucesso, o recálculo de alertas de `recommendations`; disparado por um único Cron Job nativo do Render (AD-5, AD-6). `market_data` persiste timestamp de última atualização bem-sucedida.
- **Hospedagem**: deploy via git-push para o Render; Postgres gerenciado; sem ambiente de staging no v1 (AD-6, AD-12).
- **Envelope operacional** (AD-12): backup no padrão do plano Render aceito como está; heartbeat/notificação de falha do job diário; deploy contínuo direto a produção é decisão consciente.
- **Enforcement mecânico de fronteiras** (AD-14): contrato `import-linter` em CI/pre-commit barrando import cross-app de `models.py`/`provider.py` fora da interface pública, inclusive em `migrations/` e em `market_data` (nem para logging).
- **Posse de dado derivado** (AD-11, AD-13, AD-15): `portfolio` é dono do registro de posição (compra) e de venda, do cálculo canônico de "valor atual" e do booleano "real vs. simulado" — `tax`/`recommendations`/`dashboard` só consomem via `portfolio.services`.
- **Exclusão de conta** (AD-16): sempre `CASCADE` até o fim da árvore de FKs a partir de `accounts.User`, sem `PROTECT`/`SET_NULL` em dado financeiro pessoal no v1.
- **Autenticação**: sessão nativa do Django, sem OAuth/JWT/MFA no v1 (AD-10).
- **Renderização**: 100% Django Template Language server-side, sem framework JS de terceiros; JS vanilla só para slider do gate e focus-trap (módulo compartilhado `static/js/focus-trap.js`) (AD-9).

### UX Design Requirements

**Design tokens (DESIGN.md) a implementar como CSS custom properties / arquivos estáticos:**
- UX-DR1: Paleta de cores completa light + dark (bg, card, border, ink, ink-muted, terracotta[-soft/-strong], moss[-light/-soft], amber[-soft/-ink], red[-soft/-strong], overlay) como CSS custom properties, com troca de tema via `prefers-color-scheme` (sem toggle manual persistido no v1).
- UX-DR2: Escala tipográfica (brand, heading, label, body, meta, caption, action, figure-lg, figure-sm) — pilha de sistema para texto geral, `ui-monospace` reservada a valores numéricos tabulares (moeda/percentual/indicador).
- UX-DR3: Escalas de `rounded` (sm/md/lg/xl/full) e `spacing` (1–7 + tokens nomeados: card-gap, card-padding, chip-padding, gate-padding, content-margin) como variáveis CSS reutilizáveis.

**Componentes (DESIGN.md.Components / EXPERIENCE.md Component Patterns) — cada um uma unidade de implementação própria:**
- UX-DR4: Componente **Card recibo** — contêiner base de seção do Painel Único.
- UX-DR5: Componente **Linha de recibo** (receipt-row) — rótulo + sub-rótulo + valor tabular monoespaçado, divisor tracejado.
- UX-DR6: Componente **Termômetro** (indicador de status) — tubo/cápsula + preenchimento em degradê + bulbo + selo de texto (Favorável/Atenção/Alerta), nunca só cor.
- UX-DR7: Componente **Trilha de transparência** (accordion `<details>/<summary>` nativo, um nível, dois grupos fundamentalista/técnico).
- UX-DR8: Componente **Gate de concentração** (modal bloqueante) — campo de total em R$ + slider de percentual + caixa de aviso + dois botões; conteúdo completo da recomendação só libera após confirmação.
- UX-DR9: Componente **Projeção de retorno e IR** — dois cenários lado a lado (isenção aplicável/não aplicável) + recomendação de consulta a contador.
- UX-DR10: Componente **Guia rápido** (tour inline na primeira visita ao Painel Único, não reaparece depois).
- UX-DR11: Componente **Registro de posição** (form/modal leve: data, preço, quantidade).
- UX-DR12: Componente **Chip de confirmação** e **Pílula de disclaimer**.
- UX-DR13: Componente **Input field** genérico (Cadastro, Configuração de Perfil, Registro de Posição, campo de total do gate).
- UX-DR14: Componentes de **Botão** primário e ghost.

**Acessibilidade (EXPERIENCE.md Accessibility Floor):**
- UX-DR15: Contraste AA em todo par texto/fundo, light e dark, incluindo selo do termômetro e textos de aviso/disclaimer.
- UX-DR16: Status nunca comunicado só por cor (selo de texto obrigatório).
- UX-DR17: Ordem de foco = ordem de leitura no Painel Único (Tab/Shift+Tab sem saltos).
- UX-DR18: Focus-trap completo no modal do gate de concentração (foco automático ao abrir, preso dentro do modal, Esc fecha sem confirmar, clique fora NÃO fecha, lista de focáveis recalculada a cada mudança de estado) via módulo JS compartilhado único.
- UX-DR19: Foco visível (anel de foco AA) em todo elemento interativo, em ambos os temas.
- UX-DR20: Alvos de toque ≥44px; formulários com labels associados e erros inline (nunca só cor).

**Responsivo (EXPERIENCE.md Responsive & Platform):**
- UX-DR21: Layout desktop (coluna única de cards, texto+termômetro lado a lado) e mobile web (mesma ordem, texto+termômetro empilhado) — breakpoint ~640px `[ASSUMPTION]`, mesma informação em ambos, sem versão reduzida.

**Tom e rotulagem (EXPERIENCE.md Voice and Tone / Rotulagem):**
- UX-DR22: Todo copy de recomendação usa linguagem não-imperativa ("os dados indicam"), nunca "compre"; disclaimer sempre tecido no texto.
- UX-DR23: Regra de rotulagem "Dado Real vs. Simulado" aplicada de forma consistente a todo número de performance (status, lucro líquido, alerta) conforme existência de registro de posição/venda.

**IA (EXPERIENCE.md Information Architecture) — 7 superfícies, cada uma um candidato a story de tela:**
- UX-DR24: Cadastro (Signup), Login, Recuperação de senha, Configuração de Perfil, Painel Único, Edição de Perfil, Gestão de Conta e Consentimento.

### FR Coverage Map

FR1: Epic 1 - Cadastro via e-mail e senha
FR2: Epic 1 - Definição de perfil de risco e horizonte
FR3: Epic 2 - Painel único consolidado
FR4: Epic 2 - Comparação de rendimento perdido (poupança/CDI)
FR5: Epic 2 - Recomendação combinada com explicação
FR6: Epic 2 - Aviso de concentração
FR7: Epic 2 - Trilha de transparência
FR8: Epic 2 - Projeção de retorno e IR
FR9: Epic 2 - Disclaimer educacional embutido
FR10: Epic 3 - Alerta de mudança relevante
FR11: Epic 3 - Indicador visual de status
FR12: Epic 1 - Consentimento e exclusão de conta
FR13: Epic 3 - Registro de posição

## Epic List

### Epic 1: Cadastro e Perfil
Usuário cria conta, define perfil de risco/horizonte de investimento, e consente com o uso de dados financeiros (ou solicita exclusão da conta depois).
**FRs covered:** FR1, FR2, FR12

### Epic 2: Painel Único — Recomendação Completa
Usuário vê, numa única tela, quanto perdeu ficando parado, recebe uma recomendação personalizada com aviso de concentração obrigatório, pode expandir a trilha de transparência completa e ver a projeção de retorno já considerando o IR — a experiência central do produto (UJ-2).
**FRs covered:** FR3, FR4, FR5, FR6, FR7, FR8, FR9

### Epic 3: Acompanhamento e Posição
Usuário registra que comprou (ou vendeu) de verdade, e volta depois pra ver o status atualizado da recomendação — silêncio quando nada mudou, alerta quando algo relevante mudou.
**FRs covered:** FR10, FR11, FR13

## Epic 1: Cadastro e Perfil

Usuário cria conta, define perfil de risco/horizonte de investimento, e consente com o uso de dados financeiros (ou solicita exclusão da conta depois).

### Story 1.1: Cadastro com e-mail, senha e consentimento explícito

Como uma visitante,
Eu quero criar uma conta com e-mail e senha e aceitar explicitamente o uso dos meus dados financeiros,
Para que eu possa começar a usar o sistema com meu consentimento registrado.

**Acceptance Criteria:**

**Given** que estou na página de cadastro
**When** informo e-mail, senha e aceito o texto específico sobre uso de dados financeiros
**Then** minha conta é criada, a senha é armazenada com hash (nunca texto plano), e sou autenticada automaticamente
**And** se eu não aceitar o consentimento, o cadastro não é concluído
**And** se o e-mail já existir, vejo um erro claro sem expor se a conta pertence a outra pessoa
**And** o formulário segue o componente Input field (UX-DR13), com labels associados e erros inline (nunca só cor)

### Story 1.2: Login com e-mail e senha

Como uma usuária cadastrada,
Eu quero entrar com e-mail e senha,
Para acessar minha conta em visitas futuras.

**Acceptance Criteria:**

**Given** que já tenho uma conta
**When** informo e-mail e senha corretos
**Then** sou autenticada via sessão nativa do Django (AD-10) e redirecionada ao Painel Único
**And** credenciais incorretas mostram um erro genérico, sem indicar se o e-mail existe

### Story 1.3: Recuperação de senha

Como uma usuária que esqueceu a senha,
Eu quero recuperá-la por e-mail,
Para não ficar trancada fora da conta.

**Acceptance Criteria:**

**Given** que esqueci minha senha
**When** solicito recuperação informando meu e-mail
**Then** recebo um e-mail com link para redefinir a senha, válido por tempo limitado
**And** ao definir a nova senha, a política mínima padrão do Django é aplicada (AD-10)

### Story 1.4: Configuração de perfil de risco e horizonte

Como uma usuária autenticada,
Eu quero declarar meu perfil de risco, horizonte de investimento e, opcionalmente, quanto tenho parado na poupança,
Para receber uma recomendação adequada.

**Acceptance Criteria:**

**Given** que acabei de concluir o cadastro
**When** sou redirecionada automaticamente pra configuração de perfil (sem tela intermediária)
**Then** posso escolher perfil (conservador/moderado/arrojado), horizonte (curto/médio/longo) e, opcionalmente, valor de poupança
**And** ao concluir, caio direto no Painel Único
**And** posso editar essas informações depois a partir do Painel Único

### Story 1.5: Exclusão de conta e dados associados

Como uma usuária autenticada,
Eu quero excluir minha conta e os dados associados,
Para exercer meu direito sobre meus dados.

**Acceptance Criteria:**

**Given** que estou autenticada
**When** solicito exclusão da conta na Gestão de Conta e confirmo
**Then** a conta e todos os dados ligados a ela são apagados em cascata (AD-16), minha sessão encerra, e não consigo mais fazer login
**And** existe um passo de confirmação explícito antes da exclusão efetiva

## Epic 2: Painel Único — Recomendação Completa

Usuário vê, numa única tela, quanto perdeu ficando parado, recebe uma recomendação personalizada com aviso de concentração obrigatório, pode expandir a trilha de transparência completa e ver a projeção de retorno já considerando o IR.

**Pré-requisito do épico (não é uma story):** a escolha do provedor de dados de mercado (cotações/fundamentos — PRD §8 item 2, Deferred na arquitetura) precisa estar resolvida antes de iniciar a Story 2.1. Não é trabalho de dev a ser descoberto durante a story — é uma decisão de produto/negócio (custo, licenciamento, viabilidade) que bloqueia a implementação de `market_data.provider`.

### Story 2.1: Ingestão e cache de dados de mercado

Como o sistema,
Eu quero ingerir e cachear diariamente cotações, fundamentos e séries de referência (CDI/poupança),
Para que as próximas stories tenham dados frescos e confiáveis pra trabalhar.

**Acceptance Criteria:**

**Given** que o Cron Job diário do Render dispara o orquestrador (AD-5)
**When** `market_data` busca dados do provedor externo (via `provider.py`, único ponto de acesso — AD-4) e das séries de CDI/poupança
**Then** os dados são cacheados no Postgres com timestamp de última atualização bem-sucedida
**And** se a ingestão falhar, nenhum outro módulo troca dados obsoletos por "atuais" silenciosamente, e um heartbeat de falha é registrado (AD-12)
**And** o recálculo de `recommendations` só roda se a ingestão terminar com sucesso (AD-5)

### Story 2.2: Painel Único com "quanto você perdeu parado"

Como uma usuária autenticada,
Eu quero ver, na minha primeira visita ao painel, quanto meu dinheiro parado deixou de render comparado à poupança e ao CDI,
Para entender o custo real da inércia.

**Acceptance Criteria:**

**Given** que completei meu perfil (Epic 1)
**When** acesso o Painel Único pela primeira vez
**Then** vejo, numa única tela sem navegação, a comparação de rendimento perdido — poupança primeiro, depois CDI — com o disclaimer educacional tecido no texto (nunca em rodapé)
**And** se não informei valor de poupança, vejo um valor ilustrativo com convite a inserir o real
**And** o card usa o padrão "Linha de recibo" (UX-DR5) com números tabulares monoespaçados
**And** essa comparação nunca aparece isolada — a tela já emenda direto na recomendação (Story 2.3)
**And** esta é a primeira tela renderizada, então estabelece a fundação do design system: tokens de cor (light + dark via `prefers-color-scheme`), tipografia, espaçamento e raio (UX-DR1-3), e os componentes de Botão primário/ghost (UX-DR14) reutilizados por todo o resto do Painel Único
**And** o layout já nasce responsivo (desktop: coluna única; mobile web: mesma ordem, empilhado — UX-DR21)

### Story 2.3: Recomendação personalizada com explicação

Como uma usuária com perfil definido,
Eu quero receber uma recomendação de ação com explicação em linguagem simples,
Para decidir com confiança.

**Acceptance Criteria:**

**Given** que meu perfil é conservador/longo prazo (único calibrado no v1)
**When** o painel carrega a recomendação
**Then** vejo uma única ação recomendada com explicação não-imperativa ("os dados indicam...")
**And** o texto nunca chama a recomendação de "adequada"/"compatível" com meu perfil sem a ressalva de concentração na mesma frase (FR5, §8 item 8)
**And** o disclaimer educacional está embutido no texto (FR9)
**And** se meu perfil for moderado/arrojado/curto/médio prazo, vejo "ainda não suportado neste perfil" em vez de recomendação mal calibrada

### Story 2.4: Gate de concentração

Como uma usuária vendo uma recomendação,
Eu quero confirmar quanto pretendo investir no total e quanto dessa recomendação representa,
Para entender o risco de concentração antes de ver os detalhes completos.

**Acceptance Criteria:**

**Given** que uma recomendação foi exibida
**When** tento ver os detalhes completos (trilha/projeção)
**Then** um modal bloqueante pede o valor total (R$) que pretendo investir em renda variável e o percentual desta recomendação nesse total
**And** o modal avisa explicitamente que uma ação avulsa não é, por definição, conservadora — mesmo compatível com meu perfil
**And** só depois de confirmar, a trilha de transparência e a projeção são liberadas

**Critério de aceite de acessibilidade (não-opcional — só marca "pronto" se todos passarem):**
**Given** que o modal está aberto
**Then** o foco move automaticamente pro primeiro elemento interativo
**And** o foco fica preso dentro do modal (Tab/Shift+Tab não escapam)
**And** Esc fecha o modal sem confirmar (o gate permanece pendente)
**And** clique fora do modal explicitamente NÃO fecha
**And** a lista de elementos focáveis é recalculada a cada mudança de estado interno (UX-DR18) — testar isso é parte do Definition of Done da story, não um extra opcional

### Story 2.5: Trilha de transparência expansível

Como uma usuária que confirmou o gate de concentração,
Eu quero expandir e ver exatamente quais dados fundamentaram a recomendação,
Para confiar na decisão.

**Acceptance Criteria:**

**Given** que confirmei o gate de concentração
**When** expando "ver a trilha de transparência completa"
**Then** vejo indicadores fundamentalistas e padrões técnicos com valor, fonte e data de referência — nunca só o nome do indicador
**And** o componente usa `<details>/<summary>` nativo (UX-DR7), sem JS de terceiros
**And** fechado por padrão, um nível só, sem accordion aninhado

### Story 2.6: Projeção de retorno e cálculo de IR

Como uma usuária vendo a trilha expandida,
Eu quero ver a projeção de retorno e quanto pagaria de IR,
Para decidir com clareza fiscal.

**Acceptance Criteria:**

**Given** que a trilha de transparência está expandida
**When** vejo a seção de projeção
**Then** vejo retorno médio projetado (curto/médio/longo prazo) em `Decimal`, líquido de alíquota de swing trade (15%)
**And** vejo dois cenários lado a lado pra isenção de R$20k/mês ("se esta for sua única venda" vs. "se você já vendeu outras ações") — nunca um número único definitivo
**And** vejo uma recomendação explícita de consulta a um contador antes de declarar
**And** como não há posição registrada ainda (Epic 3), todo número é rotulado como simulação ("como teria se comportado")

## Epic 3: Acompanhamento e Posição

Usuário registra que comprou (ou vendeu) de verdade, e volta depois pra ver o status atualizado da recomendação — silêncio quando nada mudou, alerta quando algo relevante mudou.

### Story 3.1: Registro de posição (compra)

Como uma usuária que decidiu investir,
Eu quero registrar que comprei a ação recomendada (data, preço, quantidade),
Para que o sistema passe a acompanhar minha posição real.

**Acceptance Criteria:**

**Given** que vi uma recomendação completa
**When** confirmo que comprei, informando data, preço e quantidade
**Then** o registro é salvo em `portfolio` (dono único — AD-11), amarrado à recomendação e à minha conta
**And** a partir daí, os números ligados a essa recomendação deixam de ser rotulados como simulação e passam a refletir minha posição real
**And** sem esse registro, a evolução exibida continua rotulada como "como teria se comportado" (UX-DR23)

### Story 3.2: Indicador visual de status (termômetro)

Como uma usuária com uma recomendação ativa,
Eu quero ver o status dela de forma visual e simples,
Para entender a situação sem interpretar números técnicos.

**Depende de:** Story 3.1 (a função canônica `portfolio.services.get_current_value()` usada aqui é criada nessa story).

**Acceptance Criteria:**

**Given** que tenho uma recomendação ativa (com ou sem posição registrada)
**When** vejo o Painel Único
**Then** o status é comunicado por um termômetro (favorável/atenção/alerta) sempre acompanhado de selo de texto — nunca só cor (UX-DR16)
**And** os valores numéricos completos só aparecem ao expandir a trilha de transparência
**And** o valor "atual" exibido vem sempre da função canônica de `portfolio.services` (AD-15), nunca recalculado localmente por `dashboard`

### Story 3.3: Alerta de mudança relevante

Como uma usuária que já tem uma recomendação,
Eu quero ser avisada só quando algo relevante mudar,
Para não ser incomodada à toa.

**Acceptance Criteria:**

**Given** que não houve mudança relevante desde minha última visita
**When** abro o Painel Único
**Then** não vejo nenhum alerta, badge ou notificação — o silêncio é o estado esperado
**And** se algo relevante mudou (preço/indicador), vejo um único alerta visível no painel, nunca uma notificação externa
**And** o alerta só é calculado sobre dados atualizados com sucesso pelo job diário (nunca sobre cotação potencialmente obsoleta — AD-5)

### Story 3.4: Registro de venda e isenção real de IR

Como uma usuária que vendeu uma posição registrada,
Eu quero registrar a venda,
Para que o cálculo de isenção e DARF passe a refletir minhas vendas reais dentro do sistema.

**Acceptance Criteria:**

**Given** que tenho uma posição registrada (Story 3.1)
**When** registro uma venda (data, preço, quantidade)
**Then** o registro é salvo em `portfolio` como dono único (AD-13), referenciando a posição de origem
**And** o cálculo de isenção de R$20k/mês passa a somar vendas registradas no sistema naquele mês (continua avisando que vendas fora da plataforma não são visíveis)
**And** `tax` lê esse registro só pela interface pública de `portfolio`, nunca cria model próprio
