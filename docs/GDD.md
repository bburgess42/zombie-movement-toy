# Zombie Platformer — Game Design Document
## Paper Prototype Balance Report & Production GDD

**Version:** Post-balance tuning (Feb 2026)
**Prototype:** Browser-based HTML/JS paper prototype with Monte Carlo validation
**Repo:** `bburgess42/zombie-platformer`
**Purpose:** Validated balance framework ready for production game build

---

## 1. Core Concept

**Genre:** Turn-based tactical survival / stealth platformer
**Perspective:** Top-down grid (50x16 tiles in prototype)
**Player Role:** A sentient zombie trying to escape a hostile area while managing sanity

**One-line pitch:** You're a zombie who's still thinking — eat civilians to stay sane, avoid hunters, navigate monsters, and reach the exit before your mind goes.

**Core fantasy:** You are a zombie clinging to the last threads of consciousness. You must reach the exit before your mind is gone — but survival demands horrific choices. Every civilian you eat buys time but makes you more of the monster you're trying not to become.

**Core tension:** Eating civilians restores sanity but alerts nearby hunters. Moving fast burns through the map but leaves you exposed. Losing sanity makes you stronger but uncontrollable. Every decision is a tradeoff.

---

## 2. Core Loop

```
Enter Map -> Navigate toward Exit -> Manage Sanity -> Eat/Fight/Flee -> Reach Exit or Die
```

**Per-round structure:**
1. **Zombie turn** — Player moves (up to movement limit) and takes one action (attack, eat, or wait)
2. **Hunter phase** — All hunters execute AI (respond to alerts, pursue zombie, guard civilians, fight monsters)
3. **Monster phase** — All monsters execute AI (wander, attack nearby entities)
4. **Civilian phase** — All civilians execute AI (flee from threats, seek safety near hunters)
5. **End of round** — Sanity drains, starvation check, new round begins
6. **Involuntary action check** — At Slipping/Feral tiers, zombie may act against player's will

**Win condition:** Zombie reaches exit tile alive
**Lose conditions:**
- Sanity reaches 0 (mind lost)
- HP reaches 0 (killed)
- Starvation (sanity reaches 0 while no civilians remain — accelerated drain)

---

## 3. The Sanity System

The central mechanic. Sanity is a depleting resource that determines the zombie's capabilities and behavior.

### 3.1 Sanity Tiers

| Tier | Sanity Range | Movement | Attack (d6 >=) | Behavior |
|---|---|---|---|---|
| **Lucid** | >= 7 | 4 tiles | 3+ hits | Full player control |
| **Slipping** | 4-6 | 5 tiles | 2+ hits | Involuntary eat check (d6: 1-2 triggers) |
| **Feral** | 1-3 | 6 tiles | 2+ hits | Involuntary attack + eat check (d6: 1-4 triggers) |
| **Gone** | 0 | 0 | never | Game over |

**Key design insight:** Lower sanity = faster and more dangerous in combat, but less controllable. This creates a genuine dilemma: letting sanity drop gives tactical advantages (more movement, better hit rate) but risks losing control entirely.

### 3.2 Sanity Economy

| Source | Value | Notes |
|---|---|---|
| Starting sanity | **12** | ~13 rounds before Gone at base drain |
| Drain per round | **0.90** | Constant bleed; ~13 rounds Lucid-to-Gone without eating |
| Sanity per eat | **4** | Restores 33% of max; need 2-3 eats for full recovery |
| Max sanity | 12 | Cannot exceed starting value |
| Starvation multiplier | **1.5x** | Drain becomes 1.35/round when all civilians dead |

### 3.3 Sanity Timeline (no eating)

| Round | Sanity | Tier |
|---|---|---|
| 1 | 12.0 | Lucid |
| 6 | 7.5 | Lucid |
| 7 | 6.6 | Slipping |
| 9 | 4.8 | Slipping |
| 10 | 3.9 | Feral |
| 13 | 1.3 | Feral |
| 14 | 0.4 | Feral |
| 15 | 0 | Gone |

### 3.4 Involuntary Actions

When the zombie enters Slipping or Feral tier, it may act against the player's will at the start of its turn:

**Feral:**
- If any creature is adjacent, involuntarily attack the nearest one
- If a civilian is adjacent, roll d6: on 1-4, involuntarily eat them (turn consumed)

**Slipping:**
- If a civilian is adjacent, roll d6: on 1-2, involuntarily eat them (turn consumed)

Involuntary eats still restore sanity and trigger the death scream mechanic.

---

## 4. Entity Types

### 4.1 Zombie (Player)

