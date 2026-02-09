# Mechanic Specification: Core Game Loop

**Game:** Brains for Breakfast
**Author:** Brent Burgess
**Version:** 1.1
**Date:** Feb 2026
**Status:** Proposed

---

## 1. Player Fantasy

**What the player gets to feel:** The tension of a gentleman zombie — powerful, dignified, and losing his mind. You feel the pull between maintaining composure (Lucid: precise, controlled, safe) and unleashing raw undead power (Feral: fast, dangerous, unpredictable). Every civilian you eat buys another moment of clarity — but the scream attracts danger. Every second without food is a step toward becoming the monster you're fighting not to be.

**Power fantasy:** You are stronger than the things that hunt you — if you're willing to give up control. At Feral, you outrun guards, leap gaps they can't cross, and (potentially) stomp them flat. The power is always available. The cost is always real.

**Skill expression:** A beginner plays safe — stays Lucid, eats the unguarded civilians first, avoids all guards. An expert manages the sanity curve: lures a guardaway from a civilian then circles back to eat, deliberately drops to Slipping for a speed boost through a guardcorridor, threads a Feral jump through a gap that Lucid can't reach, and knows when to stop eating and sprint for the exit. The expert's zombie looks reckless but is precisely controlled chaos.

---

## 2. Core Description

**One-sentence summary:** The player navigates platformer levels as a zombie whose sanity constantly drains, eating civilians to restore sanity while avoiding guardsthat guard those civilians, reaching the level exit before sanity or HP hits zero.

**Core Loop of This Mechanic:**

1. **Explore/Navigate** — Player moves through the level toward the exit using platformer movement (run, jump). Movement feel changes continuously with sanity: faster/higher but less controllable as sanity drops.
2. **Find Civilians** — Player identifies civilian locations. Some civilians are unguarded (safe to eat), others are near guards(risky). Civilians flee when the zombie approaches, requiring the player to chase or corner them.
3. **Eat Civilian** — Player overlaps a civilian to consume it. Sanity restores by `SANITY_PER_EAT` (capped at max). The civilian disappears and a **death scream** alerts all guardswithin `SCREAM_ALERT_RANGE`, creating a burst of danger. The player must decide: eat now (risk scream alerting guards) or skip this one (risk running out of sanity)?
4. **Avoid/Evade Guards** — Guards patrol near civilians they guard (leash behavior) and chase the player on detection or scream alert. Contact deals HP damage. Player chooses: dodge around (costs time/sanity), take the hit and push through (costs HP), lure the guardaway then circle back to eat, or find an alternate route.
5. **Reach Exit** — Player arrives at the level exit zone. Level complete. Stats displayed. Next level loads (or victory screen).

**Failure branches:**
- Sanity hits 0 at any point → "MIND LOST" → Game over
- HP hits 0 at any point → "GAME OVER" → Game over

**Frequency:** The full loop executes once per level (2-5 minutes). Within a level, sub-loops repeat continuously:
- Navigate: constant (every frame of play)
- Find/eat civilian: 3-5 times per level (one per civilian)
- Guard encounter: 4-8 times per level (multiple guards, re-encounters during chase, scream responses)
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

### 3.2 Civilian Consumption (New — Sprint 1)

#### Inputs
| Input | Context | Timing |
|-------|---------|--------|
| Walk into civilian (AABB overlap) | Civilian must be alive. Civilian flees when zombie is within 96px, requiring pursuit or cornering. | Instant on overlap — but civilian is moving away |

#### Outputs (Success)
| Outcome | Description | Feedback |
|---------|-------------|----------|
| Sanity restored | `zombie.sanity += SANITY_PER_EAT` (4), capped at `ZOMBIE_MAX_SANITY` (12) | Civilian disappears. Sanity bar visibly jumps. Movement feel shifts toward Lucid. |
| Death scream triggered | All guardswithin `SCREAM_ALERT_RANGE` (128px) become alerted for `SCREAM_ALERT_DURATION` (3s) | Audio/visual scream indicator. Alerted guardschange behavior (rush toward scream location). Player sees guardsconverging — time to run. |
| Guard guard released | The guardthat was guarding this civilian reassigns to the nearest remaining civilian | Guard patrol path visibly changes. Remaining civilians become more heavily guarded. |

