---
title: Extração de Fonte para UX Discovery — Sistema de Recomendação de Ações
source-1: _bmad-output/planning-artifacts/prds/prd-stock-recommendation-system-2026-08-30/prd.md
source-2: _bmad-output/brainstorming/brainstorm-recomendacao-acoes-leigos-2026-08-25/brainstorm-intent.md
purpose: Insumo para DESIGN.md (identidade visual) e EXPERIENCE.md (IA/comportamento)
---

# 1. Jornadas-Chave do Usuário (verbatim, PRD §2.3)

Estas devem migrar para a seção "Key Flows" do EXPERIENCE.md com os mesmos nomes/IDs (UJ-1, UJ-2, UJ-3).

## UJ-1. Marta cria sua conta e define seu perfil.

- **Persona + contexto:** Marta, leiga em investimentos, tem dinheiro parado na poupança há anos por medo e falta de conhecimento. Chegou ao site por indicação de alguém.
- **Estado de entrada:** não autenticada, acessando pelo navegador (desktop ou mobile web).
- **Caminho:** Acessa a página inicial → cria conta com e-mail e senha → responde poucas perguntas objetivas (perfil de risco: conservador/moderado/arrojado; horizonte de investimento: curto/médio/longo prazo) → informa opcionalmente quanto tem hoje parado na poupança.
- **Clímax:** ao concluir, cai direto no Painel Único (FR-3) — nenhuma etapa intermediária, nenhum formulário longo.
- **Resolução:** Marta está no painel, pronta para ver o UJ-2.
- **Edge case:** se ela pular o valor da poupança, a tela de comparação (FR-4) usa um valor de exemplo ilustrativo e a convida a inserir o valor real depois.

## UJ-2. Marta enfrenta o custo de não ter feito nada — e recebe uma recomendação.

- **Persona + contexto:** mesma Marta, agora dentro do Painel Único pela primeira vez.
- **Estado de entrada:** autenticada, perfil já preenchido (UJ-1).
- **Caminho:** o painel abre mostrando, em destaque, quanto o valor informado por ela já deixou de render comparado à poupança e, logo abaixo, ao CDI (FR-4) — com o disclaimer educacional embutido no texto, não em rodapé (FR-9). Na sequência, vê **uma** recomendação de ação, compatível com seu perfil conservador/longo prazo, com explicação resumida do porquê (FR-5). Antes de qualquer ação, o sistema pede que ela confirme qual percentual do total pretende alocar ali, com um aviso claro de concentração (FR-6).
- **Clímax:** ela expande a recomendação e vê a trilha de transparência — quais indicadores fundamentalistas e quais padrões gráficos pesaram (FR-7) — e a projeção de retorno médio (curto/médio/longo prazo) já líquida de IR estimado (FR-8).
- **Resolução:** Marta sai da sessão sem ter comprado nada ainda, mas entendendo exatamente por que aquela ação foi sugerida e quanto ela pagaria de imposto se decidisse agir — a decisão continua sendo dela.
- **Edge case:** se os dados do provedor financeiro estiverem indisponíveis para o ativo, o painel comunica isso explicitamente em vez de mostrar uma recomendação incompleta ou desatualizada.

## UJ-3. Marta volta duas semanas depois.

- **Persona + contexto:** já tem uma recomendação registrada; não é notificada com frequência.
- **Estado de entrada:** autenticada, retornando ao painel.
- **Caminho:** login → painel mostra o status atualizado da recomendação anterior em linguagem visual simples (semáforo/termômetro, FR-11) — sem pedir nenhuma ação se estiver tudo dentro do esperado.
- **Clímax:** o silêncio é o produto funcionando — nenhum alerta, nenhuma pressão para agir.
- **Resolução:** ela fecha o painel tranquila; só recebe um alerta ativo se algo relevante mudou (FR-10).

---

# 2. Sinais Visuais e de Tom (explícitos e implícitos)

