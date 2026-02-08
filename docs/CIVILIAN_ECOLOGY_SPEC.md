# Economy Specification: Civilian-Threat Ecology

**Game:** Brains for Breakfast
**Author:** Brent Burgess
**Version:** 1.0
**Date:** Feb 2026
**Skill Applied:** gamedev-systems-balance (Economy Spec)
**Lenses Applied:** #33 (Meaningful Choices), #34 (Triangularity), #45 (Economy), #9 (Elemental Tetrad)

---

## Overview

This document replaces the static brain pickup system defined in CORE_LOOP_SPEC.md with a **living civilian ecology**. Civilians are the zombie's food source — walking, fleeing entities that must be caught and eaten to restore sanity. Threats guard civilians via a leash/patrol mechanic, making eating inherently risky. The death scream mechanic alerts nearby threats when the zombie eats, creating a cascading risk.

**Why living civilians instead of static pickups:** Static brains reduce the core loop to "walk over glowing item." Living civilians add:
- **Pursuit** — civilians flee, requiring the player to chase or corner them
- **Risk** — threats cluster around civilians, making food dangerous to access
- **Death scream** — eating triggers an alert, punishing careless feeding
- **Spatial dynamics** — civilians move, changing the level's resource landscape over time
- **Moral weight** — "I'm eating people" is the gentleman zombie's central tension

**What this document does NOT include:**
- Starvation mechanic (explicitly cut — no accelerated drain when all civilians dead)
- Three-way faction combat (threats don't attack civilians, civilians don't fight)
- Sanity-dependent faction reactions (threats always chase zombie regardless of sanity)

---

## 1. Resources

### Resource Inventory

| Resource | Role | Source(s) | Sink(s) | Cap | Persistence |
|----------|------|-----------|---------|-----|-------------|
| **Sanity** | Core survival currency — depletes over time, restored by eating civilians. Determines movement feel (power vs. control tradeoff). | Eating civilians (+4 per eat). Level start (12). | Continuous time drain (-0.15/s). | 12 (max sanity) | Resets each level |
| **HP** | Damage buffer — lost to threat contact. | Level start (5). No in-level restore source. | Threat damage (-1 per hit). | 5 (max HP) | Carries across levels |
| **Civilians** | Finite food supply — walking sanity pickups. The zombie's reason to engage with the level instead of sprinting to the exit. | Level start (placed by design, 4-6 per level). Non-renewable. | Eaten by zombie (consumed permanently). | Fixed per level | N/A (single level) |
| **Threat attention** | Implicit resource — how many threats are actively chasing the zombie vs. passively guarding. | Death scream (alerts threats in range). Proximity detection. | Alert timer expiry. Breaking line-of-sight / leaving detection range. | All threats can be alerted simultaneously | Per-encounter |

### Resource Design Verification

| Check | Sanity | HP | Civilians | Threat Attention |
|-------|--------|-----|-----------|-----------------|
| Player makes choices about this resource? | **Yes** — when to eat, whether to risk low sanity for power | **Yes** — whether to take a hit to save time or dodge and lose sanity to time | **Yes** — which civilians to eat, when, and in what order | **Yes** — whether to eat near threats (triggering scream) or lure threats away first |
| Can the player run out? | **Yes** — game over at 0 | **Yes** — game over at 0 | **Yes** — finite per level | **Yes** — all threats can be lured to one area, leaving other areas unguarded |
| Can the player have too much? | **Soft cap** — eating at sanity 10+ wastes restore | **No** — HP has no downside at max | **No** — can't have "too many" civilians, but player may choose to leave some uneaten | **Yes** — too many alerted threats = overwhelmed |
| Source clear to player? | **Yes** — eat civilians | **Yes** — don't get hit (no healing source) | **Yes** — visible in level | **Yes** — eat = scream = alerts |
| Sink clear to player? | **Yes** — visible drain + vignette | **Yes** — threat contact | **Yes** — eating consumes them | **Yes** — timer expires, flee range |
| Interacts with other resources? | **Yes** — civilians are the source. HP loss comes from threats that guard civilians. | **Yes** — threats guard civilians (risk HP to access food) | **Yes** — eating triggers scream (threat attention) | **Yes** — attention draws threats away from guarding other civilians |

