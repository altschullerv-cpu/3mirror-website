# 3Mirror Website — Session Handoff

**Date:** 2026-05-20
**Last commit:** `6181a4e`
**Branch:** `main` — fully pushed to origin, auto-deploys to Vercel

---

## Project

Single-file vanilla HTML/CSS/JS landing page.

| Path | Purpose |
|---|---|
| `C:\Users\altsc\Desktop\Creaio.ai\3mirror-website\index.html` | Entire site — all HTML, CSS, JS in one file |
| `.claude/launch.json` | Dev server configs (vercel-dev :3000, python-http :8000, npx-serve :5000) |
| `.vercel\project.json` | `projectId: prj_jTbtPXHc0ygkI5f6zBu0PUXhkfrL` |

**Local preview:** python-http server on port 8000. Start via `preview_start("python-http")`, then use `preview_screenshot` / `preview_eval`.

---

## What was done this session

### 1. Insight cards — complete overhaul

5 cards, full-width overlapping deck. Final CSS state:

```css
.insights {
  display: flex; width: 100%; align-items: stretch;
  margin-top: 40px; padding: 40px 0 60px;
}
.insight-card {
  background: var(--bg-elev);   /* #111111, opaque */
  border: 1px solid var(--border);
  border-left-width: 3px;
  border-left-color: rgba(255,255,255,0.18);
  border-radius: 10px;
  padding: 22px 24px 20px;
  display: flex; flex-direction: column; gap: 14px;
  width: 28%; flex-shrink: 0; min-height: 360px;
  transition: transform 0.3s ease, box-shadow 0.3s ease, background 0.3s ease;
  cursor: pointer;
  box-shadow: 0 14px 30px rgba(0,0,0,0.55),
              inset -1px 0 0 rgba(255,255,255,0.09);  /* right-edge highlight */
}
.insight-card + .insight-card { margin-left: -10%; }
/* Stack: first card on top */
.insight-card:nth-child(1) { z-index: 5; }
.insight-card:nth-child(2) { z-index: 4; }
.insight-card:nth-child(3) { z-index: 3; }
.insight-card:nth-child(4) { z-index: 2; }
.insight-card:nth-child(5) { z-index: 1; }
/* Hover = lift only, NO z-index change */
.insight-card:hover:not(.is-active) {
  transform: translateY(-22px);
  box-shadow: 0 28px 44px rgba(0,0,0,0.65),
              inset -1px 0 0 rgba(255,255,255,0.09);
}
/* Click = come to front */
.insight-card.is-active {
  transform: translateY(-32px) scale(1.06) !important;
  z-index: 100 !important;
  background: var(--bg-elev-2);
  box-shadow: 0 40px 80px rgba(0,0,0,0.8),
              0 0 0 1px rgba(255,255,255,0.1),
              0 0 70px rgba(212,196,164,0.06);
}
```

Click-to-activate JS (end of `<body>`):
```javascript
(function insightCardClicks() {
  var cards = document.querySelectorAll('.insight-card');
  if (!cards.length) return;
  cards.forEach(function (card) {
    card.addEventListener('click', function (e) {
      e.stopPropagation();
      var wasActive = card.classList.contains('is-active');
      cards.forEach(function (c) { c.classList.remove('is-active'); });
      if (!wasActive) card.classList.add('is-active');
    });
  });
  document.addEventListener('click', function () {
    cards.forEach(function (c) { c.classList.remove('is-active'); });
  });
})();
```

Card HTML structure (v2):
```html
<div class="insight-card iv-low reveal">
  <div class="iv-header">
    <span class="iv-phrase">"I'm confident in this"</span>
    <div class="iv-meta">
      <span class="iv-score">0.12</span>
      <span class="iv-dot">·</span>
      <span class="iv-tag">aligned</span>
    </div>
  </div>
  <div class="iv-section">
    <div class="iv-label">Conclusion</div>
    <p class="iv-body">...</p>
  </div>
  <div class="iv-section iv-action">
    <div class="iv-label">Action</div>
    <p class="iv-body">...</p>
  </div>
</div>
```

The 5 cards (in order, first = top of stack):
| Card | Score | Tag | Class |
|---|---|---|---|
| "I'm confident in this" | 0.12 | aligned | iv-low |
| "I'm not sure yet" | 0.41 | decided | iv-mod |
| "This is great news" | 0.08 | aligned | iv-low |
| "I disagree" | 0.73 | uncertain | iv-high |
| "No problem at all" | 0.55 | tense | iv-mod |

---

### 2. RGC section — copy rewrite

