---
name: 'Review Adversarial — Architecture Spine (Sistema de Recomendação de Ações)'
type: review
lens: adversarial
target: ../ARCHITECTURE-SPINE.md
status: draft
created: '2026-09-07'
---

# Review Adversarial — Architecture Spine

Papel assumido: adversário. Para cada achado abaixo, construí duas implementações de "um nível abaixo" (dois módulos/devs) que obedecem a letra de uma ou mais AD e ainda assim colidem — em forma de dado compartilhado, dono duplo de uma entidade, caminho de mutação de estado conflitante, ou uma Rule ambígua o suficiente pra ser satisfeita de dois jeitos contraditórios. Total de achados: **13**.

---

## 1. `services.py`/`selectors.py` não é interface — é convenção sem enforcement

**Duas implementações:** Dev A (`portfolio`) põe toda a lógica de negócio em `services.py` (`create_position()`, `get_position_for_user()`) e trata `models.py` como puro schema. Dev B (`tax`), pressionado por prazo, escreve a lógica de cálculo de IR como métodos/classmethods em `Position.objects` (manager customizado) dentro de `models.py`, e cria um `services.py` que é só um re-export de uma linha (`get_position = Position.objects.get_for_user`). Ambos apontam para a mesma Consistency Convention ("comunicação entre apps só via função pública de `services.py`/`selectors.py`") como cumprida — nenhuma AD define o que "interface pública" proíbe dentro do próprio app dono, só regula o *import* entre apps.

**AD/convenção cumprida na letra:** Consistency Conventions → "Acesso entre módulos"; AD-3.

**Furo:** a convenção nunca é testada — não há `__all__`, não há regra de lint (ex. `import-linter`/`flake8-tidy-imports`), não há teste de arquitetura que falhe se alguém importar `outro_app.models` fora de `services.py`. Qualquer builder pode alegar conformidade só por ter um arquivo chamado `services.py`, mesmo que ele não seja de fato a única porta de entrada.

**Fix:** adicionar uma AD (ou reforçar AD-3) que exija enforcement mecânico — ex. `import-linter` com contrato de camadas rodando no CI/pre-commit, falhando o build se qualquer app importar `models.py`/managers de outro app fora de `services.py`/`selectors.py`. Sem isso, a convenção é prosa, não arquitetura.

---

## 2. Leitura direta de `models.py` de outro app nunca foi proibida — só a escrita

**Duas implementações:** Dev A (`recommendations`) precisa do preço em cache de uma ação. Em vez de chamar `market_data.services.get_quote()`, faz `from market_data.models import Quote; Quote.objects.filter(...)` diretamente — argumenta que é *leitura*, não escrita, e a Consistency Convention diz literalmente "nunca import direto de `models.py` de outro app **para escrita**" (grifo na palavra escrita). Dev B, no mesmo app, sempre chama `market_data.services.get_quote()`. Ambos compilam, ambos passam qualquer review superficial, ambos "não escrevem" no model de outro app.

**AD/convenção cumprida na letra:** AD-4 (só proíbe acesso ao *provider*, não ao model interno de `market_data`); Consistency Conventions → "Acesso entre módulos" (a frase restringe só escrita).

**Furo:** a AD-4 quebra na prática — se `market_data` mudar o schema de `Quote` (renomear campo, trocar índice, mudar semântica de "preço mais recente"), o código do Dev A quebra silenciosamente sem que `market_data` saiba que tem esse consumidor. A camada anti-corrupção vale zero se leitura direta de `models.py` for uma porta lateral aberta.

**Fix:** tighten a regra para "nenhum import de `models.py` de outro app, para leitura ou escrita, sob nenhuma circunstância — inclusive em management commands e migrations de dados". Toda leitura cross-app passa por `services.py`/`selectors.py`, sem exceção redigida como "só escrita".

---

## 3. Logging cross-módulo: duas leituras opostas do "grafo sem seta = proibido"