---

## 2. Flow Diagram

### Primary Flow: Sanity Economy

```
[TIME] --drains--> [SANITY] --motivates--> [FIND CIVILIAN]
                      ^                          |
                      |                     [APPROACH]
                      |                          |
                [SANITY +4]              [RISK: THREATS GUARD]
                      ^                          |
                      |                     [EAT CIVILIAN]
                      |                          |
                [CIVILIAN CONSUMED] <----    [DEATH SCREAM]
                                                 |
                                          [THREATS ALERTED]
                                                 |
                                          [THREATS CHASE ZOMBIE]
                                                 |
                                          [ZOMBIE TAKES DAMAGE or ESCAPES]
```

**Flow Description:** Time drains sanity continuously. The player must eat civilians to restore sanity. Civilians are guarded by threats that patrol nearby. Eating triggers a death scream that alerts threats in range, creating a burst of danger after every meal. The player must balance the urgency of eating (sanity is draining) against the danger of eating (threats will converge). Escaping after eating costs time, which costs sanity — creating a tension loop where the solution to one problem (low sanity) temporarily worsens another (threat pressure).

### Secondary Flow: Threat Attention Economy

```
[THREATS] --guard--> [CIVILIANS]
    |                     ^
    |               [LEASH PULLS]
    |               [THREAT BACK]
    |                     |
[DEATH SCREAM] --alerts--> [THREATS CHASE ZOMBIE]
    |                               |
    |                        [ZOMBIE ESCAPES]
    |                               |
    |                     [ALERT TIMER EXPIRES]
    |                               |
    |                     [THREAT RETURNS TO GUARD]
    v
[CIVILIAN EATEN = ONE FEWER GUARDED POINT]
    |
    v
[REMAINING THREATS REDISTRIBUTE TO REMAINING CIVILIANS]
```

**Flow Description:** Threats are tethered to civilians via the leash mechanic. When a civilian is eaten, the threat that was guarding it has no anchor — it must find another civilian to guard or default to patrol behavior. As the civilian population decreases, threats consolidate around remaining civilians, making later eats progressively harder. This creates a natural difficulty curve within each level without any explicit scaling.

### Cross-Resource Interactions

- **Sanity ↔ Civilians:** Eating civilians restores sanity. Sanity drain creates the urgency to eat. Without drain, civilians are irrelevant. Without civilians, drain is a death sentence.
- **HP ↔ Threat Attention:** Taking damage costs HP. Threat attention is triggered by eating (death scream) or proximity. HP is the cost of failing to manage threat attention.
- **Civilians ↔ Threats:** Threats guard civilians. Eating a civilian removes a guard anchor, redistributing threat coverage. The civilian population shapes the threat landscape.
- **Sanity ↔ Threat Avoidance (implicit):** Low sanity = faster movement (easier to outrun threats) but less control (drift may push you into threats). The sanity system and threat system interact through the movement feel, not through direct mechanical coupling.

---

## 3. Faction Interaction Matrix

### Entity Stats (from GDD, adapted to real-time)

**Civilians:**

| Stat | Value | Notes |
|------|-------|-------|
| HP | 1 | Instant kill on eat (one overlap = consumed) |
| Move speed | 60 px/s | Slow — about 20% of zombie Lucid speed. Can be caught but requires closing distance. |
| Flee range | 96 px (3 tiles) | Starts fleeing when zombie is within this distance |
| Seek range | 128 px (4 tiles) | Moves toward nearest threat if one is within range (seeking protection) |
| Wander speed | 30 px/s | Idle movement when no zombie threat and no nearby threat to seek |

