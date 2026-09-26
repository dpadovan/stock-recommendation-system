# Intenção do Produto: Sistema de Recomendação de Ações para Leigos

## Contexto / Problema

Sistema greenfield que recomenda ações (curto/médio/longo prazo) combinando análise técnica e fundamentalista, voltado a investidores leigos — pessoas que não entendem de mercado financeiro.

O concorrente real não é a falta de informação, é a **inércia/segurança psicológica da poupança**. A dor central do usuário é **ansiedade + culpa** por deixar dinheiro parado. O valor do produto está em **absorver a responsabilidade da decisão**, dando permissão simples e justificada para agir.

A real inovação não é o motor de análise técnica+fundamentalista (commoditizado) — é a **camada de redução de ansiedade**: painel calmo, linguagem visual simples, monitoramento contínuo silencioso.

## Usuário-alvo (realidade emocional)

- **Diz:** investir é arriscado e complicado demais.
- **Faz:** evita o assunto por completo, deixa tudo parado na poupança (inércia como defesa).
- **Sente:** ansiedade e culpa por não fazer nada com o dinheiro.
- Por trás do "é complicado" há **medo de julgamento/vergonha**, não só medo de perder dinheiro.
- **Job a ser demitido:** o cunhado "que entende de bolsa" — o sistema substitui esse conselheiro informal, e precisa dar ao usuário prova/justificativa para defender a decisão perante família e amigos.

## Requisitos não-negociáveis

- **Transparência total** dos dados usados na análise (trilha de transparência/auditoria) — exigência levantada pela voz do "trader cético".
- **Cálculo correto de IR sobre ganho de capital**, considerando: isenção de R$20k/mês em vendas no mercado à vista, alíquota diferenciada day trade (20%) vs swing trade (15%), DARF mensal, compensação de prejuízos entre operações. "Lucro líquido" não pode ser só venda menos compra.
- **Disclaimer de caráter educacional/informativo** — não é consultoria de investimentos regulamentada (exigência da CVM). Esse mesmo disclaimer resolve dois problemas ao mesmo tempo: a exigência regulatória e o medo de julgamento/vergonha do usuário.
- Cortar **day trade** do escopo (ver Won't) desarma a complexidade fiscal do cálculo de IR, tornando-o tratável na v1.

## Direção de produto escolhida

Combo vencedor da análise morfológica: **perfil conservador + horizonte de longo prazo + monitoramento contínuo do portfólio + painel visual interativo e intuitivo** (linguagem tipo semáforo/termômetro).

Dimensões consideradas: perfil de risco (conservador/moderado/arrojado) e nível de interação/automação (passivo com alerta pontual / ativo com chat conversacional / monitoramento contínuo).

## Escopo (MoSCoW)

**Must**
- Recomendação combinada técnica + fundamentalista, com explicação.
- Trilha de transparência dos dados usados.
- Disclaimer educacional (não é consultoria).
- Cálculo correto de IR sobre ganho de capital.
- Personalização por perfil de risco e horizonte de investimento.
- Painel visual central.

**Should**
- Monitoramento contínuo com alertas.
- Linguagem visual tipo semáforo/termômetro.

**Could**
- Modo "explica pro cunhado" (relatório simples e compartilhável).
- Chat conversacional ativo.
- Formato em áudio.

**Won't (por agora)**
- Suporte a day trade.
- Execução de ordens / integração com corretora.

## Insights-chave (síntese final)

1. O modo "explica pro cunhado", a transparência dos dados e o "demitir o cunhado" são, na verdade, **uma única feature**: um relatório rastreável e compartilhável, visto por três ângulos diferentes.
2. O disclaimer educacional da CVM resolve **dois problemas com a mesma frase**: a exigência regulatória e o medo de julgamento/vergonha do usuário.
3. Cortar day trade do escopo **desarma a complexidade fiscal** do cálculo de IR, tornando-o tratável na v1.
4. O diferencial real do produto não é o motor de análise (commoditizado) — é a **camada de redução de ansiedade**: painel calmo, linguagem semáforo/termômetro, monitoramento contínuo silencioso. Direção escolhida: conservador + longo prazo + monitoramento contínuo + painel visual interativo.
