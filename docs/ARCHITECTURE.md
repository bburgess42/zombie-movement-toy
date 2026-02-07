# Zombie Movement Toy — Architecture

**Last updated:** Feb 2026 — Level Generator build

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
9. **Collision Detection** — AABB vs tile grid (Y-first resolution)
10. **Jump System** — Coyote time, jump buffer, variable height
11. **Input Drift** — Random impulses at Slipping/Feral tiers
12. **Rendering** — Canvas drawing (tiles, zombie, direction indicator)
13. **UI — Debug Panel** — Real-time movement diagnostics
14. **UI — Sanity Slider** — Primary testing tool
15. **UI — Tuning Panel** — Live constant adjustment + level generator controls
16. **Game Loop** — requestAnimationFrame with dt capping
17. **Initialization** — Setup and loop start

## Key Architecture Decisions

### Constants as `let` instead of `const`
The tuning panel needs to write to movement constants at runtime. Using `let` allows direct mutation. The constants are still grouped at the top of the file in a clearly labeled section — they're "constants" in design intent, not in language enforcement.

### Y-before-X collision resolution
Resolving vertical collisions before horizontal prevents the common "stuck on platform edge" bug where horizontal resolution fights with gravity. When the zombie is falling and sliding into a platform corner, Y resolution places them on top first, then X resolution has nothing to resolve.

### Separate jumpPressed vs jumpHeld
Variable jump height requires knowing when the jump key is *released* (to cut velocity), while jump buffering requires knowing when it was first *pressed* (to start the buffer timer). These are separate input signals — edge-triggered vs level-triggered.

### Map stored as string array
The tile map is stored as an array of strings rather than a 2D boolean array in source. This makes the level layout visually obvious in code — you can read the ASCII art directly. It's parsed into a 2D boolean grid at load time for efficient collision checks.

### eval() in tuning panel
The tuning panel uses `eval()` to read/write global variables by name. This is acceptable here because the variable names are hardcoded string literals in the `tuningParams` array — no user input reaches eval. The alternative (a Map or object wrapper) would require refactoring all physics code to use property access instead of bare variable names.

## Physics Model

Frame-rate independent via `dt` multiplication everywhere. The update order is:

```
gravity → input acceleration → friction → speed clamp → position update → collision → jump timers → jump execution
```

This order ensures:
- Gravity is applied before collision (so landing detection works)
- Collision happens after position update (so the zombie is never rendered inside a tile)
- Jump timers update after collision (so grounded state is fresh)
- Jump execution is last (so the zombie can jump the same frame they land)

## Drift Model

Input drift applies random horizontal velocity impulses on a timer. The timer resets to a random interval within the tier's range after each impulse. This creates irregular, unpredictable jolts rather than periodic wobble.

Feral tier additionally has an **input delay** on direction reversals: if the zombie is moving right and the player presses left, there's a 0.05s window where the left input is ignored. This simulates the zombie's body resisting direction changes — momentum commits you.

## Sanity Sliding Scales

Jump velocity and deceleration interpolate smoothly across the full 0-12 sanity range, not stepped by tier. This avoids jarring threshold effects when crossing tier boundaries.

- **FERAL_JUMP_MULT** (1.6): At sanity 0, jump velocity is 1.6x base. Linear interpolation from sanity 12 (1.0x) to sanity 0 (1.6x). Result: Feral zombie can reach platforms 9.7 tiles high vs Lucid's 4 tiles.
- **FERAL_DECEL_MULT** (0.5): At sanity 0, deceleration is 0.5x base. Result: Feral zombie slides further, making precise platforming harder.

The `getSanityT()` function provides the interpolation factor: `(12 - sanity) / 12`, yielding 0 at full Lucid and 1 at sanity 0.

## Sanity Gauntlet Map Design

The 40x25 map is purpose-built to test the core design tension: **ability vs control**. Each tier of platforms is placed at a height that requires a specific sanity range to reach. The player must lower their own sanity (via the slider) to jump higher, but doing so degrades their control (more drift, less air control, input delay).

Row 21 includes a 10-tile horizontal gap that Lucid max range (~8 tiles) cannot cross, forcing Feral speed. This tests both vertical AND horizontal ability gating.

## Level Generator

### Mutable map variables
`MAP_COLS`, `MAP_ROWS`, and `tileMap` are `let` rather than `const` so the generator can resize the world at runtime. The original Sanity Gauntlet is preserved as `PRESET_GAUNTLET` and can be restored via the "Gauntlet" button.

### Physics envelope calculation
`calcJumpEnvelope(sanity)` derives reachable tile distances directly from the game's physics constants using kinematics: peak height = v²/(2g), horizontal range = maxSpeed × timeToPeak. This ensures generated platforms are reachable at the specified minimum sanity level without hardcoded tile-step values.

### Section-based terrain
Levels are divided into 3-5 sections, each randomly assigned one of five archetypes: **canyon** (ground gap + bridge), **tower** (vertical stack), **open** (scattered platforms), **corridor** (low ceiling), **staircase** (ascending/descending chain). No back-to-back repeats, and at least one tower or canyon is guaranteed. Each section gets its own ground treatment (gaps, raised bumps, partial coverage). Section boundaries always have 2 tiles of ground on each side for walkability.

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

## Ground Detection Fix

The zombie's height is 48px (1.5 tiles). Using `zombie.y + zombie.height - 1` for ground detection caused grounded state flicker because 48px doesn't align to the 32px tile grid. Fix: use `zombie.y + zombie.height` (the actual feet pixel) for collision. This ensures consistent ground detection regardless of entity height.
