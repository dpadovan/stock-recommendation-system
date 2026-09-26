# Reconciliação: ARCHITECTURE-SPINE vs. fontes upstream

**Spine avaliado:** `_bmad-output/planning-artifacts/architecture/architecture-stock-recommendation-system-2026-09-07/ARCHITECTURE-SPINE.md`
**Fontes:**
- `_bmad-output/planning-artifacts/prds/prd-stock-recommendation-system-2026-08-30/prd.md`
- `_bmad-output/planning-artifacts/ux-designs/ux-stock-recommendation-system-2026-09-05/DESIGN.md`
- `_bmad-output/planning-artifacts/ux-designs/ux-stock-recommendation-system-2026-09-05/EXPERIENCE.md`

Checado contra a seção **Deferred** do spine antes de qualquer item abaixo ser listado — itens já lá (limiares de FR-6/FR-10, provedor de dados, criptografia de campo, MFA, CVM, compensação de prejuízos, cobertura de perfil, app mobile, monetização) **não** são repetidos aqui.

---

## Gaps encontrados

- **FR-1 (política mínima de senha) — delegação explícita para a arquitetura, não cumprida.**
  O PRD diz literalmente: "Conta não é criada com... senha abaixo de um mínimo de segurança definido na arquitetura." O spine (AD-10) só diz que a autenticação usa "o sistema de auth embutido do Django (sessão de servidor, hashing de senha embutido)" — nunca define o mínimo em si (comprimento, complexidade, ou mesmo "aceitamos os `AUTH_PASSWORD_VALIDATORS` default do Django"). Isso não está na seção Deferred nem em nenhuma AD. É um requisito que o próprio PRD empurrou para a arquitetura resolver, e o spine simplesmente não pegou a bola.
  **Severidade: alta** — é um FR Must (FR-1), testável, e a lacuna é sobre segurança de conta que guarda dados financeiros.

- **FR-4 (comparação de rendimento perdido: poupança + CDI) — sem módulo dono claro; fonte de dado fora do escopo da AD-4.**
  Nenhum módulo do Structural Seed lista FR-4 (accounts cobre FR-1/2/12, market_data cobre cotações/fundamentos, recommendations cobre FR-5/7/10/11, portfolio cobre FR-6/13, tax cobre FR-8, dashboard só monta FR-3). O PRD diz que FR-4 usa "as mesmas séries históricas já exigidas" por FR-5/FR-8, mas CDI e rentabilidade da poupança são séries de referência macroeconômica (ex. API do Banco Central), categoricamente diferentes de cotação/fundamento de ação — e a AD-4 (camada anti-corrupção) está escopada explicitamente a "cliente do provedor externo de cotações e fundamentos", não a esse tipo de dado. Ninguém é dono de buscar/cachear CDI e taxa de poupança, e a AD-4 como redigida não cobre essa integração.
  **Severidade: alta** — FR-4 é Must (§6.1), é a primeira tela que o usuário vê (UJ-2), e a lacuna é estrutural (precisa de uma segunda integração externa não coberta por nenhuma AD).

- **NFR de frescor de dados (§10, PRD) — AD-5 não fecha o requisito, falta lógica de falha/retry e de "última atualização com sucesso".**
  O PRD exige: se a cotação tiver mais de 1 dia útil de atraso ou o provedor estiver indisponível, o painel comunica isso explicitamente (também presente como State Pattern em EXPERIENCE.md: "Dados do provedor indisponíveis ou desatualizados (> 1 dia útil)"). AD-5 define *como* o job roda (management command + Cron do Render, uma vez por dia, sem fila), mas não diz: (a) o que acontece se o job falhar (sem retry — a próxima tentativa só ocorre no próximo dia, o que sozinho já pode violar "1 dia útil" se cair num fim de semana/feriado); (b) quem calcula "dia útil" (calendário/feriados não é trivial e não está previsto em nenhum módulo); (c) que `market_data` deve persistir um timestamp de "última atualização bem-sucedida" para que `recommendations`/`dashboard` comparem contra o limiar e decidam mostrar o estado de "dados desatualizados". Nada disso está no Structural Seed nem em Deferred.
  **Severidade: alta** — é uma NFR explícita ligada a um Estado de UX já especificado; sem esse mecanismo, o requisito "comunica isso explicitamente" não tem como ser implementado de forma confiável.

