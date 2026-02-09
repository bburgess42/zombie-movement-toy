# Mechanic Specification: Civilian Variants

**Game:** Brains for Breakfast
**Author:** Brent Burgess
**Version:** 1.0
**Date:** Feb 2026
**Status:** Proposed (Stretch goal)

---

## 1. Player Fantasy

**What the player gets to feel:** "I know which civilians are worth the risk." The satisfaction of reading a tactical situation and choosing the right target based on current sanity, nearby guards, HP, and time remaining.

**Power fantasy:** Predatory intelligence. The player isn't a mindless zombie lunging at anything that moves. They're a calculating survivor triaging targets by type, weighing cost against reward, and executing the smart play under pressure.

**Skill expression:** A beginner eats whatever civilian is closest. An expert identifies that they need the gold Brainiac to survive but waits until the nearest hunter moves away from its guard position, or deliberately chases a green Sprinter early when sanity is high and time is plentiful, banking efficiency for later.

---

## 2. Core Description

**One-sentence summary:** Three civilian types (Standard, Sprinter, Brainiac) replace the uniform civilian pool, each offering different sanity rewards, flee behaviors, and danger profiles, forcing the player to choose targets based on context rather than proximity alone.

**Core Loop of This Mechanic:**
1. Player spots civilians on the map and visually identifies their type by color (blue = Standard, green = Sprinter, gold = Brainiac)
2. Player assesses current state: sanity level, HP, nearby guards, distance to exit, time pressure
3. Player selects a target type based on risk/reward tradeoff appropriate to the situation
4. Player pursues and eats the target, dealing with type-specific consequences (chase duration, scream radius, guard proximity)
5. Player adapts to the outcome: repositions after scream, manages new guard responses, reassesses remaining civilian pool

**Frequency:** Every time the player needs to eat (2-4 times per level), they make a variant-aware targeting decision. Target identification is continuous throughout navigation.

---

## 3. Input / Output

### Inputs

| Input | Context | Timing |
|-------|---------|--------|
| Move toward target | Player navigates to chosen civilian | Any time during zombie turn |
| Eat action (overlap with adjacent civilian) | Adjacent to a civilian of any type | Once per turn, consumes entire turn |

Inputs are identical to the base eat mechanic. The variant system changes outputs, not inputs.

### Outputs (Success)

| Variant | Sanity Restored | Scream Range | Special Consequence |
|---------|----------------|--------------|---------------------|
| **Standard** (blue) | +4 | 4 tiles (128px) | Base death scream, normal hunter response |
| **Sprinter** (green) | +2 | 4 tiles (128px) | Low reward but safe positioning (Sprinters flee far from guards) |
| **Brainiac** (gold) | +6 | 6 tiles (192px) | 50% larger scream radius; always placed near guards, guaranteeing hunter proximity |

### Outputs (Failure)

| Scenario | Description | Feedback |
|----------|-------------|----------|
| Sprinter outruns zombie | Player spent turns chasing but the Sprinter fled beyond catch range | Sanity drained during pursuit with no payoff; log messages show Sprinter fleeing |
| Brainiac scream draws hunters | Eating a Brainiac alerts hunters from farther away | Multiple hunters converge; larger alert radius shown in log |
| Target killed by monster | A monster reaches the civilian before the zombie | Civilian lost, food supply reduced; "killed by monster" log entry |
| Chase costs too much sanity | Time spent pursuing drains sanity below a critical threshold | Sanity tier drops during pursuit; involuntary action may trigger |

---

## 4. Variables and Parameters

### Per-Variant Stats

