# Higiene do Sono — Reorganização Interativa

> Status: implementado em 2026-08-07.

Data: 2026-08-07

## Contexto

O guia de Higiene do Sono é uma página única (`index.html`) com 11 seções em sequência linear. Ele já tem 3 ferramentas interativas — tracker de hábitos de 7 dias (localStorage), autodiagnóstico com pontuação, e exercício de respiração guiada 4-7-8 — mas todas ficam depois de 6 seções de texto corrido, incluindo as duas mais densas do site ("Por que importa", cheia de fisiologia/hormônios, e "Os pilares", com 6 subtemas empilhados). O dono do produto (FGT, treinador/nutricionista) acredita que os alunos não estão lendo por excesso de texto antes de chegar nas partes úteis.

**Objetivo:** reduzir a sensação de "muito texto" trazendo as ferramentas pra mais cedo, comprimindo as seções densas em accordion/abas (progressive disclosure), cortando repetição de conteúdo, e tornando o diagnóstico um gancho ativo que direciona o aluno — sem perder informação nem a leitura fluida do texto que permanece em prosa.

**Escopo:** reorganização de estrutura + progressive disclosure + edição de texto para concisão, tudo dentro do `index.html` único existente (sem build, sem novas dependências). Tracker, diagnóstico e respiração mantêm exatamente sua lógica/persistência atual (`localStorage`, chaves e IDs internos inalterados) — só mudam de lugar na página e ganham a melhoria descrita abaixo no diagnóstico.

## Nova ordem das seções (nav + anchors)

| # | Seção | Anchor | Mudança |
|---|---|---|---|
| 1 | Comece aqui | `#comece` | inalterada (já é curta e funciona como gancho) |
| 2 | Como está o seu sono? (diagnóstico) | `#diagnostico` | **movida** de posição 8 → 2; ganha CTA de resultado (ver abaixo) |
| 3 | Seu tracker de 7 dias | `#tracker` | **movida** de posição 7 → 3 |
| 4 | O seu dia para dormir bem (timeline) | `#dia` | movida de posição 3 → 4; conteúdo inalterado (já é visual/escaneável) |
| 5 | Por que o sono decide seus resultados | `#porque` | movida de posição 2 → 5; vira **accordion de 3 cards** (ver abaixo) |
| 6 | Os pilares, explicados | `#pilares` | movida de posição 4 → 6; vira **abas de 6 itens** (ver abaixo) |
| 7 | Referência rápida (tabela + faça/evite) | `#ref` | movida de posição 4→7; conteúdo inalterado (já é escaneável) |
| 8 | Respiração 4-7-8 | `#respiracao` | movida de posição 10 → 8 |
| 9 | Seu contexto (filhos pequenos / sem tempo) | `#contexto` | **fusão** de `#filhos` + `#tempo` em uma seção com **abas de 2 itens** |

A seção "Resumo" (`#resumo`) é **removida como seção própria** — seu papel de fechamento é absorvido pelo resultado do diagnóstico (que já entrega uma mensagem final personalizada) e pela lista consolidada de hábitos (ver abaixo). A frase de fechamento atual do Resumo ("Sono bem feito é a alavanca mais barata e poderosa que você tem...") migra para o footer, logo acima do aviso legal existente.

O nav (`<nav class="nav">`) passa de 11 para 9 links, na nova ordem acima. O scrollspy (`IntersectionObserver` já existente) continua funcionando sem mudança de lógica — só a lista de seções que ele observa muda.

## Lista de hábitos consolidada

Hoje a mesma lista de "hábitos prioritários" aparece 3 vezes, reformulada: caixa de 3 itens em "Comece aqui", caixa de 5 itens em "Sem tempo", e 6 frases no "Resumo". A versão de referência única passa a ser a caixa de **3 itens já existente em "Comece aqui"** (é a mais curta, a primeira que o aluno vê, e cobre o essencial: horário fixo, luz da manhã, cortar cafeína/telas).

- A caixa de 5 itens da aba "Sem tempo" (dentro de `#contexto`) é reescrita para **não repetir os 3 já ditos na abertura** — vira um complemento curto: "Você já viu os 3 ajustes de maior impacto na abertura. Pra quem tem pouquíssimo tempo, adicione mais dois: [item 4] e [item 5]" (os dois itens que não estão na lista de 3: "trate o sono como compromisso agendado" e "ajuste o ambiente uma vez").
- As "6 frases" do Resumo são retiradas como lista própria — a única frase que não é redundante com o resto do conteúdo (a de fechamento, sobre alavanca barata e poderosa) migra para o footer conforme acima; o restante já está coberto pelas outras seções.

## Accordion — "Por que importa" (`#porque`)