- **Modo escuro obrigatório (EXPERIENCE.md, Foundation) — mecanismo de armazenamento/serviço da preferência de tema não endereçado.**
  EXPERIENCE.md exige paridade total light/dark desde o v1, mas não diz se existe alternância manual ou apenas `prefers-color-scheme`. Dado que AD-9 fixa renderização 100% server-side (DTL, sem framework JS) e AD-1/AD-2 fixam Postgres como único sistema de registro, uma alternância manual persistida (para não "piscar" o tema errado no primeiro paint server-rendered) implicaria decisão arquitetural — campo em `accounts` vs. cookie lido no request vs. só client-side via `localStorage`/CSS — que nenhuma AD, nem o Structural Seed (nenhum campo de preferência em `accounts/models.py`), aborda. Se a intenção for só CSS media query sem toggle, isso deveria estar dito explicitamente; hoje é silêncio, não decisão.
  **Severidade: média** — não bloqueia o build (dá pra resolver via CSS puro sem toggle), mas se o toggle for esperado, a lacuna é real e toca o modelo de dados de `accounts`.

- **FR-8 (tax) — acesso a dados de `market_data`/`recommendations` sob o grafo de dependência da AD-3 fica implícito, não regrado.**
  O grafo da AD-3 permite `tax -> accounts` e `tax -> portfolio`, mas não lista nenhuma aresta `tax -> market_data` nem `tax -> recommendations`. Porém FR-8 precisa de cotação atual (para lucro líquido simulado sem posição registrada) e de dados de projeção de retorno ligados à recomendação — dados que só existem em `market_data`/`recommendations`. O spine não deixa explícito que o padrão esperado é `dashboard` (a "camada de montagem final") buscar esses dados nos módulos de origem e *passá-los como parâmetro* para as funções puras de `tax.services`, em vez de `tax` importar esses módulos diretamente. Isso é resolvível dentro das regras da AD-3, mas fica implícito — um dev solo seguindo o grafo ao pé da letra pode travar tentando decidir se abre uma exceção nova na AD-3 ou usa o padrão de orquestração via `dashboard`.
  **Severidade: média** — não é uma contradição fatal (há um caminho válido: orquestração via `dashboard`), mas a spine não o declara, o que é exatamente o tipo de ambiguidade que uma AD deveria fechar.

- **FR-3 ("guia rápido não reaparece em visitas seguintes") — estado de "já visto o tour" sem dono de persistência.**
  O Structural Seed não dá `models.py` ao módulo `dashboard` (só `views.py`/`templates/`), o que é coerente com "dashboard só monta a visão". Mas então onde persiste a flag "usuário já viu o guia rápido"? Nenhuma AD ou módulo assume essa responsabilidade explicitamente (candidato natural seria um campo em `accounts`, mas isso não está dito).
  **Severidade: baixa** — é pouco provável que trave o build (resolução óbvia: campo booleano em `accounts`), mas é um requisito testável do FR-3 que ficou sem dono declarado.

---

## Itens verificados e considerados corretamente cobertos (não são gap)

- **Focus trap do modal de concentração + `<details>`/`<summary>` nativos:** AD-9 já abre exceção explícita para "JavaScript vanilla... slider de percentual do gate de concentração, focus-trap do modal" — não há tensão real com "vanilla JS only", pois focus trap é implementável sem framework, e a AD já reconhece isso nominalmente.
- **NFR de hashing de senha:** coberto por AD-10 (auth nativo do Django).
- **NFR de disponibilidade (sem meta formal):** compatível com a ausência de qualquer AD de SLA/uptime no spine — silêncio aqui é consistente com o PRD, não uma omissão.
- **FR-13/AD-11 (registro de posição) e rotulagem real vs. simulado:** bem coberto e citado explicitamente na AD-11.
