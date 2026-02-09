# Sanity-Movement Balance Report

**Produced by:** systems-balance skill
**Date:** Feb 2026
**Lenses applied:** #33 (Meaningful Choices), #34 (Triangularity), #21 (Flow), #9 (Elemental Tetrad)
**References:** balance-principles.md (feedback loops, meaningful choices, cost curves)

---

## 1. System Purpose

The sanity system exists to create one player decision:

> **"How much control am I willing to trade for power?"**

**Design intent:** Lower sanity = more powerful movement (faster, higher jumps, stronger acceleration) but less precise control (drift, reduced air steering, input delay). The risk/reward ratio should make players **WANT to flirt with low sanity**, not avoid it.

**Interaction map:**
```
Sanity (input) → getSanityT() → movement params (speed, jump, accel, decel, air control, gravity)
                               → drift system (impulse, interval, input delay)
                               → juice feedback (vignette, shake, visual effects)
```

The system has one tuning knob (sanity 0-12) that drives two output categories: **gains** (movement power) and **costs** (control loss). For the system to create meaningful choices, gains and costs must scale proportionally across the entire range.

---

## 2. Diagnostic Checklist

| Question | Finding | Verdict |
|----------|---------|---------|
| Is there a dominant strategy? | **Yes.** Sanity 7 is always optimal when the challenge is achievable at that level. | FAIL |
| Is any resource infinitely superior? | Sanity 7-12 provides increasing power at zero cost. | FAIL |
| Is there a steady state where the player stops engaging? | Yes — player sets sanity to 7 and never changes unless forced. | FAIL |
| Does the system produce different outcomes in different contexts? | Partially. Vertical challenges force lower sanity. Horizontal/precision challenges favor higher sanity. | PARTIAL PASS |
| Are costs proportional to gains at every stage? | **No.** Gains scale smoothly; costs jump at tier boundaries. | FAIL |

**Overall: The tradeoff curve is broken.** The curve has three specific problems detailed below.

---

## 3. Movement Gains by Sanity Level

All parameters interpolate smoothly via `getSanityT() = (12 - sanity) / 12`.

| Sanity | t | Max Speed | Accel | Decel | Air Ctrl | Jump Vel | Gravity | Jump Peak | Horiz Range* |
|--------|---|-----------|-------|-------|----------|----------|---------|-----------|-------------|
| 12 | 0.000 | 300 | 2400 | 3200 | 0.80 | 600 | 1400 | 4.0 tiles | 8.0 tiles |
| 10 | 0.167 | 325 | 2640 | 2933 | 0.73 | 660 | 1435 | 4.7 tiles | 9.3 tiles |
| **7** | **0.417** | **363** | **3000** | **2533** | **0.63** | **750** | **1488** | **5.9 tiles** | **11.4 tiles** |
| 6 | 0.500 | 375 | 3120 | 2400 | 0.60 | 780 | 1505 | 6.3 tiles | 12.1 tiles |
| 4 | 0.667 | 400 | 3360 | 2133 | 0.53 | 840 | 1540 | 7.2 tiles | 13.6 tiles |
| **3** | **0.750** | **413** | **3480** | **2000** | **0.50** | **870** | **1558** | **7.6 tiles** | **14.3 tiles** |
| 1 | 0.917 | 438 | 3720 | 1733 | 0.43 | 930 | 1593 | 8.5 tiles | 15.9 tiles |

*Horizontal range = max speed × total airborne time, assuming max speed at launch. Real range is lower due to acceleration time.

**Key observation:** The gain curve is smooth and linear. Every tick of the slider provides proportional improvement. This is well-designed.

---

## 4. Control Costs by Sanity Level

Here the problems emerge. Costs are **not smooth** — they jump at tier boundaries.

| Sanity | Tier | Drift Impulse | Avg Interval | Airborne Mult | Input Delay | Effective Grounded Cost* | Effective Airborne Cost* |
|--------|------|---------------|-------------|---------------|-------------|------------------------|------------------------|
| 12-7 | Lucid | **0** | ∞ | — | 0 | **0 tiles** | **0 tiles** |
| 6.99 | Slipping | 100 px/s | 1.75s | ×2 | 0 | ~0.05 tiles | ~0.1 tiles |
| 5 | Slipping | 100 px/s | 1.75s | ×2 | 0 | ~0.05 tiles | ~0.1 tiles |
| 4.01 | Slipping | 100 px/s | 1.75s | ×2 | 0 | ~0.05 tiles | ~0.1 tiles |
| **3.99** | **Feral** | **500 px/s** | **1.3s** | **×2** | **0.03s** | **~1.5 tiles** | **~2.8 tiles** |
| 3 | Feral | 500 px/s | 1.3s | ×2 | 0.03s | ~1.5 tiles | ~2.8 tiles |
| 1 | Feral | 500 px/s | 1.3s | ×2 | 0.03s | ~1.5 tiles | ~2.8 tiles |

