# Higiene do Sono — Reorganização Interativa Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Reorganize the single-file Higiene do Sono guide so its 3 existing interactive tools (tracker, diagnostic, breathing) appear early, convert its two densest sections into accordion/tabs, merge two sections into a tabbed "Seu contexto", remove a 3x-repeated habits list down to one canonical version, and make the diagnostic result recommend a next section — all while keeping every existing `localStorage` key, JS function name, and interactive-tool behavior unchanged.

**Architecture:** Everything stays in the single `index.html` file (no build step, no new dependencies). Work proceeds in an order that keeps every intermediate commit fully working: first a pure structural reorder (Task 1, no content change), then section-by-section content transforms that each add their own small, self-contained CSS/JS (Tasks 2-4), then the two content-only cleanups (Tasks 5-6), then the diagnostic enhancement (Task 7), then a full regression pass (Task 8).

**Tech Stack:** Vanilla HTML/CSS/JS (no framework), existing Google Fonts (DM Serif Display, Manrope — already loaded), Playwright for the manual regression pass (already available in this environment from prior sessions in this workspace).

## Global Constraints

- No build tools, no bundler, no new npm packages, no new CDN dependencies.
- Single `index.html` file — do not split into separate `.js`/`.css` files.
- All UI copy stays in Portuguese (pt-BR), same direct/conversational tone ("eu"/"você") as the existing text.
- The 3 existing interactive tools (tracker, diagnostic, breathing) keep their exact `localStorage` key (`fgt_sono_tracker_v1`), exact data arrays (`HABITS`, `DAYS`, `SIGNS`, `PHASES`), and exact function names (`renderTracker`, `tkSave`, `tkCount`, `updateResult`, `runPhase`, `startBreath`, `stopBreath`) — anyone who already used the tracker or diagnostic keeps their saved data working after this change.
- No automated test framework — verification is manual, in-browser. Serve via `python3 -m http.server`.
- Where text is trimmed/rewritten for concision, it must read as fluid prose (complete sentences, natural voice) — not fragmented into bullet telegram-style unless the source was already a list.

---

## Task 1: Reorder sections and nav (structural move, zero content change)

**Files:**
- Modify: `index.html` (`<nav class="nav" id="nav">` block, currently lines 171-183; the 11 top-level `<section>` blocks in the body, currently lines 189-457)

**Interfaces:**
- Produces: the new section order that every later task assumes — `comece → diagnostico → tracker → dia → porque → pilares → ref → respiracao → filhos → tempo → resumo` (Tasks 2-6 each further transform one or more of these sections without changing this base ordering, except Task 4 which relocates `filhos`/`tempo` again as part of merging them).
- Consumes: nothing from other tasks.

This task only **moves** existing blocks — it does not add, remove, or edit a single word of content. That makes the result independently testable: the page must look and behave identically to before, just in a new scroll order.

- [ ] **Step 1: Replace the nav with the reordered version**

Find (currently lines 171-183):

```html
    <nav class="nav" id="nav">
      <a href="#comece">Comece aqui</a>
      <a href="#porque">Por que importa</a>
      <a href="#dia">Seu dia</a>
      <a href="#ref">Referência</a>
      <a href="#pilares">Os pilares</a>
      <a href="#filhos">Filhos pequenos</a>
      <a href="#tempo">Sem tempo</a>
      <a href="#tracker">Tracker</a>
      <a href="#diagnostico">Seu sono</a>
      <a href="#respiracao">Respiração</a>
      <a href="#resumo">Resumo</a>
    </nav>
```

Replace with:

```html
    <nav class="nav" id="nav">
      <a href="#comece">Comece aqui</a>
      <a href="#diagnostico">Seu sono</a>
      <a href="#tracker">Tracker</a>
      <a href="#dia">Seu dia</a>
      <a href="#porque">Por que importa</a>
      <a href="#pilares">Os pilares</a>
      <a href="#ref">Referência</a>
      <a href="#respiracao">Respiração</a>
      <a href="#filhos">Filhos pequenos</a>
      <a href="#tempo">Sem tempo</a>
      <a href="#resumo">Resumo</a>
    </nav>
```

(Only the `<a>` order changed — same 11 links, same `href`s, same text. This keeps the nav consistent with the new section order below without yet touching the sections Task 4/5 will later merge or remove.)

- [ ] **Step 2: Reorder the section blocks**

The body currently has these 11 top-level blocks in this order, each starting at its `<!-- COMMENT -->` marker and ending at its matching `</section>` closing tag:

1. `<!-- COMECE -->` / `<section id="comece">` (lines 189-203)
2. `<!-- POR QUE -->` / `<section id="porque">` (lines 205-225)
3. `<!-- SEU DIA -->` / `<section id="dia">` (lines 227-242)
4. `<!-- REFERÊNCIA -->` / `<section id="ref">` (lines 244-285)
5. `<!-- PILARES -->` / `<section id="pilares">` (lines 287-343)
6. `<!-- FILHOS -->` / `<section id="filhos">` (lines 345-371)
7. `<!-- SEM TEMPO -->` / `<section id="tempo">` (lines 373-390)
8. `<!-- TRACKER -->` / `<section id="tracker">` (lines 392-412)
9. `<!-- DIAGNÓSTICO -->` / `<section id="diagnostico">` (lines 414-424)
10. `<!-- RESPIRAÇÃO -->` / `<section id="respiracao">` (lines 426-439)
11. `<!-- RESUMO -->` / `<section id="resumo">` (lines 441-457)

Cut each block whole (its comment marker through its closing `</section>` tag, inclusive — do not alter anything inside any block) and reassemble them, in place of the original 11, in this new order:

1. `<!-- COMECE -->` (unchanged, stays first)
2. `<!-- DIAGNÓSTICO -->`
3. `<!-- TRACKER -->`
4. `<!-- SEU DIA -->`
5. `<!-- POR QUE -->`
6. `<!-- PILARES -->`
7. `<!-- REFERÊNCIA -->`
8. `<!-- RESPIRAÇÃO -->`
9. `<!-- FILHOS -->`
10. `<!-- SEM TEMPO -->`
11. `<!-- RESUMO -->` (unchanged, stays last)

After this move, the `<div class="wrap">...</div>` still contains exactly these 11 sections, just reordered — same total content, same closing `</div>` before `<footer>`.

- [ ] **Step 3: Manual verification**

Serve the app:

