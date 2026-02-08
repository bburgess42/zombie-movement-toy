# Zombie Movement Toy — Current State

**Version:** Sprint 2 complete (threat AI + game flow)
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
Sanity 0-12, controlled by slider. Tiers are cosmetic labels; ALL movement and drift parameters interpolate smoothly across the full range (no tier stepping).

| Tier | Range | Label Only |
|------|-------|------------|
| Lucid | 7-12 | Green tint |
| Slipping | 4-6.99 | Yellow tint |
| Feral | 0.01-3.99 | Orange/red tint |
| Gone | 0 | No movement, "MIND LOST" overlay |

### Input Drift (Smooth System)
Drift impulse and interval interpolate smoothly via `getSanityT()` with a t² curve — gentle at high sanity, devastating at low sanity. This replaces the old tier-based system that had a "Lucid free zone" (sanity 7-12 = zero cost) and a "Feral cliff" (28× cost increase at the 4→3.99 boundary).

- **Impulse**: `DRIFT_BASE_IMPULSE * t²` — ranges from ~1 px/s at sanity 11 to 500 px/s at sanity 0
- **Interval**: linear from 5.0s at sanity 12 to 0.8s at sanity 0
- **Position nudge**: instant displacement (`impulse * DRIFT_NUDGE_SCALE`) that bypasses speed clamp, making both same-direction and reverse drift visible
- **Velocity residue**: small momentum change (`impulse * DRIFT_VELOCITY_RESIDUE`) for physical feel
- **Airborne multiplier**: 2x impulse while not grounded (drift is scariest mid-jump)
- **Visual feedback**: zombie flashes white, directional vignette, impulse-scaled screen shake
- **Input delay**: smooth onset starting at sanity 6, scaling to 0.03s at sanity 0 (was Feral-only)

### Collision Detection
- AABB vs tile grid
- Separate horizontal/vertical resolution (X first, then Y)
- World boundary clamping
- No clipping through platforms

### Level Generator
Procedural physics-aware platform generation in the tuning panel. Calculates jump envelope from actual physics constants (JUMP_VELOCITY, GRAVITY, FERAL_JUMP_MULT) to place platforms that are reachable at the specified minimum sanity level.

- **Seeded PRNG** (splitmix32): reproducible layouts from seed
- **Pacing-aware sequencer** (Schell interest curve): sections follow a beat pattern based on section count:
  - 3 sections: hook(0.5) → climax(0.9) → resolution(0.3)
  - 4 sections: hook(0.5) → rising(0.6) → climax(0.9) → resolution(0.3)
  - 5 sections: hook(0.5) → rising(0.65) → valley(0.25) → climax(0.9) → resolution(0.3)
- **Weighted archetype selection**: each beat has preferred archetypes (hook/climax favor canyon/tower, valley/resolution favor open/corridor)
- **Difficulty modulation**: each archetype adjusts parameters based on beat difficulty (gap widths, platform sizes, ceiling height, ground coverage, step rise/run)
- **Transition zones**: 3-tile ground at section boundaries + perch platform when difficulty delta > 0.3
- **Pacing toggle**: `PACING_ENABLED` flag preserves old random behavior when disabled
- **Difficulty scale**: `PACING_DIFFICULTY_SCALE` (0.5-1.5) multiplies beat difficulties for easier/harder levels
- **Section-based terrain**: 3-5 sections per level, each an archetype (5 types):
  - **Canyon** — ground gap with bridge platform above, upper reward platform
  - **Tower** — vertical stack of alternating offset platforms, side approach
  - **Open** — partial ground with gaps/raised bumps, scattered platforms at various heights
  - **Corridor** — low ceiling with gaps, cramped lower path vs risk/reward upper path
  - **Staircase** — ascending or descending chain of step platforms
- **Platform personality**: mixed sizes (25% tiny 1-2 tiles, 30% medium 3-4, 45% wide 5-7+)
- **Ground variation**: per-section ground treatment (gaps, partial, raised), transition zones at boundaries
- **Landmarks**: 1-2 per level (tall pillars, single floating tiles, overhangs)
- **BFS reachability validation**: removes any platform unreachable via the physics envelope
- **Gauntlet preset**: one-click restore of the hardcoded Sanity Gauntlet map

Generator UI: Width (20-60), Height (15-35), Density (0.1-0.9), Min Sanity (1-12), Seed, Pacing toggle, Difficulty Scale (0.5-1.5)

