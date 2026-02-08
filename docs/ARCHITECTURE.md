# Zombie Movement Toy — Architecture

**Last updated:** Feb 2026 — Level generator pacing upgrade

## Design Philosophy

This is a **movement toy**, not a game. The architecture is intentionally minimal: a single HTML file with all code inline. This constraint is a feature — it makes the prototype trivially portable, shareable, and eliminable. No build tools, no dependencies, no server.

The goal is to answer one question: **Does this feel like the right foundation for a real-time zombie platformer?**

## File Structure

Everything lives in `index.html`. The code is organized into clearly separated sections in this order:

1. **Styles** — CSS for UI overlay elements (including generator button/input styles)
2. **Tunable Constants** — All movement parameters (the tuning panel reads/writes these)
3. **Tile Map** — Preset Gauntlet map as `PRESET_GAUNTLET`, mutable `MAP_DATA`/`tileMap`
4. **Level Generator** — Seeded PRNG, physics envelope, section-based generation, BFS validation
5. **Game State** — Mutable runtime state (zombie position, velocity, input, drift)
6. **Sanity Tier Helpers** — Functions to get tier-adjusted movement values
7. **Input Handling** — Keyboard event listeners, key state tracking
8. **Physics Update** — Core movement loop (gravity, accel, friction, clamp, position)
9. **Collision Detection** — AABB vs tile grid (X-first resolution)
10. **Jump System** — Coyote time, jump buffer, variable height
11. **Input Drift** — Smooth impulses via t² curve (position nudge + velocity residue)
12. **Juice Systems** — Squash/stretch, particles, screen shake, vignette
13. **Rendering** — Canvas drawing (tiles, zombie with squash/stretch, particles, vignette)
14. **UI — Debug Panel** — Real-time movement diagnostics
15. **UI — Sanity Slider** — Primary testing tool
16. **UI — Tuning Panel** — Live constant adjustment + juice params + level generator controls
17. **Game Loop** — requestAnimationFrame with dt capping
18. **Initialization** — Setup and loop start

## Key Architecture Decisions

### Constants as `let` instead of `const`
The tuning panel needs to write to movement constants at runtime. Using `let` allows direct mutation. The constants are still grouped at the top of the file in a clearly labeled section — they're "constants" in design intent, not in language enforcement.

### X-before-Y collision resolution
Each axis is moved and resolved independently: horizontal first, then vertical. Resolving X first prevents the "wall eats jump" bug where pressing into a wall causes the zombie's X to overlap the wall column, and the subsequent ceiling check sees the wall tile and kills the upward velocity. By resolving X first, horizontal position is corrected before any vertical checks run.

### Separate jumpPressed vs jumpHeld
Variable jump height requires knowing when the jump key is *released* (to cut velocity), while jump buffering requires knowing when it was first *pressed* (to start the buffer timer). These are separate input signals — edge-triggered vs level-triggered.

### Map stored as string array
The tile map is stored as an array of strings rather than a 2D boolean array in source. This makes the level layout visually obvious in code — you can read the ASCII art directly. It's parsed into a 2D boolean grid at load time for efficient collision checks.

### eval() in tuning panel
The tuning panel uses `eval()` to read/write global variables by name. This is acceptable here because the variable names are hardcoded string literals in the `tuningParams` array — no user input reaches eval. The alternative (a Map or object wrapper) would require refactoring all physics code to use property access instead of bare variable names.

## Physics Model

Frame-rate independent via `dt` multiplication everywhere. The update order is:

```
processInput → gravity → input acceleration → friction → speed clamp → X move + X collision → Y move + Y collision → reversal juice → jump timers (landing juice) → jump execution (jump juice) → drift (Feral shake) → juice update (lerp, particles, shake decay) → render (shake → tiles → particles → zombie squash/stretch → arrow → restore → vignette)
```

This order ensures:
- Gravity is applied before collision (so landing detection works)
- X is moved and resolved before Y (prevents "wall eats jump" — see collision section)
- Collision happens after each axis move (so the zombie is never rendered inside a tile)
- Jump timers update after collision (so grounded state is fresh)
- Jump execution is last (so the zombie can jump the same frame they land)

## Drift Model (Smooth System)

