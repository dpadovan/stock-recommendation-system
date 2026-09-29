# Epic 1 Context: Cadastro e Perfil

<!-- Compiled from planning artifacts. Edit freely. Regenerate with compile-epic-context if planning docs change. -->

## Goal

Este épico entrega a porta de entrada do produto: uma visitante cria conta com e-mail e senha, consente explicitamente com o uso de seus dados financeiros e de perfil de risco, declara perfil de risco/horizonte de investimento (e opcionalmente quanto tem parado na poupança), e pode depois excluir sua conta e dados associados. É pré-requisito de tudo o mais — sem conta, perfil e consentimento registrados, nenhuma recomendação (Epic 2) pode ser gerada, e o produto não pode alegar base legal para tratar dados financeiros sensíveis do usuário.

## Stories

- Story 1.1: Cadastro com e-mail, senha e consentimento explícito
- Story 1.2: Login com e-mail e senha
- Story 1.3: Recuperação de senha
- Story 1.4: Configuração de perfil de risco e horizonte
- Story 1.5: Exclusão de conta e dados associados

## Requirements & Constraints

- Cadastro exige aceite de um texto **específico** sobre uso de dados financeiros/perfil de risco — nunca um checkbox genérico de "termos de uso". Sem esse aceite, o cadastro não se conclui.
- E-mail duplicado no cadastro gera erro claro, sem revelar se a conta pertence a outra pessoa. Credenciais incorretas no login geram erro genérico, sem indicar se o e-mail existe.
- Senha nunca é armazenada em texto plano (hash com salt).
- Recuperação de senha acontece por e-mail, com link válido por tempo limitado.
- Perfil de risco (conservador/moderado/arrojado), horizonte (curto/médio/longo) e valor de poupança (opcional) ficam associados à conta, persistem entre sessões e são editáveis depois. O formulário coleta todas as combinações, mesmo sabendo que o motor de recomendação (Epic 2) só está calibrado para conservador/longo prazo.
- Exclusão de conta exige um passo de confirmação explícito antes de ser efetivada; após confirmar, a sessão encerra e login deixa de funcionar.
- Sucesso de cadastro autentica automaticamente e segue direto para configuração de perfil, sem tela intermediária; ao concluir a configuração de perfil, o usuário cai direto no Painel Único (Epic 2) — sem etapa extra em nenhum dos dois pontos.

## Technical Decisions

- App `accounts` é dono de: autenticação, perfil de risco/horizonte, consentimento LGPD e a flag "guia rápido já visto" (esta última é estado de UI por usuário, não dado de perfil, mas mora aqui por ser 1:1 com o usuário).
- `accounts` está na camada 0 do grafo de dependências — não importa nenhum outro módulo interno (nem para logging). Outros módulos podem depender de `accounts`, nunca o inverso.
- Autenticação usa sessão nativa do Django (sem OAuth/login social, sem JWT/MFA no v1). Política de senha usa os `AUTH_PASSWORD_VALIDATORS` padrão do Django (mínimo 8 caracteres, não similar a dados do usuário, não em lista de senhas comuns, não inteiramente numérica) — sem validador customizado.
- Valor de poupança usa `Decimal`/`NUMERIC`, nunca `float`, normalizado com `ROUND_HALF_UP` no ponto de cálculo.
- Exclusão de conta é sempre `CASCADE` até o fim da árvore de FKs a partir de `accounts.User` — nunca `PROTECT`/`SET_NULL` em dado financeiro pessoal, mesmo por prudência de auditoria (essa regra também vale para módulos futuros — `portfolio`, `tax` — que ligarem FKs a `accounts.User`).
- Fronteira de módulo é reforçada mecanicamente via `import-linter` em CI/pre-commit: nenhum app pode importar `models.py` de outro app fora de `services.py`/`selectors.py`, nem em migrations.
- Sem criptografia adicional de campo no v1 (piso é o do provedor de hospedagem: criptografia em repouso + TLS). Logs nunca contêm PII/dado financeiro em texto livre — sempre referenciam `user_id`.
- Validação de formulário via Django Forms/ModelForms; erros exibidos inline junto ao campo.
- Sem ambiente de staging — deploy via git-push direto a produção.

## UX & Interaction Patterns

- Superfícies deste épico: Cadastro (Signup), Login, Recuperação de senha, Configuração de Perfil, Edição de Perfil e Gestão de Conta e Consentimento — cada uma com rota própria (diferente do Painel Único, que é single-surface).
- Componente **Input field**: fundo/borda padrão, destaque de borda no foco, rótulo sempre visível acima do campo (nunca só placeholder), valores numéricos em fonte monoespaçada tabular. Serve Cadastro, Configuração de Perfil e (mais tarde) Registro de Posição.
- Formulários têm labels associados programaticamente a cada campo; erros aparecem inline junto ao campo, nunca só por cor de borda.
- Piso de acessibilidade aplicável: contraste AA texto/fundo em light e dark; foco visível (anel de foco AA) em todo elemento interativo; alvos de toque ≥44px.
- Texto de consentimento no cadastro deve soar como aviso genuíno e específico sobre dados financeiros/perfil de risco, não como termo de uso frio e genérico — mesmo princípio de tom aplicado no restante do produto (evitar linguagem de "conselheiro"/pressão).
- Os tokens de design (cores, tipografia, espaçamento, raio) e os componentes de Botão primário/ghost só são formalmente estabelecidos na Story 2.2 (Epic 2), que é a primeira tela a fixar o design system "Recibo Caloroso". As telas deste épico (Cadastro, Login, etc.) precisam de UI funcional antes disso — considerar se usam uma versão mínima/provisória de estilo ou se a ordem de implementação é ajustada.

## Cross-Story Dependencies

- Story 1.1 → Story 1.4: sucesso no cadastro autentica automaticamente e encadeia direto na configuração de perfil, sem tela intermediária.
- Story 1.4 → Epic 2 (Painel Único): ao concluir a configuração de perfil, o usuário é redirecionado direto ao Painel Único — este épico não implementa o painel, apenas o ponto de entrada nele.
- Story 1.5 (exclusão de conta) depende de todo módulo futuro que referenciar `accounts.User` (ex.: `portfolio`, `tax` em épicos posteriores) manter `on_delete=CASCADE` — a regra é fixada aqui, mas sua efetividade completa só se confirma quando esses módulos existirem.
- Botão primário/ghost e tokens visuais formais chegam só na Story 2.2 (Epic 2); as telas deste épico são construídas antes dessa fundação de design system estar oficialmente fixada.
