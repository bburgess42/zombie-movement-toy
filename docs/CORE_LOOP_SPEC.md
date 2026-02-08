# Mechanic Specification: Core Game Loop

**Game:** Brains for Breakfast
**Author:** Brent Burgess
**Version:** 1.0
**Date:** Feb 2026
**Status:** Proposed

---

## 1. Player Fantasy

**What the player gets to feel:** The tension of a gentleman zombie — powerful, dignified, and losing his mind. You feel the pull between maintaining composure (Lucid: precise, controlled, safe) and unleashing raw undead power (Feral: fast, dangerous, unpredictable). Every brain you eat buys another moment of clarity. Every second without food is a step toward becoming the monster you're fighting not to be.

**Power fantasy:** You are stronger than the things that hunt you — if you're willing to give up control. At Feral, you outrun threats, leap gaps they can't cross, and (potentially) stomp them flat. The power is always available. The cost is always real.

**Skill expression:** A beginner plays safe — stays Lucid, eats every brain immediately, avoids all threats. An expert manages the sanity curve: deliberately drops to Slipping for a speed boost through a threat corridor, eats a brain at the last possible moment to recover, threads a Feral jump through a gap that Lucid can't reach. The expert's zombie looks reckless but is precisely controlled chaos.

---

## 2. Core Description

**One-sentence summary:** The player navigates platformer levels as a zombie whose sanity constantly drains, eating brain pickups to restore sanity while avoiding threats that deal HP damage, reaching the level exit before sanity or HP hits zero.

**Core Loop of This Mechanic:**

1. **Explore/Navigate** — Player moves through the level toward the exit using platformer movement (run, jump). Movement feel changes continuously with sanity: faster/higher but less controllable as sanity drops.
2. **Find Brains** — Player identifies brain pickup locations. Brains are placed at decision points — some on safe paths, others near threats or requiring risky jumps.
3. **Eat Brain** — Player walks into a brain to consume it. Sanity restores by `BRAIN_SANITY_RESTORE` (capped at max). The brain disappears. The player must decide: eat now (waste potential if sanity is high) or save it for later (risk dying before returning)?
4. **Avoid/Evade Threats** — Threats patrol platforms and chase the player on detection. Contact deals HP damage. Player chooses: dodge around (costs time/sanity), take the hit and push through (costs HP), or find an alternate route (costs time exploring).
5. **Reach Exit** — Player arrives at the level exit zone. Level complete. Stats displayed. Next level loads (or victory screen).

**Failure branches:**
- Sanity hits 0 at any point → "MIND LOST" → Game over
- HP hits 0 at any point → "GAME OVER" → Game over

**Frequency:** The full loop executes once per level (2-5 minutes). Within a level, sub-loops repeat continuously:
- Navigate: constant (every frame of play)
- Find/eat brain: 3-5 times per level (one per brain pickup)
- Threat encounter: 4-8 times per level (multiple threats, re-encounters during chase)
- Sanity check (implicit): every second (drain is continuous)

---

## 3. Input / Output

### 3.1 Movement / Exploration (Existing — Validated)

#### Inputs
| Input | Context | Timing |
|-------|---------|--------|
| Arrow keys / WASD (horizontal) | Always valid while alive | Continuous hold; acceleration-based |
| Space / Up / W (jump) | Valid when grounded or within coyote time (0.1s) | Tap for short jump, hold for full jump. Jump buffer: 0.1s |

#### Outputs (Success)
| Outcome | Description | Feedback |
|---------|-------------|----------|
| Horizontal movement | Zombie accelerates in input direction, up to sanity-scaled max speed | Squash/stretch on direction reversal, speed lines at >80% max speed, dust particles |
| Jump | Zombie launches upward at sanity-scaled jump velocity | Launch squash, dust burst, gravity pulls back down |
| Air control | Zombie steers mid-air at reduced rate (sanity-scaled) | Reduced responsiveness communicates air state |

