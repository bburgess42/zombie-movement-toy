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
- [ ] Playtest Lucid tier — does it feel safe and precise?
- [ ] Playtest Feral tier — does it feel fast but dangerous?
- [ ] Evaluate jump arc satisfaction
- [ ] Test momentum feel (too slidey? too sticky?)
- [x] Evaluate input drift (annoying vs. interestingly scary?) — retuned: bigger/rarer impulses, airborne amplification, visual flash
- [ ] Record tuned constant values

## Potential Refinements (only if movement feel needs it)
- [ ] Adjust base constants after playtesting
- [x] Fine-tune drift intervals/impulse magnitudes — Feral 500 px/s every 0.8-1.8s, Slipping 100 px/s every 1.0-2.5s
- [ ] Evaluate Feral input delay duration
- [x] Consider adding visual feedback for drift impulses — white flash with tier-colored outline
- [ ] Test platform edge collision behavior

## Upcoming
- (nothing currently planned)

## DO NOT BUILD
- Enemies, combat, AI, eating mechanic
- Civilians, hunters, monsters
- Sanity drain over time
- Win/lose conditions
- Sound, sprites, animations, save/load
