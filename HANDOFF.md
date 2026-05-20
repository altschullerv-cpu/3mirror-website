# 3Mirror Website — Session Handoff

**Date:** 2026-05-20
**Branch:** `main` — fully pushed to origin, auto-deploys to Vercel

---

## Project

Single-file vanilla HTML/CSS/JS landing page.

| Path | Purpose |
|---|---|
| `C:\Users\altsc\Desktop\Creaio.ai\3mirror-website\index.html` | Entire site — all HTML, CSS, JS in one file |
| `.claude/launch.json` | Dev server configs (vercel-dev :3000, python-http :8000, npx-serve :5000) |
| `.vercel\project.json` | `projectId: prj_jTbtPXHc0ygkI5f6zBu0PUXhkfrL` |

**Local preview:** python-http server on port 8000. Start via `preview_start("python-http")`, then use `preview_eval`.
`preview_screenshot` does NOT work reliably (renderer timeout). Use `preview_eval` to verify JS state; user opens localhost:8000 in browser.

---

## Session 1 (commits up to 6181a4e)

### Insight cards — initial overhaul
5 cards, full-width overlapping deck. Hover = lift, click = bring to front.

### RGC section — first copy rewrite
Tagline, 3 description paragraphs, matrix structure established.

---

## Session 2 (commit a9fb376)

### Insight cards — UX overhaul
Hover lift: `translateY(-78px)`. Mouseleave instant deactivate. Active state white glow.

### Hero mirrors — complete fix
Opaque surfaces, rounded corners, backface visibility via JS rAF loop (CSS `backface-visibility` unreliable in Chrome with `preserve-3d`).

### 3Mirror + RGC sections — full text rewrites
See git commit `a9fb376` for full diff.

---

## Session 3 (2026-05-20)

### 1. Merged flamboyant-cohen-245a9f → main
3 commits (c0ba874, a9fb376, 9ad6169) fast-forwarded into main, pushed to origin.

### 2. launch.json fix
`vercel-dev` config: `--listen` flag was unknown. Fixed to `["dev", "--yes", "-l", "3000"]`.

### 3. RGC description — removed one paragraph
Deleted: *"That is the zero point of the scale."*
Now 4 paragraphs (was 5): zero-intro → TTS precision → real person above zero → RGC scale.

### 4. Mirror rotation — slowed down
`axis-spin` duration: **28s → 45s**

### 5. Mirror numbers removed
Deleted `<span class="idx">01/02/03</span>` from all 3 mirror labels. Only tags remain: Semantic / Acoustic / Context.

### 6. Formula block — beam abandoned, static frame
**Long saga:** tried conic-gradient + CSS animation, SVG dasharray comet, JS perimeter math with `@property`. All had issues (speed uneven / ugly / broken inheritance). User decided to drop beam entirely.

**Current state — formula block:**
```css
.formula-block {
  margin: 40px auto 0;
  max-width: 760px;
  padding: 32px 32px 28px;
  border: 1px solid var(--border-strong);
  border-radius: 14px;
  box-shadow: 0 0 30px rgba(212,196,164,0.06), 0 0 80px rgba(212,196,164,0.03);
  text-align: center;
  font-family: "JetBrains Mono", monospace;
  color: var(--text);
  line-height: 1.7;
}
.formula-line {
  font-size: 28px;   /* was 18px */
  font-weight: 500;
  letter-spacing: -0.005em;
  margin-bottom: 4px;
}
.formula-line .accent { color: var(--accent); }
```

No `::before`, no `@property`, no JS for beam.

### 7. Formula text updated
```
Voice − Text = Divergence
Divergence + Context = Conclusion
```
(was "Text − Sound = Divergence")

### 8. Patent badges shortened

| Location | Old | New |
|---|---|---|
| Hero eyebrow | `Patent Pending · 2026` (pill badge) | **Removed entirely** |
| #threemirror patent-badge | `Patent Pending · US PPA #64/016,499 · Priority March 2026` | `Patent Pending · #64/016,499` |
| #rgc patent-badge | `Patent Pending · US PPA #64/018,274 · Priority March 2026` | `Patent Pending · US #64/018,274` |

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

- `h2.section-title` uses `-webkit-text-fill-color: transparent` + `background-clip: text` for gradient. Plain `color:` has no effect.
- Chart.js CDN `<script>` must be placed directly before the inline init `<script>` at end of `<body>` — no `defer`, no async.
- Mirror `.surface` visibility controlled by JS rAF loop, NOT by CSS `backface-visibility` (unreliable in Chrome with `preserve-3d`).
- `.vercelignore` excludes `.claude/` to prevent 386MB worktree bloat on deploy.
- `preview_screenshot` consistently times out. Use `preview_eval` for verification.

---

## Commit history (recent)

```
ac73eaa  Beam: switch to pure CSS animation via @property
b2bc30f  Fix beam animation: add @property declaration for --beam-angle
9ad6169  Update HANDOFF.md: add session 2 summary
a9fb376  3Mirror + RGC: text rewrites, mirror fixes, card UX overhaul
c0ba874  Insight cards: increase hover lift to -74px
44dce8b  Add HANDOFF.md session summary for next chat
```
