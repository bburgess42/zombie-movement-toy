# Brains for Breakfast — Milestone Plan

**Last Updated:** Feb 2026
**Project:** Brains for Breakfast v1
**Scope Reference:** `docs/SCOPE_V1.md`
**Skill Applied:** gamedev-production
**Sizing:** Relative (S/M/L) — no hour estimates. Calibrated from Milestone 1.

---

## Milestone Overview

| # | Name | Status | Purpose | Playable Deliverable |
|---|------|--------|---------|---------------------|
| 1 | **Vertical Slice** | DONE | Prove the core loop is fun | 1 level, all core systems, rectangle art |
| 2 | **Polished Demo** | IN PROGRESS (nearly complete) | Prove the game looks, sounds, and feels like a real game | 1 hand-crafted demo level, pixel art, audio, full game flow |
| 3 | **Content + Ship** | NOT STARTED | Build remaining content and ship | 3-5 levels, second guard type, deploy to itch.io |

**Key change from original 4-milestone plan:** Milestones 2 (Pre-Production Gate) and 3 (Content + Identity) were merged and restructured. The original Milestone 2 focused on a second level and go/no-go gate; Milestone 3 focused on art, audio, and content. In practice, the project pulled art and audio forward into a "Polished Demo" milestone that proves the game's identity with one high-quality level rather than two placeholder levels. The original Milestone 4 (Polish + Ship) was absorbed into the new Milestone 3.

---

## Milestone 1: Vertical Slice --- DONE

**Purpose:** Prove the core loop — navigate, eat civilians, manage sanity, survive guards, reach exit — is fun. This was the single most important milestone. If the loop wasn't fun with rectangles, art wouldn't save it.

**Previous Milestone:** Movement Toy (complete)
**Next Milestone:** Polished Demo

### Delivered

All primary and secondary goals were met across 3 sprints:

- Sanity drains in real time (0.15/s, configurable via tuning panel)
- Player has health (5 HP, iframes, game over at 0)
- 5 civilians with flee AI, sanity restored on eat (+4, cap 12)
- Death scream mechanic alerts guards within 128px
- 3 guards with patrol, chase, and scream response AI
- Sanity-dependent pounce: Feral (sanity < 4) kills guards on contact
- Level 1 "First Run" (50x18 tiles, 4 sections, 5 civilians, 3 guards)
- Level exit zone with "Level Complete" screen + stats
- Win/lose states: Mind Lost (sanity 0), Game Over (HP 0), Level Complete (exit reached)
- Player-facing HUD: sanity bar (tier-colored) + HP hearts
- All juice systems from movement toy preserved (squash/stretch, dust, screen shake, vignette, speed lines)

### Exit Criteria — ALL MET

- [x] Sanity drains over time and is restored by eating civilians
- [x] Player takes damage from guards and dies at 0 HP
- [x] At least one guard patrols and chases the player
- [x] Level has a clear start and end, with a "Level Complete" screen on reaching the exit
- [x] All three failure states work (sanity 0, HP 0) with restart capability
- [x] The complete level is playable start-to-finish in 2-5 minutes
- [x] The core loop is testable and fun
- [x] No critical bugs (crashes, softlocks, clipping through level)
- [x] Movement feel has not regressed from the toy (verified at all sanity tiers)

---

## Milestone 2: Polished Demo --- IN PROGRESS (nearly complete)

**Purpose:** Transform the rectangle-art vertical slice into a game that looks, sounds, and feels like a real product. One hand-crafted demo level with pixel art, procedural audio, full game flow, and narrative flavor. This proves the game's identity and polish pipeline before committing to multi-level content production.

**Previous Milestone:** Vertical Slice (complete)
**Next Milestone:** Content + Ship

### Primary Goals