**Duas implementações:** `market_data.update_market_data` (comando de management, AD-5) quer logar qual usuário disparou um refresh manual. Dev A interpreta a regra "market_data não depende de nenhum outro módulo interno" de forma absoluta: passa só um `user_id` inteiro cru pro log, nunca importa nada de `accounts`. Dev B interpreta que logging é "infraestrutura", não "lógica de negócio", e portanto não conta como a dependência que AD-3 proíbe — importa `accounts.services.get_username(user_id)` só para enriquecer a mensagem de log.

**AD/convenção cumprida na letra:** AD-3 ("market_data é o único módulo com exceção explícita: não depende de nenhum outro módulo interno") — o texto não distingue "dependência de negócio" de "dependência utilitária de logging".

**Furo:** o grafo do AD-3 não cobre a categoria "acoplamento incidental para observabilidade". Um builder motivado (Dev B) consegue justificar um `import accounts.services` dentro de `market_data` citando a mesma AD que Dev A cita pra recusar qualquer import. Isso quebra a promessa central de AD-3 ("market_data roda de forma autônoma") sem violar o texto.

**Fix:** adicionar uma frase explícita em AD-3: "a proibição de dependência de `market_data` é absoluta e cobre também logging/observabilidade — nenhum import de outro app interno, em nenhuma circunstância; contexto de usuário passa como dado primitivo (id, string), nunca via chamada a serviço de outro app."

---

## 4. AD-11 trava a posse do dado bruto, mas não da valorização derivada — drift entre `tax` e `dashboard`

**Duas implementações:** `dashboard` calcula "valor atual da posição" ao vivo, a cada request: `quantidade (portfolio) × preço atual (market_data)`. `tax`, para gerar o resumo de IR no momento da consulta, calcula e **cacheia** um "valor de referência" em campo próprio (ex. `TaxSnapshot.valor_atual`) no momento do último job diário (AD-5), porque recalcular a cada request seria caro para o cálculo de ganho de capital. No dia em que a cotação muda entre o job noturno e o acesso do usuário ao painel, `dashboard` mostra um valor e `tax` mostra outro — para a mesma posição, no mesmo Painel Único (FR-3 exige montagem única e coerente).

**AD/convenção cumprida na letra:** AD-11 ("`portfolio` é o único dono do registro... preço de entrada, data de entrada e quantidade... `tax` e `dashboard` leem esse registro via a interface pública"). Nenhum dos dois módulos grava em `Position` — ambos só leem o registro bruto e cada um computa seu próprio derivado.

**Furo:** AD-11 define posse do dado *bruto* de entrada, não posse do *cálculo derivado* (valor atual, lucro não realizado). Isso é exatamente o ponto cego citado no brief — nada impede dois donos concorrentes de "valor atual da posição".

**Fix:** nova AD atribuindo a uma função única (ex. `portfolio.services.get_current_value(position)`, já que `portfolio` é quem já possui a `Position` e está na camada certa do grafo AD-3 para consumir `market_data`) o cálculo canônico de valor derivado; `tax` e `dashboard` chamam essa função em vez de recalcular. Se cache for necessário por custo, o cache também vive em `portfolio`, com TTL explícito — nunca duplicado em `tax`.

---

## 5. "Dado Real vs. Simulado": o flag também não tem dono único

**Duas implementações:** `dashboard` calcula `tem_posicao = portfolio.services.has_position(user, recommendation)` e rotula todo número da tela como "real" com base nisso. `tax`, para decidir se aplica a isenção de ganho de capital e o texto "como teria se comportado" na sua própria seção de projeção, implementa sua própria checagem — por exemplo, exige adicionalmente que a posição tenha mais de um dia de existência (para evitar calcular ganho de capital sobre uma posição registrada há segundos), tratando-a como "ainda simulada" até esse critério bater. Resultado: a mesma posição pode aparecer como "real" no card de recomendação (`dashboard`) e como "simulação" na seção de projeção de IR (`tax`), na mesma tela — violando diretamente a regra do `EXPERIENCE.md` de que "nenhum estado real e nenhum estado simulado pode compartilhar a mesma frase" e de que o rótulo é transversal a FR-8/FR-10/FR-11.