## 2.1 "Painel único"
- **Painel Único** é o termo oficial (Glossário, §3 do PRD): "a interface central do produto: uma única tela onde o usuário acessa perda por inércia, recomendação, trilha de transparência e projeção."
- FR-3 (Painel único consolidado): tudo em uma única tela, **sem navegação entre páginas separadas**. Expansão/detalhe acontece **inline** (ex: acordeões, modais) — nunca uma página nova.
- Na primeira visita, um **guia rápido (tour inline)** apresenta as seções do painel; não reaparece em visitas seguintes salvo se o usuário pedir para revê-lo. `[NOTE FOR PM no PRD]`: formato exato do tour (destaques na tela vs. vídeo curto) é decisão de UX, não travada.
- Feature 4.2 chama o Painel Único de "manifestação direta da camada de redução de ansiedade": deve se manter **calmo por padrão**, e só chamar atenção quando algo relevante de fato mudou.

## 2.2 Semáforo / termômetro (linguagem visual simplificada)
- Feature 4.9 "Linguagem Visual Simplificada" (Should, FR-11): usa metáforas visuais (semáforo, termômetro) para comunicar status, **reduzindo dependência de números e gráficos técnicos crus como linguagem principal**.
- FR-11: status de qualquer recomendação comunicado **por padrão** via indicador visual simples (ex: semáforo verde/amarelo/vermelho); dados numéricos completos disponíveis **apenas ao expandir** (liga com FR-7, trilha de transparência).
- Consequência testável de FR-11: nenhuma tela do Painel Único exibe números técnicos crus como primeira camada de informação sem o equivalente visual simplificado ao lado.
- Se FR-11 for cortado (Should cortável): painel mostra recomendação/status em **texto simples**, sem semáforo/termômetro — enfraquece a tese de "painel calmo" mas não quebra o produto (§6.1).
- Brainstorm: "painel visual interativo e intuitivo (linguagem tipo semáforo/termômetro)" é parte do "combo vencedor" da direção de produto escolhida.

## 2.3 Tom "calmo" / silêncio como produto
- Visão (§1 do PRD): a camada de redução de ansiedade é "um painel único e calmo, linguagem visual simples (...), monitoramento contínuo silencioso".
- UJ-3: "o silêncio é o produto funcionando — nenhum alerta, nenhuma pressão para agir." Ela "fecha o painel tranquila".
- FR-10 (Monitoramento/Alerta): sem eventos relevantes, **o painel não gera nenhum alerta ou notificação**.
- Contra-métrica SM-C3: frequência de alertas não deve ser otimizada para cima — retenção via alertas mais frequentes contradiz a tese de "silêncio é o produto".
- Contra-métrica SM-C1: frequência de novas recomendações por usuário não deve subir — um número alto sugeriria o produto empurrando decisões, oposto da tese de reduzir ansiedade.

## 2.4 Regras de linguagem / disclaimer (não-imperativa)
- FR-9 (Disclaimer educacional embutido): o texto de toda recomendação e da tela de comparação (FR-4) comunica, **de forma integrada (não em rodapé ou modal separado)**, que o sistema é educacional/informativo e a decisão final é do usuário.
- Consequência testável: **nenhuma recomendação usa linguagem imperativa** do tipo "compre esta ação" — usa formulações como **"os dados indicam"** ou equivalente.
- Disclaimer aparece na primeira visita ao Painel Único **e em todo ponto de recomendação** (FR-5).
- FR-4: o texto inclui, **na mesma tela** (não em rodapé ou link separado), a frase de que rentabilidade passada não garante rentabilidade futura, integrada ao disclaimer (FR-9).
- FR-5: explicação usa **linguagem sem jargão técnico não explicado** — qualquer termo técnico é definido inline ou vinculado ao Glossário.
- FR-5: recomendação nunca é chamada de "adequada" ou "compatível" com o perfil conservador de forma irrestrita — reconhece explicitamente que ação avulsa não é, por si só, investimento conservador.
- FR-6 (Aviso de concentração): texto **afirma explicitamente** que uma ação avulsa não é, por definição, investimento conservador — não é apenas alerta de percentual.
- Brainstorm — insight-chave 2: "o disclaimer educacional da CVM resolve dois problemas com a mesma frase: a exigência regulatória e o medo de julgamento/vergonha do usuário." Isso é um argumento de tom, não só de compliance — a linguagem do disclaimer deve soar acolhedora, não apenas legal.