| # | Goal | Status | Deliverable |
|---|------|--------|-------------|
| 1 | Pixel art sprites | DONE | Zombie (3 sanity states x 3 walk frames), civilian (2 poses), guard (3 states: patrol/chase/alert) |
| 2 | Environment tileset | DONE | 4 tile variants (ground, wall, platform, surface) replacing colored blocks |
| 3 | City skyline background | DONE | Parallax stars + building silhouettes behind gameplay area |
| 4 | Procedural SFX (Web Audio API) | DONE | 8 sound effects: jump, land, eat, scream, damage, guard alert, die, level complete |
| 5 | 2-layer music system | DONE | Calm + tense layers crossfaded by sanity via Web Audio oscillators |
| 6 | Title screen + game flow | DONE | "Brains for Breakfast" title, intro text card, pause menu (Escape), full flow: Title -> Intro -> Playing -> GameOver/LevelComplete -> Title |
| 7 | Inner monologue system | DONE | Event-triggered zombie thoughts: near civilian, after eating, damage, sanity thresholds, near exit |
| 8 | Environmental signs | DONE | 3 readable signs placed in the demo level |
| 9 | Demo level | DONE | 60x18 hand-crafted level: 7 civilians, 4 guards, 3 signs, designed pacing |
| 10 | Polished HUD | DONE | Gradient sanity bar with tier glow, pixel-art heart shapes for HP |
| 11 | Game over screens | DONE | Full game over/level complete overlays with "R to Restart, T for Title" |
| 12 | Death animation delay | DONE | Brief delay before allowing restart after death |
| 13 | Dev mode hidden | DONE | Debug/tuning panels hidden by default, Ctrl+Shift+D to toggle |
| 14 | Mute toggle | DONE | M key or on-screen button to mute/unmute all audio |

### Exit Criteria

**Milestone 2 is COMPLETE when:**

- [x] Pixel art replaces all rectangles (zombie, civilians, guards, tiles)
- [x] Procedural SFX play for all critical actions
- [x] Music plays during gameplay and responds to sanity
- [x] Title screen, intro, pause, and game over screens all work
- [x] Demo level is hand-crafted and paced for a complete experience
- [x] Inner monologue triggers add narrative flavor
- [x] HUD is polished (gradient bar, heart shapes)
- [x] Dev tools are hidden from players by default
- [x] Full game flow works: Title -> Intro -> Play -> End -> Title
- [ ] Balance tuning pass on the demo level
- [ ] No critical or major bugs
- [ ] Movement feel verified at all sanity tiers (regression check)

### Remaining Work

- Balance tuning pass on the demo level (sanity economy, guard placement, pacing)
- Bug sweep and regression testing
- Movement feel verification at all sanity tiers

---

## Milestone 3: Content + Ship

**Purpose:** Build remaining levels, add the second guard type, final polish, and deploy. This is the production + ship milestone — executing against the proven design and polish pipeline established in Milestone 2.

**Previous Milestone:** Polished Demo
**Next Milestone:** Ship (this IS the ship milestone)

### Primary Goals

| # | Goal | Deliverable | Size | How to Verify |
|---|------|-------------|------|---------------|
| 1 | Additional levels (3-5 total) | Each level with distinct character, civilian routing, guard placement, difficulty target. Later levels require lower sanity for traversal. | L | Each level playable start-to-finish. Full game playthrough takes 15-20 minutes. |
| 2 | Second guard type | Different behavior from patrol guard. Creates routing decisions: "go through the patrollers or the chasers?" | M | Both guard types in later levels. Player can distinguish them by behavior and appearance. |
| 3 | Level progression | Completing one level loads the next. Victory screen after final level. State management between levels. | S | Seamless transitions. No dead ends. |
| 4 | Difficulty curve | Smooth progression from inviting (Level 1) to demanding (final level). | M | Full playthrough difficulty feels intentional, not arbitrary. |
| 5 | Balance sweep | Final tuning pass on sanity drain, civilian placement, guard behavior across all levels. | S | Full playthrough feels fair. No dominant strategy. No frustration spikes. |
| 6 | Bug fix sprint | All critical and major bugs fixed. Minor bugs documented for v1.1. | M | Zero critical, zero major remaining. |
| 7 | Playtest session | At least 1 full playthrough by someone who hasn't seen the game. Observe without guidance. | S | New player completes the game or fails in a learnable way. No confusion about objectives. |
| 8 | Deploy | Build uploaded to itch.io (or equivalent). Playable in-browser. | S | URL works. Game loads. Game is playable start-to-finish. |

