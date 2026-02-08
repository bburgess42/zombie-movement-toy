# Brains for Breakfast — v1 Scope Document

**Date:** Feb 2026
**Phase:** Pre-Production (Phase 0 — Scope Lock)
**Essential Experience:** "A gentleman zombie navigating a decaying city, where losing your mind makes you more powerful but less controllable."
**Skill Applied:** gamedev-production (Mid-Project Reassessment)

---

## 1. Existing Asset Inventory

Everything below currently exists in `zombie-movement-toy/index.html` (1615 lines, single-file HTML5 Canvas prototype).

| System | Completion | Status | Notes |
|--------|-----------|--------|-------|
| **Acceleration-based movement** | 100% | Tuned | Acceleration, deceleration, air control, max speed clamping. Frame-rate independent. |
| **Jump system** | 100% | Tuned | Variable height, coyote time (0.1s), jump buffering (0.1s). One-cut fix for spam-jump bug. |
| **Collision detection** | 100% | Solid | AABB vs tile grid, X-before-Y resolution, world boundary clamping. No clipping. |
| **Sanity tier system** | 100% | Tuned | Smooth interpolation across 0-12 range. Lucid/Slipping/Feral/Gone tiers for drift only. All movement params interpolate continuously. |
| **Input drift** | 100% | Tuned | Slipping: 100 px/s every 1.0-2.5s. Feral: 500 px/s every 0.8-1.8s. Airborne 2x multiplier. Visual flash feedback. |
| **Feral input delay** | 100% | Tuned | 0.03s direction reversal delay at Feral tier. |
| **Sanity sliding scales** | 100% | Tuned | Speed, accel, decel, air control, jump velocity, gravity all interpolate via getSanityT(). 6 FERAL_*_MULT constants. |
| **Tile map + Sanity Gauntlet** | 100% | Done | 40x25 hardcoded level testing vertical/horizontal ability gating. |
| **Level generator** | 100% | Done | Seeded PRNG, 5 section archetypes, BFS reachability, platform personality, landmarks. |
| **Sanity slider** | 100% | Done | Top bar, real-time, shows value + tier name. Primary testing tool. |
| **Debug panel** | 100% | Done | Velocity, grounded, coyote, buffer, air control, drift, jump vel, decel. |
| **Tuning panel** | 100% | Done | All constants adjustable via sliders + level generator controls. |
| **Gone overlay** | 100% | Done | "MIND LOST" text when sanity = 0. |
| **Input handling** | 100% | Done | WASD + Arrow Keys + Space. Blur handler prevents stuck keys. |

**Inventory summary:** The movement toy is complete and validated. All movement systems are tuned through multiple playtest passes. This is a strong foundation — the 80/20 of a platformer is movement feel, and it's proven.

### What Does NOT Exist

| System | Completion | Required for v1? |
|--------|-----------|-------------------|
| Enemies / Faction AI | 0% | Yes |
| Civilian consumption (sanity restore) | 0% | Yes |
| Level objectives (start/exit) | 0% | Yes |
| Multiple levels / progression | 0% | Yes |
| Player health / damage | 0% | Yes |
| Sanity drain over time | 0% | Yes |
| Art assets (sprites, tiles, backgrounds) | 0% | Yes |
| Audio / Music | 0% | Likely |
| HUD (game HUD, not debug) | 0% | Yes |
| Menus (title, pause, game over) | 0% | Yes |
| Save / Load | 0% | Maybe |
| Environmental narrative | 0% | Maybe |
| Civilian type variants | 0% | Maybe |
| Full three-faction ecology (cut — bounded civilian-threat ecology is Must Have) | 0% | No |

---

## 2. Feature Prioritization (MoSCoW)

The essential experience is tested against Lens #2 (Essential Experience): every feature must serve the fantasy of a zombie who gains power by losing control.

### MUST HAVE (Core)

Without these, the game concept does not function. Cut any of these and the essential experience breaks.