Drift uses a **smooth curve** rather than tier-based constants. The balance report (docs/SANITY_BALANCE_REPORT.md) identified four problems with the old tier system: a "Lucid free zone" (sanity 7-12 = zero drift cost), negligible Slipping drift, a "Feral cliff" (28× cost increase at the tier boundary), and invisible same-direction drift (speed clamp eats pure velocity impulses).

### Impulse curve
`getDriftImpulse()` returns `DRIFT_BASE_IMPULSE * t²` where t = `getSanityT()`. The squared exponent means drift starts very gently at high sanity (~1 px/s at sanity 11) and accelerates into devastating at low sanity (500 px/s at sanity 0). Below 5 px/s, drift is skipped entirely.

### Interval
`getDriftInterval()` returns a linear interpolation from `DRIFT_MAX_INTERVAL` (5.0s) at sanity 12 to `DRIFT_MIN_INTERVAL` (0.8s) at sanity 0. The actual interval has ±30% random variance around this value.

### Position nudge + velocity residue
The old system applied drift as a pure velocity impulse. Same-direction drift was invisible because the speed clamp fired before the next movement frame. The new system uses two channels:
1. **Position nudge**: `zombie.x += direction * impulse * DRIFT_NUDGE_SCALE` — instant displacement that bypasses speed clamp. Both directions create observable movement.
2. **Velocity residue**: `zombie.vx += direction * impulse * DRIFT_VELOCITY_RESIDUE` — small momentum change for physical feel, corrected by acceleration/friction over subsequent frames.

### Drift collision resolution
Position nudge can push the zombie into walls. Collision is resolved **inline** using the drift direction (not `zombie.vx`, which may point the other way). This avoids the `resolveHorizontal()` function, which uses `zombie.vx` for direction and would check the wrong side for counter-direction nudges.

### Airborne amplification
Drift impulse is multiplied by `DRIFT_AIRBORNE_MULT` (2.0) while the zombie is not grounded. This makes drift terrifying during jumps (where it can kill you) and ignorable on flat ground (where it just nudges you).

### Visual feedback
When drift fires, `drift.flashTimer` is set to 0.15s. The renderer draws the zombie as a white flash that fades over that duration, with a tier-colored outline to maintain visibility. Screen shake scales with impulse magnitude (`min(4, impulse / 100)`). Directional vignette opacity scales with `drift.lastImpulse / 300`.

### Smooth input delay
Direction reversal delay now uses `getInputDelay()` which returns 0 above `INPUT_DELAY_ONSET_T` (sanity 6) and scales smoothly to `FERAL_INPUT_DELAY` (0.03s) at sanity 0. This replaces the old Feral-only hard gate. The visual (darken + jitter) triggers whenever `drift.inputDelayTimer > 0`, regardless of tier.

## Sanity Sliding Scales

ALL movement parameters interpolate smoothly across the full 0-12 sanity range, not stepped by tier. This avoids jarring threshold effects when crossing tier boundaries and eliminates dead zones where the slider does nothing (playtest finding: tier-stepped speed/accel/air control made the progression feel flat between boundaries).

The `getSanityT()` function provides the interpolation factor: `(12 - sanity) / 12`, yielding 0 at full Lucid and 1 at sanity 0. Each `get*()` helper uses the formula `BASE * (1 + t * (FERAL_*_MULT - 1))`.

- **FERAL_SPEED_MULT** (1.5): Max speed at sanity 0 is 1.5x base.
- **FERAL_ACCEL_MULT** (1.6): Acceleration at sanity 0 is 1.6x base. Combined with BASE_ACCELERATION bump (1800→2400), starts feel snappy at all sanities.
- **FERAL_AIR_CONTROL_MULT** (0.5): Air control at sanity 0 is 0.5x base (0.8 × 0.5 = 0.4).
- **FERAL_JUMP_MULT** (1.6): Jump velocity at sanity 0 is 1.6x base. Feral zombie can reach ~9.7 tiles vs Lucid's ~4 tiles.
- **FERAL_DECEL_MULT** (0.5): Deceleration at sanity 0 is 0.5x base. Feral zombie slides further, making precise platforming harder.
- **FERAL_GRAVITY_MULT** (1.15): Gravity at sanity 0 is 1.15x base. Tightens jump arcs at low sanity so Feral feels punchy, not floaty (playtest finding: low-sanity jumps felt balloon-like without this).

