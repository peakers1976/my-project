# Visual Priority Plan — The Last Village

A single ranked plan tying together [visual_problems.md](visual_problems.md), [quick_wins.md](quick_wins.md), and [high_impact_upgrades.md](high_impact_upgrades.md) into "what to do, in what order."

## Top 5 visual problems (the ones hurting perceived quality most)

1. **Every entity is a flat, single-color primitive with no shading, glow, or shadow** — the root cause of the "programmer art / placeholder" impression (problems #1-2, #9-10 in [visual_problems.md](visual_problems.md)).
2. **Zero animation/feedback on hits, deaths, gold pickup, and upgrades** — state changes are instant and silent, so nothing feels satisfying (problems #19-25).
3. **The HUD and world labels look like debug overlays, not designed UI** — no panels, no icons, default system font (problems #12-18).
4. **No shape/silhouette differentiation between object types** — everything is "circle + text label," so nothing has a visual identity (problems #3-4).
5. **The environment is a flat color + faint grid with no sense of place** — no town/wilderness distinction, no props, no depth (problems #26-28).

## Top 5 quick wins (do first, small effort, immediate visible difference)

1. Add soft glow (gold, bullets, core) + ground-contact shadows under every entity.
2. Two-tone shading pass (highlight/shadow) on every existing shape instead of flat fill.
3. Style the HUD as a real panel with icons and a type-size hierarchy (Town HP most prominent).
4. Swap the default font stack for one deliberate Google Font, applied to both CSS and canvas text.
5. Add background chips behind all world-space cost/level labels for legibility and "UI" feel.

*(Full detail and code locations for each: [quick_wins.md](quick_wins.md).)*

## Top 5 high-impact upgrades (bigger effort, biggest quality jump)

1. A reusable particle/effects system for hit sparks, death bursts, floating gold text, and upgrade flashes.
2. Distinct hand-drawn silhouettes per entity type (core/forge/tower/fast enemy/strong enemy) instead of circles.
3. A proper animated HUD + world-label UI kit (icon set, number tweening, affordability color states).
4. A layered environment pass: town-vs-wilderness ground treatment plus static decorative props.
5. Screen shake + full-screen hit/level-up flash tied to core damage and upgrades.

*(Full detail and code locations for each: [high_impact_upgrades.md](high_impact_upgrades.md).)*

## What should be changed first

**Do the 5 quick wins first, as one batch.** They are all CSS/canvas-drawing-only changes (no new systems, no new state), can be done in an afternoon, and directly address problems #1 and #3 above — the two biggest contributors to the "cheap" impression. This gets the most visible improvement for the least engineering risk before touching any game logic-adjacent code.

**Then do high-impact upgrade #1 (particle/effects system).** This is the single highest-leverage item in the whole audit: it fixes problem #2 (no feedback/juice) in one reusable piece of infrastructure, reused across four different moments (hit, death, gold, upgrade). Pair it with high-impact upgrade #5 (screen shake/flash) since they share the same trigger points and are cheap to build together — this is explicitly called out in the sequencing note in [high_impact_upgrades.md](high_impact_upgrades.md).

**Then do high-impact upgrade #2 (entity silhouettes).** This is the most labor-intensive item (iterative shape design) but is what finally gives the game an actual visual identity rather than "colored circles with labels" — it's sequenced after the juice system because juice is cheaper to build and more universally noticeable moment-to-moment, but silhouettes matter more for first-impression screenshots/trailers.

## What can wait until later

- **High-impact upgrade #3 (full animated UI kit / icon system)** — the quick-win HUD panel styling gets most of the visible benefit cheaply; the fuller icon/tweening system is a refinement, not a first-priority fix.
- **High-impact upgrade #4 (environment/props pass)** — background is the least-attended-to part of the screen during actual play (players focus on entities and UI far more), so while it's a real upgrade to "sense of place," it's correctly last: it has no bearing on how "cheap" the moment-to-moment action feels, which is the main complaint this audit addresses.
- Any further art-style unification work (e.g., custom illustrated icon sets replacing inline SVG, more elaborate parallax backgrounds) — worth revisiting only after the above are in place and the team can see whether the flat-vector direction still needs more identity work once shading/glow/juice/silhouettes are done.

## Suggested execution order (summary)

1. Quick wins (all 5) — one focused pass, CSS + draw() tweaks only.
2. Particle/effects system + screen shake/flash (high-impact #1 + #5 together).
3. Entity silhouettes (high-impact #2).
4. Full UI kit polish (high-impact #3).
5. Environment/props pass (high-impact #4).