#### Outputs (Failure)
| Outcome | Description | Feedback |
|---------|-------------|----------|
| Sanity already full | Sanity is at or near 12; restore amount is partially or fully wasted | Civilian still consumed — death scream still fires. Eating at full sanity wastes the food AND triggers danger. Double penalty for poor timing. |
| Civilian escapes | Civilian flees to a position the zombie can't easily reach (e.g., near a guardcluster) | Civilian is still alive but now in a more dangerous location. Chase cost time (= sanity). Player must decide: keep pursuing or give up. |

### 3.3 Guard Avoidance (New — Sprint 2)

#### Inputs
| Input | Context | Timing |
|-------|---------|--------|
| Movement (all movement inputs) | Always; guardavoidance IS movement | Continuous |
| Positioning relative to guards| Player chooses paths that avoid guardpatrol routes and detection ranges | Strategic; decided over seconds, not frames |

#### Outputs (Success)
| Outcome | Description | Feedback |
|---------|-------------|----------|
| Dodge/evade | Player passes through guardarea without contact | No damage taken. Guard may chase but player outruns or breaks line-of-sight. Clean pass feels skillful. |

#### Outputs (Failure)
| Outcome | Description | Feedback |
|---------|-------------|----------|
| Take damage | Guard contacts zombie | HP decreases by `GUARD_DAMAGE`. Zombie enters invincibility window (flashing, `INVINCIBILITY_DURATION`). Screen shake. HP display decreases. Player learns the guard's pattern. |
| Death (HP) | HP reaches 0 from guarddamage | "GAME OVER" overlay. Game stops. Press R to restart. |

### 3.4 Level Exit (New — Sprint 2)

#### Inputs
| Input | Context | Timing |
|-------|---------|--------|
| Walk into exit zone (AABB overlap) | Exit zone is always active | Instant on overlap |

#### Outputs (Success)
| Outcome | Description | Feedback |
|---------|-------------|----------|
| Level complete | Game state transitions to "Level Complete" | Overlay: "LEVEL COMPLETE." Stats displayed (time, civilians eaten, HP remaining). Press Enter to continue / Press R to replay. |

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
| `SANITY_PER_EAT` | 4 | 3-6 | Sanity restored per civilian consumed | GDD validated: 4 = 33% of max. 3 eats = full recovery. Makes each civilian meaningful but not game-changing. |

### 4.2 Health System

