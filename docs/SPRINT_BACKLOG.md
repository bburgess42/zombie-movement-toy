# Brains for Breakfast — Sprint Backlog (Milestone 1: Vertical Slice)

**Last Updated:** Feb 2026
**Milestone:** 1 — Vertical Slice
**Milestone Goal:** Prove the core loop (navigate → eat civilians → manage sanity → survive threats → reach exit) is fun.
**Skill Applied:** gamedev-production (Sprint Kickoff)

---

## Sprint Structure

Milestone 1 is broken into 3 sprints. Each sprint ends with something testable.

| Sprint | Focus | Testable Output |
|--------|-------|-----------------|
| **Sprint 1** | Game systems foundation | Zombie drains sanity, eats civilians, has health, can die. Playable on the existing Gauntlet map. |
| **Sprint 2** | Threat AI + game flow | Threats patrol and chase. Level has entrance/exit. Win/lose states work. |
| **Sprint 3** | Level assembly + integration test | One complete level purpose-built for the core loop. HUD. Full playtest. |

---

## Sprint 1: Game Systems Foundation

**Goal:** "The zombie has a survival problem. Sanity drains. Eating civilians restores it. The zombie can take damage and die. All of this is testable on the existing Gauntlet map with rectangles."

**How this advances the milestone:** Delivers goals 1 (sanity drain), 2 (health), 3 (civilians), and partially 6 (lose states). After this sprint, the resource loop exists even without threats.

### Tasks

