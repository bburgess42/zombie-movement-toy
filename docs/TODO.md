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
| **Drift (Smooth)** | | |
| DRIFT_BASE_IMPULSE | 500 px/s | Max impulse at sanity 0 (t² curve) |
| DRIFT_IMPULSE_EXPONENT | 2 | Curve shape: gentle start, sharp end |
| DRIFT_MIN_INTERVAL | 0.8s | Shortest interval (at sanity 0) |
| DRIFT_MAX_INTERVAL | 5.0s | Longest interval (at sanity 12) |
| DRIFT_AIRBORNE_MULT | 2.0 | Impulse doubled while airborne |
| DRIFT_NUDGE_SCALE | 0.1 | Position nudge per px/s impulse |
| DRIFT_VELOCITY_RESIDUE | 0.3 | Velocity fraction of impulse |
| FERAL_INPUT_DELAY | 0.03s | Max delay at sanity 0 |
| INPUT_DELAY_ONSET_T | 0.5 | Delay starts at sanity 6 |

## Phase 1: Movement Feel Pass
- [x] Step 1.1: Feedback matrix audit (docs/FEEDBACK_MATRIX.md)
- [x] Phase A: Critical juice implementation
  - Squash/stretch (jump launch, landing, direction reversal)
  - Dust particles (jump, land, reversal — count/spread scale with intensity)
  - Screen shake (heavy landings, Feral drift — exponential decay)
  - Sanity vignette (radial gradient, tier-colored, scales with getSanityT())
  - Tuning panel: 6 new juice sliders (Scale Lerp, Jump Sqsh X/Y, Land Str Y, Land Shake, Vignette)
  - Bugs fixed: reversal detection timing (prevVxSign before accel), vignette in screen-space (outside shake transform)
- [x] Phase B: HIGH priority feedback
  - Feral input delay visual (darken 20% + ±1px jitter during 30ms direction block)
  - Directional drift camera shake (bias toward drift direction, random + constant pull)
  - Feral drift directional vignette (one-sided orange flash on drift side)
- [x] Phase C: MEDIUM priority feedback
  - Speed lines: 1-2 faint tier-colored horizontal lines trail behind zombie at >80% max speed
  - Landing intensity scaling: already done in Phase A (stretch/dust/shake scale with landVy)
- [ ] Phase D: Audio (deferred to Milestone 3)
- [x] Step 1.2: Sanity-movement balance analysis (docs/SANITY_BALANCE_REPORT.md)
  - Diagnostic: 4 of 5 checks FAIL — tradeoff curve is broken
  - Problem 1: Lucid free zone (sanity 7-12 = zero cost, +48% jump)
  - Problem 2: Slipping drift negligible (100 px/s can't reverse zombie at speed)
  - Problem 3: Feral cliff (28× cost increase for 5.6% gain crossing 4→3.99)
  - Problem 4: Same-direction drift invisible (speed clamp eats it)
  - Rec 1 (HIGH): Smooth drift via getSanityT()² curve (eliminate tiers for drift)
  - Rec 2 (MED): Fix drift asymmetry (position nudge + velocity residue)
  - Rec 3 (LOW): Smooth input delay onset
- [x] Implement balance recommendations (from SANITY_BALANCE_REPORT.md)
  - Rec 1: Smooth drift via getSanityT()² curve (replaces tier-based drift)
  - Rec 2: Position nudge + velocity residue (fixes same-direction drift invisibility)
  - Rec 3: Smooth input delay onset (starts at sanity 6, scales to 0.03s at sanity 0)
  - Bug fix: inline collision resolution in updateDrift (drift direction, not zombie.vx)
  - New tuning panel: 9 drift sliders replacing old 4 tier-based sliders
- [x] Step 1.3: Level generator pacing upgrade
  - Pacing-aware sequencer follows Schell's interest curve (hook → rising → valley → climax → resolution)
  - Weighted archetype selection per beat (canyon/tower for hooks/climaxes, open/corridor for valleys/resolution)
  - Difficulty modulation per archetype (gap widths, platform sizes, ceiling height, ground coverage, step dimensions)
  - Transition zones at section boundaries (3-tile ground + perch when difficulty delta > 0.3)
  - PACING_ENABLED toggle preserves old random behavior
  - PACING_DIFFICULTY_SCALE slider (0.5-1.5) for easier/harder levels
  - UI: Pacing checkbox + Difficulty Scale slider in tuning panel
- [x] Step 1.4: Toy validation playtest
  - Lucid: sluggish start + weak jump → bumped BASE_ACCELERATION 2400→2800, JUMP_VELOCITY -600→-650
  - Slipping: good drift balance (pass)
  - Feral: needed more speed, less drift → FERAL_SPEED_MULT 1.5→1.7, DRIFT_MIN_INTERVAL 0.8→1.0
  - Progression 12→0: initially steppy, resolved by tuning pass
  - Juice systems: all pass (squash/dust, drift VFX, speed lines)
  - Generator pacing: subtle arc, clean layouts (pass)
  - Verdict: ready for Sprint 1

## Phase 2: Core Loop Implementation
- [x] Step 2.1: Core loop mechanic spec (docs/CORE_LOOP_SPEC.md)
  - Mechanic spec following gamedev-design-doc template
  - All 8 template sections filled (no TBD placeholders)
  - 5 decision points with Lens #33 (Meaningful Choices) analysis
  - Parameters grounded in GDD paper prototype data
  - 13 edge cases prioritized (Must/Nice/Won't-handle)
  - Sanity economy model with timeline walkthrough
  - Implementation notes mapped to Sprint Backlog tasks 1.1-3.7

## Upcoming
- Sprint 1: Game systems foundation (sanity drain, health, brain pickups)

## Pre-Production Documents Created
- [x] docs/SCOPE_V1.md — MoSCoW scope document (10 Must/7 Should/5 Stretch/12 Won't)
- [x] docs/MILESTONE_PLAN.md — 4 milestones (Vertical Slice → Pre-Prod Gate → Content → Ship)
- [x] docs/SPRINT_BACKLOG.md — 3 sprints for Milestone 1
- [x] docs/FEEDBACK_MATRIX.md — Movement feedback audit (9 gaps identified)

## DO NOT BUILD
- Enemies, combat, AI, eating mechanic
- Civilians, hunters, monsters
- Sanity drain over time
- Win/lose conditions
- Sound, sprites, animations, save/load