## Sanity Gauntlet Map Design

The 40x25 map is purpose-built to test the core design tension: **ability vs control**. Each tier of platforms is placed at a height that requires a specific sanity range to reach. The player must lower their own sanity (via the slider) to jump higher, but doing so degrades their control (more drift, less air control, input delay).

Row 21 includes a 10-tile horizontal gap that Lucid max range (~8 tiles) cannot cross, forcing Feral speed. This tests both vertical AND horizontal ability gating.

## Level Generator

### Mutable map variables
`MAP_COLS`, `MAP_ROWS`, and `tileMap` are `let` rather than `const` so the generator can resize the world at runtime. The original Sanity Gauntlet is preserved as `PRESET_GAUNTLET` and can be restored via the "Gauntlet" button.

### Physics envelope calculation
`calcJumpEnvelope(sanity)` derives reachable tile distances directly from the game's physics constants using kinematics: peak height = v²/(2g), horizontal range = maxSpeed × timeToPeak. All three values (jump velocity, gravity, max speed) use the same smooth sanity interpolation as the runtime helpers. This ensures generated platforms are reachable at the specified minimum sanity level without hardcoded tile-step values.

### Pacing-aware sequencer
When `PACING_ENABLED` is true, the generator follows Schell's interest curve instead of random archetype selection. `PACING_BEATS` maps section count (3/4/5) to beat sequences (hook, rising, valley, climax, resolution), each with a difficulty value 0-1. `BEAT_ARCHETYPE_WEIGHTS` defines weighted preferences per beat — hooks and climaxes favor canyon/tower (visually striking, challenging), valleys and resolutions favor open/corridor (restful). `selectArchetype()` performs weighted random selection with no back-to-back repeats.

Each archetype's `generateSection()` accepts a `difficulty` parameter that modulates its structural parameters: canyon gap widths and bridge heights scale up; tower step heights increase and platform widths decrease; open ground coverage probability drops from 75% to 43%; corridor ceilings lower and gaps reduce; staircase rise/run increase and platforms narrow. `PACING_DIFFICULTY_SCALE` (0.5-1.5) multiplies all beat difficulties, clamped to 1.0.

When `PACING_ENABLED` is false, the old random selection with tower/canyon guarantee is preserved exactly.

### Transition zones
`applyTransitionZone()` replaces the old 2-tile ground continuity at section boundaries. It places 3 tiles of ground at each boundary. When the difficulty delta between adjacent sections exceeds 0.3, it also places a perch platform 3 tiles above ground to help players prepare for the difficulty shift.

### Section-based terrain
Levels are divided into 3-5 sections, each assigned one of five archetypes: **canyon** (ground gap + bridge), **tower** (vertical stack), **open** (scattered platforms), **corridor** (low ceiling), **staircase** (ascending/descending chain). Each section gets its own ground treatment (gaps, raised bumps, partial coverage) modulated by its beat difficulty.

### Platform personality
Platform lengths are drawn from a weighted distribution: 25% tiny (1-2 tiles), 30% medium (3-4 tiles), 45% wide (5-7+ tiles, density-scaled). This prevents the uniform look of fixed-width platforms.

### Landmarks
1-2 landmarks per level add visual distinctiveness: tall pillars (1-wide, 3-6 tiles high), single floating tiles, and overhangs (platform with pillar hanging from one end).

### BFS reachability validation
After generation, a BFS flood from ground level checks every platform for reachability using the physics envelope as edge weights. Unreachable platforms are removed. This guarantees all remaining platforms can be reached at the specified minimum sanity.

### Extracted helpers
`resizeCanvas()` and `findAndSetSpawn()` were extracted from the init code so the generator can call them after building a new map. `resizeCanvas()` also updates the sanity bar's max-width to match the canvas.

### Seeded PRNG
`splitmix32` provides deterministic random numbers from a seed, making layouts reproducible. The UI exposes both a seed input and a "Random" button that picks a new seed.

## Variable Jump Height — One-Cut Fix