*Effective cost = horizontal displacement lost per drift event (reverse-direction only; same-direction drift is eaten by speed clamp — see §5).

### The Three Problems

**Problem 1: The Lucid Free Zone (sanity 7-12)**
Sanity 7 provides +48% jump peak, +21% max speed, and +25% acceleration over sanity 12 — with **zero drift cost**. This is pure power. A knowledgeable player would never stay at sanity 12 when 7 gives a 48% jump height increase for free.

Violates: Lens #33 (Meaningful Choices) — the choice between sanity 12 and 7 is false. 7 is strictly better.

**Problem 2: Slipping Drift is Negligible**
Slipping drift impulse (100 px/s) cannot reverse the zombie's direction at typical movement speeds (300+ px/s). A grounded zombie at sanity 5 moving at max speed (388 px/s) absorbs a 100 px/s drift and only slows to 288 px/s — barely noticeable. The zombie never actually changes direction from Slipping drift.

Effective cost of Slipping tier: **near zero**. Players get sanity 4.01 benefits (7.2-tile jumps, 400 px/s speed) with negligible risk.

**Problem 3: The Feral Cliff**
Crossing from sanity 4 (Slipping) to sanity 3.99 (Feral) causes:
- Drift impulse: 100 → 500 px/s (**5× increase**)
- Input delay: 0 → 0.03s (new mechanic appears)
- Airborne effective cost: ~0.1 → ~2.8 tiles (**28× increase**)

The gain for crossing this boundary: jump peak goes from 7.2 → 7.6 tiles (+5.6%). The player pays a 28× increase in drift cost for a 5.6% movement gain. This is massively disproportionate.

---

## 5. Drift Displacement Analysis

Drift fires as a velocity impulse (`zombie.vx += direction * impulse`). Because drift is applied AFTER the speed clamp in the game loop (`updatePhysics → updateDrift`), the behavior is asymmetric:

### Same-direction drift (50% of events)
The drift-inflated velocity exceeds maxSpeed. Next frame, the clamp fires before movement: `zombie.vx = Math.max(-maxSpeed, Math.min(maxSpeed, zombie.vx))`. The zombie moves at normal max speed. **Net displacement: zero.**

This means 50% of drift events are invisible. Drift is pure downside with no upside — it can hurt you but never help you.

### Reverse-direction drift (50% of events)
The zombie's velocity reverses. Next frame, the clamp doesn't fire because the magnitude may still be within maxSpeed (just in the wrong direction). Acceleration from player input gradually brings velocity back. During recovery:

**Feral grounded reverse drift (impulse 500, holding input):**
- Starting vx: ~-90 (after impulse reverses from +413)
- Accel per frame: 55.7 px/s (Feral accel × dt)
- Frames to recover: ~8 frames (0.133s)
- Backwards displacement: **~28px (0.87 tiles)**
- Re-acceleration cost: **~21px (0.66 tiles)**
- **Total cost: ~1.53 tiles**

**Feral airborne reverse drift (impulse 1000, holding input):**
- Starting vx: ~-588 (after impulse)
- Accel per frame: 27.8 px/s (Feral accel × air control × dt)
- Frames to recover: ~15 frames (0.25s)
- Backwards displacement: **~52px (1.6 tiles)**
- Re-acceleration cost: **~39px (1.2 tiles)**
- **Total cost: ~2.8 tiles**

**Slipping grounded reverse drift (impulse 100, holding input):**
- Starting vx: +288 (100 px/s is not enough to reverse from 388)
- No reversal — zombie just slows momentarily
- **Total cost: ~0.05 tiles (imperceptible)**

### Drift probability during a jump

| Tier | Jump Duration | Avg Drift Interval | P(≥1 drift during jump) |
|------|--------------|-------------------|------------------------|
| Lucid | 0.86-1.01s | ∞ | 0% |
| Slipping | 1.04-1.09s | 1.75s | ~45% (but negligible effect) |
| Feral | 1.12-1.17s | 1.3s | ~60-86% (devastating effect) |

---

## 6. Tradeoff Curve Visualization