| Feature | Size | Justification |
|---------|------|---------------|
| **Movement system** (exists) | Done | The foundation. Proven and tuned. |
| **Sanity tier system** (exists) | Done | The core tension mechanic. Proven. |
| **Sanity drain over time** | S | The clock that drives urgency. Without drain, there's no pressure to eat. Paper prototype validated 0.90/round; real-time equivalent needed. |
| **Civilian entities (living food)** | L | Living NPCs that flee from the zombie and seek safety near threats. Eating a civilian restores sanity and triggers a death scream that alerts nearby threats. Replaces static brain pickups — adds pursuit, risk/reward (scream), and moral weight. Without food sources, sanity only goes down. |
| **Player health + damage** | S | The zombie can die. Without mortality, there's no Thrill of Danger. Paper prototype validated 5 HP. |
| **At least one threat type** | M | Something that hurts the player and guards civilians. Threats patrol near assigned civilians (leash behavior), chase the zombie on detection, and respond to death screams. One type is enough for v1. |
| **Level start + exit** | S | The objective: get from A to B. Without this, there's no goal. |
| **At least 3 playable levels** | L | Enough content to demonstrate the experience. Procedural generator already exists — this is level design using existing tools, not new tech. |
| **Basic HUD** | S | Sanity meter + health. The player needs to know their state. Current debug panel is designer-facing, not player-facing. |
| **Game over / win states** | S | Sanity = 0 → lose. HP = 0 → lose. Reach exit → win. Clear feedback. |
| **Basic title screen + restart** | S | Player needs to be able to start and restart. Minimal — not a menu system, just a way in. |
| **Art: player character** | M | Replace the green rectangle. The zombie needs visual identity. Sanity tiers should be visually distinct. |
| **Art: environment tileset** | M | Replace the colored blocks. The "decaying city" needs to read as a city. |
| **Art: threat sprites** | S | Whatever the threat type is needs to be visually identifiable. |

### SHOULD HAVE (Expected)

Players of platformers will expect these. Their absence would be noticed but the game technically works without them.

| Feature | Size | Justification |
|---------|------|---------------|
| **Two threat types** (e.g., patrollers + chasers) | M | One threat type works but feels thin. Two creates tactical variety — Schell's Lens #33 (Meaningful Choices): "Do I go through the patrollers or the chasers?" |
| **Pause menu** | S | Players expect to pause. |
| **SFX (critical sounds)** | M | Footsteps, jump, land, eat civilian, death scream, take damage, die. Movement feel is dramatically enhanced by audio (Swink: feedback layer). |
| **Music (at minimum: 1 track)** | M | A single adaptive or looping track. Silence in a platformer reads as broken, not atmospheric. |
| **Difficulty scaling across levels** | S | Levels should get harder. The generator supports min-sanity targeting — this is tuning, not new code. |
| **Juice: screen shake, landing squash** | S | The movement toy feels "dead" without feedback (per Phase 1 roadmap). A handful of juice effects close the feedback loop (Swink). |
| **Sanity visual effects** | S | Screen distortion, color shift, or vignette as sanity drops. Communicates state without HUD. Sells the "losing your mind" fantasy. |
| **Civilian + threat placement tuning** | S | Civilians and their threat guards need to be placed thoughtfully — balanced ratio of unguarded (safe) and guarded (risky) civilians to create routing decisions. |
| **Threat AI: enhanced behaviors** | M | Beyond basic guard/chase (which is now Must Have), add patrol pattern variety, group coordination, or sanity-dependent detection range. |
| **Death animation / respawn** | S | Feedback on dying. Currently nothing happens — the game just stops. |

### NICE TO HAVE (Bonus)

Enhance the experience. First candidates for cutting.

