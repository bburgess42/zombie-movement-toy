# Zombie Movement Toy — TODO

## Completed
- [x] Project setup (git, docs, .gitignore)
- [x] Core movement system (acceleration, deceleration, gravity)
- [x] Jump system (variable height, coyote time, jump buffer)
- [x] Collision detection (AABB vs tile grid)
- [x] Sanity tier system (Lucid/Slipping/Feral/Gone)
- [x] Input drift (Slipping + Feral)
- [x] Feral input delay on direction reversal
- [x] Test level (40x15 hardcoded tile map)
- [x] Sanity slider (real-time, top bar)
- [x] Debug info panel (velocity, grounded, timers)
- [x] Tuning panel (collapsible, all constants)
- [x] Gone overlay ("MIND LOST")
- [x] WASD + Arrow key + Space input
- [x] Frame-rate independent physics (dt everywhere)
- [x] Sanity sliding scales (jump velocity + deceleration interpolate smoothly across sanity 0-12)
- [x] Sanity Gauntlet map (40x25, platforms require progressively lower sanity to reach)
- [x] Collision fix for 1.5-tile zombie height (ground detection uses feet pixel)
- [x] Level generator (procedural physics-aware platform generation)
  - Seeded PRNG (splitmix32) for reproducible layouts
  - Section-based terrain: 5 archetypes (canyon, tower, open, corridor, staircase)
  - Physics envelope from actual kinematics (v²/2g for peak height)
  - BFS reachability validation removes unreachable platforms
  - Platform personality: mixed sizes (tiny/medium/wide)
  - Ground variation per section, landmarks (pillars, floaters, overhangs)
  - UI: Width, Height, Density, Min Sanity, Seed sliders + Generate/Random/Gauntlet buttons
  - Gauntlet preset restores the original handcoded map

## Playtesting Tasks
- [x] Playtest Lucid tier — felt sluggish, fixed: BASE_ACCELERATION 1800→2400
- [x] Playtest Feral tier — reversal too slow, fixed: FERAL_INPUT_DELAY 0.05→0.03
- [x] Evaluate jump arc satisfaction — floaty at low sanity, fixed: FERAL_GRAVITY_MULT 1.15
- [x] Test momentum feel (too slidey? too sticky?) — feels good after tuning pass
- [x] Evaluate input drift (annoying vs. interestingly scary?) — retuned: bigger/rarer impulses, airborne amplification, visual flash
- [x] Evaluate tier progression — dead zones fixed: all params now smooth interpolation (no tier stepping)
- [x] Test edge cases — spam jump bug found and fixed (jumpCut flag prevents repeated velocity cut)
- [x] Record tuned constant values — see below

## Potential Refinements (only if movement feel needs it)
- [x] Adjust base constants after playtesting — BASE_ACCELERATION bumped, FERAL_INPUT_DELAY tightened
- [x] Fine-tune drift intervals/impulse magnitudes — Feral 500 px/s every 0.8-1.8s, Slipping 100 px/s every 1.0-2.5s
- [x] Evaluate Feral input delay duration — 0.05→0.03s
- [x] Consider adding visual feedback for drift impulses — white flash with tier-colored outline
- [x] Test platform edge collision behavior — no issues found
- [x] Convert tier-stepped params to smooth interpolation — speed, accel, air control, gravity all interpolate via getSanityT()
- [x] Fix variable jump height spam bug — jumpCut flag ensures one cut per jump

## Tuned Constants (post-playtest)

| Constant | Value | Notes |
|----------|-------|-------|
| **Base Movement** | | |
| BASE_MAX_SPEED | 300 px/s | Unchanged |
| BASE_ACCELERATION | 2400 px/s² | Was 1800 — snappier starts/reversals |
| BASE_DECELERATION | 3200 px/s² | Unchanged |
| BASE_AIR_CONTROL | 0.8 | Unchanged |
| **Jump** | | |
| JUMP_VELOCITY | -600 px/s | Unchanged |
| GRAVITY | 1400 px/s² | Unchanged (but scaled by sanity at runtime) |
| COYOTE_TIME | 0.1s | Unchanged |
| JUMP_BUFFER_TIME | 0.1s | Unchanged |
| **Sanity Sliding Scales** | | All interpolate smoothly sanity 12→0 |
| FERAL_SPEED_MULT | 1.5 | Max speed at sanity 0 = 450 px/s |
| FERAL_ACCEL_MULT | 1.6 | Accel at sanity 0 = 3840 px/s² |
| FERAL_AIR_CONTROL_MULT | 0.5 | Air control at sanity 0 = 0.4 |
| FERAL_JUMP_MULT | 1.6 | Jump vel at sanity 0 = -960 px/s |
| FERAL_DECEL_MULT | 0.5 | Decel at sanity 0 = 1600 px/s² |
| FERAL_GRAVITY_MULT | 1.15 | Gravity at sanity 0 = 1610 px/s² |
| **Drift** | | |
| SLIPPING_DRIFT_IMPULSE | 100 px/s | Every 1.0-2.5s |
| FERAL_DRIFT_IMPULSE | 500 px/s | Every 0.8-1.8s |
| DRIFT_AIRBORNE_MULT | 2.0 | Impulse doubled while airborne |
| FERAL_INPUT_DELAY | 0.03s | Was 0.05 — direction reversal delay |

## Upcoming
- (nothing currently planned)

## DO NOT BUILD
- Enemies, combat, AI, eating mechanic
- Civilians, hunters, monsters
- Sanity drain over time
- Win/lose conditions
- Sound, sprites, animations, save/load