| # | Task | Domain | Size | Dependency | Status |
|---|------|--------|------|-----------|--------|
| **1.1** | **Add real-time sanity drain** | systems-balance | S | None | [ ] |
| | Sanity decreases at a configurable rate per second (e.g., `SANITY_DRAIN_RATE = 0.9` per second, matching paper prototype's 0.9/round scaled to real-time). Add constant to the tunable constants section. Add drain to the game loop (after physics, before render). Drain stops at 0. Gone overlay already triggers at sanity 0 — verify it still works. Add drain rate slider to tuning panel. | | | | |
| **1.2** | **Add player health system** | code-architecture | S | None | [ ] |
| | Add `zombie.hp` and `zombie.maxHP` to game state (default: 5 HP, matching GDD). Add `damageZombie(amount)` function that reduces HP and triggers a brief invincibility window (0.5-1s, flashing). Add HP constant to tuning panel. No damage sources yet — just the system. | | | | |
| **1.3** | **Implement game-over state for HP = 0** | code-architecture | S | 1.2 | [ ] |
| | When `zombie.hp <= 0`, show a "GAME OVER" overlay (similar to existing Gone overlay). Stop the game loop or block input. Add a "Press R to Restart" handler that resets zombie position, HP, sanity, and all state. Verify: Gone overlay (sanity 0) and Game Over overlay (HP 0) are distinct — different text, different color. | | | | |
| **1.4** | **Create civilian entity with flee AI** | code-architecture | L | None | [ ] |
| | Define a civilian entity: `{ x, y, width: 24, height: 24, vx, alive: true, state: 'wander' }`. AI states: WANDER (slow random patrol, 30 px/s), FLEE (run from zombie at 60 px/s when zombie within 96px), SEEK (drift toward nearest threat within 128px for protection). Reverse at platform edges (never fall off). Render as a blue/cyan rectangle with "C" label. Store in `civilians[]` array. Place 4-5 test civilians on the existing Gauntlet map. Add constants to tuning panel: `CIVILIAN_FLEE_SPEED`, `CIVILIAN_FLEE_RANGE`, `CIVILIAN_SEEK_RANGE`, `CIVILIAN_WANDER_SPEED`. | | | | |
| **1.5** | **Implement civilian eat + death scream** | systems-balance | M | 1.1, 1.4 | [ ] |
| | In the game loop, check AABB overlap between zombie and each alive civilian. On overlap: restore sanity by `SANITY_PER_EAT` (default: 4, matching GDD), cap at 12, set `civilian.alive = false`. Trigger death scream: find all threats within `SCREAM_ALERT_RANGE` (128px) of eaten civilian's position, set their `alertTimer = SCREAM_ALERT_DURATION` (3s) and `alertTarget` to civilian's position. Visual feedback: civilian disappears, scream indicator pulse. Add `SANITY_PER_EAT`, `SCREAM_ALERT_RANGE`, `SCREAM_ALERT_DURATION` to tuning panel. | | | | |
| **1.6** | **Design decision: sanity drain rate for real-time** | design-doc | S | 1.1 | [ ] |
| | The paper prototype drains 0.9 sanity per round (~13 rounds to Gone). In real-time, "a round" is roughly 3-5 seconds of play. Test drain rates: 0.15/s (80s to Gone), 0.2/s (60s to Gone), 0.3/s (40s to Gone). Play the Gauntlet with civilians and each rate. Evaluate: Is there enough time to chase and eat civilians? Is there urgency? Does the player NEED to eat? Does the death scream create tension? Record the chosen value. | | | | |
| **1.7** | **Integrate sanity slider with drain** | ui-ux | S | 1.1 | [ ] |
| | Decide: does the manual sanity slider coexist with auto-drain? Recommended: add a "Drain: ON/OFF" toggle next to the slider. When drain is ON, the slider still works but sanity also drains. When OFF, it's the original toy behavior (slider-only). This preserves the toy's tuning utility while enabling game testing. | | | | |

### Sprint 1 Cut Line

**Above the line (must do):** Tasks 1.1, 1.2, 1.3, 1.4, 1.5
**Below the line (do if time):** Tasks 1.6, 1.7

Without 1.1-1.5, the sprint goal isn't met. Tasks 1.6 and 1.7 are refinement — drain rate tuning and UI polish can carry over.

### Sprint 1 Verification

After Sprint 1, open index.html and test:
- [ ] Sanity drains over time without touching the slider
- [ ] Civilians wander on platforms, flee when zombie approaches, reverse at edges
- [ ] Walk into a civilian → sanity increases, civilian disappears, death scream visual fires
- [ ] Sanity hits 0 → "MIND LOST" overlay appears
- [ ] Manually reduce HP (via console or test damage) → HP 0 → "GAME OVER" overlay
- [ ] Press R (or equivalent) → game resets to start
- [ ] Movement feel has NOT changed (regression check: run, jump, drift at all tiers)

---

## Sprint 2: Threat AI + Game Flow

**Goal:** "Something is trying to hurt the zombie. The level has a start and end. The player can win by reaching the exit or lose by dying."

**How this advances the milestone:** Delivers goals 4 (threat type), 5 (entrance/exit), 6 (win/lose states complete). After this sprint, all core systems exist — the only missing piece is a purpose-built level.

### Tasks

| # | Task | Domain | Size | Dependency | Status |
|---|------|--------|------|-----------|--------|
| **2.1** | **Design decision: threat interaction model** | design-doc | S | None | [ ] |
| | Answer: Can the zombie kill threats, or is it pure avoidance? Options: (A) **Stomp-to-kill** — jump on head kills threat, touching from side/below damages zombie. Classic platformer. (B) **Pure avoidance** — threats cannot be killed, only dodged. Emphasizes movement mastery. (C) **Sanity-dependent** — at Feral tier, the zombie can stomp; at Lucid, it can't. Ties into the power/control tradeoff. Document the decision and rationale in a section at the top of the task file or GDD. This decision shapes ALL of Sprint 2. | | | | |
| **2.2** | **Create threat entity data structure** | code-architecture | S | None | [ ] |
| | Define threat entity: `{ x, y, width: 32, height: 32, vx, hp, state, guardedCivilian, detectionRange, speed, damage, alertTimer, alertTarget }`. Store in `threats[]` array. Render as a red rectangle with "!" label. Add configurable constants: `THREAT_SPEED`, `THREAT_CHASE_SPEED`, `THREAT_DETECTION_RANGE`, `THREAT_DAMAGE`, `THREAT_GUARD_RANGE`, `THREAT_LEASH_DISTANCE`, `THREAT_PATROL_RANGE`. At level start, assign each threat to guard its nearest same-platform civilian. Add to tuning panel. | | | | |
| **2.3** | **Implement threat guard patrol behavior** | code-architecture | M | 2.2, 1.4 | [ ] |
| | State: GUARD_PATROL. Threat patrols within `THREAT_GUARD_RANGE` (64px) of its assigned civilian. If no civilian assigned (all dead), fall back to standard patrol between `patrolLeft` and `patrolRight`. Reverses direction at patrol bounds and platform edges. When guarded civilian is eaten, reassign to nearest remaining civilian (prefer civilians not already guarded by 2+ threats). Render facing direction indicator. | | | | |
| **2.4** | **Implement threat chase + scream response** | code-architecture | M | 2.3 | [ ] |
| | State: CHASE. When zombie is within `THREAT_DETECTION_RANGE` AND within `THREAT_LEASH_DISTANCE` of guarded civilian, switch to CHASE. Move toward zombie at `THREAT_CHASE_SPEED`. If zombie exceeds leash distance from guarded civilian, break chase and return to GUARD_PATROL. State: RESPOND_TO_SCREAM. When death scream fires within `SCREAM_ALERT_RANGE`, set `alertTimer` and move to `alertTarget` (scream location) at chase speed. Attack zombie if encountered en route. After alert timer expires, return to GUARD_PATROL. Scream response overrides chase. | | | | |
| **2.5** | **Implement threat-zombie collision damage** | code-architecture | S | 2.2, 1.2 | [ ] |
| | AABB overlap check between zombie and each threat. On overlap: call `damageZombie(THREAT_DAMAGE)`. Respect invincibility window (zombie flashes, no re-damage during iframes). If using stomp-to-kill (decision 2.1): check if zombie is falling and above threat's midpoint → kill threat instead of damaging zombie. | | | | |
| **2.6** | **Implement threat collision with tiles** | code-architecture | S | 2.2 | [ ] |
| | Threats need basic tile collision: stay on platforms, don't walk through walls. Simpler than zombie collision — threats don't jump. Just horizontal movement + ground check. If no ground ahead, reverse direction (don't walk off ledges). | | | | |
| **2.7** | **Add level entrance and exit zones** | code-architecture | S | None | [ ] |
| | Define `levelEntrance = { x, y }` and `levelExit = { x, y, width: 32, height: 64 }`. Render exit as a distinct colored rectangle (gold/yellow with "EXIT" label). Zombie spawns at entrance. When zombie overlaps exit → trigger level complete. | | | | |
| **2.8** | **Implement "Level Complete" state** | code-architecture | S | 2.7 | [ ] |
| | When zombie reaches exit: show "LEVEL COMPLETE" overlay. Stop game loop or block input. Display stats (time, civilians eaten, HP remaining). "Press R to Restart" or "Press Enter to Continue" (for now, just restart — level progression comes in Milestone 2). | | | | |
| **2.9** | **Place threats + civilians on Gauntlet map** | level-design | S | 2.3, 2.6, 1.4 | [ ] |
| | Add 3 threats and 5 civilians to the existing Gauntlet map. Place 2 civilians far from threats (unguarded/safe). Place 3 civilians near threats (guarded/risky). Verify threat guard assignment picks the correct civilian. One threat on the ground runway, one on a Lucid-tier platform, one guarding a civilian near a vertical challenge. | | | | |
| **2.10** | **Integration test: full loop on Gauntlet** | qa-testing | S | All above | [ ] |
| | Play the Gauntlet map with all systems: sanity drains, civilians restore sanity, threats patrol and chase, exit exists. Can you survive from entrance to exit? Is it fun? Document observations. This is NOT the final level — it's a systems integration test. | | | | |

### Sprint 2 Cut Line

**Above the line (must do):** Tasks 2.1, 2.2, 2.3, 2.4, 2.5, 2.6, 2.7, 2.8
**Below the line (do if time):** Tasks 2.9, 2.10

Chase + scream response (2.4) is above the line because it is the mechanical heart of the civilian ecology's risk/reward loop — without it, the death scream fires but threats do not respond, making the scream a visual-only event with no gameplay consequence. Placement (2.9) and integration testing (2.10) can be quick or deferred.

### Sprint 2 Verification

After Sprint 2, open index.html and test:
- [ ] Red threat rectangles patrol near their assigned civilians
- [ ] Threats do not walk through walls or fall off platforms
- [ ] Touching a threat damages the zombie (HP decreases, iframes flash)
- [ ] Threats detect and chase the zombie within leash distance
- [ ] Threats break chase and return to guard when zombie exceeds leash distance
- [ ] Eating a civilian triggers a scream — nearby threats respond and move to scream location
- [ ] After scream alert expires, threats return to guard behavior
- [ ] When a guarded civilian is eaten, the threat reassigns to another civilian
- [ ] (If stomp-to-kill) Jumping on a threat kills it
- [ ] Exit zone is visible and distinct
- [ ] Walking into exit → "Level Complete" overlay
- [ ] "GAME OVER" (HP 0), "MIND LOST" (sanity 0), "LEVEL COMPLETE" (exit) all work and offer restart
- [ ] Movement feel has NOT changed (regression check)
- [ ] Threats + civilians + death scream + sanity drain + exit = the core loop is playable

---

## Sprint 3: Level Assembly + Integration

**Goal:** "One purpose-built level demonstrates the complete core loop. A first-time player can play from start to exit (or die trying) and the experience answers: is this fun?"

**How this advances the milestone:** Delivers goal 7 (complete test level), goal 8 secondary (basic HUD), and the milestone exit criteria (playable, testable vertical slice).

### Tasks

| # | Task | Domain | Size | Dependency | Status |
|---|------|--------|------|-----------|--------|
| **3.1** | **Design Level 1 layout** | level-design | M | Sprint 2 complete | [ ] |
| | Design a purpose-built level for the vertical slice. NOT the Gauntlet — a level designed for the core loop. Requirements: (1) Linear left-to-right flow with entrance and exit. (2) 5 civilians placed to create routing decisions: 2 unguarded (safe, near main path) and 3 guarded (risky, near threats). (3) 3 threats placed to guard civilians and create obstacles. (4) At least one section requiring low sanity to traverse (testing the power/control tradeoff). (5) Completable in 2-5 minutes. Use the generator to create a base layout, then hand-tune. Dimensions: 40-50 tiles wide, 15-20 tiles tall. Sketch the layout with annotations before building. See CIVILIAN_ECOLOGY_SPEC.md for placement guidelines. | | | | |
| **3.2** | **Build Level 1 in code** | level-design | M | 3.1 | [ ] |
| | Encode the designed layout as a tile map string array (like the Gauntlet). Add civilian positions and threat positions as data. Threat-civilian guard assignments happen automatically at level load (each threat guards nearest same-platform civilian). Replace the default map load with Level 1. Keep the Gauntlet available via the "Gauntlet" button. | | | | |
| **3.3** | **Add basic player-facing HUD** | ui-ux | S | 1.2, 1.1 | [ ] |
| | Replace or supplement the debug panel with a player-facing HUD: (1) Sanity bar — horizontal bar, color matches tier (green→yellow→orange→red). (2) HP display — simple hearts or numbered display. Position at top-left, non-intrusive. The debug panel should still be accessible via the tuning toggle for development, but the HUD is what a player sees. | | | | |
| **3.4** | **Add title/start screen** | ui-ux | S | None | [ ] |
| | Overlay on the canvas: "BRAINS FOR BREAKFAST" title, "Press ENTER to Start" prompt. Game doesn't begin until player presses Enter. Minimal — text on a dark background. Can reuse the Gone overlay pattern. | | | | |
| **3.5** | **Balance tuning pass** | systems-balance | M | 3.2 | [ ] |
| | Play Level 1 repeatedly and tune: (1) Sanity drain rate — is there urgency without desperation? (2) Sanity restore amount — does eating feel meaningful? (3) Civilian placement — does the player have eat-order choices (free vs. guarded)? (4) Death scream range — does eating feel risky near threats? (5) Threat speed, detection range, leash distance — learnable and fair? (6) Threat guard reassignment — does consolidation make later eats harder? (7) Level length — 2-5 minutes? Record all tuned values. | | | | |
| **3.6** | **Threat behavior tuning** | systems-balance | S | 2.3, 2.4 | [ ] |
| | Tune threat constants for the level: patrol speed, chase speed, detection range, patrol range. Threats should be avoidable by a skilled player at any sanity. At low sanity, threats should be harder to dodge (reduced air control, drift) but the player moves faster (can outrun). The interplay between sanity-driven movement and threat avoidance IS the core tension. | | | | |
| **3.7** | **Full vertical slice playtest** | qa-testing | S | All above | [ ] |
| | Play the complete experience: Title → Level 1 → Win or Lose → Restart. Play 5+ times. Answer the milestone's core question: "Is the core loop fun?" Document: (1) What works — what moments feel good? (2) What's broken — bugs, frustrations, confusion. (3) What's missing — what would make it better? (Note: "missing" items go to the backlog, not to this sprint.) (4) Tuned constant values — record the final values after tuning. | | | | |
| **3.8** | **Update docs** | production | S | 3.7 | [ ] |
| | Update `docs/CURRENT_STATE.md` with all new systems (sanity drain, health, civilians, threats, level structure, HUD, game states). Update `docs/TODO.md` with completed items and any new items discovered during playtesting. Update `docs/ARCHITECTURE.md` with new sections (threat AI, civilian AI, game states). | | | | |

### Sprint 3 Cut Line

**Above the line (must do):** Tasks 3.1, 3.2, 3.5, 3.7
**Below the line (do if time):** Tasks 3.3, 3.4, 3.6, 3.8

A purpose-built level + balance pass + playtest is the minimum. HUD and title screen are polish. Docs should be updated but won't block the milestone.

### Sprint 3 Verification

After Sprint 3, the **Milestone 1 exit criteria** are evaluated:
- [ ] Sanity drains over time and is restored by eating civilians
- [ ] Player takes damage from threats and dies at 0 HP
- [ ] At least one threat patrols and chases the player
- [ ] Level has a clear start and end, with a "Level Complete" screen on reaching the exit
- [ ] All three failure states work (sanity 0, HP 0) with restart capability
- [ ] The complete level is playable start-to-finish in 2-5 minutes
- [ ] The core loop is testable: "Am I having fun?"
- [ ] No critical bugs (crashes, softlocks, clipping)
- [ ] Movement feel has not regressed from the toy

---

## Task Dependency Graph

```
Sprint 1 (Systems Foundation)
─────────────────────────────
1.1 (sanity drain) ──┐
                      ├──→ 1.5 (eat + scream) ──→ 1.6 (drain rate tuning)
1.4 (civilian AI) ───┘                             1.7 (slider integration)
1.2 (health) ──→ 1.3 (game over)


Sprint 2 (Threats + Flow)
─────────────────────────
2.1 (design decision) ──→ shapes 2.5
1.4 (civilians) ──→ 2.2 (threat entity + guard assign) ──→ 2.3 (guard patrol) ──→ 2.4 (chase + scream) ──→ 2.9 (placement)
                                                       └──→ 2.6 (tile collision) ──┘                          │
1.2 (health) ──────────────────────────────────────────────→ 2.5 (damage) ────────────────────────────────────→│
2.7 (entrance/exit) ──→ 2.8 (level complete) ────────────────────────────────────────────────────────────────→ 2.10 (integration)


Sprint 3 (Level + Integration)
──────────────────────────────
3.1 (design level) ──→ 3.2 (build level) ──→ 3.5 (balance) ──→ 3.7 (playtest) ──→ 3.8 (docs)
                                               3.6 (threat tune) ──┘
3.3 (HUD) ──────────────────────────────────────────────────────────┘
3.4 (title screen) ─────────────────────────────────────────────────┘
```

**Critical path:** 1.4 → 1.5 → 2.2 → 2.3 → 2.4 → 2.5 → 3.1 → 3.2 → 3.5 → 3.7

---

## Cross-Sprint Design Decisions to Make Early

These decisions affect multiple tasks. Resolve them before or during Sprint 1 to avoid rework.

| Decision | Options | Affects Tasks | Recommended |
|----------|---------|--------------|-------------|
| **Threat interaction model** | (A) Stomp-to-kill, (B) Pure avoidance, (C) Sanity-dependent | 2.1, 2.5, 3.1, 3.5 | **(C) Sanity-dependent** — ties directly into the essential experience. At Feral you're powerful enough to stomp; at Lucid you must dodge. Creates the power/control decision. But (B) is simpler to implement and still works. |
| **Sanity drain: paused during Gone?** | (A) Drain stops at 0, (B) Drain continues below 0 | 1.1, 1.3 | **(A)** — sanity 0 = game over. No below-zero. |
| **Health persistence between levels** | (A) HP carries over, (B) HP resets each level | 1.2, future M2 | **(A)** for now — carries tension across levels. Revisit in Milestone 2. |
| **Sanity reset between levels** | (A) Sanity carries over, (B) Sanity resets to 12 | 1.1, future M2 | **(B)** — each level is a fresh survival challenge. Clean slate. Revisit in Milestone 2. |
| **Drain + slider coexistence** | (A) Drain replaces slider, (B) Toggle, (C) Drain only in "game mode" | 1.1, 1.7 | **(B) Toggle** — preserves the toy's utility for tuning while enabling game testing. One checkbox: "Auto Drain." |
| **Threat guard assignment** | (A) Nearest civilian, (B) Manually assigned in level data | 2.2, 2.3 | **(A) Nearest same-platform civilian** — automatic at level load. Simpler, and placement is the level design lever. Manual assignment is unnecessary complexity for v1. |

---

## Velocity Calibration Note

This is the first sprint of Brains for Breakfast. There is no velocity data yet. After Sprint 1, measure:
- How many tasks were completed vs. planned?
- How accurate were the S/M/L estimates?
- What took longer than expected? What was faster?

Use Sprint 1 actuals to calibrate Sprint 2 and 3 estimates. If Sprint 1 takes significantly longer than expected, reduce Sprint 2 and 3 scope before starting them — don't assume you'll "catch up."

---

*Produced using the gamedev-production skill (Sprint Kickoff framework). Task domains map to gamedev skills: code-architecture, systems-balance, level-design, ui-ux, design-doc, qa-testing, production.*