#### Outputs (Failure)
| Outcome | Description | Feedback |
|---------|-------------|----------|
| Drift override | At low sanity, random impulses push the zombie off-course | Position nudge + velocity change, directional vignette flash, camera shake (bias toward drift direction) |
| Input delay | At very low sanity (below 6), direction reversal is delayed by up to 0.03s | Screen darkens 20%, ±1px jitter during delay |
| Fall into pit / wall collision | Zombie hits a wall or falls off a platform | Collision stops movement; falling continues until landing or death zone |

### 3.2 Brain Pickup (New — Sprint 1)

#### Inputs
| Input | Context | Timing |
|-------|---------|--------|
| Walk into brain (AABB overlap) | Brain must be active (not yet consumed) | Instant on overlap |

#### Outputs (Success)
| Outcome | Description | Feedback |
|---------|-------------|----------|
| Sanity restored | `zombie.sanity += BRAIN_SANITY_RESTORE`, capped at `ZOMBIE_MAX_SANITY` (12) | Brain disappears, brief flash/pulse at pickup location. Sanity bar visibly jumps. Movement feel shifts toward Lucid (smoother, more controlled). |

#### Outputs (Failure)
| Outcome | Description | Feedback |
|---------|-------------|----------|
| Sanity already full | Sanity is at or near 12; restore amount is partially or fully wasted | Brain still consumed — no saving it. Sanity bar barely moves. This IS the failure state: eating too early wastes the resource. Player learns to wait. |

### 3.3 Threat Avoidance (New — Sprint 2)

#### Inputs
| Input | Context | Timing |
|-------|---------|--------|
| Movement (all movement inputs) | Always; threat avoidance IS movement | Continuous |
| Positioning relative to threats | Player chooses paths that avoid threat patrol routes and detection ranges | Strategic; decided over seconds, not frames |

#### Outputs (Success)
| Outcome | Description | Feedback |
|---------|-------------|----------|
| Dodge/evade | Player passes through threat area without contact | No damage taken. Threat may chase but player outruns or breaks line-of-sight. Clean pass feels skillful. |

#### Outputs (Failure)
| Outcome | Description | Feedback |
|---------|-------------|----------|
| Take damage | Threat contacts zombie | HP decreases by `THREAT_DAMAGE`. Zombie enters invincibility window (flashing, `INVINCIBILITY_DURATION`). Screen shake. HP display decreases. Player learns the threat's pattern. |
| Death (HP) | HP reaches 0 from threat damage | "GAME OVER" overlay. Game stops. Press R to restart. |

### 3.4 Level Exit (New — Sprint 2)

#### Inputs
| Input | Context | Timing |
|-------|---------|--------|
| Walk into exit zone (AABB overlap) | Exit zone is always active | Instant on overlap |

#### Outputs (Success)
| Outcome | Description | Feedback |
|---------|-------------|----------|
| Level complete | Game state transitions to "Level Complete" | Overlay: "LEVEL COMPLETE." Stats displayed (time, brains eaten, HP remaining). Press Enter to continue / Press R to replay. |

#### Outputs (Failure)
| Outcome | Description | Feedback |
|---------|-------------|----------|
| N/A | There is no way to fail at the exit. Getting there IS the challenge. | — |

---

## 4. Variables and Parameters

All defaults are grounded in GDD paper prototype data (27,000 Monte Carlo simulations) and adapted from turn-based to real-time.

### 4.1 Sanity System

| Parameter | Default Value | Range | Description | Balance Notes |
|-----------|--------------|-------|-------------|---------------|
| `ZOMBIE_MAX_SANITY` | 12 | 10-15 | Maximum sanity value | GDD validated: 12 gives ~13 rounds turn-based. Starting sanity = max. |
| `SANITY_DRAIN_RATE` | 0.15 /s | 0.1-0.4 | Sanity lost per second (continuous) | Paper prototype: 0.9/round. At ~5s/round equivalent, 0.15-0.2/s gives 60-80s Lucid→Gone. Start conservative (0.15 = 80s); tune via task 1.6. |
| `BRAIN_SANITY_RESTORE` | 4 | 3-6 | Sanity restored per brain consumed | GDD validated: 4 = 33% of max. 3 brains = full recovery. Makes each brain meaningful but not game-changing. |

