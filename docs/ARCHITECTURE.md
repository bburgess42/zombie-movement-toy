# Zombie Movement Toy — Architecture

**Last updated:** Feb 2026 — Initial build

## Design Philosophy

This is a **movement toy**, not a game. The architecture is intentionally minimal: a single HTML file with all code inline. This constraint is a feature — it makes the prototype trivially portable, shareable, and eliminable. No build tools, no dependencies, no server.

The goal is to answer one question: **Does this feel like the right foundation for a real-time zombie platformer?**

## File Structure

Everything lives in `index.html`. The code is organized into clearly separated sections in this order:

1. **Styles** — CSS for UI overlay elements
2. **Tunable Constants** — All movement parameters (the tuning panel reads/writes these)
3. **Tile Map** — Hardcoded test level as ASCII strings
4. **Game State** — Mutable runtime state (zombie position, velocity, input, drift)
5. **Sanity Tier Helpers** — Functions to get tier-adjusted movement values
6. **Input Handling** — Keyboard event listeners, key state tracking
7. **Physics Update** — Core movement loop (gravity, accel, friction, clamp, position)
8. **Collision Detection** — AABB vs tile grid (Y-first resolution)
9. **Jump System** — Coyote time, jump buffer, variable height
10. **Input Drift** — Random impulses at Slipping/Feral tiers
11. **Rendering** — Canvas drawing (tiles, zombie, direction indicator)
12. **UI — Debug Panel** — Real-time movement diagnostics
13. **UI — Sanity Slider** — Primary testing tool
14. **UI — Tuning Panel** — Live constant adjustment
15. **Game Loop** — requestAnimationFrame with dt capping
16. **Initialization** — Setup and loop start

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
