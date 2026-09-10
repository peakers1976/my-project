# Visual Summary — The Last Village

Scope: visual quality only (art, color, UI, animation, environment). Gameplay/logic is out of scope and is solid — this audit is about how it *looks*, not how it plays.

## 1. Overall visual impression

**Yes, it currently reads as a programmer-art prototype, not a game.** Every entity (player, enemies, towers, core, forge, bullets, gold) is a flat-filled circle with a solid outline, drawn with the Canvas 2D primitive API (`arc`/`fillRect`). There is no shading, no texture, no outline styling variance, and no depth cues anywhere. This is the single biggest driver of the "cheap/placeholder" feeling — it looks like a physics-diagram mockup of a game rather than the game itself.

**What makes it feel low-end, specifically:**
- Every object is a perfect circle or rectangle with one flat fill color — no gradient, no texture, no highlight/shadow to suggest volume.
- All labels (`CORE`, `FORGE`, `L1`, `Build (15g)`, etc.) are plain `ctx.fillText` in Arial/default canvas font — no drop shadow, no background chip, no icon, just floating text.
- The background is a single flat color (`#2b3a24`) with a faint 1px grid overlay — it reads as "unrendered graph paper," not a village or battlefield.
- There is zero animation on anything except position: no idle bob, no muzzle flash, no death effect, no screen shake, no hit-flash. Enemies simply vanish when killed; gold just fades from existence into the HUD number.
- The HUD is unstyled text stacked in the corner with a text-shadow — no panel, no icon, no background container distinguishing it from the game world behind it.
- The win/lose overlay is a semi-transparent black box with default-looking heading text and a flat green button — functional but visually generic (looks like a browser `alert()` with CSS applied).

**What already looks good / worth keeping:**
- The color-coding logic is sound: gold/yellow for currency and player bullets, blue for towers/tower bullets, red for "strong" enemies vs. yellow for "fast" ones, green for player/health. This palette *logic* is a good foundation — the audit below focuses on styling it, not replacing it.
- The HUD's use of color-coded spans (gold/hp/timer) and the low-HP color swap (`green → red`) is a nice, cheap piece of feedback design already in place.
- Range indicators (faint circle under leveled towers) are a subtle, non-cluttering touch that's already the right idea, just needs polish.
- The core interaction model — labels floating directly under clickable world objects — is a genuinely good UX pattern for "no menu" design; it just needs a visual upgrade, not a redesign.

**What to improve first:** the entity rendering itself (Section 2/3 below) and basic hit/death juice (Section 5). Those two changes alone will move the game further than any UI polish, because they're what's on-screen 100% of the time.

## 2. Art style

There is currently **no art style** — there's a rendering *technique* (flat vector primitives) but no deliberate stylistic choice being made with it. Flat-shape/vector art is a legitimate, low-effort-to-execute style (think geometric/"vector minimalist" indie games), but right now it's applied inconsistently and without the finishing touches that make that style read as intentional rather than incomplete:

- No consistent outline weight/treatment — some objects have thick strokes (core: 4px), some thin (bullets: none), some dashed (empty tower spot).
- No shared shape language — core and forge and towers are all "circle with a smaller circle/label inside," so structurally important objects don't read as visually distinct roles at a glance; only color and label text differentiate them.
- Text labels use the browser default font stack, which will render differently across Chrome/Edge/Firefox and OS. Nothing about the text (weight, letter spacing, color) currently reinforces a style.

**Recommended direction:** commit to a clean, flat "geometric vector" style on purpose — it's the cheapest style to execute at high quality and fits an HTML canvas game well. Concretely: give every entity type a distinct silhouette (not just color) — e.g. core = hex/fort shape, towers = squat turret shape with a barrel, forge = anvil-like silhouette, enemies = spiky shape for "strong" vs. rounded for "fast" — plus a consistent 2-tone shading rule (base fill + one shade darker for a bottom/rim shadow) applied everywhere. That one shading rule, applied consistently, is what makes flat-vector art look "designed" instead of "sketched."

## 3. Color and lighting