## 2.5 Arquitetura da informação: telas/ordem
- FR-4 (Tela "Quanto Você Perdeu Parado"): primeiro conteúdo visto após cadastro. Poupança aparece **antes** do CDI na ordem de exibição. "A comparação nunca aparece isolada: a mesma tela emenda direto na recomendação (FR-5), por design" — mostrar "quanto você perdeu" sem uma ação imediata na sequência reforçaria culpa/ansiedade em vez de resolvê-la. Isso é uma restrição de sequenciamento de tela forte para a IA.
- Ordem implícita do fluxo dentro do Painel Único (via UJ-2 + FR-3): (1) comparação de perda por inércia (FR-4, com disclaimer embutido) → (2) recomendação única resumida (FR-5) → (3) confirmação de percentual + aviso de concentração (FR-6, **gate obrigatório antes de ver detalhe completo**) → (4) recomendação completa desbloqueada: trilha de transparência expansível (FR-7) + projeção de retorno líquida de IR (FR-8).
- FR-6: "Recomendação completa (FR-5, FR-7, FR-8) só é exibida após essa confirmação [percentual de alocação]." Ou seja, o aviso de concentração funciona como **etapa de confirmação bloqueante (gate)**, não como banner dispensável.
- FR-3: guia rápido de uso também mora dentro do Painel Único.

---

# 3. FRs com Implicação Direta de UI/Interação

