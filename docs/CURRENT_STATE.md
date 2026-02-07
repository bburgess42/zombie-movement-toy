# Zombie Movement Toy — Current State

**Version:** Initial build
**Last updated:** Feb 2026

## Overview

Single-file HTML/JS movement prototype (`index.html`). Opens directly in any modern browser — no build step, no server, no dependencies.

## Systems Implemented

### Movement (acceleration-based)
- Horizontal acceleration/deceleration (not instant velocity)
- Gravity-based vertical physics
- Frame-rate independent (all physics multiplied by `dt`)
- Air control multiplier (reduced accel while airborne)
- Max speed clamping per sanity tier

### Jump System
- Variable jump height (releasing jump early halves upward velocity)
- Coyote time: 0.1s grace period after leaving ground
- Jump buffering: 0.1s pre-landing input memory
- Supports Space, W, and ArrowUp

### Sanity Tier System
Sanity 0-12, controlled by slider. Tiers:

| Tier | Range | Effects |
|------|-------|---------|
| Lucid | 7-12 | Base values, crisp control |
| Slipping | 4-6.99 | 1.25x speed, 1.3x accel, 0.85x air control, subtle drift |
| Feral | 0.01-3.99 | 1.5x speed, 1.6x accel, 0.5x air control, heavy drift, input delay |
| Gone | 0 | No movement, "MIND LOST" overlay |

### Input Drift
- Slipping: random ±50 px/s impulse every 0.5-1.5s
- Feral: random ±150 px/s impulse every 0.3-0.8s + 0.05s direction reversal delay

### Collision Detection
- AABB vs tile grid
- Separate vertical/horizontal resolution (Y first)
- World boundary clamping
- No clipping through platforms

### UI
- **Sanity slider**: top bar, range 0-12, real-time, shows value + tier name
- **Debug panel**: top-right overlay, monospace, shows velocity/grounded/coyote/buffer/air control/drift
- **Tuning panel**: collapsible left panel with sliders for all movement constants
- **Gone overlay**: "MIND LOST" text when sanity = 0
- **Controls hint**: bottom text showing key bindings

### Test Level
40x15 tile grid (1280x480 canvas). Hardcoded layout with:
- Full-width ground floor
- Platforms at various heights
- Gaps of varying width
- Vertical shaft sections
- Long flat runway

## Constants (defaults)

| Constant | Value | Unit |
|----------|-------|------|
| BASE_MAX_SPEED | 300 | px/s |
| BASE_ACCELERATION | 1800 | px/s² |
| BASE_DECELERATION | 2400 | px/s² |
| BASE_AIR_CONTROL | 0.8 | multiplier |
| JUMP_VELOCITY | -500 | px/s |
| GRAVITY | 1400 | px/s² |
| COYOTE_TIME | 0.1 | seconds |
| JUMP_BUFFER_TIME | 0.1 | seconds |
| TILE_SIZE | 32 | px |

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