| Stat | Value |
|---|---|
| HP | 5 |
| Starting Sanity | 12 |
| Movement | 4/5/6 (by tier) |
| Attack | d6 >= 3/2/2 (by tier), 1 damage |

**Actions per turn:** Move (up to movement limit) + one action (attack adjacent, eat adjacent civilian, or wait). Eating consumes the entire turn.

### 4.2 Hunters

| Stat | Value |
|---|---|
| HP | 3 |
| Movement | 2 tiles |
| Attack | d6 >= 4, 1 damage |
| Aggro Range | 5 tiles |
| Fight Range | 3 tiles (monsters) |
| Guard Range | 2 tiles (civilians) |
| Leash Distance | 5 tiles (won't stray far from civilians) |

**AI Priority (highest to lowest):**
1. **Respond to death scream** — If alerted, move toward scream location for 2 turns
2. **Pursue zombie** — If zombie within aggro range, chase and attack
3. **Fight monsters** — If monster within fight range, engage
4. **Guard civilians** — Stay near nearest civilian (max 2 guards per civilian, surplus hunt threats)
5. **Patrol** — Move toward nearest civilian if too far

Hunters are the zombie's primary antagonist. They cluster around civilians, making eating dangerous. The leash mechanic means more civilians = more spread-out hunter coverage.

### 4.3 Monsters

| Stat | Value |
|---|---|
| HP | 3 |
| Movement | 2 tiles |
| Attack | d6 >= 4, 1 damage |

**AI:** Wander randomly, attack any adjacent entity (including zombie, hunters, civilians). Monsters are chaotic neutral — they kill civilians (destroying the zombie's food), kill hunters (helping the zombie), and attack the zombie (hurting it). They create unpredictability.

### 4.4 Civilians

| Stat | Value |
|---|---|
| HP | 1 |
| Movement | 1 tile |
| Flee Range | 3 tiles |
| Seek Range | 4 tiles |

**AI:** Flee from zombie/monsters within flee range, seek safety near hunters within seek range. 1 HP means any hit kills them. They're the zombie's food source and the hunters' wards.

---

## 5. Death Scream Mechanic

When the zombie eats a civilian (by any method — player action, AI decision, or involuntary), all hunters within **4 tiles** (Manhattan distance) become alerted:

- Alerted hunters gain `alertedTo` coordinates and `alertTurns = 2`
- For 2 turns, responding to the scream is their highest-priority AI action
- They move toward the eat location and attack the zombie if adjacent
- Alert clears when they arrive or 2 turns expire

**Design purpose:** Makes eating a risk/reward decision. The zombie needs to eat to survive, but eating attracts danger. Encourages eating when hunters are far away and punishes careless feeding.

---

## 6. Starvation Mechanic

When all civilians on the map are dead (eaten by zombie, killed by monsters), the sanity drain multiplies by **1.5x** (0.90 becomes 1.35 per round).

- Creates a death spiral when food runs out
- Tracked as `lose_starvation` outcome (distinct from regular sanity loss)
- Particularly punishing with low civilian counts and high monster counts (monsters consume the food supply)
- Interactive game shows warning: "All civilians dead -- accelerated sanity drain!"

---

## 7. Map Design

### 7.1 Current Prototype

- **Grid:** 50 wide x 16 tall
- **Tile types:** Ground (passable), Wall (impassable)
- **Platformer layout:** Ground layer at bottom, platforms at various heights, connected by ladders/gaps
- **Entry point:** Left side of map
- **Exit point:** Right side of map
- **Pathfinding:** BFS on walkable tiles (ground, ladders, platforms)

### 7.2 Map Generation

Procedural section-based generation with 5 terrain types:
- **Canyon**: Ground gaps forcing vertical traversal
- **Tower**: Stacked platforms for climbing
- **Tunnel**: Low ceiling creating upper/lower paths
- **Open**: Sparse platforms, wide sightlines
- **Maze**: Dense platforms with pillars

Post-generation passes: ground gap filling, edge alignment, ladder punching, overhang placement, BFS reachability pruning.

### 7.3 Placement Rules

- Zombie spawns at entry point (left)
- Exit point on right side
- Entities placed randomly on walkable tiles, avoiding overlap
- Hunters cluster near civilians due to leash mechanic

### 7.4 Map Editor

Prototype includes a built-in map editor for designing custom layouts. Maps can be tested in interactive play or MC simulation.

---

## 8. Validated Balance Parameters

All values validated by Monte Carlo simulation (27,000 games across 27 parameter combos).

### 8.1 Default Configuration

```
zombieHP: 5          zombieStartSanity: 12     sanityDrainPerRound: 0.90
sanityPerEat: 4      starvationDrainMultiplier: 1.5    screamAlertRange: 4
lucidThreshold: 7    slippingThreshold: 4      feralThreshold: 1
zombieLucidMove: 4   zombieSlippingMove: 5     zombieFeralMove: 6
civilianCount: 10    civilianHP: 1             civilianFleeRange: 3
civilianSeekRange: 4 hunterCount: 8            hunterHP: 3
hunterMovement: 2    hunterAggroRange: 5       hunterFightRange: 3
hunterGuardRange: 2  hunterLeashDistance: 5    monsterCount: 4
monsterHP: 3         monsterMovement: 2
```

### 8.2 Difficulty Presets

| Preset | Key Overrides | Design Intent |
|---|---|---|
| **Easy** | 6 civs, 1 HP hunters, 3 aggro, 15 sanity | Forgiving; learn mechanics |
| **Default** | As above | Balanced; ~30-40% win rate |
| **Hard** | 3 civs, 3 hunters, 1.5 drain, 6 aggro, 10 sanity | Punishing; expert play required |
| **Chaos** | 4 monsters, 1 hunter, 6 civs, 1.5 drain | Unpredictable; monster-dominated |

### 8.3 Monte Carlo Results Summary (27,000 sims, Feb 2026)

**Sweep parameters:** civilianCount [2, 6, 12], monsterCount [2, 6, 12], hunterCount [2, 6, 12]

**Win rates by composition:**

| Civs | Few Enemies (M2/H2) | Mid (M6/H6) | Many Enemies (M12/H12) | Average |
|---:|---:|---:|---:|---:|
| 2 | 55.9% | 37.1% | 26.6% | ~35% |
| 6 | 52.1% | 35.6% | 21.4% | ~33% |
| 12 | 48.6% | 39.3% | 20.1% | ~34% |

**Key findings:**
- Win rates range 20-56% across all combos (healthy range for skill expression)
- Civilian count is roughly neutral (not irrelevant, not inverted)
- More hunters/monsters = lower win rate (expected difficulty scaling)
- Starvation deaths: 0-8% in low-civilian combos (working as designed)
- Tier distribution: ~66% Lucid / 17% Slipping / 16% Feral (improved from baseline 85/8/7)
- AI exit pursuit: ~28-47% of decisions (zombie has enough runway to plan)
- Zero timeout games (all games resolve naturally)

**Death cause distribution (averaged):**

| Cause | Low Enemies | Mid Enemies | High Enemies |
|---|---|---|---|
| Sanity drain | 40-90% | 25-45% | 5-20% |
| HP (killed) | 5-20% | 25-45% | 55-75% |
| Starvation | 0-8% | 0-3% | 0-5% |

Games against few enemies tend to be long sanity wars; games against many enemies are short brutal combats.

---

## 9. Strategic Depth

### 9.1 Player Decisions

1. **Route planning:** Direct path to exit vs. detour for civilians
2. **When to eat:** Early eating (safe but wastes time) vs. late eating (risky but efficient)
3. **Sanity management:** Stay Lucid for control, or drop to Slipping/Feral for speed/combat bonuses
4. **Fight or flee:** Engage blocking enemies (costs HP, gains path) vs. find alternate route (costs time/sanity)
5. **Scream awareness:** Eat when hunters are far, avoid eating near hunter clusters

### 9.2 Emergent Dynamics

- **Monster chaos:** Monsters kill hunters (opening paths) and civilians (destroying food) unpredictably
- **Hunter clustering:** The leash mechanic creates natural "safe zones" around civilian groups and "open lanes" elsewhere
- **Starvation spiral:** Monsters killing all civilians before the zombie can eat creates a death clock
- **Involuntary betrayal:** Feral-tier zombie may attack a needed ally (monster fighting a hunter) or eat a civilian at the worst possible time

---

## 10. Known Balance Observations & Future Tuning Levers

### 10.1 What's Working

- **Starvation mechanic** — Clean, thematic, appropriately rare (0-8%)
- **Death scream** — Creates meaningful eat-timing decisions at range 4
- **Win rate range** — 20-56% gives room for player skill expression
- **Sanity drain** — At 0.90/round, creates consistent ~13-round clock
- **Monster ecosystem** — Adds chaos without dominating outcomes
- **Movement/combat tier scaling** — Feral zombie is faster and hits harder, creating a genuine "risk the madness" option

### 10.2 Areas for Production Tuning

- **Tier distribution** — Still Lucid-heavy at 66%. If production wants more Slipping/Feral gameplay, reduce starting sanity to 10-11 (at cost of lower win rates). The tier thresholds (7/4/1) are another lever.
- **Involuntary actions** — Currently rare (~0.17 attacks/game, ~0.02 eats/game). In production with real-time gameplay and more encounters, these may fire more often. If not, lower Feral/Slipping thresholds or broaden trigger conditions.
- **Eating rate** — AI zombie only eats in ~0.5-2% of decisions. Human players will likely eat more strategically. Production should track this and adjust `sanityPerEat` if needed.
- **Civilian relevance** — Currently neutral (not positive). To make civilians a net positive for the zombie, could add: sanity bonus for being near civilians, civilians that drop items, or civilians that can be "saved" for bonus.

### 10.3 Tuning Levers (ranked by impact)

| Lever | Effect | Current | Range to Try |
|---|---|---|---|
| `zombieStartSanity` | Total game length, tier time | 12 | 10-15 |
| `sanityDrainPerRound` | Urgency, tier progression | 0.90 | 0.75-1.25 |
| `sanityPerEat` | Value of eating, recovery speed | 4 | 3-6 |
| `hunterCount` | Map pressure, combat frequency | 8 | 2-12 |
| `screamAlertRange` | Eat risk, hunter response | 4 | 2-8 |
| `starvationDrainMultiplier` | Death spiral severity | 1.5 | 1.25-2.0 |
| `hunterLeashDistance` | Hunter coverage spread | 5 | 3-8 |
| `hunterAggroRange` | Detection danger zone | 5 | 3-8 |

---

## 11. Target Forms of Fun (Heeter/Garneau 16 Forms)

- **EXTENSIVE**: Intellectual Problem Solving, Thrill of Danger, Discovery, Application of Ability
- **MODERATE**: Power, Advancement & Completion, Immersion
- **LIGHT**: Competition (vs AI), Comedy (dark humor of the premise)
- **ABSENT**: Social Interaction (single player), Physical Activity (turn-based), Love, Altruism (you're eating people), Creation

---

## 12. Production Considerations

### 12.1 What Transfers Directly from Prototype

- All numeric balance values (validated by simulation)
- Entity AI behavior priorities and logic
- Sanity tier system and thresholds
- Combat system (d6, hit thresholds, 1 damage)
- Starvation and death scream mechanics
- Difficulty presets
- Map generation algorithm (section-based procedural)

### 12.2 What Needs Production Design

- **Real-time vs. turn-based:** Prototype is strict turn-based. Production could be real-time with pause, or action-based with "turns" as action points.
- **Visual design:** Tile art, animations, UI/UX, sanity visual effects (screen distortion at Feral?)
- **Audio:** Scream SFX, ambient sanity-tier music shifts, combat sounds
- **Level design:** Hand-crafted maps vs. procedural generation. Prototype uses random platformer layouts.
- **Progression:** Multiple levels? Increasing difficulty? Unlockable abilities?
- **Narrative:** Why is the zombie sentient? What's beyond the exit? Who are the hunters?
- **Involuntary action UX:** How to communicate loss of control to the player (screen shake, forced camera movement, brief cutscene?)
- **Scream visualization:** Alert radius indicator, hunter reaction animations
- **Save/load:** Not in prototype
- **Multiplayer potential:** Could one player control hunters while another controls zombie?

### 12.3 Recommended Engine

The prototype was designed with Godot 4.3 compatible patterns (grid-based, tile-based, entity-component). GDScript implementation should map 1:1 from the JS prototype logic.

---

## 13. Appendix: Balance Iteration History

| Version | Change | Result |
|---|---|---|
| Initial | Sanity 15, drain 0.75, eat 5 | 85% Lucid, civilians irrelevant, no starvation |
| Drain 0.90 | Increased drain to 0.90 | Win rates dropped, still Lucid-heavy |
| Sanity 10 + starvation + scream(8) | Full balance pass | Tiers 50/28/22 but win rates crashed to 3-27%, civilians inverted |
| Scream range 8->4 | Reduced alert range | No meaningful change (scream barely fired) |
| **Sanity 12 (current)** | Increased starting sanity | Win rates 20-56%, tiers 66/17/16, civilians neutral, starvation 0-8% |

### Prototype Technical Details

Single file: `index.html` (~3500 lines). All game logic, rendering, AI, MC simulation, map editor in one file.

**How to run MC simulations:**
- **In-browser:** Open `index.html`, click Monte Carlo, configure sweep, run
- **Auto-sweep:** Open `index.html#automc` for automated 27-combo sweep
- **Headless:** Run `run-mc.py` to execute via Edge headless, outputs JSON