**AD/convenção cumprida na letra:** AD-11 (cada módulo só lê `Position` via interface pública de `portfolio` — nenhum dos dois grava).

**Furo:** AD-11 não atribui posse da *decisão derivada* "esta posição conta como real para fins de exibição/cálculo" — só da entidade bruta. `EXPERIENCE.md` exige consistência de rotulagem entre módulos, mas nenhuma AD garante uma fonte única de verdade para o booleano usado nessa rotulagem.

**Fix:** estender a mesma AD do item 4 (ou uma nova) para cobrir também o flag "real vs. simulado": função canônica em `portfolio.services.is_real(position)` (ou equivalente), única fonte consumida por `tax`, `recommendations` e `dashboard`. Qualquer critério adicional (ex. carência mínima) vira parâmetro dessa função central, não uma reimplementação local.

---

## 6. AD-4 barra o `provider.py`, mas não impede um segundo caminho de acesso a dados de mercado "cru"

**Duas implementações:** Dev A, em `recommendations.engine`, precisa de um campo de fundamentos que `market_data.services` ainda não expõe. Em vez de esperar por uma nova função de serviço, importa `market_data.provider` diretamente "só dessa vez, é um campo simples" — nada no Python impede essa importação, e a Rule da AD-4 ("somente o app `market_data` importa/chama o cliente do provedor") é uma afirmação de intenção, não uma restrição executável. Dev B, no mesmo cenário, abre uma PR pedindo que `market_data.services` exponha o campo novo. Ambos "resolvem o mesmo problema"; só um viola a letra da AD, mas nada no repositório teria detectado a violação do Dev A antes de um code review humano atento.

**AD/convenção cumprida na letra:** tecnicamente Dev A *viola* AD-4 — mas o ponto adversarial é que a AD não é enforced por nada além de review manual, então "cumprir AD-4" e "violar AD-4 sem ser pego" são indistinguíveis em CI.

**Furo:** mesmo furo estrutural do item 1: nenhuma AD do spine é acompanhada de um mecanismo de verificação automática. Isso vale em dobro para AD-4, que existe especificamente para isolar o maior risco de viabilidade do produto (troca futura de provedor, citado em Deferred).

**Fix:** regra de import-linter (mesmo mecanismo do item 1) com um contrato específico: `provider.py` só pode ser importado de dentro do próprio pacote `market_data`. Isso transforma a Rule de intenção em invariante verificável a cada PR/CI run — sem isso, AD-4 é a AD mais crítica do documento e a menos protegida.

---

## 7. Ordem de execução dos dois cron jobs diários nunca foi travada

**Duas implementações:** Dev A configura o Cron Job do Render de `market_data.update_market_data` para `06:00` e o de `recommendations` (recálculo de status/alerta, FR-10) para `06:15`, assumindo que "diário" implica uma ordem implícita mercado→recomendação. Dev B, responsável só por `recommendations`, configura o cron desse job para `05:00` (antes do horário de mercado abrir, por hábito de outros projetos), sem saber que precisa rodar *depois* de `market_data`. Nenhuma AD diz "job B só roda após confirmação de sucesso do job A"; ambos os agendamentos "usam Cron Jobs nativos do Render disparando um management command", cumprindo AD-5 e AD-6 à risca.

**AD/convenção cumprida na letra:** AD-5 ("atualização de dados de mercado e verificação de alertas rodam via management command disparado por agendador externo") e AD-6 (Cron Jobs nativos do Render) — nenhuma menciona ordem ou dependência de sucesso entre os dois jobs.

**Furo:** `recommendations` pode recalcular status/alerta (FR-10) contra cotação do dia anterior sem nenhum erro visível — um alerta "mudança relevante" pode disparar ou deixar de disparar com base em dado velho, silenciosamente, porque a ordem dos crons é uma decisão de infraestrutura tratada como detalhe de deploy, não como invariante de arquitetura.

**Fix:** AD-5 precisa declarar explicitamente a ordem ("recommendations roda estritamente após o término bem-sucedido de market_data") e o mecanismo de garantia — ex. os dois passos dentro do *mesmo* management command/mesma execução de cron (um comando orquestrador que chama os dois em sequência e aborta o segundo se o primeiro falhar), em vez de dois Cron Jobs independentes no Render sem sinalização de sucesso entre si.