```
GAIN (movement power)                          COST (control loss)
▲                                               ▲
│                                 ╱             │                    ╱
│                               ╱               │                  ╱
│                             ╱                 │                ╱  Feral drift
│                           ╱  smooth           │              ╱   (500 px/s + delay)
│                         ╱    linear           │            ╱
│                       ╱      gains            │          ╱
│                     ╱                         │        ╱
│                   ╱                           │      ╱
│                 ╱                             │  ···· Slipping drift
│               ╱                               │  ·    (100 px/s, negligible)
│             ╱                                 │  ·
│           ╱                                   │  ·
│         ╱                                     │··
│       ╱                                       │
│     ╱                                         │  Zero cost
│   ╱                                           │  (Lucid free zone)
│ ╱                                             │
└─────────────────────────── Sanity →           └─────────────────────────── Sanity →
12    10     7     6     4     3     1           12    10     7     6     4     3     1
```

**The gain curve is a smooth ramp. The cost curve is a staircase with two steps and a flat section. They don't match.**

For meaningful choices (Lens #33), the cost curve should approximate the gain curve's shape: smooth, proportional, always increasing.

---

## 7. Dominant Strategy Analysis

### Current dominant strategies

| Challenge Type | Optimal Sanity | Why |
|---------------|---------------|-----|
| Any challenge achievable at Lucid | **7** (lowest Lucid) | Maximum free power, zero drift |
| Vertical challenge requiring Slipping | **4.01** (highest Slipping before Feral) | Maximum Slipping power, negligible drift |
| Vertical challenge requiring Feral | **Just barely Feral** (3.5-3.99) | Minimum time in Feral, same drift at all Feral sanity |
| Horizontal gap requiring speed | **7** or **4.01** | Speed gain from lower sanity is marginal vs. drift cost |

**The test (from balance-principles.md):** "Would a knowledgeable player ever choose the other option?"

- Would a player choose sanity 12 over 7? **No.** 7 is strictly better.
- Would a player choose sanity 5 over 4.01? **No.** 4.01 gives more power with the same tier cost.
- Would a player choose sanity 2 over 3.99? **Rarely.** The marginal gain is tiny, the cost is identical.

The only genuine decision point is: **"Do I need Feral-level power for this challenge?"** That's a binary choice, not the smooth risk/reward slider the design intends.

---

## 8. Recommendations

### Recommendation 1: Smooth the Drift System (HIGH PRIORITY)

**Problem:** Tier-based drift activation creates cost cliffs.
**Fix:** Make drift impulse and interval interpolate smoothly with `getSanityT()`, matching how movement parameters already work.

**Proposed implementation:**
```
driftImpulse = DRIFT_BASE_IMPULSE * getSanityT()²
driftInterval = DRIFT_MAX_INTERVAL - (DRIFT_MAX_INTERVAL - DRIFT_MIN_INTERVAL) * getSanityT()
inputDelay = FERAL_INPUT_DELAY * Math.max(0, getSanityT() - 0.5) * 2
```

**Proposed constants:**
| Constant | Value | Effect |
|----------|-------|--------|
| DRIFT_BASE_IMPULSE | 500 | Max impulse at sanity 0 (same as current Feral) |
| DRIFT_MIN_INTERVAL | 0.8 | Shortest interval at sanity 0 |
| DRIFT_MAX_INTERVAL | 5.0 | Longest interval at sanity 12 (drift fires but very rarely) |
| DRIFT_ONSET_SANITY | 10 | Sanity below which drift begins |

**Why squared:** Using `getSanityT()²` for impulse creates a convex curve — drift starts very gently and accelerates. At sanity 9, impulse would be only ~35 px/s (barely noticeable). At sanity 3, it would be ~280 px/s (scary). At sanity 1, ~420 px/s (devastating). This matches the feel intent: gradual creep into danger, not a cliff.

**Sanity point sample with squared impulse:**

| Sanity | t | t² | Impulse | Interval | Feel |
|--------|---|----|---------|----------|------|
| 12 | 0.00 | 0.00 | 0 | — | Perfect control |
| 10 | 0.17 | 0.03 | 14 | 4.3s | Barely perceptible |
| 7 | 0.42 | 0.17 | 87 | 3.2s | Occasional subtle nudge |
| 5 | 0.58 | 0.34 | 170 | 2.6s | Noticeable, manageable |
| 3 | 0.75 | 0.56 | 281 | 1.9s | Scary, forces correction |
| 1 | 0.92 | 0.84 | 420 | 1.1s | Devastating |

This eliminates the Lucid free zone, eliminates the Feral cliff, and creates a smooth cost curve that matches the smooth gain curve.

**Tier boundaries still exist for other purposes** (vignette color, Gone freeze, visual identity). They just no longer drive drift mechanics.

### Recommendation 2: Fix Same-Direction Drift Asymmetry (MEDIUM PRIORITY)

**Problem:** Same-direction drift (50% of events) is invisible because the speed clamp eats it before movement.
**Fix:** Move drift application into `updatePhysics`, between input acceleration and speed clamp. Or: exempt drift-added velocity from the clamp for 1 frame.

**Simplest approach:** Apply drift impulse as a one-frame position offset instead of a velocity change:
```javascript
// In updateDrift, instead of zombie.vx += direction * impulse:
zombie.x += direction * impulse * dt;  // direct position nudge
zombie.vx += direction * impulse * 0.3; // small velocity residue for feel
```

This makes drift visible in both directions: same-direction drift pushes you past your target (overshoot), reverse-direction drift pushes you backward (undershoot). Both are costs in a precision platforming context.

**Why this matters for Triangularity (Lens #34):** Currently drift is pure downside. Making it bidirectional creates interesting moments where drift accidentally helps (pushed onto a platform you'd have missed). This turns "the zombie fought me" into "the zombie is chaotic" — more thematic for the gentleman zombie losing his mind.

### Recommendation 3: Scale Input Delay Smoothly (LOW PRIORITY)

**Problem:** Input delay appears suddenly at the Feral boundary.
**Fix:** If Recommendation 1 is implemented, input delay should also interpolate:
```
inputDelay = FERAL_INPUT_DELAY * Math.max(0, (getSanityT() - 0.5) * 2)
```
This means input delay starts at sanity 6 (t = 0.5) and reaches full 0.03s at sanity 0. Below sanity 6, the zombie's body begins resisting direction changes.

### Recommendation 4: Narrow the Lucid Free Zone (ALTERNATIVE to Rec 1)

If smooth drift is too large a change for this phase, a simpler fix:

- Move Lucid/Slipping boundary from sanity 7 to sanity 10
- Move Slipping/Feral boundary from sanity 4 to sanity 6
- Increase Slipping impulse from 100 to 250 px/s

This doesn't fix the cliff problem but reduces the free zone from 5 sanity points (7-12) to 2 (10-12) and makes Slipping drift actually perceptible.

---

## 9. Impact Assessment

| Recommendation | Effort | Impact | Risk |
|---------------|--------|--------|------|
| 1. Smooth drift | M (refactor updateDrift, remove tier checks, add new constants) | **HIGH** — eliminates all three curve problems | Movement feel changes significantly; requires retuning |
| 2. Fix drift asymmetry | S (change drift application method) | MEDIUM — makes drift feel fair and interesting | Could make drift feel too disruptive if impulse is too high |
| 3. Smooth input delay | S (one formula change) | LOW — improves Feral onramp | Minimal risk |
| 4. Boundary shift (alternative) | S (change 3 constants) | MEDIUM — reduces but doesn't eliminate cliffs | Simpler to tune, doesn't address root cause |

**Recommended implementation order:** 1 → 2 → 3 (or 4 → 2 → 3 if conservative)

---

## 10. Tuning Guide

Post-implementation, these are the most impactful knobs:

| Parameter | What It Controls | Turn Up → | Turn Down → |
|-----------|-----------------|-----------|-------------|
| DRIFT_BASE_IMPULSE | Maximum drift strength at sanity 0 | More chaotic, more terrifying | Safer, less dramatic |
| DRIFT_MAX_INTERVAL | How rare drift is at high sanity | Longer free zone (more generous) | Earlier drift onset (more punishing) |
| DRIFT_MIN_INTERVAL | How frequent drift is at sanity 0 | Less frequent (more manageable) | Constant barrage (overwhelming) |
| Impulse curve exponent (²) | How fast drift ramps up | Gentle start, explosive end | Linear ramp (more predictable) |
| DRIFT_AIRBORNE_MULT | Airborne terror factor | Jumping becomes very risky | Jumping is safe, ground drift is the guard |
| FERAL_INPUT_DELAY | Direction reversal resistance | More committed movement | More responsive movement |

---

## 11. Pre-Implementation Validation

Before implementing, these hypotheses should be testable in the toy:

1. **"At every sanity level, the player can feel the drift getting stronger."** (Currently false — sanity 8-12 feel identical, 5-7 feel identical.)
2. **"No sanity level is strictly better than an adjacent level."** (Currently false — 7 > 12, 4.01 > any Feral.)
3. **"Drift occasionally helps the player (overshoot onto a platform)."** (Currently impossible — same-direction drift is invisible.)
4. **"The player hesitates before lowering sanity."** (Currently only true at the Feral boundary.)

---

*Produced using the gamedev-systems-balance skill. Lenses applied: #33 (Meaningful Choices — are all sanity levels viable?), #34 (Triangularity — does the risk/reward create tension?), #21 (Flow — does the cost curve keep the player in the engagement zone?), #9 (Elemental Tetrad — do the numbers support the "losing your mind" fantasy?). References: balance-principles.md (feedback loops, cost curves, meaningful choices).*