| Parameter | Standard | Sprinter | Brainiac | Range | Balance Notes |
|-----------|----------|----------|----------|-------|---------------|
| `sanityRestore` | 4 | 2 | 6 | 1-8 | Must sum to ~18 for typical 5-civilian composition |
| `fleeSpeed` (px/s) | 60 | 120 | 40 | 30-150 | In prototype: movement tiles per turn (1 / 2 / 1). Sprinter moves 2 tiles when fleeing |
| `fleeRange` (detection) | 3 tiles (96px) | 4 tiles (128px) | 3 tiles (96px) | 2-6 | Sprinter detects guards earlier, flees sooner |
| `seekRange` | 4 tiles | 0 (never seeks) | 4 tiles | 0-6 | Sprinter never moves toward hunters; relies on distance alone |
| `screamRange` | 4 tiles | 4 tiles | 6 tiles | 2-8 | Brainiac's scream is 50% larger |
| `color` | `#8be9fd` (cyan) | `#50fa7b` (green) | `#f1fa8c` (gold) | -- | Must be visually distinct at tile scale |

### Level Composition

| Parameter | Default | Range | Description | Balance Notes |
|-----------|---------|-------|-------------|---------------|
| `standardCount` | 2 | 0-5 | Standard civilians per level | Reliable baseline |
| `sprinterCount` | 2 | 0-5 | Sprinter civilians per level | Time-costly, safe option |
| `brainiacCount` | 1 | 0-3 | Brainiac civilians per level | High-risk, high-reward |
| `totalCivilians` | 5 | 3-8 | Sum of all types | Must match `civilianCount` config |
| `totalSanityAvailable` | 18 | 12-32 | Sum of all sanity restores | Slightly below base 20 (5x4) to tighten economy |

### Placement Rules

| Parameter | Standard | Sprinter | Brainiac | Notes |
|-----------|----------|----------|----------|-------|
| Placement bias | Random (existing logic) | Prefer positions far from guards | Within 5 tiles of a hunter or monster | Brainiac placement guarantees nearby danger |
| Min distance from zombie spawn | 5 tiles | 8 tiles | 3 tiles | Sprinters placed far; Brainiacs available early but risky |

---

## 5. Edge Cases