---

## 8. Cache de `market_data`: Redis "não é banco de dados", então nada impede reintroduzi-lo pela porta dos fundos

**Duas implementações:** Dev A, otimizando `market_data.services.get_quote()`, introduz `django.core.cache` com backend Redis gerenciado, justificando que AD-2 fala só de "sistema de registro" (o Postgres) e AD-5 só proíbe fila/broker assíncrono — um cache de leitura não é nenhum dos dois. Dev B resolve o mesmo problema de performance com uma tabela Postgres de cache (já coberta por AD-2, sem infra nova). Ambos "não violam" nenhuma AD lida literalmente.

**AD/convenção cumprida na letra:** AD-2 ("PostgreSQL é o único banco de dados... nenhum módulo introduz um datastore alternativo sem nova AD" — cache não é framed como "datastore de registro") e AD-5 (proíbe fila/broker, não cache).

**Furo:** o racional de fundo repetido nas duas ADs e no memlog é "complexidade operacional desnecessária para um dev solo" — Redis reintroduz exatamente esse tipo de peça de infra que o projeto todo foi desenhado para evitar, mas nenhuma AD nomeia "cache" explicitamente, deixando a porta aberta por omissão de vocabulário.

**Fix:** ampliar o "Prevents" de AD-2 (ou AD-5) para cobrir explicitamente qualquer serviço de cache externo (Redis, Memcached etc.) — não só bancos de registro e filas — travando que qualquer cache no v1 é implementado sobre Postgres ou em memória de processo (`LocMemCache`), nunca um serviço gerenciado adicional, sem nova AD.

---

## 9. Cascata de exclusão de conta (FR-12) entre `accounts`, `portfolio` e `tax` nunca foi decidida

**Duas implementações:** Dev A, em `portfolio.models.Position`, define `user = models.ForeignKey(..., on_delete=models.CASCADE)` — ao excluir a conta (FR-12: "exclusão da conta e dos dados associados"), todas as posições somem, cumprindo a letra da exclusão. Dev B, em `tax`, para preservar histórico de cálculo de IR por prudência regulatória (mesmo com MFA e CVM fora do escopo v1, ver Deferred), usa `on_delete=models.PROTECT` no FK que liga um registro fiscal ao usuário — argumentando que dado fiscal não deveria desaparecer graciosamente. No dia em que um usuário pede exclusão, o `CASCADE` de `portfolio` dispara, mas o `PROTECT` de `tax` lança `IntegrityError` e a operação de exclusão falha inteira (ou, se a ordem de cascade for diferente, deixa registros de `tax` órfãos apontando para um `Position` já deletado).

**AD/convenção cumprida na letra:** nenhuma AD trata FK `on_delete` para dados financeiros; AD-11 só define quem lê/escreve `Position` em operação normal, não o que acontece na exclusão de conta.

**Furo:** FR-12 (exclusão de conta e dados associados) é um requisito funcional explícito no PRD/UX, mas o spine não tem nenhuma AD sobre estratégia de exclusão/retenção em cascata para dados financeiros — cada app decide seu próprio `on_delete` de forma consistente *localmente* e incompatível *globalmente*.

**Fix:** nova AD definindo a estratégia de exclusão para FR-12 across `accounts`/`portfolio`/`tax` — ex. "exclusão de conta é sempre `CASCADE` até o fim da árvore de FKs, sem `PROTECT`/`SET_NULL` em nenhum dado financeiro pessoal, mesmo para fins de auditoria futura (a exigência de retenção regulatória, se vier, é tratada como nova AD quando a classificação CVM for revisitada — ver Deferred)".

---

## 10. Arredondamento de `Decimal`: AD-7 barra `float`, mas não trava a estratégia de `quantize`/rounding mode