| Feature | Size | Cutting Framework |
|---------|------|-------------------|
| **Multiple civilian types with abilities** | L | 1. Does removing change core? **No** — single civilian type (sanity restore) delivers the core loop. 2. Will players notice? **Some will** — but first-time players won't miss what they never saw. 3. Simplify to 30%? **Yes** — one special civilian type (e.g., speed boost on eat) instead of a full system. 4. Post-launch? **Yes** — trivially addable. **Decision: Defer to v1.1. If time permits, add ONE special civilian type.** |
| **Three-faction ecology (full)** | XL | 1. Core? **No** — the bounded civilian-threat ecology (civilians + threat guarding + death scream) is now in Must Have. The FULL three-faction ecology (sanity-dependent reactions, inter-faction combat, population dynamics) remains cut. 2. Notice? **No**. 3. Simplify? **Done** — simplified to civilian-threat guard relationship without faction combat or sanity-dependent NPC reactions. 4. Post-launch? **Yes**. **Decision: Full ecology cut. Bounded civilian-threat ecology promoted to Must Have. See CIVILIAN_ECOLOGY_SPEC.md.** |
| **Environmental narrative** | M | 1. Core? **No** — the essential experience is about movement feel, not story. 2. Notice? **Depends** — some players care about "why am I here?" 3. Simplify? **Yes** — a few text signs or environmental details instead of a Player Knowledge Map. 4. Post-launch? **Yes**. **Decision: Defer. Add 2-3 environmental text hints if time permits, but no authored narrative system.** |
| **Save / Load** | M | 1. Core? **No** — with 3 short levels, session length is ~15-20 min. 2. Notice? **Maybe** — depends on level length. 3. Simplify? **Yes** — save level progress only (which level you're on), not mid-level state. 4. Post-launch? **Yes**. **Decision: Defer. localStorage level checkpoint only if levels exceed 5 min each.** |
| **Adaptive music** | L | 1. Core? **No** — a single track works. 2. Notice? **Yes, musicians will.** Most players won't. 3. Simplify? **Yes** — two layers (calm/tense) crossfaded by sanity level. 4. Post-launch? **Yes**. **Decision: If music exists at all, do the simple 2-layer version. Otherwise defer.** |
| **Level generator pacing upgrade** | M | 1. Core? **No** — the generator works now. 2. Notice? **Subtle** — some generated levels feel flat, but hand-tuning seeds covers this. 3. Simplify? **Yes** — curate 3-5 good seeds instead of fixing the algorithm. 4. Post-launch? **Yes**. **Decision: Defer algorithm work. Curate seeds for v1 levels.** |
| **Squash/stretch animation** | S | 1. Core? **No.** 2. Notice? **Subconsciously.** 3. Simplify? **Already simple.** 4. Post-launch? **Yes.** **Decision: Include if time permits. Very small effort, significant feel improvement.** |

### WON'T HAVE (Explicitly Cut for v1)

These are out of scope. Listing them prevents scope creep.

| Feature | Cutting Framework |
|---------|-------------------|
| **Multiplayer** | 1. Core? No. 2. Notice? No — single-player platformer. 3. Simplify? No. 4. Post-launch? Theoretically. **Cut. Not this game.** |
| **Crafting / inventory system** | 1. Core? No. 2. Notice? No. 3. Simplify? No. 4. Post-launch? Maybe. **Cut. No inventory. Brains are consumed on pickup.** |
| **Dialogue system** | 1. Core? No. 2. Notice? No — the zombie is a gentleman of action, not words. 3. Simplify? N/A. 4. Post-launch? Maybe. **Cut.** |
| **Map / minimap** | 1. Core? No — levels are small enough to see. 2. Notice? No. 3. Simplify? N/A. 4. Post-launch? Yes. **Cut.** |
| **Unlockable abilities / progression** | 1. Core? No — each level is self-contained. 2. Notice? Some players want progression. 3. Simplify? No — meta-progression is either there or not. 4. Post-launch? Yes. **Cut for v1. Each level starts fresh.** |
| **Controller support** | 1. Core? No. 2. Notice? Controller players will. 3. Simplify? Yes, but not for an HTML5 prototype. 4. Post-launch? Yes. **Cut. Keyboard only for v1.** |
| **Accessibility options** | 1. Core? No. 2. Notice? Players who need them will. 3. Simplify? Partially — colorblind mode is simple. 4. Post-launch? Yes. **Defer to v1.1. Note: this deserves priority in a post-launch patch.** |
| **Online leaderboards** | 1. Core? No. 2. Notice? No. 3. Simplify? No. 4. Post-launch? Yes. **Cut.** |
| **Boss fights** | 1. Core? No — the core tension is navigating while managing sanity, not combat encounters. 2. Notice? Some will expect a boss at the end. 3. Simplify? No — a bad boss is worse than no boss. 4. Post-launch? Yes. **Cut. The final level IS the boss — a gauntlet that demands Feral-level movement mastery.** |
| **Procedural generation for shipped levels** | 1. Core? No — the generator is a dev tool. 2. Notice? No. 3. Simplify? N/A. 4. Post-launch? Yes (daily challenge mode?). **Cut. Generator is used to AUTHOR levels, not to ship random levels.** |
| **NPC interaction / trading** | 1. Core? No. 2. Notice? No. 3. Simplify? N/A. 4. Post-launch? Maybe. **Cut.** |

---

## 3. v1 Feature List (Locked Scope)

### The v1 Game

**Brains for Breakfast v1** is a short-form browser platformer (~15-20 min) where you play a zombie navigating 3-5 levels of a decaying city. You eat civilians to maintain sanity. Lower sanity makes you faster and stronger but harder to control. Reach the exit of each level before your mind is gone.

### Core Systems (Must Ship)

1. **Movement** — Exists. Proven. No changes needed.
2. **Sanity** — Exists (tier system, smooth interpolation, drift). Add real-time drain.
3. **Health** — New. Simple HP counter. Take damage from threats. Die at 0.
4. **Civilians** — New. Living NPCs placed in levels. Flee from zombie, seek safety near threats. Eating restores sanity, triggers death scream alerting nearby threats.
5. **One threat type** — New. AI entity that damages the player. Guards civilians (leash/patrol), chases on detection, responds to death screams.
6. **Level structure** — Entrance, exit, civilian placements, threat placements. 3 levels minimum.
7. **HUD** — Sanity bar, health display. Minimal.
8. **Win/lose states** — Exit reached = level complete. HP 0 or Sanity 0 = game over.
9. **Title screen** — Start button. Nothing more.
10. **Art** — Player sprite (4 sanity states), environment tileset, threat sprite, civilian sprite. Geometric/stylized, not realistic.

### Expected Systems (Should Ship)

11. **Second threat type** — Different behavior from first. Creates routing choices.
12. **SFX** — Critical sounds only (jump, land, eat, damage, death).
13. **Music** — One looping track minimum.
14. **Juice** — Screen shake on damage, landing feedback, sanity visual effects.
15. **Pause** — Pause button/key.
16. **Difficulty scaling** — Later levels are harder (more threats, tighter platforms, lower min-sanity requirements).
17. **Death/respawn feedback** — Clear game-over screen with restart option.

### Stretch Goals (Include If Time Permits)

18. **One special civilian type** — E.g., "Adrenaline civilian" that grants temporary speed boost when eaten.
19. **Environmental text** — 2-3 signs/notes in levels that hint at backstory.
20. **Squash/stretch** — Landing and jump animation.
21. **Simple 2-layer music** — Calm layer + tense layer crossfaded by sanity.
22. **Save checkpoint** — Remember which level the player reached (localStorage).

---

## 4. Scope Size Estimate

Using relative sizing. Baseline: the movement toy took ~4 sessions to build and tune (all systems). One session = ~2-4 hours of effective dev time.

| Feature | Size | Notes |
|---------|------|-------|
| Sanity drain (real-time) | S | Timer + drain rate. Constants already defined in GDD. |
| Health system | S | HP variable, damage function, death check. |
| Civilian entities + death scream | L | Civilian entity with flee/seek/wander AI, eat collision, sanity restore, death scream event system, threat guard reassignment. |
| Threat type 1 (guard + patrol + chase + scream response) | L | State machine: guard patrol → detect → chase → respond to scream. Leash to civilian. Collision damage. Guard reassignment when civilian dies. |
| Level structure (3 levels) | L | Use generator to create base layouts, hand-tune, add civilian/threat placements. Entrance/exit markers. Level loading. |
| HUD | S | Sanity bar + HP display. Canvas overlay or DOM. |
| Win/lose states | S | State checks + overlay screens. |
| Title screen | S | DOM overlay with start button. |
| Art: player sprite | M | Gentleman zombie in 4 sanity states. AI-generated or geometric. |
| Art: tileset | M | City/building tiles. ~20 tile variants. |
| Art: threat + civilian sprites | S | 1-2 threat sprites + civilian sprite. |
| **Must Have Total** | **~4L** | (increased from ~3L due to civilian AI + threat guarding) |
| Threat type 2 | M | Second AI behavior variant. |
| SFX | M | Source/create ~10 sounds. Web Audio API integration. |
| Music | M | Source/create 1 track. Audio element or Web Audio. |
| Juice effects | S | Screen shake, landing squash, sanity vignette. |
| Pause menu | S | Key handler + overlay. |
| Difficulty scaling | S | Tuning pass on level parameters. |
| Death/respawn | S | Animation + restart flow. |
| **Should Have Total** | **~2L** | |
| **Grand Total (Must + Should)** | **~5L** | |

---

## 5. Risk Assessment (Lens #16: Risk Mitigation)

| Risk | Severity | Mitigation |
|------|----------|------------|
| **Art pipeline undefined** — No art has been created. No pipeline proven. | HIGH | Resolve in pre-production. Test AI generation + post-processing pipeline before committing to production. If pipeline fails, ship with geometric/stylized art (colored shapes with personality). |
| **Audio pipeline undefined** — No audio experience. | MEDIUM | Use free/CC0 sound libraries. If audio proves too costly, ship without music and with minimal SFX. The game works without audio. |
| **Scope creep from roadmap ambition** — The full roadmap has 9 phases and ~35 steps. v1 only needs Phases 0-2 partially. | HIGH | This scope document IS the mitigation. The locked feature list above is the v1. Everything else is v1.1+. |
| **Single-file architecture scaling** — 1615 lines is manageable. 5000+ may not be. | MEDIUM | Accept messy code if it works (production anti-pattern: "Rewriting Working Code"). Only refactor if bugs become frequent due to complexity. The single-file constraint is a feature for distribution. |
| **Movement feel regression** — Adding game systems on top of the tuned movement could break the feel. | MEDIUM | Regression test movement feel after every system addition. The tuning panel exists specifically for this. If feel degrades, roll back the last addition. |
| **Motivation loss** — Solo dev fatigue is real (Schreier: Blood, Sweat, and Pixels). | MEDIUM | Keep milestones small and playable. Every milestone should produce something you want to show someone. If passion dies, apply Lens #18 and cut back to the spark. |

---

## 6. What Phase Is This Project In?

**Current phase: Late Pre-Production.**

The movement toy is a proven core mechanic prototype. Per the Cerny Method:
- Core mechanic validated? **YES** — movement feel is tuned and playtested.
- Core loop proven? **NO** — no objectives, no threats, no civilians. The loop (navigate → eat → manage sanity → survive) is designed but not built.
- Art pipeline tested? **NO** — zero art assets exist.
- Tech approach proven? **MOSTLY** — HTML5 Canvas is proven. AI entity management, level loading, and audio integration are unproven.

**Next step:** Build the vertical slice (1 level with all Must Have systems at prototype quality) to prove the core loop. Then pass the production readiness gate.

---

## 7. Cutting Framework Summary

The 80/20 of Brains for Breakfast v1:

**The 20% that delivers 80% of the experience:**
- Movement + sanity (exists)
- Civilians + sanity drain + death scream (the core resource loop with risk/reward)
- One threat type that guards civilians (creates danger co-located with food)
- 3 levels with an exit (provides structure and completion)

**Everything else is enhancement.** A version with just these four pillars, using rectangles and no audio, would still answer the question: "Is this game fun?"

Ship the smallest thing that's fun. Then layer on Should Haves. Then stretch goals. If at any point the game stops being fun, something in the Must Haves is broken — fix that, don't add more features.

---

*Produced using the gamedev-production skill (Mid-Project Reassessment framework). Lenses applied: #2 (Essential Experience), #14 (Problem Statement), #16 (Risk Mitigation), #18 (Passion), #33 (Meaningful Choices), #45 (Economy — dev time as resource).*
