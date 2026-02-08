# Zombie Movement Toy — Current State

**Version:** Playtest tuning pass
**Last updated:** Feb 2026

## Overview

Single-file HTML/JS movement prototype (`index.html`). Opens directly in any modern browser — no build step, no server, no dependencies.

## Systems Implemented

### Movement (acceleration-based)
- Horizontal acceleration/deceleration (not instant velocity)
- Gravity-based vertical physics
- Frame-rate independent (all physics multiplied by `dt`)
- Air control multiplier (reduced accel while airborne)
- Max speed clamping (sanity-interpolated)
- Gravity scaling via sanity (tighter arcs at low sanity)

### Jump System
- Variable jump height (releasing jump early halves upward velocity, once per jump)
- Coyote time: 0.1s grace period after leaving ground
- Jump buffering: 0.1s pre-landing input memory
- Supports Space, W, and ArrowUp

### Sanity Tier System
Sanity 0-12, controlled by slider. Tiers determine drift behavior; all other movement parameters interpolate smoothly across the full range (no tier stepping).

| Tier | Range | Drift Effects |
|------|-------|---------------|
| Lucid | 7-12 | No drift |
| Slipping | 4-6.99 | Subtle drift (±100 px/s every 1.0-2.5s) |
| Feral | 0.01-3.99 | Heavy drift (±500 px/s every 0.8-1.8s) + 0.03s direction reversal delay |
| Gone | 0 | No movement, "MIND LOST" overlay |

### Input Drift
Tuned for bigger, rarer impulses (scary events, not constant annoyance). Amplified while airborne so drift is terrifying mid-jump but ignorable on flat ground. Visual flash on drift fire so the player can distinguish "drift pushed me" from "I messed up."

- Slipping: random ±100 px/s impulse every 1.0-2.5s
- Feral: random ±500 px/s impulse every 0.8-1.8s + 0.03s direction reversal delay
- Airborne multiplier: 2x impulse while not grounded
- Visual feedback: zombie flashes white briefly when drift fires

### Collision Detection
- AABB vs tile grid
- Separate horizontal/vertical resolution (X first, then Y)
- World boundary clamping
- No clipping through platforms

### Level Generator
Procedural physics-aware platform generation in the tuning panel. Calculates jump envelope from actual physics constants (JUMP_VELOCITY, GRAVITY, FERAL_JUMP_MULT) to place platforms that are reachable at the specified minimum sanity level.

- **Seeded PRNG** (splitmix32): reproducible layouts from seed
- **Section-based terrain**: 3-5 sections per level, each a random archetype (no back-to-back repeats, at least one tower or canyon guaranteed):
  - **Canyon** — ground gap with bridge platform above, upper reward platform
  - **Tower** — vertical stack of alternating offset platforms, side approach
  - **Open** — partial ground with gaps/raised bumps, scattered platforms at various heights
  - **Corridor** — low ceiling with gaps, cramped lower path vs risk/reward upper path
  - **Staircase** — ascending or descending chain of step platforms
- **Platform personality**: mixed sizes (25% tiny 1-2 tiles, 30% medium 3-4, 45% wide 5-7+)
- **Ground variation**: per-section ground treatment (gaps, partial, raised), section boundaries get 2-tile ground for walkability
- **Landmarks**: 1-2 per level (tall pillars, single floating tiles, overhangs)
- **BFS reachability validation**: removes any platform unreachable via the physics envelope
- **Gauntlet preset**: one-click restore of the hardcoded Sanity Gauntlet map

Generator UI sliders: Width (20-60), Height (15-35), Density (0.1-0.9), Min Sanity (1-12), Seed

### UI
- **Sanity slider**: top bar, range 0-12, real-time, shows value + tier name
- **Debug panel**: top-right overlay, monospace, shows velocity/grounded/coyote/buffer/air control/drift
- **Tuning panel**: collapsible left panel with sliders for all movement constants + level generator controls
- **Gone overlay**: "MIND LOST" text when sanity = 0
- **Controls hint**: bottom text showing key bindings