**Tagline** (`.section-tagline`): *"A calibration matrix tool."*

**3 description paragraphs** (`.description`):
1. TTS model = precision by design, no internal state, non-precision is a bug
2. Human speech = two channels always, body leaks what the person hides
3. How RGC works: feeds phrases → extracts acoustic reference per emotional tag → scale 0.0–1.0, 180K combinations

---

### 3. RGC Matrix — current row structure

| Row | Values | Style |
|---|---|---|
| TTS Voice A | 0.00 × 5 | `cell-zero` |
| TTS Voice B | 0.00 × 5 | `cell-zero` |
| TTS Voice C | 0.00 × 5 | `cell-zero` |
| Averaged reference | 0.00 × 5 | `cell-zero` |
| **— 3Mirror Analysis —** | *(separator)* | `row-separator-label` |
| Human voice | 0.12 / 0.41 / 0.08 / 0.73 / 0.55 | `row-real` |
| Tag (Emotional Direction) | aligned / decided / aligned / uncertain / tense | `row-direction` |
| **— Conclusion & Action —** | *(separator)* | `row-separator-label` |
| Conclusion | Sincere / Hidden opinion / Sincere / Masked doubt / Concealed issue | italic, 0.55 opacity |
| Action | Reliable / Ask directly / Reliable / Address the doubt / Follow up | 0.55 opacity |

Direction tag colors: `aligned` = blue (`#6dcabe`), `decided` = amber (`#e0b878`), `uncertain` = red (`#e88888`), `tense` = amber (`#e0b878`).

Matrix borders: `rgba(95,184,176,0.22)` teal throughout.

---

### 4. Other elements in RGC section

**180K callout** (`.matrix-note`) — below matrix, outside `.matrix-wrap`:
- Green left border `#1D9E75`, `rgba(29,158,117,0.04)` background, 12px text

**"How to read this"** (`.matrix-callout`) — above matrix:
- Explains 3 sections: RGC Matrix (zero point) → 3Mirror Analysis (divergence + severity + direction) → Conclusion & Action (plain language + recommended response)

**PSD scale bar** — above matrix-callout, shows 0.0 aligned → 0.3 slight → 0.6 moderate → 1.0 high with gradient

**Word-by-word divergence graph** — Chart.js 4.4.7 (loaded sync, NO `defer`), `id="divergenceWave"` canvas inside `#divergenceGraph`

---

## CSS design tokens

```css
:root {
  --bg: #0a0a0a;
  --bg-elev: #111111;
  --bg-elev-2: #141414;
  --border: rgba(255,255,255,0.08);
  --border-strong: rgba(255,255,255,0.16);
  --text: #ffffff;
  --text-2: #b8b8b8;
  --text-3: #888888;
  --text-4: #555555;
  --accent: #d4c4a4;
  --c-zero-bg: #14302e;
  --c-zero-border: rgba(95,184,176,0.22);
  --c-zero-text: #6dcabe;
  --c-low-bg: #182542;
  --c-low-border: rgba(122,156,212,0.22);
  --c-low-text: #92b3e6;
  --c-mod-bg: #3a2a14;
  --c-mod-border: rgba(212,168,106,0.22);
  --c-mod-text: #e0b878;
  --c-high-bg: #3a1818;
  --c-high-border: rgba(212,120,120,0.24);
  --c-high-text: #e88888;
}
```

---

## Known technical notes

- `h2.section-title` uses `-webkit-text-fill-color: transparent` + `background-clip: text` for gradient. Plain `color:` has no effect on it.
- Chart.js CDN `<script>` must be placed directly before the inline init `<script>` at end of `<body>` — no `defer`, no async.
- Mirror cards use `transform-style: preserve-3d` + `backface-visibility: hidden`. The `.label` element lives inside `.surface` so it hides/shows with the surface automatically.
- `.vercelignore` excludes `.claude/` to prevent 386MB worktree bloat on deploy.

---

## Commit history (recent)

```
6181a4e  RGC section: rewrite description copy, update tagline
48980b8  Insight cards: increase hover lift to -22px
cbb2447  Insight cards: hover = lift only (no z-index change), click = bring to front
1ac4e8a  Insight cards: remove rotation, add right-edge highlight
1cf7cd8  RGC insight cards: reversed fan direction
4b61453  RGC: insight deck UX, matrix updates, How to read this above matrix
e3f225e  RGC: expanded insight cards with conclusions and actions, 180K callout
899b008  3Mirror: divergence/conclusion formula; RGC reorder
ae6d599  Fix Chart.js loading
5c0ef2a  RGC: divergence graph, direction row, conclusion
```
