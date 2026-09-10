# High-Impact Upgrades — The Last Village

Larger (but still scoped, no-rewrite) improvements that require more than a styling tweak — new small systems or a batch of related drawing work — but deliver the biggest jump in perceived quality. None of these require replacing the canvas-2D approach or redesigning the game.

## Top 5 high-impact upgrades

### 1. A lightweight particle/effects system for hits, deaths, gold, and upgrades
Add one small reusable `particles[]` array + `spawnParticle(x, y, opts)` helper and an `updateParticles(dt)`/`drawParticles()` pass alongside the existing entity arrays (mirrors the existing `bullets[]` pattern already in the codebase, so it fits the architecture). Use it for:
- A burst of small fading dots when a bullet hits an enemy (`updateBullets`, line 428-449).
- A radial burst + fade-out when an enemy dies, instead of instant removal (`updateEnemies`, line 401-426).
- A floating "+3g" text that rises and fades when gold is collected (`updateGoldDrops`, line 451-465).
- A brief expanding ring/flash on the object when a tower/core/weapon upgrade succeeds (`upgradeWeapon`/`repairCore`/`buildOrUpgradeTower`, lines 236-262).
This single system, reused four ways, is the highest-leverage investment in the whole audit — it directly fixes problems #19-23 in [visual_problems.md](visual_problems.md) at once.
- Effort: ~2-3 hours for the system plus all four hookups. Impact: very high — this is what separates "functional prototype" from "satisfying game feel."

### 2. Distinct silhouettes per entity type instead of "circle + label"
Replace the generic circle-based rendering for core, forge, towers, and the two enemy types with small hand-built vector shapes drawn via `ctx.beginPath()`/`lineTo`/`arc` combinations — e.g., core as a hex "fort" outline with a flag, forge as an anvil silhouette with an orange "ember" glow, towers as a turret base + barrel that rotates toward its target, fast enemies as a sharp arrow/dart shape, strong enemies as a hunched, spiked blob. This directly fixes problems #3-4 and #1 in [visual_problems.md](visual_problems.md) and gives the game an actual visual identity instead of "everything is a circle."
- Where: `draw()` — replace the arc-based rendering blocks (lines 514-624) with per-type path drawing functions.
- Effort: ~3-4 hours (mostly iteration/tuning of shapes). Impact: very high — this is the difference between "vector art style" and "physics diagram."

### 3. A designed HUD + world-label component system with animated counters
Beyond the quick-win panel styling, build this into a proper small "UI kit": consistent icon set (inline SVG, not emoji, for crisp scaling and theme control), a tween helper for number changes (HUD gold/HP count up/down smoothly instead of snapping, fixing problem #25), and a shared `drawWorldLabel(text, cost, x, y, affordable)` helper that renders all build/upgrade/repair labels with consistent chip styling, an icon, and a color state (greyed out when unaffordable vs. bright gold when affordable) — this affordability color cue is currently completely absent and is a meaningful gameplay-communication upgrade as well as a visual one.
- Where: new small helper functions near `updateHUD()`/`draw()`, applied at all label call sites (lines 527-594) plus the `#hud` markup.
- Effort: ~2-3 hours. Impact: high — improves moment-to-moment clarity and polish simultaneously.

### 4. A layered environment pass: town-zone ground treatment + static decorative props
Add a second, larger radial "town" ground layer around the core (subtly different color/texture from the wilderness) so the play space reads as a place, plus scatter a small fixed set of static decorative props (trees, rocks, fences, a couple of small hut silhouettes) outside the town ring, generated once at world-init and drawn behind entities each frame. Also add faint long shadows for all props/entities cast in a consistent light direction to unify the lighting model established in the quick wins.
- Where: new `initEnvironment()` generating a static prop list at game start (alongside `makeInitialState()`, line 149), plus a new draw pass before entities in `draw()` (before line 514).
- Effort: ~3-4 hours. Impact: high for atmosphere/immersion, though lower urgency than #1-#3 since it's the least-focused-on part of the screen during actual play.

### 5. Screen shake, hit-flash, and camera feedback layer
Add a small camera-offset "shake" system (a decaying random offset added to `camera.x/y` in `worldToScreen`, triggered on core damage and strong-enemy hits) plus a full-screen color flash (a translucent colored `fillRect` overlay that fades out over ~150ms) triggered on taking core damage (red) and on leveling up a tower/weapon (gold/white). This adds a physical, felt weight to the game's key moments that currently register only as HUD number changes.
- Where: new `shake` state + modification to `worldToScreen()` (line 483-488), plus a new full-screen overlay draw call at the end of `draw()` (after line 658), triggered from `updateEnemies()` (core damage, line 411-415) and the three upgrade functions (lines 236-262).
- Effort: ~1.5-2 hours. Impact: high relative to effort — cheap system, but only "high impact" when combined with #1 (particles) since shake alone without any accompanying particle/flash can feel gimmicky.

## Sequencing note
Upgrades #1 and #5 share infrastructure (both hook into the same event moments: hit, death, gold pickup, upgrade, core damage) — implementing them together in one pass is more efficient than doing them separately. #2 (silhouettes) and #4 (environment) are independent visual-content work that can be done in parallel by a second pass without touching the same code paths.
