# Visual Problems — The Last Village

Concrete, specific issues found in [index.html](../index.html), grouped by area. Each item names the location in code so it's easy to find and fix.

## Entities / rendering (draw(), lines ~497-661)

1. **All entities are flat single-color primitives with no shading.** Player, enemies, towers, core, forge, bullets, and gold are all `ctx.fill()` circles/rects with one solid color and (sometimes) a stroke outline. Nothing has a highlight, gradient, or shadow to suggest form or volume. This is the #1 cause of the "unfinished/placeholder" impression.
2. **No drop shadows / ground contact.** No entity casts a shadow onto the ground beneath it, so nothing looks like it's standing "on" the world — everything looks pasted flat onto the background layer.
3. **Text-only differentiation between important structures.** Core, forge, and towers are all "circle + label" shapes (lines 514-534, 536-558, 560-602) — without reading the text, a player can't tell what an object is by silhouette alone. There's no distinct shape language per object type.
4. **Enemies have no visual distinction beyond color and size.** Fast (`#e8c34a`, r=11) and strong (`#c24a3a`, r=16) enemies (line 134-136) are both plain circles — differentiated only by hue/radius, not silhouette, making them easy to visually confuse at a glance during chaotic moments.
5. **Bullets are undifferentiated dots.** Player and tower bullets (line 626-633) are 4px circles distinguished only by color (`#fff59d` vs `#9ad6ff`) with no trail, glow, or elongation to suggest motion/speed.
6. **Health bars are utilitarian rectangles.** `drawHealthBar()` (line 490-495) is a black background rect + solid color fill rect — no border, no segment/tick marks, no smooth drain animation (updates instantly to new HP value).
7. **World-space UI text has no background/legibility treatment.** Labels like `Build (15g)`, `Repair (15g)`, `L1` (lines 527, 532, 549, 554, 574, 586, 591) are raw `ctx.fillText` calls with no background chip or outline — they can wash out against the background and look like debug overlay text rather than game UI.
8. **Player has no muzzle flash or recoil.** `fireBullet()` (line 266-279) spawns a bullet at the player's exact position with no visual acknowledgment at the firing point.

## Color & lighting

9. **Overall value contrast is low.** Background `#2b3a24` and grid `rgba(255,255,255,0.04)` (lines 498, 502) are both dark, desaturated, and close in value — the world reads as murky/flat rather than crisp. Important objects (core `#8a6d3b`, towers `#4a6fa5`) are only moderately brighter than the ground, so nothing "pops."
10. **No glow on high-importance elements.** Gold (`#ffd54a`, line 607), the core, and player bullets — the elements most tied to reward/feedback — have zero glow/bloom, which is one of the cheapest premium-feeling effects to add and is currently entirely absent.
11. **Static grid background reads as a debug tool, not art.** The `1px, rgba(255,255,255,0.04)` grid (lines 501-512) looks like graph paper / level-editor guide lines left visible in a shipped build.

## UI (CSS + HUD, lines 6-87)

12. **HUD has no visual container.** `#hud` (line 17-27) is plain text with only a `text-shadow` for legibility — no background panel, no border, no icons, so it doesn't read as a designed UI layer, and text-shadow alone is a fragile way to keep it legible over a bright background.
13. **No icons anywhere in the UI.** Gold, Town HP, and Time are all text-only labels ("Gold:", "Town HP:", "Time left:") — no coin/shield/clock glyph, which is a fast, low-cost visual-maturity signal that's currently missing entirely.
14. **Default system font stack used throughout.** `font-family: 'Segoe UI', Arial, sans-serif` (line 12) plus unstyled canvas text (`ctx.font = 'bold 14px Arial'` etc., e.g. line 525) means the game uses whatever default UI font each OS/browser provides — it will look different per machine and reads as "unstyled," not deliberately chosen.
15. **No visual hierarchy between HUD stats.** Gold, Town HP, and Time (lines 76-78) are all the same font size/weight aside from color — the single most critical stat (town HP) doesn't stand out from the others.
16. **Button styling is minimal.** `#overlay button` (line 60-69) is a flat-colored rounded rectangle with only a flat hover color swap — no shadow, no gradient, no press/hover animation beyond the color change, no icon.
17. **Overlay screen is generic.** `#overlay` (line 46-57) is a semi-transparent black box with plain white heading text — functionally works but has no game-specific art direction (no border, no themed background texture, no icon/crest).
18. **Hint bar looks like a leftover dev tooltip.** `#hint` (line 33-44) is small, low-contrast, unstyled text in a translucent box in the corner — reads as a debug/help string rather than a designed onboarding element.

## Animation & feedback (game loop, lines 316-465)

19. **No hit feedback.** `updateBullets()` (line 428-449) deducts HP from an enemy on collision with zero visual acknowledgment (no flash, no impact particle, no knockback).
20. **No death effect.** `updateEnemies()` (line 401-426) removes dead enemies from the array in the same frame they hit 0 HP — they simply disappear; no particle burst, fade, or death animation of any kind.
21. **No damage feedback on the core.** The core loses HP from melee attacks (line 411-415) with no shake, flash, or floating damage number — the only feedback is the HUD number ticking down.
22. **No reward feedback for gold pickup.** `updateGoldDrops()` (line 451-465) silently increments `state.gold` — no floating "+3g" text, no sparkle/pop animation, no HUD counter animation (number just jumps to the new value).
23. **No upgrade/build/repair confirmation effect.** `upgradeWeapon()`, `repairCore()`, `buildOrUpgradeTower()` (lines 236-262) instantly mutate state with no visual celebration (no particle burst, no flash, no scale-pulse on the object) — the most emotionally important moments in the core loop (spending gold to get stronger) currently have no "juice."
24. **No screen shake or camera feedback** anywhere (e.g., on taking core damage, on a big kill, on a strong enemy attack) — all feedback is HUD-number-only.
25. **HUD numbers snap instead of animating.** `updateHUD()` (line 670-676) sets `textContent` directly to the new value every frame — gold and HP changes have no count-up/count-down tween, so changes can look like a jump-cut rather than a smooth, readable transition.

## Background & environment

26. **World is a single flat color + grid with no environmental detail.** `draw()` background fill (line 498) plus grid (501-512) is the entire environment — no decorative props, terrain variation, or landmarks.
27. **No visual boundary between "town" and "wilderness."** Enemies spawn from a ring at `WORLD_SPAWN_RADIUS` (620 units, line 144) but nothing in the rendering marks this transition — the player has no visual cue for "safe area" vs. "enemy territory."
28. **No parallax or depth layering.** The single ground layer moves 1:1 with the camera — there is no distant background layer to create a sense of scale or depth.