| Scenario | Expected Behavior | Priority |
|----------|-------------------|----------|
| All remaining civilians are Brainiacs | Player must accept maximum risk or attempt to reach exit without eating. No safe option exists. This is an intentional pressure state. | Must-handle |
| Sprinter outruns zombie entirely (low sanity = slow movement) | Sprinter can escape to an unreachable platform. Zombie must abandon pursuit. The Sprinter remains on the map as potential future target. At Feral tier (6 movement), catching a 2-move Sprinter is feasible. | Must-handle |
| Monster kills a Brainiac before zombie reaches it | Brainiac is lost. No scream (monsters don't trigger death scream). Food supply reduced. Player should have prioritized it if they needed the +6. | Must-handle |
| Involuntary eat targets a Sprinter (low sanity restore) | Involuntary eat applies to whatever civilian is adjacent, regardless of type. Getting only +2 from an involuntary eat on a Sprinter is a bad outcome — thematic (feral zombie can't be choosy). | Must-handle |
| Sprinter flees into a dead-end platform | Sprinter is trapped and catchable. Map geometry creates local opportunities. No special escape logic needed. | Nice-to-handle |
| Two Brainiacs placed near the same hunter cluster | Eating one draws hunters, making the second even more dangerous. Overlapping scream zones compound. Placement logic should distribute Brainiacs across different guard clusters when possible. | Nice-to-handle |
| Level has zero hunters (chaos preset) | Brainiac's "always near guards" placement falls back to near-monster placement. If no guards exist, Brainiac is placed randomly — effectively a free +6 with no danger. This is acceptable for chaos preset. | Nice-to-handle |
| All civilians killed by monsters before zombie eats any | Starvation mechanic activates (1.5x drain). Identical to base system. Variant types don't affect starvation logic. | Must-handle |

---

## 6. Interaction with Other Mechanics

| Mechanic | Interaction Type | Description |
|----------|-----------------|-------------|
| **Death Scream** | Modified | Brainiac's scream range is 6 tiles vs. the base 4. Standard and Sprinter use base range. This means eating a Brainiac can alert hunters that would be safe from a Standard eat. |
| **Hunter Guard Assignment** | Enabling | Hunters still guard nearest civilian. Brainiacs being placed near guards means hunters naturally cluster around them, creating "fortress" positions. Sprinters being far from guards means they're often unguarded — easy to reach but low reward. |
| **Sanity Economy** | Competing | Tighter total budget (18 vs 20) means the player can't afford to waste eats. A failed Sprinter chase that burns 2+ rounds of sanity drain may cost more sanity than it restores. |
| **Sanity Tiers** | Complementary | At Feral tier (6 movement), the zombie can catch Sprinters more reliably — but risks involuntary eat on the wrong target. At Lucid (4 movement), Sprinters are harder to catch but the player has full control over target selection. |
| **Involuntary Actions** | Conflicting | Involuntary eat targets the nearest adjacent civilian regardless of type. A Feral zombie adjacent to both a Sprinter (+2) and a Brainiac (+6) has no say in which gets eaten. This creates tension around positioning at low sanity. |
| **Monster Ecology** | Complementary | Monsters kill civilians indiscriminately. A monster killing a Brainiac removes the best sanity source. A monster killing a Sprinter removes a low-value target (may be beneficial). Monsters create urgency to eat high-value targets early. |
| **Starvation** | Unchanged | Starvation triggers when all civilians are dead, regardless of type. Variant types don't affect the 1.5x drain multiplier. |
| **Consolidation (Hunter surplus)** | Modified | With Brainiacs placed near guards, surplus hunters that would normally patrol instead guard the Brainiac cluster, making fortress positions denser. |

**Emergent Behaviors:**
- **Triage under pressure:** When sanity is dropping and multiple civilian types are reachable, the player must quickly evaluate: "Do I need the +6 Brainiac and can I survive the scream, or should I play it safe with the +2 Sprinter and hope I catch it in time?"
- **Monster-as-opener:** A monster killing a hunter near a Brainiac inadvertently creates an opening for the zombie to eat the Brainiac with reduced guard.
- **Sprinter bait:** At Feral tier, involuntary eat on a Sprinter (+2) is a terrible outcome. Players may deliberately avoid Sprinter-dense areas when at low sanity to prevent wasting an involuntary eat.
- **Gold rush:** If the single Brainiac is still alive late-game and the zombie is low on sanity, it becomes a high-stakes "do or die" target. The player might route the entire endgame around reaching it.

---

## 7. Success Metrics

| Metric | Target | How to Measure | Red Flag |
|--------|--------|---------------|----------|
| Variant eat distribution | 35-45% Standard, 25-35% Sprinter, 20-30% Brainiac | MC analytics: `eats_by_type / total_eats` | One type eaten >60% = dominant; one type eaten <10% = useless |
| Sprinter catch rate | 50-70% of pursuit attempts succeed | MC analytics: `sprinter_eats / sprinter_pursuit_starts` | Below 30% = uncatchable (speed too high); above 90% = no challenge |
| Brainiac eat survival rate | 40-60% of Brainiac eats don't result in death within 3 turns | MC analytics: `brainiac_eats_survived / brainiac_eats` | Below 25% = suicidal (scream too large); above 80% = not risky enough |
| Time-to-catch by type | Standard ~2 turns, Sprinter ~4 turns, Brainiac ~1.5 turns | MC analytics: average turns from pursuit start to eat | Sprinter >6 turns = too fast; Brainiac >3 turns = placement too far from paths |
| Sanity efficiency by type | Standard: +4 net, Sprinter: +0.5 to +1.5 net, Brainiac: +4.5 to +5.5 net | MC analytics: `sanity_restore - (pursuit_turns * drain_rate)` | Sprinter net negative = never worth eating; Brainiac net > Standard = always dominant |
| Player target switching | 20-40% of pursuits change target mid-chase | MC analytics (with improved AI): `target_changes / total_pursuits` | Below 5% = variants don't affect decisions; above 60% = too chaotic |

---

## 8. Implementation Notes

### Technical Considerations

**Entity structure change:** Each civilian needs a `variant` property (`"standard"`, `"sprinter"`, or `"brainiac"`) that determines its stats. In the current single-file HTML prototype, this means adding a field to the civilian object. In the Godot build, this maps to an exported enum on the Civilian scene/script.

**Systems that need variant-aware branches:**

| System | Change |
|--------|--------|
| Civilian AI (flee behavior) | Sprinter: `fleeRange` = 4 tiles (128px), moves 2 tiles when fleeing, never seeks guards for protection |
| Death scream / alert | Pass scream range as parameter per variant; Brainiac uses 6 tiles (192px) instead of base 4 |
| Eat action (sanity restore) | Look up `sanityRestore` from eaten civilian's variant instead of using a single global value |
| Involuntary eat | Same: use variant-specific `sanityRestore` |
| Entity placement / spawning | Split into three passes with variant-specific position preferences (see below) |
| Rendering / visuals | Distinct color per variant: Standard=#8be9fd (cyan), Sprinter=#50fa7b (green), Brainiac=#f1fa8c (gold) |

**Variant config constants:**

| Constant | Value | Description |
|----------|-------|-------------|
| `civilianVariants` | true | Enable variant system (false = all Standard) |
| `standardSanity` | 4 | Sanity restored by eating a Standard |
| `sprinterSanity` | 2 | Sanity restored by eating a Sprinter |
| `brainiacSanity` | 6 | Sanity restored by eating a Brainiac |
| `sprinterFleeRange` | 4 tiles | Wider guard detection |
| `sprinterFleeSpeed` | 2 tiles/move | Tiles per flee action |
| `brainiacScreamRange` | 6 tiles | 50% larger than base 4 |
| `standardCount` | 2 | Per-level count |
| `sprinterCount` | 2 | Per-level count |
| `brainiacCount` | 1 | Per-level count |

**Placement logic:**
1. Place Brainiacs first: find positions within 5 tiles of a guard. Distribute across different guard clusters.
2. Place Standards: use existing random placement logic.
3. Place Sprinters last: prefer positions far from guards (>6 tiles) and far from zombie spawn (>8 tiles).

### Prototyping Priority

**When to prototype:** After core civilian ecology (Step 2.2) is validated. This is a stretch goal that enriches an already-working system.

**Minimum prototype:** Add variant field + color rendering + per-variant sanity restore. Skip placement logic and AI changes initially — just randomize variant assignment among existing civilians.

**Key question prototype must answer:** Does variant-aware targeting create meaningfully different player decisions, or do players just eat whatever is closest regardless of type?

### Dependencies

- Requires: Base civilian system (eat action, death scream, flee/seek AI) — already implemented
- Requires: Civilian ecology spec (Step 2.2) — completed
- Blocked by: Nothing (stretch goal, independent of core loop)

### Known Risks

- **Color readability:** Three civilian colors at tile scale may be hard to distinguish. May need shape differences (circle, triangle, diamond) in addition to color.
- **AI complexity:** Variant-aware zombie AI for MC simulation adds branching to the heuristic decision tree. Risk of AI not exercising all variant types equally, skewing MC results.
- **Sprinter frustration:** If Sprinters are genuinely uncatchable at Lucid tier (4 movement vs. 2 flee-move), players may ignore them entirely, making them dead weight. Mitigation: Sprinters only move 2 tiles when actively fleeing; when wandering or idle, they move 1 tile like Standards.
- **Brainiac death spiral:** If the Brainiac's scream radius (6) consistently gets the zombie killed, players learn to never eat Brainiacs, removing 6 sanity from the economy (12 available instead of 18). Mitigation: tunable scream range, or reduce to 5 tiles if 6 proves too punishing.

---

## 9. Revision History

| Date | Version | Change | Reason |
|------|---------|--------|--------|
| Feb 2026 | 1.0 | Initial specification | Step 2.3 of design roadmap (stretch goal) |

---

## Appendix A: Lens #34 Triangularity Analysis

### Per-Variant Triangularity

**Lens #34 asks:** "Do my choices involve tradeoffs between risk and reward?"

Each variant embodies a distinct point on the risk/reward spectrum:

#### Standard (Blue) — The Baseline
- **Risk:** Moderate. Normal scream range (4 tiles). Placed randomly, so guard proximity varies.
- **Reward:** Moderate. +4 sanity, the reliable middle ground.
- **Triangularity:** Standard is the "safe" choice — not because it's risk-free, but because it's predictable. The player knows exactly what they're getting. It exists to make the other two types meaningful by contrast.
- **When dominant:** When the player has moderate sanity and moderate time. No urgency to maximize, no desperation to gamble.

#### Sprinter (Green) — Time vs. Safety
- **Risk:** Low direct danger (far from guards, unguarded). High opportunity cost (2-4 turns of pursuit = 1.8-3.6 sanity lost to drain while chasing).
- **Reward:** Low. +2 sanity. Net sanity gain after pursuit costs: approximately +0.2 to +1.1.
- **Triangularity:** The Sprinter inverts the usual risk. The danger isn't combat — it's the clock. Chasing a Sprinter is safe from hunters but burns the resource you're trying to restore. The sacrifice is time, not HP.
- **When dominant:** High sanity + plenty of time remaining. The player can afford a safe, low-yield eat because they're not under pressure. Also useful when HP is low and combat must be avoided.
- **When trap:** Low sanity + time pressure. The chase duration may drain more sanity than the +2 restores, creating a net negative. At this point, the Sprinter becomes a time sink.

#### Brainiac (Gold) — Danger vs. Efficiency
- **Risk:** Maximum. Placed near guards, 50% larger scream range (6 tiles), guaranteed hunter response post-eat.
- **Reward:** Maximum. +6 sanity (150% of Standard). Most efficient single eat in the game.
- **Triangularity:** The Brainiac is the classic "high risk, high reward" leg of the triangle. Eating one is the most efficient sanity restore available, but the player must survive the aftermath. The sacrifice is HP and safety.
- **When dominant:** Desperate situations — sanity critically low and the zombie needs maximum restore to survive. Also when a nearby hunter cluster has been thinned by monsters, creating a temporary opening.
- **When trap:** When multiple hunters are within the 6-tile scream range and the zombie has low HP. Eating the Brainiac may restore sanity but result in death-by-hunter within 1-2 turns.

### The Triangle as a Whole

```
        BRAINIAC (+6, max danger)
           /\
          /  \
         /    \
        / HIGH \
       / RISK   \
      /  HIGH    \
     /   REWARD   \
    /______________\
   /                \
STANDARD (+4)    SPRINTER (+2)
moderate risk     low risk
moderate reward   low reward
                  high time cost
```

The triangle has no dominant vertex:
- **Brainiac dominance check:** +6 sanity is huge, but the 6-tile scream and guard-adjacent placement mean the zombie often takes 1-2 HP damage in the aftermath. If taking 2 damage, the zombie is trading HP for sanity — not clearly dominant.
- **Standard dominance check:** +4 is reliable, but in a tight economy (18 total sanity), reliably mediocre may not be enough. Players who only eat Standards may run low.
- **Sprinter dominance check:** +2 is never enough to justify unless the player has sanity to burn. Net return after chase costs is marginal. But it's the only option that doesn't increase guard exposure.

**Context-sensitivity test:** The optimal choice changes with game state:
- Sanity 10, HP 4, 3 hunters nearby → eat Standard (moderate return, manageable risk)
- Sanity 3, HP 5, 1 hunter nearby → eat Brainiac (desperate, need the +6, can absorb hits)
- Sanity 8, HP 2, open path to exit → eat Sprinter (safe, low HP means can't risk combat)
- Sanity 5, HP 3, 2 Sprinters + 1 Brainiac left → genuine dilemma (chase Sprinters safely but slowly, or gamble on Brainiac?)

---

## Appendix B: Systems-Balance Analysis

### Sanity Economy Model

**Base system (no variants):** 5 civilians x +4 each = 20 total sanity available.

**Variant system:** 2 Standard (8) + 2 Sprinter (4) + 1 Brainiac (6) = **18 total sanity available**.

The 2-sanity reduction (10%) tightens the economy intentionally. With the base system, a zombie that eats 3 of 5 civilians restores 12 sanity — nearly full. With variants, eating 3 civilians yields 8-14 sanity depending on composition, making target selection matter.

### Is Any Variant Dominant?

**Net sanity analysis** (sanity restored minus sanity drained during pursuit):

| Variant | Restore | Avg Pursuit (turns) | Drain Cost (0.9/turn) | Net Sanity | Effective Rate |
|---------|---------|--------------------|-----------------------|------------|----------------|
| Standard | +4 | ~2 | -1.8 | +2.2 | 1.1/turn |
| Sprinter | +2 | ~4 | -3.6 | -1.6 to +0.4 | -0.4 to 0.1/turn |
| Brainiac | +6 | ~1.5 | -1.35 | +4.65 | 3.1/turn |

At first glance, Brainiac appears dominant (3.1 effective sanity per turn vs. 1.1 for Standard). However, the Brainiac's true cost includes:
- **HP cost:** Expected 0.5-1.5 HP damage from hunter response (1-3 hunters alerted within 6-tile range, ~50% hit rate each). At 1 damage per hit, eating a Brainiac costs approximately 1 HP on average.
- **Position cost:** Post-scream, the zombie is surrounded by converging hunters. Escaping costs additional movement turns.
- **When these costs compound:** At HP 2 or lower, a single unlucky hunter hit after a Brainiac eat means death. The +6 sanity is worthless if the zombie dies next turn.

**Verdict:** No dominant variant. Brainiac has the best sanity efficiency but the highest total cost (HP + position). Standard is reliable but insufficient alone in the tighter economy. Sprinter is only efficient when uncontested (high sanity, clear path).

### Does 18 Total Sanity Make Levels Too Hard?

**Comparison to base:**
- Base: 20 sanity available / 0.9 drain per round = ~22 rounds of drain covered by full consumption
- Variants: 18 sanity available / 0.9 drain per round = ~20 rounds of drain covered by full consumption

The 2-round difference is small (~10%). However, if the zombie eats only 3 of 5 civilians (typical based on MC data showing 0.5-2% eat rate per decision):
- Base: 3 x 4 = 12 sanity restored
- Variants (worst case: 3 Sprinters): 3 x 2 = 6 sanity restored
- Variants (best case: 1 Brainiac + 2 Standards): 6 + 4 + 4 = 14 sanity restored
- Variants (typical: 1 Standard + 1 Sprinter + 1 Brainiac): 4 + 2 + 6 = 12 sanity restored

Typical case matches the base system. Worst case (only Sprinters eaten) is significantly worse, but this only happens if the player makes poor choices or Brainiacs/Standards die to monsters. This pressure is intentional — it rewards good targeting.

**Verdict:** 18 total is viable. If MC testing shows win rates drop more than 5 percentage points vs. base, increase Sprinter restore to +3 (total becomes 20) as the first tuning lever.

### Does Sprinter Flee Speed Make Them Uncatchable?

**Movement comparison:**
- Zombie at Lucid: 4 tiles/turn. Sprinter fleeing: 2 tiles/turn. Net gain: 2 tiles/turn.
- Zombie at Slipping: 5 tiles/turn. Sprinter fleeing: 2 tiles/turn. Net gain: 3 tiles/turn.
- Zombie at Feral: 6 tiles/turn. Sprinter fleeing: 2 tiles/turn. Net gain: 4 tiles/turn.

If starting distance is 4 tiles (Sprinter's flee detection range), the zombie needs ~2 turns to close the gap at Lucid, ~1.3 turns at Slipping. This is catchable.

**Platform geometry matters:** Sprinters can only flee along a platform. If the platform is 8 tiles long and the Sprinter starts at tile 3, it can flee 5 tiles before hitting the edge. The zombie closing at 2 tiles/turn catches it in 2-3 turns.

**Escape scenario:** On a long open platform (>15 tiles), a Sprinter detected at max range could lead the zombie on a 4+ turn chase. This is the intended time-tax. The zombie loses 3.6 sanity to gain 2 — a net loss of 1.6. Smart players won't chase.

**Edge case — Sprinter flees to unreachable platform:** If a Sprinter jumps to a disconnected platform, it effectively escapes. This is acceptable: map geometry creates tactical variation. The Sprinter remains alive and may become reachable later.

**Verdict:** Sprinters are catchable but time-costly. At Lucid tier, a maximum-distance chase is a net sanity loss. At Feral tier, Sprinters are easily caught (4 tile/turn net gain). This is correct behavior: Sprinters are meant to be the "spend time, save HP" option.

### Does Brainiac Scream Range (6 tiles) Make Eating One Suicidal?

**Hunter density analysis:**

With default config (8 hunters on a 50x16 map), average hunter density is ~1 hunter per 100 tiles. Within a 6-tile Manhattan radius (area of ~72 tiles), expected hunters: ~0.72. However, Brainiacs are placed near guard clusters, so nearby hunter density is higher — estimated 1.5-2.5 hunters within 6 tiles.

**Expected outcome after Brainiac eat:**
- Hunters alerted: 1.5-2.5 (vs. 0.5-1.5 for Standard at 4-tile range)
- Alert duration: 2 turns
- Hunter movement: 2 tiles/turn, so a hunter 6 tiles away reaches the eat site in 3 turns (1 turn after alert expires)
- A hunter 4 tiles away reaches in 2 turns (during alert)
- A hunter 2 tiles away reaches in 1 turn (immediate guard)

**Survival model:**
- If 1 hunter attacks: 50% hit rate, 1 damage. Expected HP cost: 0.5.
- If 2 hunters attack: Expected HP cost: 1.0.
- If 3 hunters attack: Expected HP cost: 1.5.
- Zombie starts with 5 HP. Can survive 3-4 Brainiac eats before HP depletion (if starting at full HP with no other damage).

**Verdict:** Not suicidal. Eating a Brainiac typically costs ~1 HP, which is a meaningful but survivable price for +6 sanity. It becomes suicidal at HP 1-2, which is the intended danger threshold. If MC testing shows Brainiac eat survival below 30%, reduce scream range to 5 as first tuning lever.

### Tuning Levers (ranked by impact on variant balance)

| Lever | Current | Range | What It Tunes |
|-------|---------|-------|---------------|
| `brainiacSanity` | 6 | 4-8 | Brainiac reward — lower makes it less dominant |
| `sprinterSanity` | 2 | 1-4 | Sprinter reward — higher makes chasing more worthwhile |
| `brainiacScreamRange` | 6 | 4-8 | Brainiac risk — lower reduces hunter response |
| `sprinterFleeSpeed` | 2 | 1-3 | Sprinter catchability — lower makes them easier prey |
| `sprinterFleeRange` | 4 | 2-6 | Sprinter detection — lower means less warning, easier ambush |
| `brainiacCount` | 1 | 0-3 | Economy shape — more Brainiacs = higher ceiling, more risk |
| `standardCount` | 2 | 0-5 | Economy reliability — more Standards = more predictable |
| `sprinterCount` | 2 | 0-5 | Economy floor — more Sprinters = safer but slower |