### Juice / Feedback Systems (Phase A)
Four feedback systems that close Swink's simulation → feedback loop. Without these, movement is mechanically correct but feels "dead."

- **Squash/stretch**: Zombie scales on jump launch (1.15x wide, 0.80x tall), landing (scales with fall velocity), and high-speed direction reversal (0.90x compress). Anchored at bottom-center so feet stay planted. Lerps back to 1.0 via exponential approach (SCALE_LERP_RATE).
- **Dust particles**: Emitted on jump, landing, and direction reversal. Count/spread scale with intensity. Gravity-affected, alpha-fading, size-decaying. Flat array, < 30 particles at any time.
- **Screen shake**: Camera offset with exponential decay. Triggered on heavy landings (vy > 400 px/s) and Feral drift impulses. Stacking uses the stronger intensity (Vlambeer's dampened stacking). Feral drift shake is **directional** — biased toward the drift direction so the world lurches.
- **Sanity vignette**: Radial gradient overlay that intensifies as sanity drops. Tier-colored edges, transparent center. Drawn in screen-space (not affected by shake). Max opacity 0.25 at sanity 0.
- **Input delay visual**: During the direction reversal block, zombie darkens 20% and jitters ±1px per frame. Now triggers at any sanity where delay > 0 (from sanity 6 down). Communicates "body is fighting you" instead of feeling like input lag.
- **Drift directional vignette**: One-sided tier-colored flash on the side drift pushed toward. Fades over 0.15s. Opacity scales with impulse strength (skips if < 50 px/s). Communicates drift direction to the player.
- **Speed lines** (Phase C): 1-2 faint horizontal lines trail behind the zombie when |vx| > 80% of max speed. Tier-colored, opacity and count ramp with speed. Shimmer per frame via random y-offsets. Drawn behind zombie, inside shake transform.

### GameState + Phase Machine (Sprint 1)
Centralized `GameState` object (Strangler Fig migration). References existing globals — bare `zombie` and `sanity` remain as sources of truth during Sprint 1. New systems read/write through GameState.

- **Phases**: PLAYING, GAME_OVER_SANITY ("MIND LOST"), GAME_OVER_HP ("GAME OVER"), LEVEL_COMPLETE ("LEVEL COMPLETE")
- **Game loop gate**: Update systems only run when `GameState.phase === 'PLAYING'`
- **Restart**: R key in any game-over state calls `resetGameState()`
- **Stats tracking**: `timeElapsed`, `civiliansEaten`, `damagesTaken`, `threatsPounced`

### Health System (Sprint 1)
- `zombie.hp` / `zombie.maxHP` (default 5)
- `damageZombie(state, amount)` — iframes check, reduce HP, set invincibleTimer, shake
- `updateHealth(state, dt)` — decrements invincibleTimer
- HP=0 → GAME_OVER_HP phase (orange overlay, distinct from red MIND LOST)
- HP checked before sanity in phase transitions (edge case #3)
- Iframe flash: alternating alpha (1.0/0.3) every 0.1s during invincibility

### Sanity Drain (Sprint 1)
- `SANITY_DRAIN_RATE` (0.15/s) — continuous drain when enabled
- "Auto Drain" checkbox in sanity bar (OFF by default = toy mode)
- `updateSanityDrain(state, dt)` — decrements bare `sanity` global, syncs slider
- Sanity=0 → GAME_OVER_SANITY phase

### Civilian System (Sprint 1)
- `createCivilian(x, y)` — factory returning flat data object
- 5 civilians placed on Gauntlet platforms (2 ground, 1 Lucid, 2 Slipping)
- AI priority: FLEE zombie (within 96px, 60 px/s) > WANDER (30 px/s, random direction)
- Edge avoidance: ground-ahead + wall-ahead checks, reverse at edges
- Rendered as cyan rectangles ("C" label), orange when fleeing

### Eat Collision + Death Scream (Sprint 1)
- `aabbOverlap(a, b)` — shared AABB utility
- `checkEatCollision(state)` — overlap → civilian dies, sanity +4 (cap 12)
- `broadcastScream(state, x, y)` — alerts all threats within `SCREAM_ALERT_RANGE`, shake
- Guard reassignment on civilian death (`assignGuards` called after eat)
- Particle burst + screen shake on eat
- Eating at full sanity wastes restore but still triggers scream

### Threat System (Sprint 2)
- `createThreat(x, y)` — factory returning flat data object with AI state
- 3 threats placed on valid standing positions, spread across map
- `areSamePlatform(a, b)` — checks continuous ground between entities
- `assignGuards(state)` — each threat guards nearest same-platform civilian (prefer < 2 guards)
- **AI States**:
  - **GUARD_PATROL**: patrol within `THREAT_GUARD_RANGE` of guarded civilian (or `THREAT_PATROL_RANGE` of patrolCenter if no civilian). Follows civilian as it moves.
  - **CHASE**: zombie within `THREAT_DETECTION_RANGE` AND threat within `THREAT_LEASH_DISTANCE` of anchor. Moves at `THREAT_CHASE_SPEED`. Breaks chase when zombie exceeds leash.
  - **RESPONDING_TO_SCREAM**: highest priority. Move to alertTarget at chase speed. Timer-based, returns to GUARD_PATROL on expiry or arrival.
- Edge avoidance: ground-ahead + wall-ahead checks (same pattern as civilians)
- Rendered as colored rectangles ("!" label): red (patrol), magenta (chase), yellow (scream response). Cyan outline when guarding.

### Threat Collision + Pounce (Sprint 2)
- `checkThreatCollision(state)` — AABB overlap zombie ↔ threats
- **Pounce mechanic**: at Feral (sanity < `POUNCE_SANITY_THRESHOLD`=4), touching a threat kills it — no HP loss, particle burst, strong shake
- **Damage**: at Lucid/Slipping, touching a threat calls `damageZombie` (respects iframes)
- `threatsPounced` stat tracked

### Level Exit + Level Complete (Sprint 2)
- `findAndSetExit()` — scans rightmost columns for valid 2-tile-high open space above ground
- `checkExitZone(state)` — AABB overlap zombie ↔ exit zone
- `renderExitZone()` — gold/yellow rectangle with pulsing border + "EXIT" label, drawn after tiles but before entities
- **LEVEL_COMPLETE phase**: green overlay with stats (time, civilians eaten, HP, threats pounced). R to restart.
- Time tracking: `GameState.stats.timeElapsed += dt` in game loop
- Exit placed on all map change events (init, reset, generate, Gauntlet)

### UI
- **Sanity slider**: top bar, range 0-12, real-time, shows value + tier name
- **Auto Drain checkbox**: next to slider, toggles continuous sanity drain
- **Debug panel**: top-right overlay, monospace, shows phase/HP/civilians/threats + velocity/grounded/coyote/buffer/air control/drift
- **Tuning panel**: collapsible left panel with sliders for all movement constants + juice params + level generator + sanity system + health + civilian AI + threat AI
- **Gone overlay**: "MIND LOST" text on GAME_OVER_SANITY phase
- **HP overlay**: "GAME OVER" text (orange) on GAME_OVER_HP phase
- **Level complete overlay**: "LEVEL COMPLETE" text (green) with stats on LEVEL_COMPLETE phase
- **Controls hint**: bottom text showing key bindings (including R to restart, reach EXIT to win)

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
| BASE_ACCELERATION | 2800 | px/s² |
| BASE_DECELERATION | 3200 | px/s² |
| BASE_AIR_CONTROL | 0.8 | multiplier |
| JUMP_VELOCITY | -650 | px/s |
| GRAVITY | 1400 | px/s² |
| COYOTE_TIME | 0.1 | seconds |
| JUMP_BUFFER_TIME | 0.1 | seconds |
| TILE_SIZE | 32 | px |
| FERAL_SPEED_MULT | 1.7 | multiplier (sanity-interpolated) |
| FERAL_ACCEL_MULT | 1.6 | multiplier (sanity-interpolated) |
| FERAL_AIR_CONTROL_MULT | 0.5 | multiplier (sanity-interpolated) |
| FERAL_JUMP_MULT | 1.6 | multiplier (sanity-interpolated) |
| FERAL_DECEL_MULT | 0.5 | multiplier (sanity-interpolated) |
| FERAL_GRAVITY_MULT | 1.15 | multiplier (sanity-interpolated) |
| DRIFT_BASE_IMPULSE | 500 | px/s equivalent (max at sanity 0) |
| DRIFT_MIN_INTERVAL | 1.0 | seconds (shortest, at sanity 0) |
| DRIFT_MAX_INTERVAL | 5.0 | seconds (longest, at sanity 12) |
| DRIFT_IMPULSE_EXPONENT | 2 | curve shape (t² = gentle start) |
| DRIFT_AIRBORNE_MULT | 2.0 | multiplier (while not grounded) |
| DRIFT_NUDGE_SCALE | 0.1 | px per px/s of impulse |
| DRIFT_VELOCITY_RESIDUE | 0.3 | fraction of impulse as velocity |
| FERAL_INPUT_DELAY | 0.03 | seconds (max at sanity 0) |
| INPUT_DELAY_ONSET_T | 0.5 | getSanityT() threshold (sanity 6) |
| SCALE_LERP_RATE | 12 | per second (squash/stretch return speed) |
| JUMP_SQUASH_X | 1.15 | multiplier (horizontal stretch on jump) |
| JUMP_SQUASH_Y | 0.80 | multiplier (vertical compress on jump) |
| LAND_STRETCH_MAX_Y | 0.20 | added to 1.0 on landing |
| LAND_SQUASH_MAX_X | 0.12 | subtracted from 1.0 on landing |
| LAND_VY_SCALE | 3000 | vy divisor for landing intensity |
| LANDING_SHAKE_MIN_VY | 400 | px/s (min vy to trigger landing shake) |
| LANDING_SHAKE_INTENSITY | 4 | pixels (max shake for heaviest landing) |
| REVERSAL_SQUASH_X | 0.90 | multiplier (compress on direction reversal) |
| REVERSAL_MIN_SPEED | 200 | px/s (min speed before reversal for juice) |
| PARTICLE_GRAVITY | 100 | px/s² (downward on dust particles) |
| VIGNETTE_MAX_OPACITY | 0.25 | max vignette opacity at sanity 0 |
| SPEED_LINE_THRESHOLD | 0.8 | fraction of max speed to show lines |
| SPEED_LINE_OPACITY | 0.3 | max opacity of speed lines |
| PACING_ENABLED | true | toggle (pacing vs random) |
| PACING_DIFFICULTY_SCALE | 1.0 | multiplier (0.5-1.5) |
| MAP_COLS | 40 (default) | tiles (mutable, 20-60) |
| MAP_ROWS | 25 (default) | tiles (mutable, 15-35) |
| **Sanity System** | | |
| SANITY_DRAIN_RATE | 0.15 | sanity/s (continuous drain) |
| SANITY_PER_EAT | 4 | sanity restored per civilian |
| SCREAM_ALERT_RANGE | 128 | px (death scream radius) |
| SCREAM_ALERT_DURATION | 3.0 | seconds (Sprint 2) |
| **Health** | | |
| ZOMBIE_MAX_HP | 5 | hit points |
| ZOMBIE_START_HP | 5 | hit points |
| INVINCIBILITY_DURATION | 0.75 | seconds (iframes) |
| **Civilian AI** | | |
| CIVILIAN_FLEE_SPEED | 60 | px/s |
| CIVILIAN_FLEE_RANGE | 96 | px (flee trigger distance) |
| CIVILIAN_SEEK_RANGE | 128 | px (placeholder for Sprint 2) |
| CIVILIAN_WANDER_SPEED | 30 | px/s |
| **Threat AI** | | |
| THREAT_SPEED | 120 | px/s (patrol) |
| THREAT_CHASE_SPEED | 200 | px/s (chase) |
| THREAT_DAMAGE | 1 | HP per hit |
| THREAT_DETECTION_RANGE | 160 | px (aggro range) |
| THREAT_PATROL_RANGE | 128 | px (default patrol, no civilian) |
| THREAT_GUARD_RANGE | 64 | px (patrol near guarded civilian) |
| THREAT_LEASH_DISTANCE | 160 | px (max chase from anchor) |
| POUNCE_SANITY_THRESHOLD | 4 | sanity below which pounce active |

## File Structure

```
index.html          — The entire prototype
docs/
  CURRENT_STATE.md  — This file
  TODO.md           — Task tracking
  ARCHITECTURE.md   — Architecture decisions
  GDD.md            — Game design document (from paper prototype)
  SCOPE_V1.md       — MoSCoW scope document
  MILESTONE_PLAN.md — 4 milestones
  SPRINT_BACKLOG.md — Sprint tasks for Milestone 1
  FEEDBACK_MATRIX.md — Movement feedback audit
  SANITY_BALANCE_REPORT.md — Sanity system balance analysis
  CORE_LOOP_SPEC.md — Core game loop mechanic spec
  CIVILIAN_ECOLOGY_SPEC.md — Civilian-threat ecology design
  CIVILIAN_VARIANTS_SPEC.md — Civilian variant types (stretch goal)
  ARCHITECTURE_REFACTOR.md — Architecture refactor plan (GameState, entities, phases, migration)
handoff/            — Inter-window communication (gitignored)
```