**Duas implementações:** `tax.services.calcular_darf()` usa `valor.quantize(Decimal('0.01'), rounding=ROUND_HALF_UP)` (convenção comum de arredondamento fiscal no Brasil). `dashboard.views` exibe "lucro líquido estimado" formatando o mesmo `Decimal` sem `quantize()` explícito, deixando o arredondamento a cargo da formatação padrão do template Django (que usa `ROUND_HALF_EVEN`/banker's rounding do Python por trás, ou trunca dígitos na exibição sem normalizar o valor armazenado). Em um valor de borda (ex. `,125`), os dois números — que deveriam representar o mesmo lucro líquido — divergem em um centavo entre a tela do dashboard e o cálculo de IR da mesma recomendação.

**AD/convenção cumprida na letra:** AD-7 ("todo valor monetário ou percentual usa `Decimal`... `float` nunca é usado") — ambos usam `Decimal` do início ao fim, então ambos "cumprem" a AD.

**Furo:** AD-7 resolve o problema clássico de float, mas não define modo de arredondamento nem em que camada o `quantize` deve ocorrer (no cálculo, na persistência, ou só na exibição) — para um produto cuja identidade visual é "extrato bancário/recibo", números que não batem entre duas seções da mesma tela quebram a credibilidade central do produto sem violar nenhuma AD escrita.

**Fix:** estender AD-7 com uma regra de arredondamento única: "todo valor monetário final (para persistência e exibição) é normalizado com `.quantize(Decimal('0.01'), rounding=ROUND_HALF_UP)` no ponto de cálculo, antes de sair do módulo dono — nunca formatado/arredondado ad-hoc na camada de template."

---

## 11. Logging e PII: `AD-8` fala de repouso/trânsito, mas é mudo sobre o que entra nas mensagens de log

**Duas implementações:** Dev A, depurando um bug de cálculo de posição, loga `logger.info(f"Position criada: user={user.email}, preco={preco}, qtd={qtd}")` em `portfolio.services` — útil para debug, e nenhuma AD proíbe isso. Dev B, no mesmo app, segue a política implícita de nunca logar PII/dado financeiro identificável, logando só `logger.info(f"Position criada: user_id={user.id}")`. Ambos cumprem a única linha do spine sobre logging ("Logging padrão do Django via `logging` da stdlib, identificando o app de origem").

**AD/convenção cumprida na letra:** Consistency Conventions → "Logging"; AD-8 (que só fala de criptografia em repouso/trânsito do banco, nunca de conteúdo de log).

**Furo:** logs no Render tipicamente saem para um agregador/stream fora do controle direto de criptografia de campo — se e-mail, preço de posição ou dado de perfil de risco vazar para log em texto claro, isso contorna inteiramente o "piso de proteção" que a AD-8 declara como decisão consciente, porque a AD-8 nunca colocou log dentro do seu escopo "dados sensíveis".

**Fix:** estender o escopo de AD-8 (ou a linha de Logging da Consistency Convention) para proibir explicitamente PII/dado financeiro identificável em mensagens de log de nível `INFO` ou superior — logs referenciam sempre `user_id`/IDs internos, nunca e-mail, preço, ou valor de poupança em texto livre.

---

## 12. Migrations de dados como porta lateral para gravação cross-app em `Position`

**Duas implementações:** Dev A, precisando popular um novo campo em `Position` a partir de dados que hoje vivem em `recommendations` (ex. um backfill de "recomendação de origem" para posições legadas), escreve uma *data migration* dentro do próprio app `portfolio` que chama `portfolio.services` normalmente — sem violar nada. Dev B, resolvendo um problema simétrico a partir de `recommendations` (ex. popular um campo derivado em `Recommendation` a partir do estado de `Position`), escreve uma data migration **dentro do app `recommendations`** que importa `from portfolio.models import Position` diretamente para ler em bulk — migrations rodam fora do fluxo de request normal, e a Consistency Convention de "acesso entre módulos" nunca menciona explicitamente migrations como escopo.

**AD/convenção cumprida na letra:** Consistency Conventions → "Migrations" ("nenhuma migration manual fora do framework de migrations do Django") — ambos usam o framework de migrations do Django corretamente; nenhuma trata do *conteúdo* de código dentro de uma migration.

**Furo:** "nenhuma migration manual fora do framework" regula *como* a migration é registrada/aplicada, não *quais imports* uma migration pode fazer — deixando aberto exatamente o mesmo furo do item 2, mas num lugar historicamente esquecido em code review (ninguém revisa import boundary dentro de `migrations/0007_xxx.py` com o mesmo rigor que revisa `services.py`).

**Fix:** a mesma regra de import-linter dos itens 1/2/6 precisa cobrir explicitamente a pasta `migrations/` de cada app — nenhuma migration de um app importa `models.py` de outro app; leitura cross-app dentro de uma migration de dados só é permitida via a função pública `services.py`/`selectors.py` do app dono, chamada explicitamente a partir da migration.

---

## 13. Focus trap do gate de concentração: AD-9 nomeia o requisito, não a implementação — e o "clique fora" fica indefinido

**Duas implementações:** Dev A implementa o modal do gate de concentração com `<dialog>` nativo do HTML + `showModal()`, confiando no comportamento de foco que o próprio elemento `<dialog>` oferece nos browsers modernos, e adiciona um listener de `Esc` que cancela sem confirmar (conforme `EXPERIENCE.md`). Dev B implementa com uma `<div role="dialog" aria-modal="true">` e um trap manual via `keydown` que intercepta `Tab`/`Shift+Tab` sobre uma lista de elementos focáveis capturada uma única vez na abertura do modal — lista que fica desatualizada se o campo de total (R$) habilitar/desabilitar o botão "ajustar percentual" dinamicamente. Nenhum dos dois documenta se um clique fora do modal fecha o gate: Dev A deixa o `<dialog>` fechar em clique-fora por comportamento de biblioteca/costume, Dev B não implementa clique-fora (só `Esc` fecha), tratando o modal como preso até resposta explícita. `EXPERIENCE.md` define o comportamento de `Esc` e de `Tab`, mas nunca menciona clique-fora — cada Dev preenche essa lacuna com uma leitura oposta e igualmente "conforme".

**AD/convenção cumprida na letra:** AD-9 ("JavaScript vanilla é usado apenas onde estritamente necessário... focus-trap do modal — nunca um framework"); `EXPERIENCE.md` Accessibility Floor (foco automático, trap de Tab, Esc sem confirmar).

**Furo:** AD-9 e `EXPERIENCE.md` especificam *comportamento observável* mas não *contrato de implementação* nem cobrem clique-fora — como o gate de concentração é descrito como "bloqueio real de conteúdo" (não um banner dispensável), um clique-fora que fecha o modal (Dev A) contradiz o espírito do checkpoint obrigatório mesmo sem violar uma frase específica.

**Fix:** tighten AD-9 (ou criar AD nova de "Interação vanilla JS") especificando: (a) um único módulo compartilhado `static/js/focus-trap.js` reusado por todo modal do sistema (não reimplementado por app); (b) clique fora do modal do gate de concentração explicitamente **não fecha** o modal (só `Esc`, conforme já vale para `Esc`) — fechando a lacuna que `EXPERIENCE.md` deixou aberta; (c) a lista de elementos focáveis do trap é recalculada a cada mudança de estado do modal, não capturada uma única vez na abertura.

---

## Síntese

O padrão comum aos 13 achados: o spine é rico em *intenção* (o "Prevents" de cada AD é sempre bem articulado) mas pobre em *mecanismo de verificação* — quase toda AD de fronteira (services.py/selectors.py, AD-3, AD-4) depende de review humano disciplinado, e quase toda AD sobre dado derivado (AD-11, rotulagem real/simulado, valorização, arredondamento) resolve a posse do dado *bruto* sem resolver a posse do *cálculo* que dois módulos plausivelmente precisam fazer sobre esse dado. As duas lacunas mais graves em termos de risco de produto são #4/#5 (drift entre `tax` e `dashboard` no número que mais importa pra credibilidade do produto) e #7 (ordem dos cron jobs, que pode silenciosamente invalidar FR-10 sem nenhum erro visível).