- The base palette (olive-green ground, warm gold accents, blue tech accents, red danger) is coherent and has no clashing hues — good bones.
- Contrast is the weak point: the ground (`#2b3a24`) and grid lines (`rgba(255,255,255,0.04)`) are both dark and low-saturation, so the whole game world reads as visually flat/murky, and important objects (core, towers) don't pop far enough above it in value.
- There is no lighting model at all — no gradient fills, no radial glow, no shadows cast by objects onto the ground. Everything sits at the exact same visual "depth."
- Nothing glows. Gold, bullets, and the core (i.e., the things the player should be most drawn to look at) would benefit most from a soft radial glow, since glow is one of the cheapest ways to make small canvas shapes look premium and is very legible at a glance during fast action.
- Separation from background currently relies entirely on hue, not on value/contrast or drop shadows — a soft dark drop-shadow ellipse under every entity (core, towers, player, enemies) would immediately add a sense of ground-plane and depth for very little draw cost.

## 4. UI quality

- HUD: plain stacked `<div>` text, no panel/background, no icons — currently indistinguishable from a debug overlay. Needs a semi-opaque rounded panel, small icons (coin, heart/shield, clock) next to each stat, and a consistent type scale.
- World-space labels (`Build (15g)`, `Upgrade (30g)`, `Repair (15g)`) are raw canvas text with no background chip — they can become unreadable against busy backgrounds and look like debug labels, not UI.
- Fonts: Arial/Segoe UI system stack is used everywhere (HUD via CSS, world labels via canvas). This is the fastest way to look "unfinished" — a single distinct display/UI font (loaded via Google Fonts, which is CDN-allowed) instantly raises perceived production value.
- Buttons: only one exists (Play Again) and it's a flat-fill rectangle-with-radius — no shadow, no gradient, no hover animation beyond a flat color swap, no icon.
- Spacing/hierarchy: HUD lines are evenly spaced with no visual hierarchy (gold, HP, and timer all look equally important, same font size) — the most game-critical stat (Town HP) doesn't stand out.
- Overall the UI currently feels like a functional debug layer laid over the game, not a "designed" interface layer. This is one of the highest-leverage areas to fix because it's on-screen constantly and costs nothing in gameplay logic to improve.

## 5. Animation and visual feedback

This is the area with the largest gap between "functional" and "feels good," and currently has almost nothing:

- **Hits**: bullets disappear into an enemy with zero feedback (no flash, no impact spark, no knockback nudge).
- **Death**: enemies just vanish from the array — no death animation, no particle burst, no fade-out.
- **Gold pickup**: value silently increments in the HUD; no popup ("+3g"), no sparkle, no sound-adjacent visual cue.
- **Build/upgrade/repair**: state changes instantly with no confirmation flourish (no pulse, no particle burst, no screen flash) — a satisfying "upgrade" moment is one of the most important feelings in this genre and currently doesn't exist visually.
- **Damage taken**: core losing HP has no world-space feedback (no shake, no red flash on the core, no damage number) — the player only learns about it from the HUD number ticking down.
- **Player**: moves and rotates smoothly (this part is fine, since it's just physical motion), but has no muzzle flash when firing, no walk-cycle/bob, and no hit-reaction if it ever takes damage.
- Everything currently feels "stiff" because state changes are binary (exists → gone, HP A → HP B) with no interpolation, particles, or camera feedback layered on top.

**Verdict:** this is the single highest-leverage category for "feels premium" per unit of effort — small, cheap effects (flash-white-on-hit, radial particle burst on death, floating "+Ng" text, a brief scale-pulse on upgrade) cost very little to implement in canvas and dramatically change the perceived quality of the moment-to-moment loop.

## 6. Backgrounds and environment

- The world is currently a single flat color with a faint grid — it does not read as a "village under siege," just as an empty canvas with graph paper.
- There's no visual distinction between "inside the town" and "the wilderness enemies come from" — no decoration, no path, no tree line, no perimeter marker. A player can't visually tell where the safe zone ends.
- No parallax, no environmental props (rocks, fences, decorative huts, campfires), no shadows — the world has zero sense of depth or place.
- This is a legitimate "later" item relative to entity rendering and juice (Sections 2/5), because background work is the least noticed of the visual categories moment-to-moment (players look at entities/UI far more than empty ground) — but it's the biggest lever for "does this feel like a real place" rather than "a tech demo."
- Cheapest high-value background upgrade: a radial "town" zone (subtle color/texture change) around the core distinguishing settled land from the wilderness ring enemies spawn from, plus a handful of static decorative props (a few trees/rocks/fences scattered outside the town ring) for scale and depth — no gameplay impact, pure atmosphere.

See [visual_problems.md](visual_problems.md), [quick_wins.md](quick_wins.md), [high_impact_upgrades.md](high_impact_upgrades.md), and [visual_priority_plan.md](visual_priority_plan.md) for actionable detail.