| FR | Nome | Implicação de UI/interação |
|---|---|---|
| FR-1 | Cadastro via e-mail e senha | Fluxo de cadastro simples; após sucesso, autenticação automática segue direto para FR-2 (sem tela intermediária). Fluxo de recuperação de senha por e-mail. Sem login social/OAuth, sem MFA no v1. |
| FR-12 | Consentimento e exclusão de conta | Cadastro exibe e **exige aceite de texto específico** sobre uso de dados financeiros (não checkbox genérico de "termos de uso") — é uma etapa de UI distinta dentro do fluxo de cadastro (UJ-1). A partir do Painel Único, usuário autenticado encontra opção para excluir conta e dados associados (provavelmente em configurações/perfil, acessível a partir do painel único, não em página separada de "conta"). |
| FR-2 | Perfil de risco e horizonte | Formulário de poucas perguntas objetivas (parte do fluxo de cadastro, UJ-1); editável depois a partir do Painel Único. Combinações não calibradas (moderado/arrojado, curto/médio prazo) mostram mensagem "ainda não suportado neste perfil" em vez de recomendação — estado vazio/bloqueado a desenhar. |
| FR-3 | Painel único consolidado | Já detalhado na seção 2.5. Tudo inline (acordeões/modais), sem páginas separadas. Tour inline na primeira visita. |
| FR-4 | Comparação de perda (poupança/CDI) | Primeiro conteúdo pós-cadastro; poupança antes do CDI; disclaimer inline na mesma tela; valor ilustrativo + CTA para inserir valor real se poupança não informada; sem estado "isolado" — sempre emenda para a recomendação. |
| FR-5 | Recomendação combinada com explicação | Card/seção de recomendação única (nunca mais de uma ação por vez); explicação resumida visível "acima da dobra" dentro do painel, com termos técnicos linkáveis/definidos inline; nunca linguagem imperativa. |
| FR-6 | Aviso de concentração | **Etapa de confirmação obrigatória** (visualmente distinta de checkbox de termos) pedindo % do total a alocar, antes de desbloquear FR-5 completo/FR-7/FR-8; exibe aviso quando percentual sugere concentração excessiva (limiar ainda não definido — Questão em Aberto #5, não travar história de engenharia sem o número). |
| FR-7 | Trilha de transparência | Expansível **inline** (acordeão/modal) a partir da recomendação, dentro do mesmo Painel Único; lista indicadores fundamentalistas e padrões técnicos com valores usados (não só nomes), fonte do dado e data de referência. |
| FR-9 | Disclaimer educacional | Embutido no texto/copy, nunca em rodapé ou modal separado; aparece na primeira visita ao painel e em todo ponto de recomendação. |
| FR-10 | Alerta de mudança relevante *(Should)* | Alerta visível no Painel Único (não notificação externa obrigatória) só quando evento relevante ocorre; ausência de alerta é o estado padrão/esperado. Limiar de "relevante" não definido (Questão em Aberto #5). |
| FR-11 | Indicador visual de status *(Should)* | Semáforo/termômetro como camada visual padrão; números completos só ao expandir (liga com FR-7). |
| FR-12 | (ver acima, junto com cadastro) | — |
| FR-13 | Registro de posição | Fluxo para usuário confirmar compra (data, preço, quantidade) — provavelmente um formulário/modal acionado a partir da recomendação ou do painel. Sem esse registro, painel rotula claramente a evolução como "como teria se comportado desde a recomendação" (não resultado real) — importante diferenciação visual/textual entre dado hipotético e dado real do usuário. Afeta como FR-8/FR-10/FR-11 são rotulados ("personalizado" vs. genérico). |
| FR-8 | Projeção de retorno e IR | Mostra projeção curto/médio/longo prazo já líquida de IR estimado; dois cenários lado a lado para a isenção de R$20k/mês ("se esta for sua única venda..." vs. "se você já vendeu outras ações..."); rótulos claros de "estimativa", não garantia; texto explícito sobre limitação de compensação de prejuízos, recomendando contador. |

---

# 4. Restrições de Plataforma / Form Factor

- **MVP é web, sem app mobile nativo.** App mobile nativo é deferido para versão futura (§6.2 do PRD).
- **Web responsivo: desktop + mobile web** `[ASSUMPTION do PRD, §6.2 e §9]` — é a plataforma real do v1.
- Não há integração com Open Finance nem com corretoras — produto não executa ordens, é só recomendação/informação (§5, Não-Objetivos).
- Implicação de design: o Painel Único precisa funcionar como layout responsivo único (não duas experiências divergentes desktop/mobile), já que é a mesma interface central em ambos os form factors, com acordeões/modais como padrão de expansão em vez de páginas — isso favorece um approach mobile-first ou ao menos mobile-aware para os componentes de expansão inline.

---

# 5. Acessibilidade

**Nenhuma menção explícita a acessibilidade (WCAG, contraste, leitores de tela, navegação por teclado, tamanhos de fonte, etc.) foi encontrada em nenhum dos dois documentos-fonte.** Não invento requisitos — isso deve ser tratado como lacuna a decidir no próprio processo de UX Discovery, não como algo já definido no PRD ou no brainstorm.

Nota indireta: FR-11 e a "Linguagem Visual Simplificada" reduzem dependência de números/gráficos técnicos crus, o que tem *efeito colateral* positivo de acessibilidade cognitiva (menos carga numérica), mas isso não é enquadrado como requisito de acessibilidade no PRD — é enquadrado puramente como redução de ansiedade/simplicidade.

---

# 6. Estado Emocional do Usuário-Alvo (brainstorm-intent.md) — para tom/voz

- **Diz:** investir é arriscado e complicado demais.
- **Faz:** evita o assunto por completo, deixa tudo parado na poupança (**inércia como defesa**).
- **Sente:** **ansiedade e culpa** por não fazer nada com o dinheiro.
- Por trás do "é complicado" há **medo de julgamento/vergonha**, não só medo de perder dinheiro.
- **Job a ser demitido:** o "cunhado que entende de bolsa" — o sistema substitui esse conselheiro informal e precisa dar ao usuário **prova/justificativa para defender a decisão perante família e amigos** (job social, não só funcional).
- O concorrente real do produto **não é a falta de informação, é a inércia/segurança psicológica da poupança**.
- O valor do produto está em **absorver a responsabilidade da decisão**, dando "permissão simples e justificada para agir" — isso é uma pista de tom: a interface deve comunicar segurança e permissão, não pressão ou venda.
- PRD (Visão, §1) reforça: "o verdadeiro concorrente deste produto é a inércia, não a falta de uma ferramenta de análise."
- PRD (JTBD, §2.1):
  - Emocional: "Quero confiança para decidir, para não sentir medo de perder dinheiro nem vergonha de não entender."
  - Social: "Quero uma justificativa que eu possa defender perante família/amigos, para não depender só do palpite do cunhado que 'entende de bolsa'."
  - Funcional: "Quero saber em que ação investir, quanto devo esperar de retorno e quanto vou pagar de imposto, sem precisar aprender a interpretar balanço ou gráfico sozinha."
- Implicação de tom direta (FR-4): mostrar "quanto você perdeu" **nunca isolado** — sempre seguido imediatamente de uma recomendação acionável, porque isolar a perda reforçaria a culpa/ansiedade em vez de resolvê-la. Regra de sequência de copy tanto quanto de layout.
- Implicação de tom (FR-6): o aviso de concentração deve soar como cuidado genuíno ("essa ação sozinha carrega risco que nenhum perfil elimina"), não como aviso legal frio — é onde o produto é mais honesto sobre a limitação estrutural da ação avulsa.

---

# 7. Outras Notas Relevantes para EXPERIENCE.md / DESIGN.md

- Persona nomeada no PRD: **Marta** — usar como persona de referência em ambos os documentos de UX.
- Vocabulário oficial (Glossário §3 do PRD) a manter consistente em toda a UI: Investidor leigo, Perfil de risco, Horizonte de investimento, Ação avulsa, Aviso de concentração, Trilha de transparência, Painel Único, IR, Ganho de capital, DARF, CDI.
- Guia rápido / onboarding tour: aparece só na primeira visita ao Painel Único; revisitável sob demanda; formato (tour com destaques vs. vídeo) é decisão de UX em aberto — o UX Discovery pode/deve decidir isso.
- Rotulagem de dado hipotético vs. real (FR-13) é um padrão de UI recorrente: sempre que não houver registro de posição, qualquer número de performance deve ser marcado visualmente/textualmente como simulação ("como teria se comportado"), nunca como resultado real do usuário — vale para FR-8, FR-10, FR-11.
- Estado vazio/bloqueado a desenhar: usuário com perfil moderado/arrojado ou horizonte curto/médio prazo vê "ainda não suportado neste perfil" em vez de recomendação (FR-2/FR-5) — é uma tela real que precisa de tratamento de UX, não um edge case a ignorar.
- Estado de indisponibilidade de dados: painel comunica explicitamente quando dados do provedor estão indisponíveis/desatualizados (>1 dia útil), em vez de mostrar recomendação incompleta — outro estado de UI não-trivial a desenhar (liga com UJ-2 edge case e NFR de frescor de dados, §10 do PRD).
- Contra-métricas (SM-C1, SM-C2, SM-C3) funcionam como "guard rails" de design: qualquer padrão de UI que aumente frequência de alertas, de novas recomendações, ou que pareça consultoria não solicitada, vai contra a tese do produto — útil como checklist de revisão para o EXPERIENCE.md.
