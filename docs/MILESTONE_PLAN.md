# Brains for Breakfast — Milestone Plan

**Last Updated:** Feb 2026
**Project:** Brains for Breakfast v1
**Scope Reference:** `docs/SCOPE_V1.md`
**Skill Applied:** gamedev-production
**Sizing:** Relative (S/M/L) — no hour estimates. Calibrate after Milestone 1.

---

## Milestone Overview

| # | Name | Purpose | Size | Playable Deliverable |
|---|------|---------|------|---------------------|
| 1 | **Vertical Slice** | Prove the core loop is fun | L | 1 level, all core systems, rectangle art |
| 2 | **Pre-Production Gate** | Prove the game is worth building | M | 2 levels, tuned, all core systems solid |
| 3 | **Content + Identity** | Build the real game | XL | 3-5 levels, art, audio, second threat type |
| 4 | **Polish + Ship** | Make it shippable | M | Final build, tested, deployed |

**Pre-production exit gate** (between Milestones 2 and 3): "Two complete, playable levels with all core systems working." If this gate fails, the project pivots or is archived — do not enter production hoping it gets better.

---

## Milestone 1: Vertical Slice

**Purpose:** Prove the core loop — navigate → eat civilians → manage sanity → survive threats → reach exit — is fun. This is the single most important milestone. If the loop isn't fun with rectangles, art won't save it.

**Previous Milestone:** Movement Toy (complete)
**Next Milestone:** Pre-Production Gate

### Primary Goals

| # | Goal | Deliverable | Size | How to Verify |
|---|------|-------------|------|---------------|
| 1 | Sanity drains in real time | Sanity decreases at a configurable rate per second. Rate tunable via the existing tuning panel. | S | Slider auto-decreases. Zombie hits Gone state after expected duration. |
| 2 | Player has health | HP variable, damage function, death at 0. HP displayed on screen. | S | Take damage from threat → HP decreases. HP = 0 → game over screen. |
| 3 | Civilians exist and restore sanity | Living civilian entities with flee AI placed in the level. Zombie catches and eats civilian → sanity restored by configurable amount. Civilian disappears and triggers death scream alerting nearby threats. | L | Walk into civilian → sanity increases (capped at 12). Civilian disappears, scream indicator fires. Nearby threats respond. Debug panel confirms sanity value. |
| 4 | One threat type guards civilians and damages player | AI entity with guard patrol, leash-based chase, and death scream response. Deals damage on contact. Visually distinct rectangle (red). | M | Threat patrols near assigned civilian. Detects player within leash range → chases. Responds to death screams. Contact → player takes damage. Threat respects collision with platforms. |
| 5 | Level has entrance and exit | Spawn point + exit zone. Reaching exit → level complete screen. | S | Walk to exit → "Level Complete" overlay. Clear feedback. |
| 6 | Win/lose states work | Sanity 0 → "Mind Lost" (exists). HP 0 → "Game Over." Exit reached → "Level Complete." All states offer restart. | S | Each state reachable and restartable. No softlocks. |
| 7 | One complete test level | A level designed (via generator + hand-tuning) with civilian placements, threat placements, entrance, and exit. Tests all core systems together. | M | Play from entrance to exit. Must eat at least 1 civilian to survive. Must navigate past at least 2 threats. Completable in 2-5 minutes. |

### Secondary Goals

| # | Goal | Deliverable | Size |
|---|------|-------------|------|
| 8 | Basic HUD (not debug panel) | Player-facing sanity bar + HP display. Positioned for gameplay, not debugging. | S |
| 9 | Threat killed by jump-on-head or avoidance-only decision | Decide: can the zombie kill threats (platformer stomp)? Or is this pure avoidance? The answer determines the entire threat design. | S (design decision, not code) |

### Exit Criteria

**Milestone 1 is COMPLETE when:**

- [ ] Sanity drains over time and is restored by eating civilians
- [ ] Player takes damage from threats and dies at 0 HP
- [ ] At least one threat patrols and chases the player
- [ ] Level has a clear start and end, with a "Level Complete" screen on reaching the exit
- [ ] All three failure states work (sanity 0, HP 0) with restart capability
- [ ] The complete level is playable start-to-finish in 2-5 minutes
- [ ] The core loop is testable: "Am I having fun navigating, eating civilians, dodging threats, and managing sanity?"
- [ ] No critical bugs (crashes, softlocks, clipping through level)
- [ ] Movement feel has not regressed from the toy (test at all sanity tiers)

**Milestone 1 FAILS if:**

