# 3Mirror Website — Session Handoff

**Date:** 2026-05-20
**Last commit:** `a9fb376`
**Branch:** `claude/flamboyant-cohen-245a9f` (worktree) — not yet pushed to main

---

## Project

Single-file vanilla HTML/CSS/JS landing page.

| Path | Purpose |
|---|---|
| `C:\Users\altsc\Desktop\Creaio.ai\3mirror-website\.claude\worktrees\flamboyant-cohen-245a9f\index.html` | Working copy — all HTML, CSS, JS in one file |
| `C:\Users\altsc\Desktop\Creaio.ai\3mirror-website\index.html` | Main branch copy |
| `.claude/launch.json` | Dev server configs (vercel-dev :3000, python-http :8000, npx-serve :5000) |
| `.vercel\project.json` | `projectId: prj_jTbtPXHc0ygkI5f6zBu0PUXhkfrL` (must exist in worktree — copied from main repo) |

**Local preview:** python-http server on port 8000. Start via `preview_start("python-http")`, then use `preview_eval`.
`preview_screenshot` does NOT work (msedgewebview2 masked in Claude Code). User opens localhost:8000 in their own browser.

---

## Session 1 (2026-05-20, commits up to 6181a4e)

### Insight cards — initial overhaul
5 cards, full-width overlapping deck. Hover = lift, click = bring to front.

### RGC section — first copy rewrite
Tagline, 3 description paragraphs, matrix structure established.

### RGC Matrix — row structure
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

---

## Session 2 (2026-05-20, commit a9fb376)

### 1. Insight cards — UX overhaul

**Hover lift:** `translateY(-78px)` (was -22px)

**Transitions:**
- Default (resting, hover): `transition: transform 1s ease, box-shadow 1s ease, background 1s ease;`
- Active (fast snap in): override on `.is-active`: `transition: transform 0.3s ease, ...`
- Mouseleave (back): 1s (inherits default transition)

**Mouseleave instant deactivate:** `card.addEventListener('mouseleave', ...)` removes `.is-active` immediately — card slides back at 1s ease.

**Active state white glow:**
```css
.insight-card.is-active {
  transform: translateY(-32px) scale(1.06) !important;
  z-index: 100 !important;
  background: var(--bg-elev-2);
  transition: transform 0.3s ease, box-shadow 0.3s ease, background 0.3s ease;
  box-shadow: 0 40px 80px rgba(0,0,0,0.8),
              0 0 0 1px rgba(255,255,255,0.1),
              0 0 70px rgba(212,196,164,0.06),
              0 0 60px rgba(255,255,255,0.15),
              0 0 120px rgba(255,255,255,0.07);
}
```