### 4.2 Health System

| Parameter | Default Value | Range | Description | Balance Notes |
|-----------|--------------|-------|-------------|---------------|
| `ZOMBIE_MAX_HP` | 5 | 3-8 | Maximum hit points | GDD validated: 5 HP. Survives 5 hits from standard threats. |
| `ZOMBIE_START_HP` | 5 | 3-8 | HP at level start | Equals max for v1. Per Sprint Backlog design decision: HP carries over between levels (future milestone). |
| `INVINCIBILITY_DURATION` | 0.75 | 0.5-1.5 | Seconds of iframes after taking damage | Not in GDD (turn-based didn't need iframes). 0.75s is standard platformer convention — long enough to escape, short enough to punish standing still. |

### 4.3 Threat System

| Parameter | Default Value | Range | Description | Balance Notes |
|-----------|--------------|-------|-------------|---------------|
| `THREAT_SPEED` | 120 /s | 80-200 | Patrol movement speed (px/s) | Zombie Lucid max is 300. Threat at 120 is easily outrun. At Feral (510 px/s), threats are trivial to outrun — the challenge is control, not speed. |
| `THREAT_CHASE_SPEED` | 200 /s | 150-300 | Chase movement speed (px/s) | Faster than patrol but still slower than the zombie at any sanity. The threat's advantage is positioning, not speed. |
| `THREAT_DAMAGE` | 1 | 1-2 | HP damage per hit | GDD: all combat deals 1 damage. Keeps math simple. At 5 HP, player can take 5 hits — forgiving enough to learn patterns. |
| `THREAT_DETECTION_RANGE` | 160 | 96-256 | Pixels within which threat detects player and starts chasing | GDD: hunter aggro range is 5 tiles. At 32px/tile, that's 160px. Tune to match how "aware" threats feel. |
| `THREAT_PATROL_RANGE` | 128 | 64-256 | Pixels of patrol distance (half left, half right from spawn) | 4 tiles of patrol gives enough movement to feel alive without covering too much ground. |

### 4.4 Level Structure

| Parameter | Default Value | Range | Description | Balance Notes |
|-----------|--------------|-------|-------------|---------------|
| `BRAIN_COUNT` | 4 | 3-6 | Number of brain pickups per level | At 4 brains × 4 sanity each = 16 potential sanity restore. With 12 max and 80s drain, player has surplus IF they collect all. Collecting 60-80% (3 brains) should be enough to survive. |
| `THREAT_COUNT` | 3 | 2-5 | Number of threats per level | Start with 3 for the vertical slice. Enough to create obstacles without overwhelming. Scale up in later levels. |
| `LEVEL_TARGET_TIME` | 120-300 | 60-600 | Expected completion time in seconds | 2-5 minutes per level. At 0.15/s drain, 80s of drain budget. With 3-4 brain pickups, effective budget extends to ~160-200s. |

---

## 5. Edge Cases

| # | Scenario | Expected Behavior | Priority |
|---|----------|-------------------|----------|
| 1 | **Sanity hits 0 while airborne** | Game over triggers immediately. Zombie freezes in place (or falls to ground first, then overlay). Do not allow any further input. The "MIND LOST" overlay appears. | Must-handle |
| 2 | **Eat brain at sanity 12 (already full)** | Brain is still consumed (disappears). Sanity stays at 12. This is the player's mistake — no partial-consume or "save for later" mechanic. The waste teaches timing. | Must-handle |
| 3 | **HP hits 0 and sanity hits 0 on the same frame** | HP death takes priority (show "GAME OVER" not "MIND LOST"). Process HP check before sanity check in the game loop. Rationale: HP death is from an active threat — more dramatic and informative than passive sanity drain. | Must-handle |
| 4 | **Threat pushes zombie into wall** | Threats do not apply knockback in v1. Damage is dealt, invincibility starts, but zombie position is unchanged. The zombie occupies the same space as the threat during iframes. No physics interaction between zombie and threats beyond the damage trigger. | Must-handle |
| 5 | **Multiple threats overlap zombie during invincibility** | Only one damage event per invincibility window. Additional threats contacting the zombie during iframes deal zero damage. iframes protect against ALL sources. | Must-handle |
| 6 | **Brain spawns inside a wall or unreachable location** | Brains are hand-placed (not randomly spawned) for v1. Level designer ensures reachability. If using generated levels, BFS reachability check (already exists in generator) should validate brain positions. | Must-handle |
| 7 | **Player reaches exit with very low sanity (e.g., 0.1)** | Level complete triggers normally. Any non-zero sanity is alive. The "just barely made it" moment is a peak experience — do not rob the player of it. | Must-handle |
| 8 | **Threat walks off platform edge** | Threats reverse direction at platform edges (ground check ahead). They never fall off. This is handled in Sprint 2 task 2.6. | Must-handle |
| 9 | **Zombie takes damage while eating a brain (same frame)** | Both events process: sanity restores AND HP decreases. Eating does not grant iframes — only taking damage does. The player chose to go for the brain in a dangerous spot. | Nice-to-handle |
| 10 | **Sanity drain triggers tier transition mid-jump** | Sanity interpolates continuously — there are no hard tier transitions for movement. The jump arc smoothly adjusts as sanity changes (gravity and air control shift in real-time). Drift parameters shift smoothly too. This already works in the movement toy. | Nice-to-handle |
| 11 | **Player pauses game during invincibility** | Invincibility timer pauses with the game. Iframes resume on unpause. No free iframe exploits from pause-spamming. | Nice-to-handle |
| 12 | **All brains collected but sanity still draining** | Expected gameplay. The player must reach the exit on remaining sanity. No additional brains spawn. This creates the final-stretch tension. | Won't-handle (by design) |
| 13 | **Threat chases player into a different section of the level** | Threats have no leash in v1. If chase causes issues, add a leash distance in tuning. For now, a persistent chaser adds tension and rewards outrunning them. | Won't-handle (evaluate in playtest) |

---

## 6. Interaction with Other Mechanics

| Other Mechanic | Interaction Type | Description |
|---------------|-----------------|-------------|
| **Movement interpolation (sanity → speed/accel/jump)** | Enabling | Sanity drain directly changes how the zombie moves in real-time. As sanity drops, the zombie gets faster and jumpier — the core loop's time pressure IS the movement system's input. Eating a brain immediately shifts movement feel back toward Lucid. The player FEELS the loop in their hands. |
| **Drift system (sanity → impulse/interval)** | Complementary | Drift makes low-sanity movement dangerous. In the core loop, drift transforms threat avoidance from a simple dodge into a high-stakes gamble: at Feral, you're fast enough to outrun anything but drift might throw you INTO a threat. The t² impulse curve (from SANITY_BALANCE_REPORT) means drift cost scales proportionally with power gain — no free zones, no cliffs. |
| **Input delay (sanity → direction reversal lag)** | Complementary | Input delay at low sanity makes threat avoidance harder. Reversing direction to dodge a threat takes 0.03s longer at Feral — enough to matter in a tight encounter. Combined with drift, this creates the "losing control" fantasy without removing player agency entirely. |
| **Squash/stretch + particles (juice system)** | Complementary | Juice feedback communicates loop state: heavy landing shake after a desperate Feral jump to reach a brain, dust clouds from a panicked reversal away from a threat. The juice layer makes the loop's tension visible and visceral. |
| **Sanity vignette + visual effects** | Complementary | The vignette (already implemented) communicates sanity state passively. In the core loop, a reddening screen tells the player "find a brain soon" without requiring them to read a number. The vignette IS the loop's urgency signal. |
| **Level generator pacing (interest curve)** | Enabling | The pacing system (hook → rising → valley → climax → resolution) maps directly to loop pacing: brains should appear at valley beats (rest/reward), threats cluster at climax beats (peak tension), the exit is the resolution. Brain and threat placement should follow the level's pacing curve. |

**Emergent Behaviors:**

- **Desperation jump**: Player is at low sanity, spots a brain on a high platform. Feral jump can reach it but drift might push them off. Lucid can't reach it. The player must decide: risk the Feral jump or run for the exit on fumes. This emerges from sanity drain + movement interpolation + brain placement.
- **Threat corridor sprint**: A narrow passage has a patrolling threat. At Lucid speed (300 px/s), timing the gap is tight. At Feral speed (510 px/s), the player blows through before the threat can react — but drift might slam them into a wall mid-sprint. Speed vs. control tradeoff emerges naturally.
- **Accidental brain waste**: Player eats a brain at sanity 10, restoring only 2 points (cap at 12). Moments later, sanity drain puts them at 6. If they'd waited, the brain would have restored the full 4. The timing economy emerges from fixed drain + fixed restore + positioning.

---

## 7. Success Metrics

| Metric | Target | How to Measure | Red Flag |
|--------|--------|---------------|----------|
| **Average level completion time** | 2-5 minutes | Timer from level start to exit (or death). Average across 10+ plays. | Below 1 min = level too short or too easy. Above 7 min = level too long or sanity drain too harsh. |
| **Brain collection rate** | 60-80% of available brains per level | `brains_eaten / brains_placed` per level, averaged across sessions. | Below 40% = brains are too dangerous/inaccessible, or drain is too slow (no need to eat). Above 95% = brains are too easy to collect (no routing decisions). |
| **Death cause distribution** | HP deaths and sanity deaths roughly even (40-60% each) | Track `death_cause` per game over. Aggregate across 20+ deaths. | HP deaths > 80% = threats too punishing or drain too slow. Sanity deaths > 80% = drain too fast or brains too scarce. One cause dominating means one system overshadows the other. |
| **Sanity curve shape** | Oscillating (sawtooth) — drops then spikes on brain eat, drops again | Plot sanity over time for a full level. | Monotonically decreasing = player never eats (brains too scarce, too risky, or drain too slow to matter). Flat at 12 = drain too slow. Flat at low values = brains too common. |
| **Threat encounter survival rate** | 70-85% of threat encounters result in no damage | `clean_dodges / total_threat_encounters` per level. | Below 50% = threats too hard to avoid (detection range too wide, speed too fast). Above 95% = threats too easy (no threat). |
| **Route choice variance** | Players take different paths on repeated plays | Observation during playtesting — do players explore, or beeline? | All players take the same path every time = brain/threat placement creates a dominant route. Needs spatial redesign. |
| **Player sentiment** | "I died but I know why, and I want to try again" | Playtest observation and verbal feedback. | "That felt random" = drift too strong or threats too unpredictable. "That was unfair" = damage without visible cause. "I was bored" = drain too slow or threats too passive. |

---

## 8. Implementation Notes

### Technical Considerations

- **Sanity drain is continuous, not per-frame.** Use `zombie.sanity -= SANITY_DRAIN_RATE * dt` in the game loop (after physics, before render). The existing `dt` time-step ensures frame-rate independence.
- **Brain pickup uses existing AABB collision.** The zombie's collision system is proven. Brain collision is a simple overlap check — no new physics needed.
- **Threat AI state machine is new code.** Patrol and chase require a state machine pattern not currently in the codebase. Keep it simple: `PATROL` ↔ `CHASE` with distance-based transitions.
- **Invincibility window uses a timer pattern.** `zombie.invincibleTimer` counts down from `INVINCIBILITY_DURATION`. While > 0, skip damage. Render as flashing (alternate visible/invisible every 0.1s).
- **Game state management will expand.** Currently the game has two states: playing and Gone. Sprint 1-2 adds: playing, game_over_hp, game_over_sanity, level_complete. A simple state variable with switch-based logic suffices for v1.

### Prototyping Priority

**When to prototype:** Sprint 1 (immediate) — the core loop's resource management (drain + restore) is the #1 risk. Threat AI follows in Sprint 2.

**Minimum prototype (Sprint 1):** Sanity drains. Brains restore. HP exists and can be damaged (via console command). Game over states work. All on the existing Gauntlet map.

**Key question the prototype must answer:** "Does the sanity drain + brain restore cycle create urgency without desperation? Does the player WANT to find brains?"

### Sprint Backlog Mapping

| Spec Element | Sprint | Tasks | Dependencies |
|-------------|--------|-------|-------------|
| Sanity drain | Sprint 1 | 1.1 (drain), 1.6 (rate tuning), 1.7 (slider integration) | None |
| Health + damage | Sprint 1 | 1.2 (health system), 1.3 (game over HP=0) | None |
| Brain pickups | Sprint 1 | 1.4 (brain entity), 1.5 (collision + restore) | 1.1, 1.4 |
| Threat entity + patrol | Sprint 2 | 2.2 (data structure), 2.3 (patrol), 2.6 (tile collision) | None |
| Threat chase | Sprint 2 | 2.4 (chase behavior) | 2.3 |
| Threat damage | Sprint 2 | 2.5 (collision damage) | 2.2, 1.2 |
| Level entrance/exit | Sprint 2 | 2.7 (zones), 2.8 (level complete state) | None |
| Threat interaction model | Sprint 2 | 2.1 (design decision) | None (but shapes 2.5) |
| Level assembly + balance | Sprint 3 | 3.1-3.2 (level design), 3.5-3.6 (tuning) | All Sprint 1-2 |
| Full integration test | Sprint 3 | 3.7 (playtest) | All above |

### Dependencies

- **Requires:** Frame-rate independent game loop (exists)
- **Requires:** AABB collision system (exists)
- **Requires:** Sanity tier system with smooth interpolation (exists)
- **Requires:** Tuning panel infrastructure (exists — add new sliders)
- **Blocked by:** Nothing — all prerequisites from the movement toy are met

### Known Risks

- **Real-time drain rate is uncharted.** Paper prototype validated 0.9/round, but "a round" has no fixed real-time equivalent. The drain rate will need hands-on tuning (task 1.6). Start with 0.15/s and adjust.
- **Threat AI complexity creep.** Simple patrol + chase is sufficient for the vertical slice. Resist adding patrol patterns, alert states, or group behavior until the basic loop is proven fun.
- **Brain placement is a level design skill.** Badly placed brains break the loop regardless of systems tuning. Brains must create routing decisions (safe path vs. risky path) or the loop degenerates into "collect everything and beeline to exit."

---

## Appendix A: Lens #33 Analysis — Meaningful Choices

For each player decision point, the following analysis documents what makes the choice meaningful (or identifies where a dominant strategy might exist).

### Decision Point 1: Route Choice

**The choice:** Safe path (no brains, fewer threats) vs. risky path (brains near threats, tighter platforms).

| Question | Answer |
|----------|--------|
| What are the options? | (A) Take the safe path — avoid threats, conserve HP, but no sanity restoration. (B) Take the risky path — pass near threats, potentially take damage, but collect brains to restore sanity. |
| What information does the player have? | They can see brain locations, threat positions, and their current sanity/HP. They can estimate whether they need a brain to survive to the exit. |
| What are the consequences? | Safe path: HP preserved, sanity continues draining, may not have enough sanity to reach exit. Risky path: may lose HP, but sanity restored, movement feel improves (back toward Lucid). |
| Is the choice interesting? | **Yes, if calibrated.** The choice degenerates if: (1) drain is so slow that brains aren't needed (safe path always wins), or (2) drain is so fast that every brain is mandatory (risky path always wins). At the target drain rate (0.15/s), the player should SOMETIMES need brains and SOMETIMES be able to skip them, depending on how much sanity they've already spent. |
| Dominant strategy risk | If brain positions are all on the main path, there's no route choice. **Mitigation:** Place at least 1-2 brains on branching paths that require a detour. |

### Decision Point 2: When to Eat

**The choice:** Eat the brain now (potentially wasting sanity if near full) vs. save it for later (risk dying before returning).

| Question | Answer |
|----------|--------|
| What are the options? | (A) Eat immediately — guaranteed sanity restore, but may waste some if sanity is high. (B) Pass by now, come back later — full value restore, but may die or get cut off by threats before returning. |
| What information does the player have? | Current sanity (visible on HUD + vignette feel), brain position, distance to exit, threat positions. |
| What are the consequences? | Eat now: up to `BRAIN_SANITY_RESTORE` (4) wasted if sanity > 8. Later: full value, but brain may be inaccessible (threats repositioned, sanity too low to backtrack safely). |
| Is the choice interesting? | **Yes — classic risk/reward.** Note: in v1, brains are consumed on contact (walk-into), so "saving" a brain means deliberately not walking over it. This is spatial — the player routes AROUND brains they want to save. The physicality of the choice makes it engaging. |
| Dominant strategy risk | If levels are linear with no backtracking, "eat now" always wins (can't return). **Mitigation:** At least one brain per level should be accessible from multiple directions, enabling a save-for-later strategy. |

### Decision Point 3: Sanity Management

**The choice:** Stay Lucid (precise control, weaker movement) vs. let sanity drop (powerful but chaotic movement).

| Question | Answer |
|----------|--------|
| What are the options? | (A) Eat brains aggressively to stay in Lucid range (sanity 7-12) — full control, weaker jump/speed. (B) Let sanity drop into Slipping/Feral — faster movement, higher jumps, but drift and input delay increase. |
| What information does the player have? | Current sanity (HUD + vignette + movement feel). Upcoming level geometry (can see platforms and gaps ahead). Knowledge of whether upcoming sections require Feral-level movement to traverse. |
| What are the consequences? | Lucid: safe but slower. Cannot reach some platforms. Slipping: faster, occasional drift nudges (manageable). Feral: fastest, highest jumps, but drift is scary (t² impulse curve means Feral drift displaces 1.5-2.8 tiles per event). |
| Is the choice interesting? | **Yes — the SANITY_BALANCE_REPORT validated this.** After the smooth drift fix, every sanity level has proportional cost. There's no free zone (old Lucid 7-12 was cost-free) and no cliff (old Feral boundary was 28× cost jump). The t² drift curve means the choice is granular, not binary. |
| Dominant strategy risk | Pre-fix, sanity 7 was strictly dominant (max free power). Post-fix, the smooth drift curve eliminates this. Ongoing risk: if drain is too slow, player stays Lucid always. **Mitigation:** Ensure at least one section per level requires Slipping-tier movement to traverse optimally. |

### Decision Point 4: Fight or Flee

**The choice:** Engage the threat (if stomp mechanic exists) vs. dodge around it vs. find an alternate path.

| Question | Answer |
|----------|--------|
| What are the options? | (A) Jump on the threat (stomp-to-kill, if implemented per Sprint 2 design decision 2.1). (B) Dodge past the threat using speed/timing. (C) Find an alternate route that avoids the threat entirely. |
| What information does the player have? | Threat position and patrol pattern (observable). Own HP. Own sanity (affects speed/control for dodging). Level geometry (visible platforms). |
| What are the consequences? | Stomp: threat eliminated (path cleared permanently), but requires precise positioning — missed stomp means taking damage. Dodge: threat survives, may chase, but no HP risked if successful. Alternate route: safe but costs time (sanity drain continues). |
| Is the choice interesting? | **Depends on the interaction model decision (Sprint 2, task 2.1).** If pure avoidance (option B from backlog): choice is between dodge and alternate route — still interesting but simpler. If stomp-to-kill or sanity-dependent stomp: adds a risk/reward layer. The recommended option (C: sanity-dependent stomp) ties directly into sanity management — you must be at Feral to stomp, adding a "drop sanity to fight" decision. |
| Dominant strategy risk | If stomping is free and easy, it dominates. **Mitigation:** If stomp is implemented, require precise vertical positioning (above threat's midpoint while falling). At Feral where stomp is available, drift makes precision harder — the power/control tradeoff applies to combat too. |

### Decision Point 5: Exploration vs. Speed

**The choice:** Explore the level for more brains vs. rush to the exit.

| Question | Answer |
|----------|--------|
| What are the options? | (A) Explore — find all brains, learn level layout, maximize resources. (B) Rush — head straight for the exit, skip optional brains, race the sanity clock. |
| What information does the player have? | Visible portion of the level (can see brains and threats in view). Current sanity (are they desperate?). Whether they've collected enough brains to survive to the exit. |
| What are the consequences? | Explore: more brains collected (higher sanity buffer), but more sanity spent on travel time. More threat encounters. Rush: less sanity spent on travel, but no safety buffer. One mistake (take damage, miss a jump) could be fatal. |
| Is the choice interesting? | **Yes — this is the macro version of the core tension.** The sanity drain makes time a cost. Exploration costs time (= sanity), but the reward is more sanity (from brains). The break-even depends on brain placement density and distance. With 4 brains restoring 4 sanity each and a drain of 0.15/s, a 10-second detour for a brain costs 1.5 sanity but gains 4 — net positive. A 30-second detour costs 4.5 — barely worth it. The math creates a natural exploration radius. |
| Dominant strategy risk | If brains are always close to the main path, exploration is always worth it. **Mitigation:** Place 1-2 brains far from the main route so the detour cost is genuinely high. The player must evaluate whether they need it. |

---

## Appendix B: Sanity Economy Model

Reference model for how the sanity drain + restore cycle plays out over a level.

**Assumptions:** `SANITY_DRAIN_RATE` = 0.15/s, `BRAIN_SANITY_RESTORE` = 4, `ZOMBIE_MAX_SANITY` = 12, 4 brains available, level takes 180s (3 min).

```
Time (s)  Event              Sanity   Tier Transition
────────  ─────              ──────   ───────────────
  0       Level start        12.0     Lucid
 33       (drain only)        7.0     → Slipping onset (drift begins ramping)
 40       (drain only)        6.0
 45       Eat Brain #1       10.0 → Lucid feel returns
 53       (drain only)        8.8
 80       (drain only)        5.7     drift noticeable
 90       Eat Brain #2        9.7 → back to mild drift
120       (drain only)        5.2     Slipping feel
130       Eat Brain #3        9.2
160       (drain only)        4.7     drift getting serious
170       Skip Brain #4       4.7     (player rushes for exit)
180       Reach exit           3.2     Feral! Barely made it.
```

**Total sanity budget:** 12 (start) + 12 (3 brains × 4) = 24 sanity units
**Total sanity cost:** 180s × 0.15/s = 27 sanity units
**Deficit:** -3 → player MUST eat at least 1 brain. With 3 brains, has a 9-unit buffer. With all 4, has a 13-unit buffer (comfortable).

This model confirms: at 0.15/s drain with 4 brains, the player has meaningful choices about which brains to collect without guaranteed death if they miss some. The 60-80% brain collection target (3 of 4) leaves a comfortable margin.

---

## 9. Revision History

| Date | Version | Change | Reason |
|------|---------|--------|--------|
| Feb 2026 | 1.0 | Initial specification | Pre-Sprint 1 design lock. Documents the core loop for the vertical slice milestone. |