| Parameter | Default Value | Range | Description | Balance Notes |
|-----------|--------------|-------|-------------|---------------|
| `ZOMBIE_MAX_HP` | 5 | 3-8 | Maximum hit points | GDD validated: 5 HP. Survives 5 hits from standard guards. |
| `ZOMBIE_START_HP` | 5 | 3-8 | HP at level start | Equals max for v1. Per Sprint Backlog design decision: HP carries over between levels (future milestone). |
| `INVINCIBILITY_DURATION` | 0.75 | 0.5-1.5 | Seconds of iframes after taking damage | Not in GDD (turn-based didn't need iframes). 0.75s is standard platformer convention — long enough to escape, short enough to punish standing still. |

### 4.3 Guard System

| Parameter | Default Value | Range | Description | Balance Notes |
|-----------|--------------|-------|-------------|---------------|
| `GUARD_SPEED` | 120 /s | 80-200 | Patrol movement speed (px/s) | Zombie Lucid max is 300. Guard at 120 is easily outrun. At Feral (510 px/s), guardsare trivial to outrun — the challenge is control, not speed. |
| `GUARD_CHASE_SPEED` | 200 /s | 150-300 | Chase movement speed (px/s) | Faster than patrol but still slower than the zombie at any sanity. The guard's advantage is positioning, not speed. |
| `GUARD_DAMAGE` | 1 | 1-2 | HP damage per hit | GDD: all combat deals 1 damage. Keeps math simple. At 5 HP, player can take 5 hits — forgiving enough to learn patterns. |
| `GUARD_DETECTION_RANGE` | 160 | 96-256 | Pixels within which guarddetects player and starts chasing | GDD: hunter aggro range is 5 tiles. At 32px/tile, that's 160px. Tune to match how "aware" guardsfeel. |
| `GUARD_PATROL_RANGE` | 128 | 64-256 | Pixels of patrol distance (half left, half right from spawn) | 4 tiles of patrol gives enough movement to feel alive without covering too much ground. Used when guardhas no civilian to guard. |
| `GUARD_WATCH_RANGE` | 64 | 32-128 | Pixels — how tightly guardpatrols near its guarded civilian | 2 tiles. Tight enough to make eating risky, loose enough to allow approach angles. |
| `GUARD_LEASH_DISTANCE` | 160 | 96-256 | Pixels — max chase distance from guarded civilian | 5 tiles. Guard breaks chase and returns if zombie lures it too far. Enables "lure and eat" strategy. |
| `SCREAM_ALERT_RANGE` | 128 | 64-256 | Pixels — death scream alert radius | 4 tiles (GDD validated). All guardswithin range become alerted when zombie eats a civilian. Primary tuning knob for eat danger. |
| `SCREAM_ALERT_DURATION` | 3.0 | 1.5-5.0 | Seconds guardsinvestigate after a scream | GDD: 2 rounds ≈ 3s. How long guardspursue the scream location before returning to guard. |

### 4.4 Civilian System

| Parameter | Default Value | Range | Description | Balance Notes |
|-----------|--------------|-------|-------------|---------------|
| `CIVILIAN_COUNT` | 5 | 3-7 | Number of civilians per level | At 5 civilians × 4 sanity each = 20 potential restore. With 3 guards, 2 civilians are unguarded (free) and 3 are guarded (risky). Player eating 4 of 5 (80%) gives comfortable margin. |
| `CIVILIAN_FLEE_SPEED` | 60 | 30-100 | Civilian movement speed when fleeing (px/s) | ~20% of zombie Lucid speed. Catchable but requires closing distance. |
| `CIVILIAN_FLEE_RANGE` | 96 | 64-160 | Pixels — distance at which civilian begins fleeing from zombie | 3 tiles (GDD). Early enough to require pursuit, close enough that approach is feasible. |
| `CIVILIAN_SEEK_RANGE` | 128 | 64-192 | Pixels — distance within which civilian drifts toward nearest guardfor protection | 4 tiles (GDD). Civilians naturally cluster near their guards. |
| `CIVILIAN_WANDER_SPEED` | 30 | 15-60 | Civilian idle movement speed (px/s) | Slow amble when no zombie guardnearby. Just enough to feel alive. |

### 4.5 Level Structure

| Parameter | Default Value | Range | Description | Balance Notes |
|-----------|--------------|-------|-------------|---------------|
| `GUARD_COUNT` | 3 | 2-5 | Number of guardsper level | Start with 3 for the vertical slice. Each guards its nearest civilian. Enough to create obstacles without overwhelming. Scale up in later levels. |
| `LEVEL_TARGET_TIME` | 120-300 | 60-600 | Expected completion time in seconds | 2-5 minutes per level. At 0.15/s drain, 80s of drain budget. With 4 civilians eaten (×4 sanity), effective budget extends to ~180-200s. |

---

## 5. Edge Cases

| # | Scenario | Expected Behavior | Priority |
|---|----------|-------------------|----------|
| 1 | **Sanity hits 0 while airborne** | Game over triggers immediately. Zombie freezes in place (or falls to ground first, then overlay). Do not allow any further input. The "MIND LOST" overlay appears. | Must-handle |
| 2 | **Eat civilian at sanity 12 (already full)** | Civilian is still consumed (disappears). Sanity stays at 12. Death scream still fires. This is doubly punished — wasted food AND triggered danger. Teaches timing. | Must-handle |
| 3 | **HP hits 0 and sanity hits 0 on the same frame** | HP death takes priority (show "GAME OVER" not "MIND LOST"). Process HP check before sanity check in the game loop. Rationale: HP death is from an active guard— more dramatic and informative than passive sanity drain. | Must-handle |
| 4 | **Guard pushes zombie into wall** | Guards do not apply knockback in v1. Damage is dealt, invincibility starts, but zombie position is unchanged. The zombie occupies the same space as the guardduring iframes. No physics interaction between zombie and guardsbeyond the damage trigger. | Must-handle |
| 5 | **Multiple guardsoverlap zombie during invincibility** | Only one damage event per invincibility window. Additional guardscontacting the zombie during iframes deal zero damage. iframes protect against ALL sources. | Must-handle |
| 6 | **Civilian placed on unreachable platform** | Civilians are placed on platforms accessible to the zombie. Same-platform constraint between civilian and zombie spawn. Civilians can't jump, so they stay on their starting platform (flee along it, reverse at edges). BFS reachability check (exists in generator) validates civilian positions. | Must-handle |
| 7 | **Player reaches exit with very low sanity (e.g., 0.1)** | Level complete triggers normally. Any non-zero sanity is alive. The "just barely made it" moment is a peak experience — do not rob the player of it. | Must-handle |
| 8 | **Guard walks off platform edge** | Guards reverse direction at platform edges (ground check ahead). They never fall off. This is handled in Sprint 2 task 2.6. | Must-handle |
| 9 | **Zombie takes damage while eating a civilian (same frame)** | Both events process: sanity restores AND HP decreases. Eating does not grant iframes — only taking damage does. The player chose to eat in a dangerous spot. Death scream still fires. | Nice-to-handle |
| 10 | **Sanity drain triggers tier transition mid-jump** | Sanity interpolates continuously — there are no hard tier transitions for movement. The jump arc smoothly adjusts as sanity changes (gravity and air control shift in real-time). Drift parameters shift smoothly too. This already works in the movement toy. | Nice-to-handle |
| 11 | **Player pauses game during invincibility** | Invincibility timer pauses with the game. Iframes resume on unpause. No free iframe exploits from pause-spamming. | Nice-to-handle |
| 12 | **All civilians eaten but sanity still draining** | Expected gameplay. No special effect (no starvation mechanic). Guards with no civilian to guard default to standard patrol (no leash). Player must reach exit on remaining sanity. | Won't-handle (by design) |
| 13 | **Guard leash prevents chase** | Guard breaks chase when zombie exceeds `GUARD_LEASH_DISTANCE` (160px) from guarded civilian. Guard returns to guard. This is intentional — enables "lure and eat" strategy. If leash feels too restrictive, increase `GUARD_LEASH_DISTANCE`. | Won't-handle (evaluate in playtest) |
| 14 | **Civilian flees off platform** | Civilians reverse at platform edges (same as guards). Never fall off. Cornered civilians stop at walls — intentional design (cornering is a strategy). | Must-handle |
| 15 | **Death scream alerts guardalready chasing** | Alert overrides chase — guardmoves to scream location instead. After alert timer expires, guardreturns to guard behavior (reassigning if its civilian was the one eaten). | Nice-to-handle |

---

## 6. Interaction with Other Mechanics

| Other Mechanic | Interaction Type | Description |
|---------------|-----------------|-------------|
| **Movement interpolation (sanity → speed/accel/jump)** | Enabling | Sanity drain directly changes how the zombie moves in real-time. As sanity drops, the zombie gets faster and jumpier — the core loop's time pressure IS the movement system's input. Eating a civilian immediately shifts movement feel back toward Lucid. The player FEELS the loop in their hands. |
| **Drift system (sanity → impulse/interval)** | Complementary | Drift makes low-sanity movement dangerous. In the core loop, drift transforms guardavoidance from a simple dodge into a high-stakes gamble: at Feral, you're fast enough to outrun anything but drift might throw you INTO a guard. The t² impulse curve (from SANITY_BALANCE_REPORT) means drift cost scales proportionally with power gain — no free zones, no cliffs. |
| **Input delay (sanity → direction reversal lag)** | Complementary | Input delay at low sanity makes guardavoidance harder. Reversing direction to dodge a guardtakes 0.03s longer at Feral — enough to matter in a tight encounter. Combined with drift, this creates the "losing control" fantasy without removing player agency entirely. |
| **Civilian flee AI** | Complementary | Civilians flee when the zombie approaches, turning eating from a static pickup into a pursuit. At low sanity the zombie is faster (easier to catch) but less precise (drift may push past the civilian). At high sanity the zombie is precise but slower — may need to corner the civilian. Flee AI interacts with the movement interpolation to create sanity-dependent chase dynamics. |
| **Guard guard/leash behavior** | Enabling | Guards patrol near civilians they guard. Eating is inherently dangerous because food and danger are co-located. The leash mechanic means guardsreturn to guard posts after chasing, enabling the "lure and eat" strategy. As civilians die, guardsconsolidate — later eats are harder. The guard system is what makes the civilian economy work. |
| **Death scream** | Competing | Eating restores sanity (benefit) but triggers a scream that alerts guards(cost). This single mechanic creates the core risk/reward tension: every eat is a calculated decision about whether the sanity gain outweighs the danger response. See CIVILIAN_ECOLOGY_SPEC.md §4 for full analysis. |
| **Squash/stretch + particles (juice system)** | Complementary | Juice feedback communicates loop state: heavy landing shake after a desperate Feral jump to reach a civilian, dust clouds from a panicked reversal away from a scream-alerted guard. The juice layer makes the loop's tension visible and visceral. |
| **Sanity vignette + visual effects** | Complementary | The vignette (already implemented) communicates sanity state passively. In the core loop, a reddening screen tells the player "find food soon" without requiring them to read a number. The vignette IS the loop's urgency signal. |
| **Level generator pacing (interest curve)** | Enabling | The pacing system (hook → rising → valley → climax → resolution) maps directly to loop pacing: civilians should appear at valley beats (rest/reward), guardscluster at climax beats (peak tension), the exit is the resolution. Civilian and guardplacement should follow the level's pacing curve. |

**Emergent Behaviors:**

- **Lure and eat**: Player enters guarddetection range to pull a guard away from its civilian, then circles back to eat while the guardis chasing. Death scream fires but the lured guardis already far away. This emerges from leash + detection + scream range interaction.
- **Desperation jump**: Player is at low sanity, spots a civilian on a high platform. Feral jump can reach it but drift might push them off. Lucid can't reach it. The player must decide: risk the Feral jump or run for the exit on fumes. This emerges from sanity drain + movement interpolation + civilian placement.
- **Scream chain**: Player eats one civilian, sprints to another, eats that one too. Both screams alert overlapping sets of guards, creating a mob of pursuers. High-risk, high-reward "feeding frenzy" for massive sanity recovery.
- **Guard corridor sprint**: A narrow passage has a patrolling guard. At Lucid speed (300 px/s), timing the gap is tight. At Feral speed (510 px/s), the player blows through before the guardcan react — but drift might slam them into a wall mid-sprint. Speed vs. control tradeoff emerges naturally.
- **Guard consolidation**: After eating 3 of 5 civilians, all 3 guardsguard the remaining 2 civilians. The last civilians become mini-bosses — heavily guarded, extremely risky. The player must decide if the sanity is worth the danger.

---

## 7. Success Metrics

| Metric | Target | How to Measure | Red Flag |
|--------|--------|---------------|----------|
| **Average level completion time** | 2-5 minutes | Timer from level start to exit (or death). Average across 10+ plays. | Below 1 min = level too short or too easy. Above 7 min = level too long or sanity drain too harsh. |
| **Civilian eat rate** | 60-80% of civilians per level | `civilians_eaten / civilians_placed` per level, averaged across sessions. | Below 40% = civilians too dangerous/inaccessible, or drain too slow (no need to eat). Above 95% = eating too easy (guards ineffective, no routing decisions). |
| **Death cause distribution** | HP deaths and sanity deaths roughly even (40-60% each) | Track `death_cause` per game over. Aggregate across 20+ deaths. | HP deaths > 80% = guardstoo punishing or drain too slow. Sanity deaths > 80% = drain too fast or brains too scarce. One cause dominating means one system overshadows the other. |
| **Sanity curve shape** | Oscillating (sawtooth) — drops then spikes on eat, drops again | Plot sanity over time for a full level. | Monotonically decreasing = player never eats (civilians too risky, or drain too slow to matter). Flat at 12 = drain too slow. Flat at low values = civilians too accessible. |
| **Guard encounter survival rate** | 70-85% of guardencounters result in no damage | `clean_dodges / total_guard_encounters` per level. | Below 50% = guardstoo hard to avoid (detection range too wide, speed too fast). Above 95% = guardstoo easy (no guard). |
| **Route choice variance** | Players take different paths on repeated plays | Observation during playtesting — do players explore, or beeline? | All players take the same path every time = civilian/guard placement creates a dominant route. Needs spatial redesign. |
| **Player sentiment** | "I died but I know why, and I want to try again" | Playtest observation and verbal feedback. | "That felt random" = drift too strong or guardstoo unpredictable. "That was unfair" = damage without visible cause. "I was bored" = drain too slow or guardstoo passive. |

---

## 8. Implementation Notes

### Technical Considerations

- **Sanity drain is continuous, not per-frame.** Use `zombie.sanity -= SANITY_DRAIN_RATE * dt` in the game loop (after physics, before render). The existing `dt` time-step ensures frame-rate independence.
- **Civilian consumption uses existing AABB collision.** The zombie's collision system is proven. Civilian overlap check is simple — but civilians are moving targets (flee AI), so the check runs against a moving entity.
- **Civilian AI is new code.** Simple priority-based AI: flee from zombie > seek nearby guard> wander. Edge avoidance at platform edges. Lower complexity than guardAI.
- **Death scream is an event system.** When zombie eats, broadcast a scream event at the civilian's position. All guardswithin `SCREAM_ALERT_RANGE` subscribe and respond. Simple distance check — no line-of-sight needed.
- **Guard AI state machine is new code.** States: `GUARD_PATROL` ↔ `CHASE` ↔ `RESPOND_TO_SCREAM`. Guard patrol replaces free patrol when a civilian is assigned. Leash distance limits chase range. Reassignment triggers when guarded civilian dies.
- **Invincibility window uses a timer pattern.** `zombie.invincibleTimer` counts down from `INVINCIBILITY_DURATION`. While > 0, skip damage. Render as flashing (alternate visible/invisible every 0.1s).
- **Game state management will expand.** Currently the game has two states: playing and Gone. Sprint 1-2 adds: playing, game_over_hp, game_over_sanity, level_complete. A simple state variable with switch-based logic suffices for v1.

### Prototyping Priority

**When to prototype:** Sprint 1 (immediate) — the core loop's resource management (drain + restore) is the #1 risk. Guard AI follows in Sprint 2.

**Minimum prototype (Sprint 1):** Sanity drains. Civilians flee and can be eaten to restore sanity. HP exists and can be damaged (via console command). Game over states work. All on the existing Gauntlet map.

**Key question the prototype must answer:** "Does chasing and eating civilians feel satisfying? Does the death scream create tension? Does the player WANT to find food?"

### Sprint Backlog Mapping

| Spec Element | Sprint | Tasks | Dependencies |
|-------------|--------|-------|-------------|
| Sanity drain | Sprint 1 | 1.1 (drain), 1.6 (rate tuning), 1.7 (slider integration) | None |
| Health + damage | Sprint 1 | 1.2 (health system), 1.3 (game over HP=0) | None |
| Civilian entity + flee AI | Sprint 1 | 1.4 (civilian entity + AI), 1.8 (guard reassignment) | None |
| Civilian eat + death scream | Sprint 1 | 1.5 (eat collision + scream event) | 1.1, 1.4 |
| Guard entity + guard patrol | Sprint 2 | 2.2 (data structure + guard fields), 2.3 (guard patrol), 2.6 (tile collision) | 1.4 (civilians must exist for guard assignment) |
| Guard chase + scream response | Sprint 2 | 2.4 (chase + scream response behavior) | 2.3 |
| Guard damage | Sprint 2 | 2.5 (collision damage) | 2.2, 1.2 |
| Level entrance/exit | Sprint 2 | 2.7 (zones), 2.8 (level complete state) | None |
| Guard interaction model | Sprint 2 | 2.1 (design decision) | None (but shapes 2.5) |
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
- **Guard AI complexity creep.** Guard patrol + chase + scream response is the scope for the vertical slice. Resist adding group coordination, flanking, or inter-guard communication until the basic loop is proven fun.
- **Civilian placement is a level design skill.** Badly placed civilians break the loop regardless of systems tuning. Some civilians should be unguarded (safe, tutorial-like) and others guarded (risky). The ratio creates the eat-order decision. See CIVILIAN_ECOLOGY_SPEC.md §6 for population dynamics.

---

## Appendix A: Lens #33 Analysis — Meaningful Choices

For each player decision point, the following analysis documents what makes the choice meaningful (or identifies where a dominant strategy might exist).

### Decision Point 1: Route Choice

**The choice:** Safe path (no civilians, fewer guards) vs. risky path (civilians near guardguards, tighter platforms).

| Question | Answer |
|----------|--------|
| What are the options? | (A) Take the safe path — avoid guards, conserve HP, but no sanity restoration. (B) Take the risky path — enter guard-guarded areas, potentially take damage, but eat civilians to restore sanity. (C) Eat unguarded civilians first, then decide whether to risk guarded ones. |
| What information does the player have? | They can see civilian locations, guardguard positions, and their current sanity/HP. They can estimate which civilians are unguarded (safe) vs. guarded (risky). |
| What are the consequences? | Safe path: HP preserved, sanity continues draining, may not have enough sanity to reach exit. Risky path: may lose HP (guard damage + scream response), but sanity restored. The death scream adds cascading danger beyond just the immediate guard. |
| Is the choice interesting? | **Yes — and richer than static pickups.** The civilian system creates a natural free-first → guarded sequence. Unguarded civilians are pure gain (no risk). Guarded civilians involve the full eat risk/reward matrix (approach, scream, escape). The choice is granular: which guarded civilian, from which angle, at what sanity level? |
| Dominant strategy risk | If all civilians are unguarded, there's no risk. **Mitigation:** With 5 civilians and 3 guards, at least 3 civilians are guarded. The ratio ensures meaningful routing decisions. |

### Decision Point 2: When to Eat

**The choice:** Eat this civilian now (potentially wasting sanity + triggering scream) vs. skip and eat later or eat a different civilian.

| Question | Answer |
|----------|--------|
| What are the options? | (A) Eat now — guaranteed restore, but may waste sanity if near full, and scream alerts nearby guards. (B) Skip — save this civilian for when sanity is lower. Civilian stays alive and continues its behavior. (C) Eat a different, safer civilian instead. |
| What information does the player have? | Current sanity (HUD + vignette), civilian position, nearby guardpositions (visible), scream range (learned through play). |
| What are the consequences? | Eat now: up to `SANITY_PER_EAT` (4) wasted if sanity > 8, PLUS death scream fires regardless. Eating at full sanity = pure cost, no benefit. Later: civilian might flee to a worse position, or guardsmight reposition. Different civilian: may require detour (time = sanity). |
| Is the choice interesting? | **Yes — richer than static pickups.** The death scream adds a cost to EVERY eat, not just ones near guards. Even "safe" eats carry scream risk if a guardpatrols into range. The civilian is also moving — waiting has consequences because the civilian's position changes. |
| Dominant strategy risk | "Eat when sanity is low" seems dominant, but the civilian may flee into a harder position if you wait. **Mitigation:** Civilian flee + seek AI means waiting can make eating harder or easier depending on dynamics. No guaranteed stable state. |

### Decision Point 3: Sanity Management

**The choice:** Stay Lucid (precise control, weaker movement) vs. let sanity drop (powerful but chaotic movement).

| Question | Answer |
|----------|--------|
| What are the options? | (A) Eat civilians aggressively to stay in Lucid range (sanity 7-12) — full control, weaker jump/speed. (B) Let sanity drop into Slipping/Feral — faster movement, higher jumps, but drift and input delay increase. |
| What information does the player have? | Current sanity (HUD + vignette + movement feel). Upcoming level geometry (can see platforms and gaps ahead). Knowledge of whether upcoming sections require Feral-level movement to traverse. |
| What are the consequences? | Lucid: safe but slower. Cannot reach some platforms. Slipping: faster, occasional drift nudges (manageable). Feral: fastest, highest jumps, but drift is scary (t² impulse curve means Feral drift displaces 1.5-2.8 tiles per event). |
| Is the choice interesting? | **Yes — the SANITY_BALANCE_REPORT validated this.** After the smooth drift fix, every sanity level has proportional cost. There's no free zone (old Lucid 7-12 was cost-free) and no cliff (old Feral boundary was 28× cost jump). The t² drift curve means the choice is granular, not binary. |
| Dominant strategy risk | Pre-fix, sanity 7 was strictly dominant (max free power). Post-fix, the smooth drift curve eliminates this. Ongoing risk: if drain is too slow, player stays Lucid always. **Mitigation:** Ensure at least one section per level requires Slipping-tier movement to traverse optimally. |

### Decision Point 4: Fight or Flee

**The choice:** Engage the guard(if stomp mechanic exists) vs. dodge around it vs. find an alternate path.

| Question | Answer |
|----------|--------|
| What are the options? | (A) Jump on the guard(stomp-to-kill, if implemented per Sprint 2 design decision 2.1). (B) Dodge past the guardusing speed/timing. (C) Find an alternate route that avoids the guardentirely. |
| What information does the player have? | Guard position and patrol pattern (observable). Own HP. Own sanity (affects speed/control for dodging). Level geometry (visible platforms). |
| What are the consequences? | Stomp: guardeliminated (path cleared permanently), but requires precise positioning — missed stomp means taking damage. Dodge: guardsurvives, may chase, but no HP risked if successful. Alternate route: safe but costs time (sanity drain continues). |
| Is the choice interesting? | **Depends on the interaction model decision (Sprint 2, task 2.1).** If pure avoidance (option B from backlog): choice is between dodge and alternate route — still interesting but simpler. If stomp-to-kill or sanity-dependent stomp: adds a risk/reward layer. The recommended option (C: sanity-dependent stomp) ties directly into sanity management — you must be at Feral to stomp, adding a "drop sanity to fight" decision. |
| Dominant strategy risk | If stomping is free and easy, it dominates. **Mitigation:** If stomp is implemented, require precise vertical positioning (above guard's midpoint while falling). At Feral where stomp is available, drift makes precision harder — the power/control tradeoff applies to combat too. |

### Decision Point 5: Exploration vs. Speed

**The choice:** Explore the level for more civilians vs. rush to the exit.

| Question | Answer |
|----------|--------|
| What are the options? | (A) Explore — find all civilians, eat as many as possible, maximize sanity buffer. (B) Rush — head straight for the exit, skip remaining civilians, race the sanity clock. |
| What information does the player have? | Visible portion of the level (can see civilians and guardsin view). Current sanity (are they desperate?). How many civilians they've already eaten. Whether the remaining civilians are heavily guarded (consolidation effect). |
| What are the consequences? | Explore: more civilians eaten (higher sanity buffer), but more sanity spent on travel time, more guardencounters, more scream events. Rush: less sanity spent on travel, but no safety buffer. One mistake (take damage, miss a jump) could be fatal. |
| Is the choice interesting? | **Yes — and the civilian system adds a new dimension.** Late-game civilians are harder to eat due to guardconsolidation (see CIVILIAN_ECOLOGY_SPEC.md §6). With 5 civilians restoring 4 sanity each and a drain of 0.15/s, eating the 4th civilian nets ~3.25 sanity (4 restore - ~5s × 0.15 drain for approach). But the 5th civilian, now guarded by all 3 guards, might cost 10-15s of dangerous maneuvering = 1.5-2.25 sanity + possible HP damage. The math creates a "when to stop" decision that doesn't exist with static pickups. |
| Dominant strategy risk | If all civilians are equally easy, exploring is always worth it. **Mitigation:** Guard consolidation makes later civilians progressively harder. The natural difficulty curve within each level creates a point where the risk of eating exceeds the benefit. |

---

## Appendix B: Sanity Economy Model

Reference model for how the sanity drain + civilian eating cycle plays out over a level. See also CIVILIAN_ECOLOGY_SPEC.md §6 for the full population dynamics model.

**Assumptions:** `SANITY_DRAIN_RATE` = 0.15/s, `SANITY_PER_EAT` = 4, `ZOMBIE_MAX_SANITY` = 12, 5 civilians (2 unguarded, 3 guarded), 3 guards, level takes 180s (3 min).

```
Time (s)  Event                     Sanity  Notes
────────  ─────                     ──────  ─────
  0       Level start               12.0    5 civilians alive, 3 guardsguarding
 15       Approach free civ #1       9.8    2.2 sanity spent on travel
 18       Eat free civilian #1      12.0    +4 (capped). No scream response — no guardsnear.
 40       Approach free civ #2       8.7    Detour to second free civilian
 43       Eat free civilian #2      12.0    Full restore. Both free civilians eaten.
 70       Approach guarded civ       8.0    Now must enter guardterritory
 73       Eat guarded civ #3        12.0    SCREAM! 1-2 guardsalerted. Sprint away.
 78       Escape scream chase       11.3    Lost 5s fleeing. -0.75 sanity + maybe 1 HP.
110       Approach guarded civ       6.5    Two guardsguarding two civilians.
115       Eat guarded civ #4        10.5    SCREAM! Both remaining guardsconverge.
122       Escape (took 1 hit)        9.4    -1 HP. Two guardschasing for 3s.
160       Skip last civilian         3.7    All 3 guardsguard it. Not worth the risk.
180       Reach exit                 0.7    Barely made it. 4 of 5 civilians eaten.
```

**Total sanity budget:** 12 (start) + 16 (4 civilians × 4) = 28 sanity units
**Total sanity cost:** 180s × 0.15/s = 27 sanity units
**Deficit with 4 eats:** +1 → just barely enough.
**Deficit with 3 eats:** -3 → still survivable for a ~160s level, but no margin for error.

This model confirms: at 0.15/s drain with 5 civilians, eating 4 (80%) is sufficient but tight. The last civilian is disproportionately risky (all 3 guardsguard it) — the player should usually skip it and sprint for the exit. This creates the "when to stop" decision that the civilian ecology enables.

---

## 9. Revision History

| Date | Version | Change | Reason |
|------|---------|--------|--------|
| Feb 2026 | 1.0 | Initial specification | Pre-Sprint 1 design lock. Documents the core loop for the vertical slice milestone. |
| Feb 2026 | 1.1 | Civilian ecology integration | Replaced static brain pickups with living civilians. Added death scream, guardguard/leash, civilian flee AI. See CIVILIAN_ECOLOGY_SPEC.md for full ecology design. |