Vira 3 cards de accordion, reaproveitando o componente `.box`/`.card` já existente no design system da página (mesmos tokens de cor). Cada card:
- Cabeçalho sempre visível: emoji + título curto + indicador `+`/`−` à direita, clicável (`role="button"`, `aria-expanded`).
- Corpo colapsado por padrão, exceto o primeiro (abre com a página, dá o gancho imediato sem exigir clique).
- Cards, na ordem: **"O triângulo treino + nutrição + sono"** (aberto por padrão — o parágrafo de abertura + a tabela `.tri` existentes), **"Os números que provam isso"** (as 4 caixas `.stat` existentes), **"Por que dormir mal engorda"** (o parágrafo sobre leptina/grelina/cortisol existente).
- Conteúdo interno reaproveita os componentes visuais já existentes (`.tri`, `.stat`) sem alteração — só o texto de apoio ao redor é cortado/reescrito para ficar mais direto, mantendo leitura fluida (não vira lista de fragmentos).

## Abas — "Os pilares" (`#pilares`)

Vira navegação por abas horizontais (mesmo padrão visual do `.nav` do cabeçalho: pílulas, ativa em rosa sólido) com 6 abas, uma por subtema atual: Horário, Luz, Ambiente, Cafeína, Exercício, Ritual. Cada aba mostra o conteúdo daquele subtema; troca de aba não recarrega a página (JS simples, mesmo padrão do `switchTab` já usado no Mapa de Trocas — mostra/esconde `div`s por classe `.active`).

Dentro da aba **Luz**, o passo-a-passo textual atual de "como ativar o modo noturno" (hoje um parágrafo + instruções em prosa) é comprimido para uma lista curta de 3 passos diretos (Configurações → Tela → ativar com agendamento automático), mantendo a caixa verde "Configuração ideal" e a caixa azul "Outros recursos" como estão (já são escaneáveis).

## Abas — "Seu contexto" (`#contexto`, fusão de Filhos pequenos + Sem tempo)

Duas abas: **"Tenho filhos pequenos"** e **"Não tenho tempo"**, mesmo padrão de abas da seção Pilares. Cada aba mostra o conteúdo da respectiva seção atual (texto cortado/editado para concisão onde fizer sentido, mantendo o tom empático da seção "Filhos pequenos" e o tom direto da "Sem tempo"). A caixa de 5 hábitos da aba "Sem tempo" é reescrita conforme a seção "Lista de hábitos consolidada" acima.

## Diagnóstico ativo (`#diagnostico`)

O resultado (`updateResult()`, hoje só atualiza texto) ganha um CTA — botão ou link estilizado (reaproveitando `.btn-pink` ou `.btn-ghost` já existentes) — que muda conforme a pontuação:

| Pontuação | Mensagem (mantém o tom atual) | CTA novo |
|---|---|---|
| 0 | "Marque os sinais acima pra ver o resultado." | nenhum |
| 1 | "Seu sono provavelmente está te ajudando. Mantém o que você já faz." | botão "Ver seu tracker" → `#tracker` |
| 2–3 | "Sinal de alerta. Comece pelos 3 ajustes do topo do guia e acompanhe pelo tracker." | botão "Ver os 3 ajustes" → `#comece` |
| 4–8 | "O sono é a sua prioridade agora..." (mantém a frase e o convite pessoal existente) | botão "Ver todos os pilares" → `#pilares` |

O clique no CTA faz scroll suave até a âncora (`scrollIntoView`, já usado implicitamente pelo `scroll-behavior:smooth` do `html`). Nenhuma mudança na lógica de pontuação (`SIGNS`, `checked`, contagem) — só a função `updateResult()` ganha a renderização do botão/link condicional.

## Edição de texto

Onde o texto for reorganizado em accordion/abas ou cortado por redundância, o objetivo é **reduzir volume mantendo leitura fluida** — frases completas e naturais, não fragmentação em bullets onde hoje é prosa. Parágrafos que já são concisos (ex. "Comece aqui", a timeline "Seu dia", a tabela de referência) não são reescritos, só reposicionados. A voz/tom atual (direto, coloquial, "eu"/"você") é mantida integralmente.

## Fora de escopo

- Qualquer mudança na lógica/dados do tracker, diagnóstico ou respiração (chaves de `localStorage`, arrays `HABITS`/`SIGNS`/`PHASES` inalterados).
- Novas ferramentas interativas além do CTA do diagnóstico (sem gamificação/pontos, sem quiz de personalização de entrada — avaliado no brainstorm e descartado por ora).
- Mudança de paleta de cores, tipografia ou componentes visuais (cards, boxes, stats, tabela) — só a reorganização estrutural e o novo padrão de abas/accordion, que reaproveitam os tokens de cor já existentes no arquivo.
- SEO, analytics, ou qualquer integração externa nova.

## Restrições globais

- Um único arquivo `index.html`, sem build, sem bundler, sem novas dependências (mesmas fontes Google já carregadas).
- Todo texto em português (pt-BR), tom conversacional mantido.
- `localStorage` dos 3 recursos interativos preserva compatibilidade — quem já usou o tracker/diagnóstico antes desta mudança não perde dados salvos.
- Sem framework de testes automatizado — verificação manual, em navegador.
