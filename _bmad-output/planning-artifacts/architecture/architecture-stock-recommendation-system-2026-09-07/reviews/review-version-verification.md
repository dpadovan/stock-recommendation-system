# Review — Verificação de Versões e Realidade de Mercado (Lente: Reality-Check Técnico)

**Alvo:** `ARCHITECTURE-SPINE.md` (Sistema de Recomendação de Ações), seção "Stack" e AD-6.
**Data da verificação:** 2026-09-07 (busca ao vivo na web, conforme solicitado).

## Veredito geral

A escolha estrutural (Django + PostgreSQL + Render, deploy via git-push, Cron Jobs nativo) continua correta e sustentada pela realidade atual da plataforma, mas dois dos três pinos de versão na tabela "Stack" (Python 3.13 e PostgreSQL 16/17) já estão defasados frente ao que Render oferece hoje e precisam ser atualizados antes do build; a escolha de Django 5.2 LTS, ao contrário, se confirma como a correta mesmo com uma versão mais nova (6.0) já disponível.

## Achados

| # | Claim do spine | O que foi verificado (fonte) | Status | Severidade |
|---|---|---|---|---|
| 1 | **Python 3.13** — `[confirmar versão estável suportada pelo runtime do Render antes do build]` | A versão estável atual do Python é a 3.14 (3.14.7 lançado, com 3.14 sendo a série atual; 3.15 ainda em release candidate, previsto para outubro/2026). O runtime nativo do Render, para serviços criados a partir de 11/02/2026, já usa **Python 3.14.3 como default**; serviços criados antes disso (jun/2025–fev/2026) tinham default 3.13.4. Fontes: HeroDevs Python EOL timeline; endoflife.date/python; Render Docs — "Setting Your Python Version" (render.com/docs/python-version). | **Precisa atualizar** | **Alto** — o hedge do spine estava correto em pedir confirmação, e a confirmação muda a resposta: pinar em 3.13 hoje significa começar já uma versão atrás do default da própria plataforma de hospedagem escolhida. |
| 2 | **Django 5.2 LTS** — `[confirmar se uma versão mais recente (ex. 6.x) já é a atual recomendada antes do build]` | Existe sim uma versão mais nova, **Django 6.0.8** (release padrão, não-LTS, requer Python ≥ 3.12), mas a próxima LTS (**Django 6.2**) ainda está em desenvolvimento (dev snapshots datados de ago/2026) — ainda não lançada em versão estável. **Django 5.2.17 LTS continua sendo a LTS vigente e suportada**, com compatibilidade Python 3.10–3.14. Fontes: Django Project weblog "Django is moving to an annual release cycle" (10/ago/2026); docs.djangoproject.com release notes 5.2 e 6.0; endoflife.date/django. | **Confirmado** (a escolha, não a redação) | **Médio** — a versão escolhida (5.2 LTS) é de fato a correta para um projeto fintech solo que valoriza suporte longo (LTS ~3 anos) sobre "a mais nova"; porém a tabela deveria registrar explicitamente essa comparação (5.2 LTS vs. 6.0 não-LTS vs. 6.2 LTS ainda não lançada) em vez de deixar só o hedge em aberto, para não parecer indecisão não resolvida. |
| 3 | **PostgreSQL 16 ou 17** — `[confirmar versão gerenciada oferecida pelo Render no momento do provisionamento]` | Render Postgres já oferece **PostgreSQL 18** (changelog "PostgreSQL 18 is now available for Render Postgres databases"), que passou a ser o **default para novos bancos**. Versões majors 13–18 disponíveis para novas instâncias; 11–12 só para quem já tinha banco nessas versões. Fontes: render.com/changelog/postgresql-18-is-now-available-for-render-postgres-databases; render.com/docs/postgresql-upgrading. | **Precisa atualizar** | **Crítico** — este é o hedge que mais rapidamente se tornaria enganoso: alguém provisionando hoje a partir do spine (sem reler o hedge) receberá por padrão a versão 18, não 16/17 como a tabela sugere, então o número na tabela já não reflete o que será de fato provisionado. |
| 4 | **Render — existência, Cron Jobs nativo, git-push deploy** (AD-6) | Confirmado nos três pontos: (a) Render existe e está ativo como PaaS em 2026; (b) Cron Jobs é um tipo de serviço nativo dedicado (execução isolada, garantia de no máximo uma execução simultânea por job, billing por segundo com mínimo de US$1/mês por cron job); (c) deploy via git-push para apps Django é fluxo oficial e documentado (build script, migrations, coleta de estáticos, deploy zero-downtime automático a cada push). Fontes: render.com/docs/cronjobs; render.com/articles/how-render-handles-scheduled-tasks; render.com/docs/deploy-django; render.com/templates/django. | **Confirmado** | **Baixo** — a decisão AD-6 permanece sólida e verificada ao vivo; nenhuma mudança material (depreciação de feature, mudança de modelo que quebre a decisão) foi encontrada. Nota operacional (não bloqueadora): cron job no Render tem cobrança mínima de US$1/mês por job — vale mencionar no orçamento, não na arquitetura. |
| 5 | **Ausência de citação de fonte nos hedges originais** (todo o bloco "Stack") | O próprio spine já assume no cabeçalho da seção que os números "refletem o melhor conhecimento disponível [...] sem consulta ao vivo às páginas de release". Isso está correto como prática de honestidade, mas dois dos três hedges (achados #1 e #3) já podem ser resolvidos agora com dados verificados, restando apenas re-confirmar o pino exato no momento real do provisionamento (que pode estar alguns patches à frente mesmo dos números aqui verificados). | **Confirmado como prática, conteúdo desatualizado** | **Médio** — recomenda-se substituir os hedges por versões concretas + data de verificação, mantendo apenas uma nota final tipo "reconfirmar patch exato no dia do provisionamento", em vez do hedge aberto atual. |

## Recomendação objetiva para a tabela Stack

| Name | Sugestão com base na verificação (2026-09-07) |
| --- | --- |
| Python | 3.14 (acompanhar default do Render, hoje 3.14.3 para serviços novos) |
| Django | 5.2 LTS (mantido — 6.2 LTS ainda não lançada; 6.0 é não-LTS e não recomendado para este perfil de projeto) |
| PostgreSQL | 18 (default atual do Render Postgres para bancos novos; 13–17 ainda disponíveis se houver razão específica para não usar o mais novo) |
| Render | Sem mudança — Cron Jobs nativo e deploy git-push confirmados vigentes |

## Sources

- [HeroDevs — Python End-of-Life Dates: Every Version's Support Timeline](https://www.herodevs.com/blog-posts/python-end-of-life-dates-every-versions-support-timeline)
- [endoflife.date — Python](https://endoflife.date/python)
- [Render Docs — Setting Your Python Version](https://render.com/docs/python-version)
- [Django Project Weblog — Django is moving to an annual release cycle (2026-08-10)](https://www.djangoproject.com/weblog/2026/aug/10/annual-release-cycle/)
- [Django docs — 6.0 release notes](https://docs.djangoproject.com/en/6.0/releases/6.0/)
- [Django docs — 5.2 release notes](https://docs.djangoproject.com/en/5.2/releases/5.2/)
- [Django docs — 6.2 release notes (under development)](https://docs.djangoproject.com/en/dev/releases/6.2/)
- [endoflife.date — Django](https://endoflife.date/django)
- [Render Changelog — PostgreSQL 18 is now available for Render Postgres databases](https://render.com/changelog/postgresql-18-is-now-available-for-render-postgres-databases)
- [Render Docs — Upgrading Your Render Postgres Version](https://render.com/docs/postgresql-upgrading)
- [Render Docs — Cron Jobs](https://render.com/docs/cronjobs)
- [Render Articles — How Render handles scheduled tasks](https://render.com/articles/how-render-handles-scheduled-tasks)
- [Render Docs — Deploy a Django App on Render](https://render.com/docs/deploy-django)
- [Render Templates — Django](https://render.com/templates/django)