### Sanity Sliding Scales
ALL movement parameters interpolate smoothly across the full 0-12 sanity range via `getSanityT()`. At sanity 12 the zombie gets base values; at sanity 0 it gets the FERAL_*_MULT × base. No tier stepping — every tick of the slider changes feel. This eliminates dead zones where the slider does nothing (playtest finding: tier-stepped values made progression feel flat).

- `FERAL_SPEED_MULT` (1.5): Max speed at sanity 0
- `FERAL_ACCEL_MULT` (1.6): Acceleration at sanity 0
- `FERAL_AIR_CONTROL_MULT` (0.5): Air control at sanity 0 (0.8 × 0.5 = 0.4)
- `FERAL_JUMP_MULT` (1.6): Jump strengthens as sanity drops
- `FERAL_DECEL_MULT` (0.5): Friction weakens as sanity drops (more slidey)
- `FERAL_GRAVITY_MULT` (1.15): Gravity increases as sanity drops (tighter arcs, less floaty)

### Test Level — Sanity Gauntlet
40x25 tile grid (1280x800 canvas). The "Sanity Gauntlet" is designed so platforms are impossible at full Lucid and require progressively lower sanity to reach, forcing the player to trade control for ability.

Jump peaks by sanity (FERAL_JUMP_MULT = 1.6):
- Sanity 12 (Lucid): ~129px (4 tiles)
- Sanity 9 (Lucid): ~170px (5.3 tiles)
- Sanity 5 (Slipping): ~234px (7.3 tiles)
- Sanity 3 (Feral): ~270px (8.4 tiles)
- Sanity 1 (Feral): ~309px (9.7 tiles)

Tier layout:
- Ground (row 24): runway, any sanity
- Lucid zone (row 21): 3 rows up, easy at sanity 12; includes 10-tile gap requiring Feral speed
- Slipping zone (row 16): 5 rows up, needs sanity <= 9
- Feral zone (row 8): 8 rows up, needs sanity <= 3
- Summit (row 2): 6 rows up, already Feral-only

## Constants (defaults)

| Constant | Value | Unit |
|----------|-------|------|
| BASE_MAX_SPEED | 300 | px/s |
| BASE_ACCELERATION | 2400 | px/s² |
| BASE_DECELERATION | 3200 | px/s² |
| BASE_AIR_CONTROL | 0.8 | multiplier |
| JUMP_VELOCITY | -600 | px/s |
| GRAVITY | 1400 | px/s² |
| COYOTE_TIME | 0.1 | seconds |
| JUMP_BUFFER_TIME | 0.1 | seconds |
| TILE_SIZE | 32 | px |
| FERAL_SPEED_MULT | 1.5 | multiplier (sanity-interpolated) |
| FERAL_ACCEL_MULT | 1.6 | multiplier (sanity-interpolated) |
| FERAL_AIR_CONTROL_MULT | 0.5 | multiplier (sanity-interpolated) |
| FERAL_JUMP_MULT | 1.6 | multiplier (sanity-interpolated) |
| FERAL_DECEL_MULT | 0.5 | multiplier (sanity-interpolated) |
| FERAL_GRAVITY_MULT | 1.15 | multiplier (sanity-interpolated) |
| SLIPPING_DRIFT_IMPULSE | 100 | px/s |
| FERAL_DRIFT_IMPULSE | 500 | px/s |
| DRIFT_AIRBORNE_MULT | 2.0 | multiplier (while not grounded) |
| MAP_COLS | 40 (default) | tiles (mutable, 20-60) |
| MAP_ROWS | 25 (default) | tiles (mutable, 15-35) |

## File Structure

```
index.html          — The entire prototype
docs/
  CURRENT_STATE.md  — This file
  TODO.md           — Task tracking
  ARCHITECTURE.md   — Architecture decisions
  GDD.md            — Game design document (from paper prototype)
handoff/            — Inter-window communication (gitignored)
```