**Threats (adapted from GDD hunters):**

| Stat | Value | Notes |
|------|-------|-------|
| HP | See CORE_LOOP_SPEC | As defined in existing spec |
| Patrol speed | 120 px/s | Per CORE_LOOP_SPEC |
| Chase speed | 200 px/s | Per CORE_LOOP_SPEC |
| Detection range | 160 px (5 tiles) | Per CORE_LOOP_SPEC |
| Guard range | 64 px (2 tiles) | Stays within this distance of guarded civilian during patrol |
| Leash distance | 160 px (5 tiles) | Won't chase zombie beyond this distance from guarded civilian |
| Scream response range | 128 px (4 tiles) | Alerts to death screams within this range |
| Alert duration | 3s | Time spent pursuing scream location before returning to guard |

### Interaction Matrix

| | Zombie (Player) | Civilian | Threat |
|---|---|---|---|
| **Zombie** | — | **Eats** on contact → sanity +4, civilian removed, death scream triggered | **Takes damage** on contact (1 HP), invincibility frames |
| **Civilian** | **Flees** when zombie within flee range (96px). Runs in opposite direction. | **Clusters** — civilians don't interact with each other | **Seeks safety** — moves toward nearest threat within seek range (128px) |
| **Threat** | **Guards/Chases** — patrols near guarded civilian, chases zombie on detection, responds to death screams | **Guards** — patrols within guard range (64px) of nearest civilian. Leashed to civilian position. | **Ignores** — threats don't interact with each other in v1 |

### Behavioral Priority Lists

