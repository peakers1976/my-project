# Quick Wins — The Last Village

Low-effort, high-visibility changes. All of these are CSS/canvas-drawing tweaks only — no gameplay logic changes, no new assets required, and each is roughly a 10-40 line change in [index.html](../index.html).

## Top 5 quick wins (do these first)

### 1. Add radial glow + soft drop shadows to key entities
Before drawing gold, bullets, and the core, draw a soft radial gradient glow behind them (`ctx.createRadialGradient` + a low-alpha fill), and draw a simple dark semi-transparent ellipse ("shadow") beneath every standing entity (player, enemies, towers, core, forge) before drawing the entity itself. This is pure additive drawing — no state changes needed. Biggest visual-impact-per-line-of-code change available.
- Where: inside `draw()`, just before each entity's existing fill call (lines ~514-658).
- Effort: ~30 minutes. Impact: high — instantly adds depth and makes gold/bullets feel valuable/energetic.

### 2. Give every entity a two-tone shading pass instead of one flat fill
For each shape, draw the base fill, then an inner highlight arc (lighter shade, offset slightly toward one corner) or a darker rim stroke on the bottom half. E.g., for the player/enemy circles, add a second smaller/offset circle in a lighter tint before the outline stroke. Takes flat "programmer art" circles most of the way to looking like actual game sprites with zero new assets.
- Where: `draw()` — player (636-658), enemies (616-624), towers (560-602), core (514-534).
- Effort: ~45 minutes. Impact: high.

### 3. Style the HUD as a real panel with icons
Wrap the three HUD stats in a semi-opaque rounded panel (`background: rgba(20,24,16,0.55); border-radius: 12px; padding: 12px 16px; backdrop-filter: blur(4px);`), add a small inline SVG/emoji icon before each stat (🪙 / 🛡 / ⏱ or custom SVGs), and bump the Town HP line to a larger/bolder font than gold/timer to establish hierarchy. Also give `#hud` a subtle border (`1px solid rgba(255,255,255,0.08)`).
- Where: `#hud` CSS block (lines 17-31) and the three `<div>`s (lines 76-78).
- Effort: ~20 minutes. Impact: high — this is the part of the screen the eye rests on most.

### 4. Swap the default font stack for a distinct display/UI font
Add a Google Fonts import (CDN-allowed: `fonts.googleapis.com`/`fonts.gstatic.com`) for one solid display font (e.g. "Rubik", "Chakra Petch", or "Press Start 2P" for a more arcade feel) and apply it to `body` and to all `ctx.font` calls. A single font swap changes the entire perceived production value for a two-line change.
- Where: `<head>` (add `<link>`), `body` font-family (line 12), every `ctx.font = '...'` call in `draw()`.
- Effort: ~15 minutes. Impact: high, essentially free.

### 5. Add a background chip behind every world-space label
Before drawing text like `Build (15g)`, `Repair (15g)`, `L1`, draw a small dark rounded rectangle (or a simple `fillRect` with low-alpha black + 1px border) behind the text so it's always legible and reads as a UI element rather than debug text floating in the world.
- Where: every `ctx.fillText(...)` call tied to a cost/level label (lines ~527-533, 549-557, 573-594).
- Effort: ~30 minutes (one small helper function `drawLabelChip(x, y, text)` reused everywhere). Impact: medium-high, cheap to implement once as a helper.

## Additional quick wins worth doing in the same pass

- **Restyle the Play Again button**: add `box-shadow: 0 4px 12px rgba(0,0,0,0.4)`, a subtle gradient background instead of flat `#4a7c3c`, and a `transform: scale(1.05)` + brighter shadow on `:hover`/`:active` for a tactile press feel. (CSS-only, `#overlay button`, lines 60-69.)
- **Fade/scale-in the win/lose overlay** instead of an instant `display: flex` — add a CSS transition on opacity/transform triggered by toggling a class. (CSS + one-line JS change, lines 46-57, 467-480.)
- **Round and soften the empty tower-spot dashes**: increase dash gap/segment for a less "wireframe" look, and add a very faint pulsing alpha (via a sine function of elapsed time) so empty build spots visibly invite interaction.
- **Replace the flat grid background** with a subtly varied fill — e.g. alternate faint patches of two close ground shades in a noise-like pattern, or simply reduce grid opacity further and add 1-2 shades of mottled color blocks — so the ground doesn't read as literal graph paper.
- **Add a colored rim-light stroke matching each enemy type's palette** (e.g., strong enemies get a darker red rim, fast enemies a brighter yellow rim) to reinforce type at a glance even before the player consciously reads color.
- **Tint the low-HP HUD state more aggressively** — when Town HP is low, also pulse the HUD panel border red (`box-shadow` animation) instead of only recoloring the number, so the urgency cue isn't limited to a single small text element.