**Full JS (end of `<body>`):**
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
    card.addEventListener('mouseleave', function () {
      card.classList.remove('is-active');
    });
  });
  document.addEventListener('click', function () {
    cards.forEach(function (c) { c.classList.remove('is-active'); });
  });
})();
```

---

### 2. Hero mirrors — complete fix

**Problem:** Mirrors were semi-transparent (rgba gradients), back face showed reversed text.

**Fix 1 — Opaque surfaces:**
```css
.mirror .surface {
  background: linear-gradient(160deg,
    #3a3a46 0%, #1e1e28 20%, #111118 50%, #0d0d14 75%, #181820 100%);
}
```

**Fix 2 — Rounded corners:** `border-radius: 12px` on `.mirror` (inherits to `.surface` via `border-radius: inherit`).

**Fix 3 — Back face JS visibility toggle:**
CSS `backface-visibility: hidden` unreliable in Chrome with `preserve-3d`. Solution: rAF loop reads computed matrix transform of `.mirrors-axis`, calculates per-mirror total rotation, sets `surface.style.visibility = 'hidden'` when on back face.

```javascript
(function mirrorBackfaceLabels() {
  var axis = document.querySelector('.mirrors-axis');
  var mirrors = document.querySelectorAll('.mirror');
  if (!axis || !mirrors.length) return;
  var baseAngles = [0, 120, 240];
  function tick() {
    var mat = getComputedStyle(axis).transform;
    var axisY = 0;
    if (mat && mat !== 'none') {
      var m = mat.match(/matrix3d\(([^)]+)\)/);
      if (m) {
        var v = m[1].split(',').map(parseFloat);
        axisY = Math.round(Math.atan2(v[8], v[0]) * 180 / Math.PI);
      } else {
        var m2 = mat.match(/matrix\(([^)]+)\)/);
        if (m2) {
          var v2 = m2[1].split(',').map(parseFloat);
          axisY = Math.round(Math.atan2(-v2[2], v2[0]) * 180 / Math.PI);
        }
      }
    }
    mirrors.forEach(function(mirror, i) {
      var total = ((axisY + baseAngles[i]) % 360 + 360) % 360;
      var onBack = total > 90 && total < 270;
      var surface = mirror.querySelector('.surface');
      if (surface) surface.style.visibility = onBack ? 'hidden' : '';
    });
    requestAnimationFrame(tick);
  }
  tick();
})();
```

**`.back` element** (visible when surface hidden):
```css
.mirror .back {
  position: absolute; inset: 0;
  background: #08080c;
  border: 1px solid rgba(255,255,255,0.06);
  transform: translateZ(-1px) rotateY(180deg);
  backface-visibility: hidden;
  -webkit-backface-visibility: hidden;
}
```

---

### 3. 3Mirror section — full text rewrite

**Eyebrow:** `The Incongruence Analysis Engine`
**Title:** `3Mirror`
**Section-tagline:** deleted
**mirrors-callout:** deleted

**5 description paragraphs (current):**
1. Carl Rogers 1961, incongruence — two types (conscious / unconscious), second type = most important signal, no way to measure for 65 years
2. Modern voice AI answers "what is being expressed?" — combining words+voice natural for that question
3. 3Mirror asks different question: where does expression diverge from underneath? Words = decision, voice = what body does. Channels must stay separate.
4. Two streams run in parallel, measure distance word by word, ms by ms. Third layer (role, context) turns number into interpretation.
5. "Rogers' observation, made measurable."

**Formula block:**
```html
<div class="formula-block reveal">
  <div class="formula-line">Text − Sound = <span class="accent">Divergence</span></div>
  <div class="formula-line"><span class="accent">Divergence</span> + Context = <span class="accent">Conclusion</span></div>
</div>
```

**Key highlight pattern:** `<strong style="color:#ffffff; font-weight:600;">word</strong>` — used on `incongruence`.

---

### 4. RGC section — full text rewrite

**Eyebrow:** `Calibration matrix tool`
**Title:** `Reverse Generative Calibration (RGC)`
**Section-tagline:** deleted

**5 description paragraphs (current):**
1. "To measure divergence, we need a scale. A point on this scale where divergence is guaranteed to be absent is a **zero**." (`zero` highlighted white-bold)
2. TTS model = built for precision, no internal state, nothing to leak. Divergence in TTS without emotional tags is zero — architecturally, not approximately.
3. "That is the zero point of the scale."
4. "A real person saying the same words will land somewhere above zero. How far above — that is the measurement."
5. RGC builds this scale: hundreds of phrases through multiple TTS models, acoustic reference extracted, emotional tags that contradict meaning = opposite pole. Full matrix: up to **180,000** combinations (phrases, tags, voices, divergence levels).

**matrix-note (`.matrix-note`, below matrix):**
```html
This example uses a single zero-point reference — TTS without emotional tags. The full calibration matrix supports up to <strong>180,000</strong> combinations
```

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
- Mirror `.surface` visibility is controlled by JS (rAF loop), NOT by CSS `backface-visibility` — the CSS property is unreliable in Chrome with `preserve-3d`.
- `.vercelignore` excludes `.claude/` to prevent 386MB worktree bloat on deploy.
- `.vercel/project.json` must exist in the worktree directory (copy from main repo) — `vercel dev` fails with interactive confirmation prompt without it.
- `preview_screenshot` does NOT work in Claude Code (msedgewebview2 masked). Use `preview_eval` to verify JS state; user refreshes browser manually.

---

## Commit history (recent)

```
a9fb376  3Mirror + RGC: text rewrites, mirror fixes, card UX overhaul
6181a4e  RGC section: rewrite description copy, update tagline
48980b8  Insight cards: increase hover lift to -22px
cbb2447  Insight cards: hover = lift only (no z-index change), click = bring to front
1ac4e8a  Insight cards: remove rotation, add right-edge highlight
1cf7cd8  RGC insight cards: reversed fan direction
4b61453  RGC: insight deck UX, matrix updates, How to read this above matrix
e3f225e  RGC: expanded insight cards with conclusions and actions, 180K callout
899b008  3Mirror: divergence/conclusion formula; RGC reorder
ae6d599  Fix Chart.js loading
```