- The core loop is not fun even after a tuning pass. If navigate → eat → dodge → exit doesn't engage, the game concept needs rethinking before proceeding.
- Threats feel unfair or unlearnable — the player can't develop strategies against them.
- Sanity drain + civilian consumption doesn't create meaningful decisions (drain too slow = never eat; drain too fast = always desperate; civilians too common = no tension).

**If Milestone 1 fails:** Do NOT proceed to Milestone 2. Diagnose which element of the loop is failing (Lens #2: which part of the essential experience is broken?). Redesign that element and re-test. Budget one additional iteration cycle. If it still fails after redesign, evaluate kill criteria.

### Risks

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|------------|
| Threat AI is hard to tune (too aggressive / too passive) | Medium | High — unfun threats kill the loop | Start simple: guard patrol only. Add leash-based chase and scream response as second pass. Use detection range and leash distance as primary tuning knobs. |
| Sanity drain rate is hard to balance for real-time | Medium | Medium — paper prototype was turn-based, real-time pacing is different | Start with a slow drain (60 seconds Lucid→Gone with no eating). Tune from there. The tuning panel makes this instant. |
| Adding new systems breaks movement feel | Low | High — the proven foundation becomes unproven | Run the movement regression checklist after each system addition. If feel degrades, revert last change before debugging. |
| Scope creep ("just add one more thing to the slice") | High | Medium — milestone drags | The deliverable list above is locked. Nothing else enters this milestone. New ideas go to a "Future" column in TODO.md. |

---

## Milestone 2: Pre-Production Gate

**Purpose:** Prove the game is worth building at production quality. Two levels, tuned, all core systems solid. This is the go/no-go decision point.

**Previous Milestone:** Vertical Slice
**Next Milestone:** Content + Identity (production begins)

### Primary Goals

| # | Goal | Deliverable | Size | How to Verify |
|---|------|-------------|------|---------------|
| 1 | Second level designed and playable | A second level with different layout, threat placement, and civilian routing. Harder than Level 1. | M | Playable start-to-finish. Noticeably different from Level 1. Harder but fair. |
| 2 | Level progression works | Completing Level 1 → loads Level 2. Completing Level 2 → victory screen. | S | Seamless transition. State resets correctly between levels (HP persists? Sanity resets? Design decision made and implemented). |
| 3 | Difficulty scales between levels | Level 2 has more threats, tighter platforms, and/or requires lower sanity to traverse. | S | Level 2 is harder than Level 1 by a testable metric (more deaths, lower average completion sanity). |
| 4 | Balance tuning pass | Sanity drain rate, sanity restore amount, threat damage, threat speed, civilian placement density — all tuned so the loop feels right across both levels. | M | Play both levels 5+ times. Consistent experience: not too easy, not too hard. The sanity tradeoff feels meaningful. |
| 5 | Title screen and game flow | Title → Level 1 → Level 2 → Victory. Game Over → Restart from current level or title. | S | Complete flow works with no dead ends. |

### Secondary Goals

| # | Goal | Deliverable | Size |
|---|------|-------------|------|
| 6 | Tuning panel updated for new systems | Threat speed, detection range, sanity drain rate, sanity restore amount all tunable via sliders. | S |
| 7 | Design decision documented: threat interaction model | Is the game pure avoidance, stomp-to-kill, or something else? Decision documented in GDD with rationale. | S |

### Exit Criteria (PRE-PRODUCTION GATE)

**This is the production readiness gate. ALL of these must be true:**

- [ ] Two complete, playable levels with all core systems working
- [ ] The core loop is fun — "I want to play again" after a session
- [ ] Sanity risk/reward tradeoff creates genuine decisions (Lens #33)
- [ ] Threats are learnable and fair (Koster: player can identify and master the pattern)
- [ ] Difficulty scales between levels without feeling arbitrary
- [ ] Complete game flow works (title → play → win/lose → restart)
- [ ] No critical or major bugs
- [ ] Movement feel verified at all sanity tiers (regression check)
- [ ] Design decisions documented: threat model, level transition rules, sanity economy values
- [ ] Developer still wants to build this game (Lens #18: Passion)

**Gate Decision:**

| Verdict | Action |
|---------|--------|
| **PASS** | Enter production (Milestone 3). Scope is locked. Art and audio pipelines begin. |
| **CONDITIONAL PASS** | One specific issue identified. Fix it (budget 1 extra iteration), then proceed. |
| **FAIL — Fixable** | Core loop has a specific problem. Return to Milestone 1 scope. Redesign the failing element. Re-test. |
| **FAIL — Fatal** | The concept isn't fun despite iteration. Archive the project. The movement toy was the valuable output — carry those tuned values to a different game. |

### Risks

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|------------|
| Level 2 feels like Level 1 with harder numbers | Medium | Medium — levels need to feel distinct, not just harder | Each level should introduce a spatial challenge: Level 1 = horizontal, Level 2 = vertical. Different section archetypes. |
| Balance tuning is a black hole | High | Medium — can tune forever | Timebox to one session. "Good enough" is the bar, not "perfect." Record the tuned values and move on. |
| Gate anxiety — reluctance to commit to pass/fail | Medium | High — stuck in pre-production forever | Set a hard date for the gate review. Run the checklist mechanically. If criteria are met, proceed. Done is better than perfect. |

---

## Milestone 3: Content + Identity

**Purpose:** Build the real game. All levels, art, audio, and Should Have features. This is production — executing against the proven design.

**Previous Milestone:** Pre-Production Gate
**Next Milestone:** Polish + Ship

### Primary Goals

| # | Goal | Deliverable | Size | How to Verify |
|---|------|-------------|------|---------------|
| 1 | All levels built (3-5 total) | Each level designed with distinct character, civilian routing, threat placement, and difficulty target. Later levels require lower sanity for traversal. | L | Each level playable start-to-finish. Full game playthrough takes 15-20 minutes. |
| 2 | Player art | Gentleman zombie sprite with visual states for each sanity tier (minimum: Lucid and Feral look different). Replaces green rectangle. | M | Zombie reads clearly against backgrounds. Sanity state is visible at a glance. |
| 3 | Environment tileset | City/building tiles replacing colored blocks. Consistent style. Enough variety for 3-5 visually distinct levels. | M | Levels look like a decaying city. Tiles read clearly as solid vs. empty. |
| 4 | Threat + civilian sprites | Visual identity for threats and civilians. Threats are immediately identifiable as dangerous. Civilians are immediately identifiable as food sources. | S | First-time player understands "avoid red, chase blue" (or equivalent) within 5 seconds. |
| 5 | Second threat type | Different behavior from threat 1. Creates routing decisions: "go through the patrollers or the chasers?" | M | Both threat types in later levels. Player can distinguish them by behavior and appearance. Different strategies for each. |
| 6 | Critical SFX | Jump, land, eat civilian, take damage, die, death scream, level complete. Minimum 8-10 sound effects. | M | Game is no longer silent. Each critical action has audio feedback. |
| 7 | Music | At minimum 1 looping track. Ideally 2 layers (calm/tense) crossfaded by sanity. | M | Music plays during gameplay. Doesn't loop annoyingly within a 5-minute level. |
| 8 | Sanity visual effects | Screen vignette, color desaturation, or distortion as sanity drops. Communicates sanity state without reading the HUD. | S | Player can tell approximate sanity tier from visual feel alone. |

### Secondary Goals

| # | Goal | Deliverable | Size |
|---|------|-------------|------|
| 9 | Juice pass | Screen shake on damage, landing squash, civilian eat burst. | S |
| 10 | Pause menu | Pause key → overlay with Resume / Restart Level / Quit to Title. | S |
| 11 | Difficulty curve verified | Full game playthrough difficulty is smooth: Level 1 is inviting, final level is demanding. | S |

### Stretch Goals

| # | Goal | Deliverable | Size |
|---|------|-------------|------|
| 12 | One special civilian type | E.g., "Adrenaline civilian" — eating grants temporary speed boost with sanity cost. | M |
| 13 | Environmental text | 2-3 readable signs/notes in levels hinting at backstory. | S |
| 14 | localStorage checkpoint | Remember which level the player reached. Resume from last level on return. | S |

### Exit Criteria

**Milestone 3 is COMPLETE when:**

- [ ] All planned levels are built, playable, and have final (or near-final) art
- [ ] Player sprite, tileset, threat sprites, and civilian sprites are in-game
- [ ] Two distinct threat types with different behaviors
- [ ] Critical SFX in place for all major player actions
- [ ] At least 1 music track plays during gameplay
- [ ] Sanity visual effects communicate state
- [ ] Full game playthrough is 15-20 minutes and holds interest throughout
- [ ] No critical bugs
- [ ] Build is content-complete (all gameplay is in; only polish remains)

**Milestone 3 FAILS if:**

- Art pipeline collapses (can't produce consistent assets) — fallback: ship with geometric/stylized art. The game works with rectangles if the loop is fun.
- Audio integration causes performance problems — fallback: reduce to 3-4 critical SFX, no music.
- Content volume exceeds capacity — fallback: ship with 3 levels instead of 5. 3 good levels > 5 rushed levels.

### Risks

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|------------|
| Art pipeline is unproven | High | High — biggest unknown in the project | Prove the pipeline on player sprite FIRST before committing to full tileset. If AI generation doesn't work, commit to geometric/stylized art immediately. |
| Audio sourcing takes too long | Medium | Medium — game works without it | Use free asset libraries (freesound.org, OpenGameArt) first. Only create custom audio if library sounds don't fit. |
| Level design takes longer than expected | Medium | Medium — 5 levels is ambitious | Build levels in priority order: Level 1 (tutorial), Level 3 (climax), Level 2 (middle). Cut Level 4-5 if needed. |
| Feature creep from "just one more enemy type" | High | High — every addition cascades into balance, art, audio, level design | Scope is locked. Second threat type is the cap. Everything else is v1.1. |

---

## Milestone 4: Polish + Ship

**Purpose:** Make the game shippable. Fix bugs, tune balance, add juice, deploy.

**Previous Milestone:** Content + Identity
**Next Milestone:** Ship (this IS the ship milestone)

### Primary Goals

| # | Goal | Deliverable | Size | How to Verify |
|---|------|-------------|------|---------------|
| 1 | Bug fix sprint | All critical and major bugs fixed. Minor bugs documented for v1.1. | M | Bug list triaged. Zero critical, zero major remaining. |
| 2 | Balance sweep | Final tuning pass on sanity drain, civilian placement, threat behavior, difficulty curve. | S | Full playthrough feels fair and engaging. No dominant strategy. No frustration spikes. |
| 3 | Playtest session | At least 1 full playthrough by someone who hasn't seen the game before. Observe without guidance. | S | New player completes the game or fails in a learnable way. No confusion about objectives. No softlocks. |
| 4 | Remove debug tools | Debug panel and tuning panel hidden or removed from the shipped build. No console spam. | S | Clean build with no developer UI visible. |
| 5 | Store page / release materials | itch.io page description, 3-5 screenshots, short tagline. | S | Page exists and looks professional enough to click "Play." |
| 6 | Deploy | Build uploaded to itch.io (or equivalent). Playable in-browser. | S | URL works. Game loads. Game is playable start-to-finish. |

### Secondary Goals

| # | Goal | Deliverable | Size |
|---|------|-------------|------|
| 7 | Performance check | 60fps on mid-range hardware. No hitches on level load. | S |
| 8 | Browser compatibility | Tested in Chrome, Firefox, Edge. | S |

### Exit Criteria

**Milestone 4 is COMPLETE when:**

- [ ] Zero critical or major bugs
- [ ] At least 1 external playtest completed with no softlocks or confusion
- [ ] Debug tools removed from shipped build
- [ ] Game deployed and publicly accessible
- [ ] v1.0 git tag created
- [ ] v1.1 backlog captured (deferred features + minor bugs)

**Milestone 4 FAILS if:**

- A critical bug is discovered that requires redesigning a core system — return to Milestone 3 scope.
- Playtest reveals the game isn't fun — this should have been caught at the pre-production gate. If it wasn't, the gate criteria were too lenient.

---

## Milestone Dependencies (Full Graph)

```
Movement Toy (DONE)
    │
    ▼
Milestone 1: Vertical Slice
    │  Adds: sanity drain, health, civilians, 1 threat, 1 level, win/lose
    │
    ▼
Milestone 2: Pre-Production Gate  ◄── GO/NO-GO DECISION
    │  Adds: 2nd level, level progression, balance pass, title screen
    │  Gate: "Two complete, playable levels with all core systems working"
    │
    ▼
Milestone 3: Content + Identity
    │  Adds: all levels, art, audio, 2nd threat, SFX, music, sanity VFX
    │
    ▼
Milestone 4: Polish + Ship
       Adds: bug fixes, balance, playtest, deploy
       Output: v1.0 on itch.io
```

---

## Scope Escalation Rules

These rules prevent scope creep during production:

1. **No new Must Have features after Milestone 2 gate.** If something is truly Must Have and wasn't identified, it means the pre-production gate was wrong. Stop and reassess.
2. **No new Should Have features after Milestone 3 starts.** Should Haves are locked at production entry. New ideas go to the v1.1 backlog.
3. **Nice to Have features are only attempted if a milestone is ahead of schedule.** If a milestone is on track or behind, Nice to Haves are automatically cut.
4. **Every addition requires a subtraction.** If something new enters the current milestone scope, something of equal or greater size must be removed or deferred.
5. **The v1.1 backlog is not a promise.** It's a parking lot for good ideas. No guilt about items that never get built.

---

*Produced using the gamedev-production skill (Milestone Planning framework). Lenses applied: #2 (Essential Experience), #14 (Problem Statement), #16 (Risk Mitigation), #18 (Passion), #33 (Meaningful Choices), #45 (Economy), #90 (Playtesting budgeted into every milestone).*
