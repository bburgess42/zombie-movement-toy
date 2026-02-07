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

## DO NOT BUILD
- Enemies, combat, AI, eating mechanic
- Civilians, hunters, monsters
- Sanity drain over time
- Win/lose conditions
- Procedural generation, multiple levels
- Sound, sprites, animations, save/load