```bash
cd "Projetos GitHub/FG-Higiene-do-Sono" && python3 -m http.server 8020
```

Open `http://localhost:8020` (or drive with Playwright).

1. Scrolling the page top to bottom shows sections in the new order: Comece aqui → Seu sono (diagnóstico) → Tracker → Seu dia → Por que importa → Os pilares → Referência → Respiração → Filhos pequenos → Sem tempo → Resumo.
2. Every nav pill, in order, scrolls to and highlights the matching section (the existing `IntersectionObserver` scrollspy logic is untouched — confirm it still works with the new order).
3. Click the tracker cells, the diagnostic checklist items, and the breathing "Começar" button — all three tools behave exactly as before (no `id`, no JS function was touched in this task).
4. `localStorage` under `fgt_sono_tracker_v1` is read/written exactly as before — if you had prior data (or set some now, then reload), it persists.
5. DevTools console: zero errors.

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "refactor: reorder sections so interactive tools appear earlier"
```

---

## Task 2: Convert "Por que importa" into a 3-card accordion

**Files:**
- Modify: `index.html` (add CSS near the existing `.tri`/`.stat` rules, currently around lines 83-88; replace the `<section id="porque">` body, currently around lines 210-225 after Task 1's move — locate by the `id="porque"` and `<!-- POR QUE -->` marker, not by line number, since Task 1 shifted it; add accordion-toggle JS near the end of the `<script>` block, after the existing tracker code)

**Interfaces:**
- Produces: `.acc`, `.acc-item`, `.acc-head`, `.acc-body` CSS classes and a generic accordion click-toggle behavior (delegated per `.acc-head`, not scoped to one section) — reusable if a future section ever needs the same pattern, though no other task in this plan uses it (Tasks 3-4 use the separate tab pattern instead).
- Consumes: Task 1's reordered section (this task only edits `#porque`'s own content, not its position).

- [ ] **Step 1: Add the accordion CSS**

Insert this block into `<style>`, right after the existing `.tri .k{...}` / `.tri .v{...}` rules (the block that currently ends with `.tri .v{font-size:.88rem;color:#dcdcdc}`):

```css
  /* ===== ACCORDION ===== */
  .acc{margin:6px 0 4px}
  .acc-item{border:1px solid var(--line);border-radius:12px;margin-bottom:8px;overflow:hidden;background:var(--surface2)}
  .acc-head{padding:14px 16px;display:flex;justify-content:space-between;align-items:center;cursor:pointer;gap:10px}
  .acc-head b{font-size:.92rem;font-weight:700;color:var(--text)}
  .acc-head .plus{color:var(--pink);font-size:1.1rem;font-weight:800;flex:0 0 auto}
  .acc-body{padding:0 16px 16px}
  .acc-item:not(.open) .acc-body{display:none}
```

- [ ] **Step 2: Replace the section content**

