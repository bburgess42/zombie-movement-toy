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

## Playtesting Tasks
- [ ] Playtest Lucid tier — does it feel safe and precise?
- [ ] Playtest Feral tier — does it feel fast but dangerous?
- [ ] Evaluate jump arc satisfaction
- [ ] Test momentum feel (too slidey? too sticky?)
- [ ] Evaluate input drift (annoying vs. interestingly scary?)
- [ ] Record tuned constant values

## Potential Refinements (only if movement feel needs it)
- [ ] Adjust base constants after playtesting
- [ ] Fine-tune drift intervals/impulse magnitudes
- [ ] Evaluate Feral input delay duration
- [ ] Consider adding visual feedback for drift impulses
- [ ] Test platform edge collision behavior

## Upcoming
- [ ] Level generator (procedural physics-aware platform generation with tunable parameters)
  - Seeded PRNG for reproducible layouts
  - Density, min-sanity, width/height sliders
  - BFS reachability validation
  - Replaces need for hand-building test maps

## DO NOT BUILD
- Enemies, combat, AI, eating mechanic
- Civilians, hunters, monsters
- Sanity drain over time
- Win/lose conditions
- Sound, sprites, animations, save/load