### Secondary Goals

| # | Goal | Deliverable | Size |
|---|------|-------------|------|
| 9 | Additional environmental signs/narrative | More readable signs across levels, backstory hints. | S |
| 10 | localStorage checkpoint | Remember which level the player reached. Resume from last level on return. | S |
| 11 | Store page / release materials | itch.io page description, 3-5 screenshots, short tagline. | S |
| 12 | Performance check | 60fps on mid-range hardware. No hitches on level load. | S |
| 13 | Browser compatibility | Tested in Chrome, Firefox, Edge. | S |

### Stretch Goals

| # | Goal | Deliverable | Size |
|---|------|-------------|------|
| 14 | One special civilian type | E.g., "Adrenaline civilian" — eating grants temporary speed boost with sanity cost. | M |
| 15 | Additional music layers or tracks | Per-level musical identity or additional mood layers. | M |

### Exit Criteria

**Milestone 3 is COMPLETE when:**

- [ ] 3-5 levels built, playable, with pixel art and audio
- [ ] Second guard type implemented with distinct behavior
- [ ] Full game playthrough is 15-20 minutes and holds interest throughout
- [ ] At least 1 external playtest completed with no softlocks or confusion
- [ ] Game deployed and publicly accessible
- [ ] Zero critical or major bugs
- [ ] v1.0 git tag created
- [ ] v1.1 backlog captured (deferred features + minor bugs)

**Milestone 3 FAILS if:**

- Level design takes too long — fallback: ship with 3 levels instead of 5. 3 good levels > 5 rushed levels.
- Second guard type doesn't add meaningful decisions — fallback: ship with one guard type and more varied level layouts.
- A critical bug requires redesigning a core system — return to the affected system and fix before proceeding.

### Risks

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|------------|
| Level design takes longer than expected | Medium | Medium — 5 levels is ambitious | Build levels in priority order: tutorial, climax, middle. Cut to 3 if needed. |
| Second guard type doesn't differentiate enough | Medium | Medium — needs to create new decisions | Prototype behavior before building sprites. If it's just "faster guard," rethink. |
| Feature creep from "just one more thing" | High | High — every addition cascades | Scope is locked. Second guard type is the cap. Everything else is v1.1. |
| Balance across many levels is hard | Medium | Medium — can tune forever | Timebox to one session per level. "Good enough" is the bar. |

---

## Milestone Dependencies (Full Graph)

```
Movement Toy (DONE)
    |
    v
Milestone 1: Vertical Slice (DONE)
    |  Delivered: sanity drain, health, civilians, 1 guard type,
    |  1 level, win/lose, HUD, all juice systems
    |
    v
Milestone 2: Polished Demo (IN PROGRESS)
    |  Delivered: pixel art sprites, tileset, background, procedural SFX,
    |  2-layer music, title/intro/pause screens, inner monologue,
    |  environmental signs, demo level (60x18), polished HUD, mute toggle
    |  Remaining: balance pass, bug sweep, regression test
    |
    v
Milestone 3: Content + Ship
       Adds: 2-4 more levels, 2nd guard type, level progression,
       difficulty curve, playtest, deploy
       Output: v1.0 on itch.io
```

---

## Scope Escalation Rules

These rules prevent scope creep during production:

1. **No new Must Have features after Milestone 2 is complete.** The demo proves what the game is. New ideas go to v1.1.
2. **No new Should Have features after Milestone 3 starts.** Should Haves are locked at production entry. New ideas go to the v1.1 backlog.
3. **Nice to Have features are only attempted if a milestone is ahead of schedule.** If a milestone is on track or behind, Nice to Haves are automatically cut.
4. **Every addition requires a subtraction.** If something new enters the current milestone scope, something of equal or greater size must be removed or deferred.
5. **The v1.1 backlog is not a promise.** It's a parking lot for good ideas. No guilt about items that never get built.

---

*Produced using the gamedev-production skill (Milestone Planning framework). Lenses applied: #2 (Essential Experience), #14 (Problem Statement), #16 (Risk Mitigation), #18 (Passion), #33 (Meaningful Choices), #45 (Economy), #90 (Playtesting budgeted into every milestone).*