The variable jump height system halves upward velocity when the jump key is released (`zombie.vy *= 0.5`). The original implementation ran this check every frame, meaning rapid key release during spam-jumping would multiply the cut across several frames (0.5^3 = 12.5% of original velocity in 3 frames), producing tiny jumps. Fix: a `jumpCut` flag on the zombie state is set to `false` when a jump starts and `true` after the first cut. The cut only fires when `jumpCut` is false, ensuring exactly one 50% velocity reduction per jump regardless of input timing.

## Juice Systems Architecture

### Squash/stretch
`zombie.scaleX` and `zombie.scaleY` are set by game events (jump, land, reversal) and lerp back to 1.0 each frame via `updateJuice(dt)`. The render function applies the scale transform anchored at the zombie's bottom-center (`zCenterX`, `zBottomY`) so feet stay planted on the ground. Without this anchor, landing squash would push the zombie into the floor.

### Particles
A flat array of particle objects (< 30 at any time). Each particle has position, velocity, life/maxLife, size, and color. Updated in `updateJuice`: position moves by velocity, vy gains `PARTICLE_GRAVITY * dt`, life ticks down. Dead particles are spliced via reverse iteration (cache-friendly, avoids skip-on-splice bug). Rendered as circles with alpha = `life/maxLife * 0.7` and size = `baseSize * alpha`.

### Screen shake
`camera.shakeX/Y` offsets are applied via `ctx.translate()` at the start of `render()`, inside a `save/restore` pair. Shake intensity decays linearly over `shakeDuration`. When multiple shakes overlap, the stronger intensity wins (Vlambeer's dampened stacking — adding would escalate to nausea). Timer resets on new shake so the decay restarts.

Feral drift shake is **directional**: `camera.shakeBiasX` adds a constant pull toward the drift direction on top of the random component. This makes the world "lurch" in the direction the zombie was pushed. Directional bias follows the winning intensity — if a stronger non-directional shake (landing) overwrites a weaker directional shake (drift), bias resets to 0.

### Input delay visual
During `drift.inputDelayTimer > 0`, the render function applies two effects: (1) ±1px random jitter via an extra `ctx.translate()` inside the squash/stretch transform, and (2) a 20% black overlay (`rgba(0,0,0,0.2)`) drawn on top of the zombie fill but before the "Z" label and tier outline. No longer gated to Feral tier — triggers at any sanity where `getInputDelay() > 0` (from sanity 6 down).

### Drift directional vignette
`renderDriftVignette()` draws a one-sided linear gradient from the drift-direction edge to the canvas center. Uses `drift.flashDirection` (set when drift fires) and `drift.flashTimer` (shared with the zombie white flash) for timing. Opacity scales with `drift.lastImpulse / 300` (skips if < 50 px/s). Uses current tier color instead of hardcoded Feral orange. Drawn in screen-space after both the shake and sanity vignette restores.

### Speed lines
`renderSpeedLines()` draws 1-2 faint horizontal lines behind the zombie when `|zombie.vx| > getMaxSpeed() * SPEED_LINE_THRESHOLD`. Lines trail opposite to movement direction, starting at the zombie's trailing edge. Line count escalates from 1 to 2 at 50% intensity. Y positions have random offsets ±5-15px from zombie center to create per-frame shimmer. Opacity ramps with `intensity * (0.6-1.0 random)` so lines fade in gradually rather than popping. Color matches current tier. Drawn inside shake transform, behind zombie, after particles.

### Sanity vignette
`renderVignette()` draws a radial gradient from transparent center to tier-colored edges. Opacity scales with `getSanityT() * VIGNETTE_MAX_OPACITY`. Drawn AFTER the screen shake `ctx.restore()` so the vignette stays fixed in screen-space — the world shakes but the vignette doesn't.

### Render save/restore nesting
The render function has two nested save/restore pairs:
1. **Outer** (lines 1537/1613): screen shake offset — all world rendering is shaken
2. **Inner** (lines 1576/1599): zombie squash/stretch — only the zombie rectangle is scaled

The direction indicator arrow is between the two restores (inside shake, outside squash/stretch). The vignette is after both restores (screen-space).

## Ground Detection Fix

The zombie's height is 48px (1.5 tiles). Using `zombie.y + zombie.height - 1` for ground detection caused grounded state flicker because 48px doesn't align to the 32px tile grid. Fix: use `zombie.y + zombie.height` (the actual feet pixel) for collision. This ensures consistent ground detection regardless of entity height.