Find the `<section id="porque">` block (its full current content, after Task 1's reorder — the text is unchanged from before Task 1, only its position in the file moved):

```html
  <section id="porque">
    <p class="eyebrow">Parte 1</p>
    <h2>Por que o sono decide seus resultados</h2>
    <p>Vou ser direto: se o seu sono está ruim, você treina com o freio de mão puxado. Não importa o quão perfeito estejam o treino e a dieta.</p>
    <h3>O triângulo que você não pode quebrar</h3>
    <p>Seus resultados dependem de três coisas trabalhando juntas. Tira uma, as outras duas param de funcionar direito:</p>
    <div class="tri">
      <div class="row"><span class="k">Treino</span><span class="v">gera o estímulo e as microlesões que pedem adaptação</span></div>
      <div class="row"><span class="k">Nutrição</span><span class="v">fornece o material pra reconstruir</span></div>
      <div class="row"><span class="k">Sono</span><span class="v">é o momento em que a reconstrução realmente acontece</span></div>
    </div>
    <p>A síntese proteica — que faz o músculo crescer — acontece principalmente no sono profundo, quando o hormônio do crescimento tem seu pico. Sem sono, o estímulo existe, mas a resposta não vem completa.</p>
    <h3>Os números que provam isso</h3>
    <div class="stat"><div class="n">18%</div><div class="d">de queda na síntese proteica muscular após uma única noite mal dormida.</div></div>
    <div class="stat"><div class="n">24%</div><div class="d">de redução na testosterona — hormônio-chave de composição corporal pra homens e mulheres.</div></div>
    <div class="stat"><div class="n">21%</div><div class="d">de aumento no cortisol, que acelera a perda de massa magra e o acúmulo de gordura abdominal.</div></div>
    <div class="stat"><div class="n">7-9h</div><div class="d">é a faixa recomendada pra adultos. Isso não é negociável biologicamente.</div></div>
    <h3>Dormir mal engorda. E não é falta de disciplina.</h3>
    <p>A privação de sono mexe com dois hormônios da fome: <span class="b">a leptina cai</span> (some o sinal de saciedade) e <span class="b">a grelina sobe</span> (a fome aumenta, principalmente por doce e coisa gordurosa). Somado ao cortisol alto, o corpo estoca mais gordura. Se o emagrecimento travou e o sono está ruim, o problema pode não ser sua força de vontade. É bioquímica — e dá pra resolver.</p>
  </section>
```

Replace with:

```html
  <section id="porque">
    <p class="eyebrow">Parte 1</p>
    <h2>Por que o sono decide seus resultados</h2>
    <p class="lead">Vou ser direto: se o seu sono está ruim, você treina com o freio de mão puxado — não importa o quão perfeito estejam o treino e a dieta. Toque em cada card pra entender o porquê:</p>
    <div class="acc" id="accPorque">
      <div class="acc-item open">
        <div class="acc-head" role="button" aria-expanded="true"><b>🔺 O triângulo que você não pode quebrar</b><span class="plus">−</span></div>
        <div class="acc-body">
          <p>Seus resultados dependem de três coisas trabalhando juntas. Tire uma, as outras duas param de funcionar direito:</p>
          <div class="tri">
            <div class="row"><span class="k">Treino</span><span class="v">gera o estímulo e as microlesões que pedem adaptação</span></div>
            <div class="row"><span class="k">Nutrição</span><span class="v">fornece o material pra reconstruir</span></div>
            <div class="row"><span class="k">Sono</span><span class="v">é o momento em que a reconstrução realmente acontece</span></div>
          </div>
          <p>A síntese proteica — que faz o músculo crescer — acontece principalmente no sono profundo, quando o hormônio do crescimento tem seu pico. Sem sono, o estímulo existe, mas a resposta não vem completa.</p>
        </div>
      </div>
      <div class="acc-item">
        <div class="acc-head" role="button" aria-expanded="false"><b>📊 Os números que provam isso</b><span class="plus">+</span></div>
        <div class="acc-body">
          <div class="stat"><div class="n">18%</div><div class="d">de queda na síntese proteica muscular após uma única noite mal dormida.</div></div>
          <div class="stat"><div class="n">24%</div><div class="d">de redução na testosterona — hormônio-chave de composição corporal pra homens e mulheres.</div></div>
          <div class="stat"><div class="n">21%</div><div class="d">de aumento no cortisol, que acelera a perda de massa magra e o acúmulo de gordura abdominal.</div></div>
          <div class="stat"><div class="n">7-9h</div><div class="d">é a faixa recomendada pra adultos. Isso não é negociável biologicamente.</div></div>
        </div>
      </div>
      <div class="acc-item">
        <div class="acc-head" role="button" aria-expanded="false"><b>🍩 Por que dormir mal engorda</b><span class="plus">+</span></div>
        <div class="acc-body">
          <p>Não é falta de disciplina. A privação de sono mexe com dois hormônios da fome: <span class="b">a leptina cai</span> (some o sinal de saciedade) e <span class="b">a grelina sobe</span> (a fome aumenta, principalmente por doce e coisa gordurosa). Somado ao cortisol alto, o corpo estoca mais gordura. Se o emagrecimento travou e o sono está ruim, o problema pode não ser sua força de vontade — é bioquímica, e dá pra resolver.</p>
        </div>
      </div>
    </div>
  </section>
```

(The first card starts `open` so the page still gives an immediate answer without requiring a click — only the 2nd and 3rd cards are collapsed by default. Content is unchanged except the connecting sentences, which were tightened; every fact, number, and claim from the original is preserved.)

- [ ] **Step 3: Add the accordion-toggle JS**

Insert this block into the `<script>` element, immediately after the line `renderTracker();` (the last line of the `/* ===== TRACKER (localStorage) ===== */` block) and before the `/* ===== AUTODIAGNÓSTICO ===== */` comment:

```js
/* ===== ACCORDION TOGGLE ===== */
document.querySelectorAll('.acc-item .acc-head').forEach(head=>{
  head.addEventListener('click',()=>{
    const item = head.closest('.acc-item');
    const open = item.classList.toggle('open');
    head.setAttribute('aria-expanded', open ? 'true' : 'false');
    head.querySelector('.plus').textContent = open ? '−' : '+';
  });
});
```

- [ ] **Step 4: Manual verification**

Reload the app.

1. "Por que importa" section shows 3 cards; the first ("O triângulo...") is open, showing the triangle rows and the two paragraphs; the other two are collapsed, showing only their title and a `+`.
2. Click the 2nd card's header — it expands (shows the 4 stat boxes), its `+` becomes `−`; click again — it collapses back, `−` becomes `+`.
3. Click the 3rd card — same open/close behavior, independent of the other two (this is not an exclusive accordion — multiple cards can be open at once, matching the "explore in any order" nature of the content).
4. All original numbers (18%, 24%, 21%, 7-9h) and every sentence's information is present somewhere in the 3 cards.
5. DevTools console: zero errors.

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "feat: convert 'Por que importa' section into a 3-card accordion"
```

---

## Task 3: Convert "Os pilares" into a 6-tab panel

**Files:**
- Modify: `index.html` (add tab-panel CSS near the accordion CSS added in Task 2; replace the `<section id="pilares">` body; add tab-toggle JS near the accordion JS added in Task 2)

**Interfaces:**
- Produces: `.tabpanel-nav`, `.tabpanel` CSS classes and a generic tab-switch behavior scoped per `.tabpanel-nav` (each nav only controls the `.tabpanel` elements that are its own siblings under the same parent) — Task 4 reuses this exact same CSS/JS for the "Seu contexto" section's 2 tabs, so this task's JS must be written generically (not hardcoded to `#pilares`), and Task 4 does not re-add the JS block.
- Consumes: Task 1's reordered section.

- [ ] **Step 1: Add the tab-panel CSS**

Insert this block into `<style>`, right after the accordion CSS block added in Task 2 (after the line `.acc-item:not(.open) .acc-body{display:none}`):

```css
  /* ===== TABPANEL ===== */
  .tabpanel-nav{display:flex;gap:6px;margin-bottom:14px;flex-wrap:wrap}
  .tabpanel-nav button{padding:7px 14px;border-radius:999px;border:1px solid var(--line);background:var(--surface2);color:var(--muted);font-family:inherit;font-size:.8rem;font-weight:600;cursor:pointer;transition:all .16s}
  .tabpanel-nav button:hover{color:var(--text)}
  .tabpanel-nav button.active{background:var(--pink);border-color:var(--pink);color:#fff}
  .tabpanel{display:none}
  .tabpanel.active{display:block}
```

- [ ] **Step 2: Replace the section content**

Find the `<section id="pilares">` block (full current content, unchanged text from before Task 1's reorder):

```html
  <section id="pilares">
    <p class="eyebrow">Parte 4</p>
    <h2>Os pilares, explicados</h2>

    <h3>1. Consistência de horário — o pilar nº 1</h3>
    <p>O ajuste de maior impacto isolado. O corpo ama previsibilidade. Quando os horários variam muito, o relógio biológico fica confuso e o sono perde qualidade mesmo que a quantidade seja a mesma. <span class="b">Fixe o horário de acordar</span> e o de dormir se ajusta sozinho.</p>

    <h3>2. Gestão da luz — seu maior aliado (e vilão)</h3>
    <ul class="clean">
      <li><span class="b">De manhã:</span> luz natural nos olhos ancora o ciclo e melhora foco e humor no dia.</li>
      <li><span class="b">À noite:</span> menos luz artificial e menos luz azul de telas. A luz azul imita o meio-dia pro cérebro e trava a melatonina.</li>
    </ul>

    <h3>Como usar o modo de luz quente do celular</h3>
    <p>Largar o celular 2h antes soa lindo na teoria e é difícil na prática. Então vamos pelo que dá pra fazer já: ativar o modo de luz quente. Todo celular atual tem — ele deixa a tela amarelada e reduz bastante a luz azul que atrapalha o sono.</p>
    <p class="b" style="margin-bottom:6px">Como ativar (em qualquer aparelho):</p>
    <p style="margin-bottom:8px">Vá em <span class="b">Configurações → Tela (ou Display) → Conforto visual / Bem-estar digital</span> e procure por um destes nomes:</p>
    <ul class="clean">
      <li>"Night Shift", "Modo Noite", "Filtro de Luz Azul", "Luz Noturna", "Conforto Visual" ou "Temperatura de Cor".</li>
    </ul>
    <div class="box green">
      <div class="lab">Configuração ideal — faça uma vez e esquece</div>
      <ul>
        <li>Ative com agendamento automático: do pôr do sol ao nascer do sol</li>
        <li>Deixe a intensidade no máximo — quanto mais amarelada, melhor</li>
        <li>Combine com o modo escuro do celular</li>
        <li>Abaixe também o brilho geral à noite</li>
      </ul>
    </div>
    <p>Honestidade: o filtro reduz a luz azul, mas não elimina o estímulo mental das redes nem as notificações. Ele é uma camada de proteção, não um passe livre. O ideal continua sendo tela desligada 30 min antes de dormir.</p>
    <div class="box blue">
      <div class="lab">Outros recursos que você já tem no celular</div>
      <ul>
        <li>"Não perturbe" agendado: silencia notificações no horário de sono</li>
        <li>Limite de uso por app: bloqueia redes depois de um horário</li>
        <li>Tela em escala de cinza (em Acessibilidade): tira as cores e reduz a vontade de rolar o feed</li>
      </ul>
    </div>

    <h3>3. Ambiente de sono</h3>
    <ul class="clean">
      <li><span class="b">Temperatura:</span> 18 a 22°C. O corpo esfria pra adormecer; quarto quente atrapalha.</li>
      <li><span class="b">Escuridão:</span> cortina blackout ou máscara de dormir.</li>
      <li><span class="b">Silêncio:</span> ruído branco ou ventilador ajudam a mascarar sons.</li>
      <li><span class="b">Associação:</span> cama é pra dormir (e sexo). Trabalhar e ver série na cama confunde o cérebro.</li>
    </ul>

    <h3>4. Cafeína</h3>
    <p>Meia-vida de 6 a 8 horas. Um café às 15h ainda age às 21h. Evite pelo menos 6h antes de dormir — inclui café, chá preto, mate, pré-treino e refrigerante.</p>

    <h3>5. Exercício</h3>
    <p>Treinar melhora muito o sono. Só cuide do horário: treino intenso 1 a 2h antes de dormir eleva temperatura e cortisol. Prefira manhã ou início da tarde quando possível.</p>

    <h3>6. Ritual de desaceleração</h3>
    <p>O sistema nervoso não tem botão de desligar. Crie uma sequência curta e sempre igual: banho morno, leitura (não tela), respiração ou alongamento leve, anotar o que precisa fazer amanhã. A respiração <span class="b">4-7-8</span> logo ali embaixo é um bom começo.</p>
  </section>
```

Replace with:

```html
  <section id="pilares">
    <p class="eyebrow">Parte 4</p>
    <h2>Os pilares, explicados</h2>
    <p class="lead">Seis frentes, cada uma independente das outras — não precisa ler em ordem. Toque na que quiser entender melhor:</p>
    <div class="tabpanel-nav" id="pilaresNav">
      <button class="active" data-panel="p-horario">Horário</button>
      <button data-panel="p-luz">Luz</button>
      <button data-panel="p-ambiente">Ambiente</button>
      <button data-panel="p-cafeina">Cafeína</button>
      <button data-panel="p-exercicio">Exercício</button>
      <button data-panel="p-ritual">Ritual</button>
    </div>

    <div class="tabpanel active" id="p-horario">
      <h3>Consistência de horário — o pilar nº 1</h3>
      <p>O ajuste de maior impacto isolado. O corpo ama previsibilidade: quando os horários variam muito, o relógio biológico fica confuso e o sono perde qualidade mesmo que a quantidade seja a mesma. <span class="b">Fixe o horário de acordar</span> e o de dormir se ajusta sozinho.</p>
    </div>

    <div class="tabpanel" id="p-luz">
      <h3>Gestão da luz — seu maior aliado (e vilão)</h3>
      <ul class="clean">
        <li><span class="b">De manhã:</span> luz natural nos olhos ancora o ciclo e melhora foco e humor no dia.</li>
        <li><span class="b">À noite:</span> menos luz artificial e menos luz azul de telas — ela imita o meio-dia pro cérebro e trava a melatonina.</li>
      </ul>
      <p class="b" style="margin-bottom:6px">Como ativar o modo de luz quente do celular:</p>
      <ul class="clean">
        <li>Vá em <span class="b">Configurações → Tela → Conforto visual</span> e procure "Night Shift", "Modo Noite" ou "Filtro de Luz Azul"</li>
        <li>Ative com agendamento automático, do pôr do sol ao nascer do sol, na intensidade máxima</li>
        <li>Combine com o modo escuro do celular e abaixe o brilho geral à noite</li>
      </ul>
      <div class="box green">
        <div class="lab">Configuração ideal — faça uma vez e esquece</div>
        <ul>
          <li>Agendamento automático do pôr ao nascer do sol</li>
          <li>Intensidade no máximo — quanto mais amarelada, melhor</li>
          <li>Modo escuro + brilho geral mais baixo à noite</li>
        </ul>
      </div>
      <p>Honestidade: o filtro reduz a luz azul, mas não elimina o estímulo mental das redes nem as notificações. É uma camada de proteção, não um passe livre — o ideal continua sendo tela desligada 30 min antes de dormir.</p>
      <div class="box blue">
        <div class="lab">Outros recursos que você já tem no celular</div>
        <ul>
          <li>"Não perturbe" agendado: silencia notificações no horário de sono</li>
          <li>Limite de uso por app: bloqueia redes depois de um horário</li>
          <li>Tela em escala de cinza (em Acessibilidade): reduz a vontade de rolar o feed</li>
        </ul>
      </div>
    </div>

    <div class="tabpanel" id="p-ambiente">
      <h3>Ambiente de sono</h3>
      <ul class="clean">
        <li><span class="b">Temperatura:</span> 18 a 22°C. O corpo esfria pra adormecer; quarto quente atrapalha.</li>
        <li><span class="b">Escuridão:</span> cortina blackout ou máscara de dormir.</li>
        <li><span class="b">Silêncio:</span> ruído branco ou ventilador ajudam a mascarar sons.</li>
        <li><span class="b">Associação:</span> cama é pra dormir (e sexo). Trabalhar e ver série na cama confunde o cérebro.</li>
      </ul>
    </div>

    <div class="tabpanel" id="p-cafeina">
      <h3>Cafeína</h3>
      <p>Meia-vida de 6 a 8 horas — um café às 15h ainda age às 21h. Evite pelo menos 6h antes de dormir, incluindo café, chá preto, mate, pré-treino e refrigerante.</p>
    </div>

    <div class="tabpanel" id="p-exercicio">
      <h3>Exercício</h3>
      <p>Treinar melhora muito o sono. Só cuide do horário: treino intenso 1 a 2h antes de dormir eleva temperatura e cortisol. Prefira manhã ou início da tarde quando possível.</p>
    </div>

    <div class="tabpanel" id="p-ritual">
      <h3>Ritual de desaceleração</h3>
      <p>O sistema nervoso não tem botão de desligar. Crie uma sequência curta e sempre igual: banho morno, leitura (não tela), respiração ou alongamento leve, anotar o que precisa fazer amanhã. A respiração <span class="b">4-7-8</span> é um bom começo.</p>
    </div>
  </section>
```

(The "como ativar o modo noturno" walkthrough is compressed from 2 paragraphs + a 1-item list into a 3-item list — same 3 pieces of information: where to find the setting, what to name-search for, and to enable auto-schedule — nothing is lost, it's just no longer prose.)

- [ ] **Step 3: Add the generic tab-toggle JS**

Insert this block into the `<script>` element, immediately after the accordion-toggle block added in Task 2 (after the closing `});` of that block) and before the `/* ===== AUTODIAGNÓSTICO ===== */` comment:

```js
/* ===== TABPANEL TOGGLE ===== */
document.querySelectorAll('.tabpanel-nav').forEach(navEl=>{
  navEl.querySelectorAll('button').forEach(btn=>{
    btn.addEventListener('click',()=>{
      navEl.querySelectorAll('button').forEach(b=>b.classList.toggle('active', b===btn));
      const targetId = btn.dataset.panel;
      const panels = navEl.parentElement.querySelectorAll(':scope > .tabpanel');
      panels.forEach(p=>p.classList.toggle('active', p.id===targetId));
    });
  });
});
```

(This is generic — it works for any `.tabpanel-nav` whose buttons have `data-panel` matching the `id` of a sibling `.tabpanel` under the same parent. Task 4 relies on this exact code already being present and does not add it again.)

- [ ] **Step 4: Manual verification**

Reload the app.

1. "Os pilares" shows 6 pill buttons (Horário, Luz, Ambiente, Cafeína, Exercício, Ritual); "Horário" is active by default and its panel (the consistency paragraph) is visible; the other 5 panels are hidden.
2. Click "Luz" — its panel appears (light management content, the compressed 3-step phone walkthrough, the green and blue boxes), "Horário"'s panel disappears, and the "Luz" pill turns pink/active.
3. Click through all 6 tabs — each shows only its own content, exactly one panel visible at a time.
4. Every fact from the original section (temperature range, caffeine half-life, exercise timing, the night-mode setting names) is present somewhere in the 6 panels.
5. DevTools console: zero errors.

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "feat: convert 'Os pilares' section into a 6-tab panel"
```

---

## Task 4: Merge "Filhos pequenos" + "Sem tempo" into a tabbed "Seu contexto" section

**Files:**
- Modify: `index.html` (nav, currently the version Task 1 produced; delete the `<section id="filhos">` and `<section id="tempo">` blocks and replace both with one new `<section id="contexto">`)

**Interfaces:**
- Consumes: the `.tabpanel-nav`/`.tabpanel` CSS and the generic tab-toggle JS, both added in Task 3 — this task adds no new CSS or JS, only HTML.
- Produces: `#contexto` section with 2 tabs (`c-filhos`, `c-tempo}`) that later tasks don't reference.

- [ ] **Step 1: Update the nav**

Find (the nav Task 1 produced):

```html
    <nav class="nav" id="nav">
      <a href="#comece">Comece aqui</a>
      <a href="#diagnostico">Seu sono</a>
      <a href="#tracker">Tracker</a>
      <a href="#dia">Seu dia</a>
      <a href="#porque">Por que importa</a>
      <a href="#pilares">Os pilares</a>
      <a href="#ref">Referência</a>
      <a href="#respiracao">Respiração</a>
      <a href="#filhos">Filhos pequenos</a>
      <a href="#tempo">Sem tempo</a>
      <a href="#resumo">Resumo</a>
    </nav>
```

Replace with:

```html
    <nav class="nav" id="nav">
      <a href="#comece">Comece aqui</a>
      <a href="#diagnostico">Seu sono</a>
      <a href="#tracker">Tracker</a>
      <a href="#dia">Seu dia</a>
      <a href="#porque">Por que importa</a>
      <a href="#pilares">Os pilares</a>
      <a href="#ref">Referência</a>
      <a href="#respiracao">Respiração</a>
      <a href="#contexto">Seu contexto</a>
      <a href="#resumo">Resumo</a>
    </nav>
```

- [ ] **Step 2: Replace the two sections with the merged, tabbed section**

Find the `<section id="filhos">` block in full (unchanged text from before Task 1's reorder):

```html
  <section id="filhos">
    <p class="eyebrow">Parte 5</p>
    <h2>Para quem tem filhos pequenos</h2>
    <p>Se você tem bebê ou criança pequena em casa, eu sei que não adianta falar em "horário fixo" pra quem acorda 3 vezes por noite. A realidade é dura e não vou fingir o contrário. Mas isso não significa que você não pode fazer nada — significa que a estratégia muda.</p>
    <p>Uma meta-análise achou perda média de 43 min de sono por noite nas primeiras 16 semanas após o nascimento. O impacto maior é na qualidade: o sono fragmentado corta o sono profundo, onde acontece a recuperação física e hormonal. Não é frescura nem preguiça.</p>
    <div class="box pink">
      <div class="lab">Foque no que está sob o seu controle</div>
      <ul>
        <li>Mantenha o horário de acordar o mais fixo possível — ancora o ciclo mesmo com noites picadas</li>
        <li>Pegue luz natural de manhã, nem que seja perto da janela</li>
        <li>Capriche na qualidade dos períodos que consegue dormir: escuro, fresco, silencioso</li>
        <li>Peça ajuda e divida as noites. Isso é estratégia, não fraqueza.</li>
      </ul>
    </div>
    <h3>O cochilo certo (quando dá)</h3>
    <p>Cochilo de 20 a 30 min no início da tarde recupera parte da função cognitiva. Só não pode ser depois das 15h. Uma boa técnica é o "nap café": tome um café, deite 20 min e acorde com a cafeína no pico do efeito.</p>
    <h3>Para adormecer rápido quando a janela aparece</h3>
    <p>Às vezes o sono aparece e o corpo está agitado demais. O relaxamento muscular progressivo ajuda: vá contraindo e soltando os músculos progressivamente, dos pés à cabeça. Reduz a tensão e aumenta o sono profundo — funciona em 20 minutos, sem silêncio absoluto.</p>
    <div class="box blue">
      <div class="lab">Uma coisa importante</div>
      <ul>
        <li>Privação de sono crônica pode intensificar ansiedade e alterações de humor nesse período</li>
        <li>Se o impacto no dia a dia estiver grande, procurar apoio profissional é parte do cuidado — não exceção</li>
      </ul>
    </div>
  </section>
```

And immediately following it, find the `<section id="tempo">` block in full:

```html
  <section id="tempo">
    <p class="eyebrow">Parte 6</p>
    <h2>Para quem vive no 'não tenho tempo'</h2>
    <p>Se você acha que dorme 5 ou 6 horas e funciona bem, deixa eu te dizer com carinho: provavelmente você se acostumou com a privação e perdeu a referência de como é funcionar de verdade. A privação crônica engana a própria percepção — reflexos, memória e humor rodam abaixo do normal sem você notar.</p>
    <p><span class="b">Dormir menos não te dá mais tempo, te dá menos resultado.</span> A produtividade por hora cai: 8 horas de trabalho com sono ruim rendem menos que 6 horas bem dormidas.</p>
    <div class="box pink">
      <div class="lab">5 ajustes de alto retorno</div>
      <ol>
        <li><span class="b">Fixe o horário de acordar.</span> É a âncora de tudo. Mesmo horário, não importa a que horas dormiu.</li>
        <li><span class="b">Celular fora da cama.</span> Comece com 30 min antes. Deixe carregando fora do quarto.</li>
        <li><span class="b">Trate o sono como compromisso agendado.</span> Quando der a hora, leve a sério como uma reunião.</li>
        <li><span class="b">Ajuste o ambiente uma vez.</span> Escuro, fresco, silencioso — rende toda noite.</li>
        <li><span class="b">Cafeína só até as 14h.</span> Corte simples, impacto direto.</li>
      </ol>
    </div>
    <p>Tratar o sono como parte do seu protocolo de performance muda tudo. Você não performa apesar do descanso — você performa por causa dele.</p>
  </section>
```

Replace both blocks (together, back to back) with this single merged section:

```html
  <!-- SEU CONTEXTO -->
  <section id="contexto">
    <p class="eyebrow">Parte 5</p>
    <h2>Seu contexto</h2>
    <p class="lead">Duas situações comuns que mudam a estratégia. Toque na que for a sua:</p>
    <div class="tabpanel-nav" id="contextoNav">
      <button class="active" data-panel="c-filhos">Tenho filhos pequenos</button>
      <button data-panel="c-tempo">Não tenho tempo</button>
    </div>

    <div class="tabpanel active" id="c-filhos">
      <p>Se você tem bebê ou criança pequena em casa, eu sei que não adianta falar em "horário fixo" pra quem acorda 3 vezes por noite. A realidade é dura e não vou fingir o contrário. Mas isso não significa que você não pode fazer nada — significa que a estratégia muda.</p>
      <p>Uma meta-análise achou perda média de 43 min de sono por noite nas primeiras 16 semanas após o nascimento. O impacto maior é na qualidade: o sono fragmentado corta o sono profundo, onde acontece a recuperação física e hormonal. Não é frescura nem preguiça.</p>
      <div class="box pink">
        <div class="lab">Foque no que está sob o seu controle</div>
        <ul>
          <li>Mantenha o horário de acordar o mais fixo possível — ancora o ciclo mesmo com noites picadas</li>
          <li>Pegue luz natural de manhã, nem que seja perto da janela</li>
          <li>Capriche na qualidade dos períodos que consegue dormir: escuro, fresco, silencioso</li>
          <li>Peça ajuda e divida as noites. Isso é estratégia, não fraqueza.</li>
        </ul>
      </div>
      <h3>O cochilo certo (quando dá)</h3>
      <p>Cochilo de 20 a 30 min no início da tarde recupera parte da função cognitiva. Só não pode ser depois das 15h. Uma boa técnica é o "nap café": tome um café, deite 20 min e acorde com a cafeína no pico do efeito.</p>
      <h3>Para adormecer rápido quando a janela aparece</h3>
      <p>Às vezes o sono aparece e o corpo está agitado demais. O relaxamento muscular progressivo ajuda: vá contraindo e soltando os músculos progressivamente, dos pés à cabeça — funciona em 20 minutos, sem silêncio absoluto.</p>
      <div class="box blue">
        <div class="lab">Uma coisa importante</div>
        <ul>
          <li>Privação de sono crônica pode intensificar ansiedade e alterações de humor nesse período</li>
          <li>Se o impacto no dia a dia estiver grande, procurar apoio profissional é parte do cuidado — não exceção</li>
        </ul>
      </div>
    </div>

    <div class="tabpanel" id="c-tempo">
      <p>Se você acha que dorme 5 ou 6 horas e funciona bem, deixa eu te dizer com carinho: provavelmente você se acostumou com a privação e perdeu a referência de como é funcionar de verdade. A privação crônica engana a própria percepção — reflexos, memória e humor rodam abaixo do normal sem você notar.</p>
      <p><span class="b">Dormir menos não te dá mais tempo, te dá menos resultado.</span> A produtividade por hora cai: 8 horas de trabalho com sono ruim rendem menos que 6 horas bem dormidas.</p>
      <div class="box pink">
        <div class="lab">Além dos 3 ajustes que você já viu na abertura</div>
        <p style="font-size:.92rem;color:#e6e6e6;margin-bottom:10px">Pra quem tem pouquíssimo tempo, adicione mais dois:</p>
        <ol>
          <li><span class="b">Trate o sono como compromisso agendado.</span> Quando der a hora, leve a sério como uma reunião.</li>
          <li><span class="b">Ajuste o ambiente uma vez.</span> Escuro, fresco, silencioso — rende toda noite.</li>
        </ol>
      </div>
      <p>Tratar o sono como parte do seu protocolo de performance muda tudo. Você não performa apesar do descanso — você performa por causa dele.</p>
    </div>
  </section>
```

(The "Sem tempo" tab's tip box now has only the 2 items that don't already appear in the "Comece aqui" 3-item box — "horário fixo" and "cafeína até 14h" are dropped here since the reader already saw them at the top of the page; this is the habits-list consolidation the design calls for.)

- [ ] **Step 3: Manual verification**

Reload the app.

1. Nav shows "Seu contexto" where "Filhos pequenos"/"Sem tempo" used to be two separate entries; clicking it scrolls to the merged section.
2. The merged section shows 2 pill tabs; "Tenho filhos pequenos" is active by default, showing that content; "Não tenho tempo" panel is hidden.
3. Click "Não tenho tempo" — its content appears (including the box now showing only 2 items, referencing "os 3 ajustes que você já viu na abertura"), the other panel hides, tab switches correctly (this reuses Task 3's generic JS — confirms it works for a second, independent `.tabpanel-nav` on the same page without interfering with the "Os pilares" tabs).
4. Scroll up to "Os pilares" and confirm its tabs still work independently (switching a Pilares tab does not affect the Contexto tabs, and vice versa).
5. DevTools console: zero errors.

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: merge 'Filhos pequenos' and 'Sem tempo' into tabbed 'Seu contexto' section"
```

---

## Task 5: Remove the "Resumo" section and migrate its closing line to the footer

**Files:**
- Modify: `index.html` (nav; delete the `<section id="resumo">` block; the `<footer>` block)

**Interfaces:** none — this is the last content-structure change; Task 6 only touches the diagnostic section's JS/HTML, unrelated to this task's files.

- [ ] **Step 1: Update the nav**

Find (the nav Task 4 produced):

```html
    <nav class="nav" id="nav">
      <a href="#comece">Comece aqui</a>
      <a href="#diagnostico">Seu sono</a>
      <a href="#tracker">Tracker</a>
      <a href="#dia">Seu dia</a>
      <a href="#porque">Por que importa</a>
      <a href="#pilares">Os pilares</a>
      <a href="#ref">Referência</a>
      <a href="#respiracao">Respiração</a>
      <a href="#contexto">Seu contexto</a>
      <a href="#resumo">Resumo</a>
    </nav>
```

Replace with (drop the last link — this is now the plan's final 9-item nav):

```html
    <nav class="nav" id="nav">
      <a href="#comece">Comece aqui</a>
      <a href="#diagnostico">Seu sono</a>
      <a href="#tracker">Tracker</a>
      <a href="#dia">Seu dia</a>
      <a href="#porque">Por que importa</a>
      <a href="#pilares">Os pilares</a>
      <a href="#ref">Referência</a>
      <a href="#respiracao">Respiração</a>
      <a href="#contexto">Seu contexto</a>
    </nav>
```

- [ ] **Step 2: Delete the Resumo section**

Find and delete this entire block (unchanged since before Task 1):

```html
  <!-- RESUMO -->
  <section id="resumo">
    <p class="eyebrow">Parte 9</p>
    <h2>O resumo de tudo</h2>
    <div class="box pink">
      <div class="lab">O essencial em 6 frases</div>
      <ul>
        <li>Sono não é descanso passivo — é onde os resultados são construídos.</li>
        <li>O que mais funciona é comportamento: horário fixo, luz certa, ambiente adequado.</li>
        <li>Luz natural de manhã e escuro à noite regulam quase todo o resto.</li>
        <li>Cafeína até as 14h, telas fora da cama, quarto fresco e escuro.</li>
        <li>Nenhum suplemento, treino ou dieta substitui uma boa noite de sono.</li>
        <li>Comece com 2 ou 3 hábitos e mantenha por 14 dias. Consistência bate perfeição.</li>
      </ul>
    </div>
    <p>Sono bem feito é a alavanca mais barata e poderosa que você tem. Escolhe seus primeiros hábitos, coloca no tracker e me conta como foi a sua semana.</p>
  </section>
```

(The "6 frases" box is dropped entirely — every one of those 6 points is already covered in "Comece aqui", "Os pilares", or "Referência". Only the closing sentence is kept, migrated to the footer in Step 3.)

- [ ] **Step 3: Migrate the closing line to the footer**

Find:

```html
<footer>
  <p class="dis">Material educativo. Não substitui avaliação médica. Se você convive com insônia persistente ou sofrimento emocional, procure um profissional de saúde.</p>
  <p>FGT Assessoria Fitness · <a href="https://instagram.com/fernandogomess__" target="_blank" rel="noopener">@fernandogomess__</a></p>
</footer>
```

Replace with:

```html
<footer>
  <p class="dis" style="color:var(--text);font-weight:600">Sono bem feito é a alavanca mais barata e poderosa que você tem. Escolhe seus primeiros hábitos, coloca no tracker e me conta como foi a sua semana.</p>
  <p class="dis">Material educativo. Não substitui avaliação médica. Se você convive com insônia persistente ou sofrimento emocional, procure um profissional de saúde.</p>
  <p>FGT Assessoria Fitness · <a href="https://instagram.com/fernandogomess__" target="_blank" rel="noopener">@fernandogomess__</a></p>
</footer>
```

- [ ] **Step 4: Manual verification**

Reload the app.

1. Nav has exactly 9 items, ending with "Seu contexto" — no "Resumo" entry.
2. Scrolling past "Seu contexto" goes straight to the footer — no "O resumo de tudo" section in between.
3. Footer now shows the closing sentence ("Sono bem feito é a alavanca...") in slightly bolder/brighter text above the medical disclaimer, then the disclaimer, then the Instagram link — all 3 lines present.
4. DevTools console: zero errors.

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "refactor: remove redundant Resumo section, migrate closing line to footer"
```

---

## Task 6: Add a dynamic CTA to the diagnostic result

**Files:**
- Modify: `index.html` (the `<div class="result" id="result">` block inside `<section id="diagnostico">`; the `updateResult()` function inside `<script>`)

**Interfaces:**
- Consumes: `.btn-pink` CSS class (already exists, from the breathing-exercise button) and the `checked` array / `SIGNS` constant (already exist, untouched) — this task only extends `updateResult()`, it does not change how `checked` is populated or how the checklist items are rendered/clicked.

- [ ] **Step 1: Add the CTA container to the result markup**

Find (inside `<section id="diagnostico">`):

```html
    <div class="result" id="result">
      <div class="score" id="resScore">0 de 8 marcados</div>
      <div class="msg" id="resMsg">Marque os sinais acima pra ver o resultado.</div>
    </div>
```

Replace with:

```html
    <div class="result" id="result">
      <div class="score" id="resScore">0 de 8 marcados</div>
      <div class="msg" id="resMsg">Marque os sinais acima pra ver o resultado.</div>
      <div id="resCta"></div>
    </div>
```

- [ ] **Step 2: Update `updateResult()` to render a CTA per score band**

Find:

```js
function updateResult(){
  const n = checked.filter(Boolean).length;
  const score = document.getElementById('resScore');
  const msg = document.getElementById('resMsg');
  score.textContent = n+' de 8 marcados';
  if(n===0){ msg.textContent = 'Marque os sinais acima pra ver o resultado.'; }
  else if(n<=1){ msg.textContent = 'Seu sono provavelmente está te ajudando. Mantém o que você já faz.'; }
  else if(n<=3){ msg.textContent = 'Sinal de alerta. Comece pelos 3 ajustes do topo do guia e acompanhe pelo tracker.'; }
  else { msg.innerHTML = 'O sono é a sua prioridade agora — antes de qualquer ajuste de treino ou dieta. Me chama pra gente ver isso junto.'; }
}
```

Replace with:

```js
function updateResult(){
  const n = checked.filter(Boolean).length;
  const score = document.getElementById('resScore');
  const msg = document.getElementById('resMsg');
  const cta = document.getElementById('resCta');
  score.textContent = n+' de 8 marcados';
  let ctaHtml = '';
  if(n===0){
    msg.textContent = 'Marque os sinais acima pra ver o resultado.';
  } else if(n<=1){
    msg.textContent = 'Seu sono provavelmente está te ajudando. Mantém o que você já faz.';
    ctaHtml = '<a href="#tracker" class="btn-pink" style="display:inline-block;margin-top:12px">Ver seu tracker</a>';
  } else if(n<=3){
    msg.textContent = 'Sinal de alerta. Comece pelos 3 ajustes do topo do guia e acompanhe pelo tracker.';
    ctaHtml = '<a href="#comece" class="btn-pink" style="display:inline-block;margin-top:12px">Ver os 3 ajustes</a>';
  } else {
    msg.innerHTML = 'O sono é a sua prioridade agora — antes de qualquer ajuste de treino ou dieta. Me chama pra gente ver isso junto.';
    ctaHtml = '<a href="#pilares" class="btn-pink" style="display:inline-block;margin-top:12px">Ver todos os pilares</a>';
  }
  cta.innerHTML = ctaHtml;
}
```

(`checked`, `SIGNS`, the checklist rendering, and the click handler that calls `updateResult()` are all untouched — this only changes what `updateResult()` renders after the existing scoring logic runs.)

- [ ] **Step 3: Manual verification**

Reload the app (clear `localStorage` first for a clean run, or just click checklist items directly — the diagnostic checklist state is in-memory only, not persisted, so no prior-data concern here).

1. With 0 signs marked: no CTA button appears (only the "marque os sinais" message).
2. Mark exactly 1 sign: message changes, a "Ver seu tracker" pink button appears; click it — page scrolls smoothly to the Tracker section.
3. Uncheck, mark 2-3 signs: message changes to the alert one, button now says "Ver os 3 ajustes" and links to `#comece`; click it — scrolls to the opening section.
4. Mark 4 or more signs: message changes to the "prioridade agora" one, button says "Ver todos os pilares" and links to `#pilares`; click it — scrolls to Os Pilares (and its tab panel from Task 3 still works normally once there).
5. DevTools console: zero errors.

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: diagnostic result recommends a next section based on score"
```

---

## Task 7: Full regression pass

**Files:** none (verification only)

- [ ] **Step 1: Fresh-eyes manual regression checklist**

Serve the app (`python3 -m http.server 8020`) and, in a private/incognito window (clean `localStorage`), drive it with Playwright (reuse the cached package/browser from earlier sessions in this environment):

1. Confirm the full nav (9 items) scrolls to and highlights each of the 9 sections correctly as you scroll past them (scrollspy).
2. In "Por que importa": open and close all 3 accordion cards independently; confirm content matches the original (18%/24%/21%/7-9h stats, leptina/grelina paragraph).
3. In "Os pilares": click through all 6 tabs, confirm each shows distinct content and the compressed night-mode steps are present.
4. In "Seu contexto": click both tabs, confirm the "Sem tempo" tip box shows only 2 items and references the opening's 3 ajustes.
5. Diagnostic: mark signs to hit all 4 score bands (0, 1, 2-3, 4+) and confirm the message and CTA button/link match Task 6's mapping each time; click each CTA and confirm it scrolls to the right section.
6. Tracker: click several day/habit cells, reload the page, confirm state persisted via `localStorage` key `fgt_sono_tracker_v1` (same key as before this plan — no migration needed).
7. Breathing exercise: click "Começar", confirm the 4-7-8 countdown animation still runs, click "Parar", confirm it resets.
8. Footer: confirm the migrated closing line, the medical disclaimer, and the Instagram link are all present, in that order.
9. Resize to 375×800 (mobile): confirm the nav pills scroll horizontally without breaking layout, tab buttons in Pilares/Contexto wrap or scroll sensibly, no horizontal page overflow (`document.documentElement.scrollWidth` equals `clientWidth`).
10. Console errors: zero, across every interaction above.

- [ ] **Step 2: Update the design spec status**

Add a one-line status note at the top of `docs/superpowers/specs/2026-08-07-reorganizacao-interativa-design.md`, immediately after its title heading:

```markdown
> Status: implementado em 2026-08-07.
```

(Adjust the date if this step runs on a different day.)

- [ ] **Step 3: Final commit**

```bash
git add docs/superpowers/specs/2026-08-07-reorganizacao-interativa-design.md
git commit -m "chore: final regression pass for interactive reorganization"
```