**Civilian AI (simple, low-cost):**
1. **Flee from zombie** — if zombie within flee range, move away at flee speed (60 px/s)
2. **Seek threat protection** — if a threat is within seek range, drift toward it
3. **Wander** — slow random patrol on current platform
4. **Edge avoidance** — reverse at platform edges (civilians don't fall off)

**Threat AI (expanded from CORE_LOOP_SPEC):**
1. **Respond to death scream** — if scream within response range, move to scream location for `alert_duration` seconds. Highest priority.
2. **Chase zombie** — if zombie within detection range AND within leash distance of guarded civilian, chase at chase speed
3. **Return to leash** — if beyond leash distance from guarded civilian, break chase and return
4. **Guard patrol** — patrol back and forth within guard range of assigned civilian
5. **Reassign** — if assigned civilian is dead, reassign to nearest living civilian. If no civilians remain, default to standard patrol (no leash).

---

## 4. Death Scream Mechanic

The death scream is the risk half of the eat risk/reward equation.

### Trigger
When the zombie eats (overlaps with) a civilian, a "scream" event fires at the civilian's position.

### Effect
All threats within `SCREAM_ALERT_RANGE` (128px / 4 tiles) of the eaten civilian's position:
- Set state to `RESPONDING_TO_SCREAM`
- Set `alert_target` to the civilian's last position
- Set `alert_timer` to `SCREAM_ALERT_DURATION` (3s)
- Override current behavior (even if already chasing zombie)

### Alert Behavior
While `alert_timer > 0`:
- Move toward `alert_target` at chase speed
- If zombie is encountered along the way, attack normally
- When arriving at `alert_target` or when `alert_timer` expires, return to previous behavior (guard nearest civilian)

### Parameters

| Parameter | Default | Range | Notes |
|-----------|---------|-------|-------|
| `SCREAM_ALERT_RANGE` | 128 px | 64-256 | GDD: 4 tiles. Smaller = safer eating. Larger = more dangerous. Primary tuning knob for eat difficulty. |
| `SCREAM_ALERT_DURATION` | 3.0s | 1.5-5.0 | How long threats investigate. Longer = more time the area is dangerous. GDD: 2 rounds, ~3s equivalent. |

### Lens #34 (Triangularity) Analysis

The death scream creates a classic risk/reward triangle:

| Option | Risk | Reward |
|--------|------|--------|
| **Eat now** (threat nearby) | Scream alerts threat, chase begins immediately, possible damage | Sanity restored immediately, no backtracking needed |
| **Lure threat away first** | Costs time (sanity drain), threat may return before you eat | Safer eat, no scream response (threat out of range) |
| **Eat and run** | Must outrun alerted threats, drift at low sanity may betray you | Sanity restored, creates excitement/tension |
| **Skip this civilian** | No food this cycle, sanity continues draining | Zero risk, but accelerates toward sanity death |

No dominant strategy — the right choice depends on current sanity, current HP, threat positions, and distance to exit.

---

## 5. Conversion Rates

| Conversion | Rate | Notes |
|-----------|------|-------|
| **Time → Sanity loss** | -0.15 sanity/s | 80s from full (12) to gone (0) with no eating |
| **Civilian eaten → Sanity gained** | +4 sanity per eat | 33% of max. 3 eats = full recovery from 0. |
| **Civilian eaten → Threat attention** | All threats within 128px alerted for 3s | ~1-2 threats per eat in typical placement |
| **Threat contact → HP lost** | -1 HP per hit | 5 hits to die. Invincibility prevents multi-hit. |
| **Civilian approach → Time cost** | ~2-5s to close distance and eat | Civilian flee speed (60) vs zombie speed (300+). Chase is short but costs sanity (~0.3-0.75 sanity). |

### Time-to-Resource Guidelines

| Resource | Time to Earn Meaningful Amount | What "Meaningful" Means |
|----------|-------------------------------|------------------------|
| Sanity (+4 from eating) | 2-5s (approach + eat) | One eat buys ~27s of drain time (4 / 0.15). Net gain: ~22-25s per eat. Always worth it if safe. |
| Safe eating window | 3-8s (lure threat, eat, escape) | Time from starting approach to being safe again. Longer if death scream attracts threats. |
| Level completion | 120-300s total | Full level with 4-6 civilians available. Player eats 3-4 (60-80%). |

---

## 6. Civilian Population Dynamics

### Level Composition (Default)

| Parameter | Default | Range | Notes |
|-----------|---------|-------|-------|
| `CIVILIAN_COUNT` | 5 | 3-7 | Number of civilians per level. Replaces `BRAIN_COUNT` from CORE_LOOP_SPEC. |
| `THREAT_COUNT` | 3 | 2-5 | Per CORE_LOOP_SPEC. Each threat guards nearest civilian at level start. |

### Guard Assignment

At level start, each threat is assigned to guard the nearest civilian:
- With 5 civilians and 3 threats: 3 civilians are guarded, 2 are unguarded
- Unguarded civilians are "free food" — safe to eat, no scream risk (no threats in range)
- **This creates the route choice:** eat the free civilians first (safe but may be out of the way) or eat guarded civilians (risky but convenient)

### Population Over Time

As civilians are eaten:

| Civilians Remaining | Threats Guarding | Effect |
|--------------------|--------------------|--------|
| 5 (start) | 3 guard, 2 free | Some safe options, some risky |
| 4 (ate 1 free) | 3 guard, 1 free | One safe option left |
| 3 (ate both free) | 3 guard, 0 free | All remaining civilians are guarded — every eat is risky |
| 2 (ate 1 guarded) | 3 threats, 2 guarded, 1 reassigned | Threats consolidate. Two clusters of guards. |
| 1 (ate 2 guarded) | All 3 threats guard last civilian | Final civilian is heavily guarded. Maximum risk for last eat. |
| 0 (all eaten) | 3 threats, no leash | Threats default to standard patrol. No food remains. Player must reach exit on remaining sanity. |

**Emergent difficulty curve:** The level gets harder as you eat. Early eats are easy (free civilians). Later eats are dangerous (consolidated guards). The last civilian is a boss-fight-level challenge. This is a natural positive feedback loop for difficulty — but it's self-limiting because the player CHOOSES when to stop eating and rush for the exit.

### Sanity Budget Model (with civilians)

Assumptions: 5 civilians, 3 threats, `SANITY_DRAIN_RATE` = 0.15/s, `SANITY_PER_EAT` = 4, level time ~180s.

```
Time   Event                  Sanity  Notes
─────  ─────                  ──────  ─────
  0    Level start            12.0    5 civilians alive, 3 threats guarding
 15    Approach free civ #1    9.8    2.2 sanity spent on travel
 18    Eat free civilian #1   13.8→12 Restore 4 (capped). No scream (no threats near).
 40    Approach free civ #2    8.7    Detour to second free civilian
 43    Eat free civilian #2   12.0    Full restore. Both free civilians eaten.
 70    Approach guarded civ    8.0    Now must enter threat territory
 73    Eat guarded civ #3     12.0    SCREAM! 1-2 threats alerted. Sprint away.
 78    Escape scream chase     11.3   Lost 5s fleeing. -0.75 sanity + maybe 1 HP.
110    Approach guarded civ    6.2    Two threats guarding two civilians.
115    Eat guarded civ #4     10.2    SCREAM! Both remaining threats converge.
122    Escape (took 1 hit)     9.1    -1 HP. Two threats chasing for 3s.
160    Skip last civilian      3.4    All 3 threats guard it. Not worth the risk.
180    Reach exit              0.4    Barely made it. 4 of 5 civilians eaten.
```

**Budget:** 12 (start) + 16 (4 eats × 4) = 28 sanity available. Cost: 180s × 0.15/s = 27. The math works: eating 4 of 5 civilians is sufficient but tight. Eating all 5 gives comfortable margin but the last one is very risky.

---

## 7. Emergent Behaviors

### Predicted Emergent Dynamics

| Behavior | How It Emerges | Player Experience |
|----------|---------------|-------------------|
| **Safe-first eating** | Unguarded civilians are risk-free. Players eat them first. | Natural tutorial: first eats are easy, teaching the mechanic before introducing risk. |
| **Threat consolidation** | As civilians die, threats reassign to remaining civilians. | Late-level eating is harder than early-level. Organic difficulty ramp within each level. |
| **Eat and run** | Player eats a guarded civilian and immediately sprints away from the scream zone. | Exciting moments of "grab the food and GO" — action-movie tension. |
| **Lure and eat** | Player enters threat detection range to pull threat away from civilian, then circles back to eat while threat is chasing. | Skilled play: manipulating AI to create safe windows. Feels clever. |
| **Civilian herding** | Civilian flee AI pushes civilians toward threats (seeking protection). A zombie approaching from one side pushes the civilian toward the threat on the other side. | Eating near threats scares civilians INTO more danger. Player may need to approach from a specific direction. |
| **Scream chain** | Player eats one civilian, scream alerts threats, player runs to another civilian and eats that one too, second scream alerts even more threats. | High-risk, high-reward "feeding frenzy" play style. Maximizes sanity gain but creates a gauntlet of alerted threats. |
| **Guard cluster bait** | All 3 threats guarding the last civilian. Player can eat the last civilian if they have enough HP to absorb hits during the escape. | End-of-level "worth it?" calculus: trading HP for sanity. |
| **Threat-free zones** | Areas with no civilians have no guarding threats. These become safe corridors. | Level readability: civilians = food = danger. Empty areas = safe passage. |

### Lens #33 (Meaningful Choices) — New Decision Points

The civilian system adds or modifies these decision points from CORE_LOOP_SPEC:

**Decision: Eat order (NEW)**
- Options: Eat free civilians first (safe) vs. eat guarded civilians first (risky, before threats consolidate)
- Information: Player can see which civilians have nearby threats
- Consequence: Free-first is safe but uses more time (sanity) on travel. Guarded-first prevents consolidation but risks HP early.
- Interesting? **Yes** — no dominant strategy. Depends on level layout and threat positions.

**Decision: Approach angle (NEW)**
- Options: Approach civilian from the side away from threats vs. approach from the side near threats
- Information: Visible threat patrol paths, civilian position
- Consequence: Wrong angle pushes civilian toward threat (flee AI). Right angle pushes civilian away from threat or into a corner.
- Interesting? **Yes** — spatial reasoning reward. Skilled players learn to "herd" civilians.

**Decision: Scream management (REPLACES static pickup)**
- Options: Eat when threats are far (safe, no scream response) vs. eat when threats are near (fast, but scream triggers)
- Information: Threat positions, scream range (learn through play)
- Consequence: Safe eating costs time (waiting for patrol cycles). Risky eating saves time but may cost HP.
- Interesting? **Yes** — the core risk/reward. Replaces the non-decision of "walk over static brain."

**Decision: When to stop eating (NEW)**
- Options: Eat all civilians (max sanity) vs. eat enough and rush exit
- Information: Remaining sanity, distance to exit, threat consolidation around remaining civilians
- Consequence: Over-eating wastes time and risks HP on heavily-guarded late civilians. Under-eating risks running out of sanity before the exit.
- Interesting? **Yes** — the last 1-2 civilians are disproportionately risky. Knowing when to stop is a skill.

---

## 8. Edge Cases

| # | Scenario | Expected Behavior | Priority |
|---|----------|-------------------|----------|
| 1 | **All civilians eaten, sanity still draining** | No special effect — no starvation mechanic. Player must reach exit on remaining sanity. Threats with no civilian to guard default to standard patrol behavior (no leash). | Must-handle |
| 2 | **Civilian flees off platform edge** | Civilian reverses at platform edges (same as threats). Civilians never fall off platforms. They're slow and cautious. | Must-handle |
| 3 | **Civilian flees into a wall** | Civilian stops at wall. If zombie is still in flee range, civilian is "cornered" — easy to eat. This is intended: cornering is a valid strategy, rewarding spatial awareness. | Must-handle |
| 4 | **Death scream with no threats in range** | Scream fires but no threats respond. This is the reward for eating isolated/unguarded civilians. Scream is always generated (for consistency) but has no effect if no threats are in range. | Must-handle |
| 5 | **Threat assigned to guard a civilian on a different platform** | Threat can only guard civilians reachable on the same platform (threats don't jump). Guard assignment at level start should only assign threats to same-platform civilians. If no same-platform civilian exists, threat defaults to standard patrol. | Must-handle |
| 6 | **Two threats guarding same civilian** | Allowed. Both patrol within guard range. Both respond to the scream when that civilian is eaten. This makes certain civilians extra dangerous — by design. | Must-handle |
| 7 | **Zombie overlaps civilian during invincibility frames** | Eating is NOT blocked by invincibility. Iframes only prevent damage, not eating. Player can eat while flashing — this enables the "take a hit, grab the food" play pattern. | Must-handle |
| 8 | **Civilian seek-threat AI and flee-zombie AI conflict** | Flee takes priority. If zombie is within flee range, civilian flees even if that means moving away from a threat. Self-preservation > seeking protection. | Must-handle |
| 9 | **Death scream during another scream's alert window** | New scream refreshes alert timer for threats already alerted. Also alerts any additional threats in range of the new scream. Alerts stack additively (new targets) but don't extend beyond the new duration (no infinite alert chains). | Nice-to-handle |
| 10 | **Threat chasing zombie when its civilian is eaten by another source** | No other source exists in v1 (only the zombie eats). But if this occurs in future, the threat should reassign to the nearest remaining civilian when its alert timer expires. | Won't-handle (v1) |

---

## 9. Inflation Controls

| Control Mechanism | What It Addresses | How It Works |
|-------------------|-------------------|-------------|
| **Sanity cap (12)** | Sanity inflation from over-eating | Eating at high sanity wastes restore. Player can't stockpile above 12. Creates timing decisions. |
| **Finite civilians** | Food supply inflation | Civilians don't respawn. Each level has a fixed food budget. Player must plan consumption across the full level. |
| **Threat consolidation** | Late-level food being too easy | As civilians are eaten, threats redistribute to remaining civilians. Later eats are harder than earlier eats. Natural inflation control. |
| **Death scream** | Risk-free eating | Every eat (near threats) has a cost: alerting nearby threats. Prevents "free lunch" — the player always pays attention to timing. |
| **No HP restore** | HP inflation | HP cannot be restored within a level (v1). Every hit permanently reduces the player's buffer. Prevents reckless play. |

---

## 10. Balance Levers

| Lever | What It Controls | Default | Range | When to Adjust |
|-------|-----------------|---------|-------|----------------|
| `CIVILIAN_COUNT` | Total food supply per level | 5 | 3-7 | If players never run out of sanity, reduce. If they always starve, increase. |
| `SCREAM_ALERT_RANGE` | How dangerous eating is | 128 px | 64-256 | If eating feels too safe (threats never respond), increase. If every eat feels suicidal, decrease. |
| `SCREAM_ALERT_DURATION` | How long threats investigate after scream | 3.0s | 1.5-5.0 | If scream feels like a non-event (threats return immediately), increase. If the player can never escape, decrease. |
| `THREAT_LEASH_DISTANCE` | How far threats stray from civilians | 160 px | 96-256 | If threats never actually chase (too short leash), increase. If threats abandon guards too easily, decrease. |
| `CIVILIAN_FLEE_SPEED` | How hard civilians are to catch | 60 px/s | 30-100 | If civilians are trivially easy to catch, increase. If chasing feels tedious, decrease. |
| `CIVILIAN_FLEE_RANGE` | How early civilians react to zombie | 96 px | 64-160 | If civilians flee before the player is close enough to engage, decrease. If they never flee, increase. |
| `SANITY_PER_EAT` | Value of each eat | 4 | 3-6 | If eating feels meaningless (tiny restore), increase. If one eat solves all problems, decrease. |
| `THREAT_GUARD_RANGE` | How tight the guard patrol is | 64 px | 32-128 | If threats wander too far from civilians (easy to eat unguarded), decrease. If threats are glued to civilians (impossible to separate), increase. |

### Tuning Priority Order

1. `CIVILIAN_COUNT` + `SANITY_PER_EAT` — these define the total sanity budget. Get this right first.
2. `SCREAM_ALERT_RANGE` — primary knob for eat danger. Tune second.
3. `THREAT_LEASH_DISTANCE` — determines whether "lure and eat" strategy is viable.
4. Everything else — fine-tuning after the big knobs are set.

---

## 11. Warning Signs of a Broken Ecology

| Warning Sign | What It Indicates | Where to Look | Fix Direction |
|-------------|-------------------|---------------|---------------|
| Player eats every civilian with zero deaths | Eating is too safe — threats don't guard effectively or scream range is too small | `SCREAM_ALERT_RANGE`, `THREAT_LEASH_DISTANCE`, threat speed | Increase scream range, tighten leash, speed up threat chase |
| Player never eats guarded civilians | Guarded civilians are too dangerous — cost always exceeds benefit | `SCREAM_ALERT_DURATION`, `THREAT_GUARD_RANGE`, threat damage | Reduce alert duration, widen guard range (easier to approach from safe angle), reduce threat damage |
| Player ignores civilians entirely and sprints to exit | Sanity drain is too slow — no need to eat | `SANITY_DRAIN_RATE`, level length | Increase drain rate or lengthen level so drain becomes relevant |
| Player always eats the first civilian they see regardless of risk | Sanity drain is too fast — every eat is life-or-death | `SANITY_DRAIN_RATE`, `SANITY_PER_EAT` | Slow drain or increase restore amount |
| Threats cluster into one mega-group after 2 eats | Guard reassignment creates death ball | Guard reassignment logic | Add spacing — reassigned threat picks nearest civilian NOT already guarded by 2+ threats |
| Civilian flee AI makes them uncatchable | Civilian effectively runs forever on a long platform | `CIVILIAN_FLEE_SPEED`, platform design | Slow civilian speed, ensure platforms have corners/walls for trapping |
| All civilians end up in the same corner | Flee + seek AI herds civilians together | Civilian AI behavior | Add randomness to wander direction, reduce seek urgency |

---

## 12. Impact on CORE_LOOP_SPEC

This ecology spec modifies the following from the existing CORE_LOOP_SPEC:

### Parameters Changed

| CORE_LOOP_SPEC Parameter | Old Value | New Value | Reason |
|-------------------------|-----------|-----------|--------|
| `BRAIN_COUNT` | 4 | Replaced by `CIVILIAN_COUNT` (5) | Living civilians replace static brains. Slightly more (5 vs 4) because catching fleeing civilians takes time, making each one slightly harder to acquire. |

### Parameters Added

| Parameter | Value | Section |
|-----------|-------|---------|
| `CIVILIAN_FLEE_SPEED` | 60 px/s | Civilian AI |
| `CIVILIAN_FLEE_RANGE` | 96 px | Civilian AI |
| `CIVILIAN_SEEK_RANGE` | 128 px | Civilian AI |
| `CIVILIAN_WANDER_SPEED` | 30 px/s | Civilian AI |
| `SCREAM_ALERT_RANGE` | 128 px | Death scream |
| `SCREAM_ALERT_DURATION` | 3.0s | Death scream |
| `THREAT_GUARD_RANGE` | 64 px | Threat guard behavior |
| `THREAT_LEASH_DISTANCE` | 160 px | Threat leash |

### Edge Cases Modified

| CORE_LOOP_SPEC Edge Case | Change |
|-------------------------|--------|
| #2 "Eat brain at sanity 12" | Same behavior: civilian consumed, sanity wasted. But now with scream risk. Eating at full sanity is even more wasteful because you pay the scream cost for zero benefit. |
| #6 "Brain spawns unreachable" | Civilian can't be unreachable because they move. But they could get cornered in an area the zombie can't reach — place civilians on platforms accessible to the zombie. Same-platform constraint. |
| #9 "Take damage while eating" | Still both events process. But now "eating" means overlapping a fleeing civilian, so this is more likely — the player may be chasing a civilian past a threat. |

### Sprint Backlog Impact

| Original Task | Change |
|--------------|--------|
| **1.4** Create brain pickup entity | **Replace with:** Create civilian entity with flee/seek/wander AI. Size increases from M to L. |
| **1.5** Brain pickup collision + sanity restore | **Modify:** Collision triggers eat + scream instead of just pickup. Add death scream event system. Size increases from S to M. |
| **2.2** Create threat entity data structure | **Add:** `guardedCivilian` field, `leashDistance`, `guardRange`. Size unchanged (S). |
| **2.3** Implement threat patrol behavior | **Modify:** Patrol within guard range of assigned civilian, not fixed patrol bounds. Size increases from M to M-L. |
| **New task** | Guard reassignment when civilian dies. S-sized. |
| **New task** | Civilian AI (flee, seek, wander, edge avoidance). M-sized. |

**Net scope impact:** Sprint 1 grows by approximately 1 M-sized task (civilian AI) and 1 S-sized task (guard reassignment). Sprint 2 tasks are modified but not significantly larger. Total milestone impact: +1 sprint of work, roughly. This is the cost of living civilians vs. static pickups.

---

## 13. Revision History

| Date | Version | Change | Reason |
|------|---------|--------|--------|
| Feb 2026 | 1.0 | Initial specification | Replaces static brain pickups with living civilian ecology. Adds death scream, threat-guarding, and civilian flee AI. |
