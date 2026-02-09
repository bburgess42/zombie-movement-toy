# Architecture Refactor Plan: Brains for Breakfast

**Last Updated:** Feb 2026
**Engine:** Custom HTML5 Canvas (single-file, no build tools)
**Language:** JavaScript (ES6, no modules)
**Repository:** github.com/bburgess42/brains-for-breakfast (future)
**Target Platform(s):** Web (any modern browser)
**Status:** Proposed (pre-Sprint 1)
**Skill Applied:** gamedev-code-architecture (4-step process: Map → Choose Patterns → Define Boundaries → Implement+Refactor)

---

## Architecture Diagram Description

### Current Architecture (Movement Toy)

```
┌─────────────────────────────────────────────────────────────────────┐
│                      index.html (~2,162 lines)                      │
│                                                                     │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │                   SCATTERED GLOBALS                          │   │
│  │  zombie{} · sanity · input{} · drift{} · particles[]       │   │
│  │  camera{} · tileMap[][] · keysDown{}                        │   │
│  │  canvas · ctx · lastTime · 40+ tunable constants (let)      │   │
│  └────────┬───────────┬───────────┬───────────┬────────────────┘   │
│           │           │           │           │                     │
│           ▼           ▼           ▼           ▼                     │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐             │
│  │ processIn│ │ updatePhy│ │ updateDri│ │ render() │             │
│  │ put()    │ │ sics(dt) │ │ ft(dt)   │ │          │             │
│  └──────────┘ └──────────┘ └──────────┘ └──────────┘             │
│       │            │            │            │                     │
│       └────────────┴────────────┴────────────┘                     │
│                         │                                           │
│           All functions read/write globals directly                │
│           No explicit ownership — any function can touch anything   │
│                                                                     │
│  ~68 top-level functions · ~50 global variables                    │
│  18 comment-banner sections · No formal system boundaries          │
└─────────────────────────────────────────────────────────────────────┘
```

**Architecture Style:** Single-file procedural with scattered global state.

**What works:** The comment-banner sections create clear visual navigation (18 sections, well-named). The update order in `gameLoop()` is correct and well-documented. Constants at the top make tuning easy. The tuning panel's `eval()`-based approach requires globals to stay as bare `let` variables.

**What doesn't work for the game:** State is scattered across 7+ top-level objects with no defined ownership. Adding civilians, guards, game phases, and health means more globals, more implicit dependencies, and no way to serialize/restore state. Every new system would read and write globals directly, creating invisible coupling.

### Target Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                      index.html (~3,500-4,000 lines)                │
│                                                                     │
│  ┌────────────────────────────────────────────┐  ┌──────────────┐  │
│  │  TUNABLE CONSTANTS (let, top-level)         │  │ RUNTIME-ONLY │  │
│  │  40+ movement/drift/juice constants         │  │ input{}      │  │
│  │  20+ new game system constants              │  │ camera{}     │  │
│  │  (eval-accessible by tuning panel)          │  │ particles[]  │  │
│  └────────────────────────────────────────────┘  │ keysDown{}   │  │
│                                                   │ canvas, ctx  │  │
│  ┌────────────────────────────────────────────┐  └──────────────┘  │
│  │              GameState                      │                    │
│  │                                             │                    │
│  │  .phase      — TITLE | PLAYING | ...        │                    │
│  │  .zombie     — position, vel, hp, sanity    │                    │
│  │  .sanity     — { value, max, drainRate, ... }│                    │
│  │  .entities   — { civilians: [], guards: [] }│                    │
│  │  .level      — { tileMap, cols, rows, ... } │                    │
│  │  .stats      — { timeElapsed, eaten, ... }  │                    │
│  └──────────┬─────────────────────────────────┘                    │
│             │                                                       │
│             │  passed as parameter                                  │
│             ▼                                                       │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │                    SYSTEM UPDATE FUNCTIONS                    │  │
│  │                                                              │  │
│  │  updateGamePhase(state)    — phase transitions               │  │
│  │  updateSanityDrain(state, dt)  — continuous drain            │  │
│  │  updateHealth(state)       — HP checks → phase change        │  │
│  │  updateCivilians(state, dt)    — flee/seek/wander AI         │  │
│  │  updateGuards(state, dt)      — guard/chase/scream AI       │  │
│  │  updatePhysics(state, dt)      — zombie movement (existing)  │  │
│  │  updateDrift(state, dt)        — drift system (existing)     │  │
│  │  updateJuice(state, dt)        — particles/shake (existing)  │  │
│  │  checkEatCollision(state)      — zombie↔civilian overlap     │  │
│  │  checkGuardCollision(state)   — zombie↔guard overlap       │  │
│  │  broadcastScream(state, x, y)  — direct function call        │  │
│  └──────────────────────────────────────────────────────────────┘  │
│             │                                                       │
│             ▼                                                       │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │                    RENDER FUNCTIONS                           │  │
│  │  render(state)  — reads state, writes to canvas only         │  │
│  │  renderHUD(state) — sanity bar, HP display                   │  │
│  └──────────────────────────────────────────────────────────────┘  │
│             │                                                       │
│             ▼                                                       │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │                    GAME LOOP                                  │  │
│  │  gameLoop() calls systems in order, passes GameState          │  │
│  └──────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────┘
```

**Architecture Style:** Centralized `GameState` object + namespaced system functions. Same single file with explicit state ownership and system boundaries.

**Why this architecture:** The game needs serializable state (for save/load), clear phase management (5 game phases), and entity management (civilians + guards). A centralized GameState gives all three without introducing modules, classes, or build tools. System functions receive state as a parameter instead of reading globals, making data flow explicit. Constants stay as top-level `let` because the tuning panel's `eval()` depends on it. Runtime-only state (input, camera, particles) stays separate since it's never serialized and is re-created each frame or on reset.

---

## Core Systems List

| System | Responsibility | Location | Dependencies |
|--------|---------------|----------|-------------|
| **Game Phase Machine** | Manages state transitions: TITLE → PLAYING → LEVEL_COMPLETE / GAME_OVER_HP / GAME_OVER_SANITY. Gates which update systems run per phase. | `updateGamePhase(state)` | GameState.phase, GameState.zombie, GameState.sanity |
| **Zombie Controller** (existing) | Player input, movement, jumping, collision. The zombie's physical presence in the world. | `processInput()`, `updatePhysics(state, dt)`, `updateJump()`, `resolveHorizontal()`, `resolveVertical()` | GameState.zombie, input, constants |
| **Sanity System** | Continuous drain, tier helpers, restore on eat. The core tension mechanic. | `updateSanityDrain(state, dt)`, `getSanityT()`, `getMaxSpeed()`, etc. | GameState.sanity, constants |
| **Health System** | HP tracking, damage with invincibility frames, death check. | `damageZombie(state, amount)`, `updateHealth(state)` | GameState.zombie.hp, GameState.phase |
| **Civilian System** | Civilian entity management, flee/seek/wander AI, edge avoidance, eat collision. | `updateCivilians(state, dt)`, `checkEatCollision(state)`, `createCivilian(x, y)` | GameState.entities.civilians, GameState.zombie |
| **Guard System** | Guard entity management, guard patrol, chase, scream response, tile collision. | `updateGuards(state, dt)`, `checkGuardCollision(state)`, `createGuard(x, y)` | GameState.entities.guards, GameState.entities.civilians, GameState.zombie |
| **Death Scream** | Broadcasts scream event when civilian is eaten. Alerts guardswithin range. | `broadcastScream(state, x, y)` — direct function call, not event bus | GameState.entities.guards |
| **Level System** | Tile map, entrance/exit zones, level loading. Wraps existing generator. | `GameState.level`, `applyLevel()`, `generateLevel()` | Existing generator code |
| **Drift System** (existing) | Smooth impulses that push zombie off-course. Scales with sanity. | `updateDrift(state, dt)` | GameState.zombie, GameState.sanity, constants |
| **Juice Systems** (existing) | Squash/stretch, particles, screen shake, vignette, speed lines. | `updateJuice(dt)`, `emitDust()`, `triggerShake()`, render helpers | particles, camera (runtime-only) |
| **Input System** (existing) | Keyboard listeners, key state tracking, process input. | `processInput()`, keydown/keyup listeners | input, keysDown (runtime-only) |
| **Rendering** (existing) | Canvas drawing: tiles, zombie, particles, vignette, overlays, HUD. | `render(state)`, `renderHUD(state)` | GameState (read-only), canvas, ctx |
| **Collision Detection** (existing) | AABB vs tile grid for zombie. Shared utility for entity-entity. | `resolveHorizontal()`, `resolveVertical()`, `aabbOverlap(a, b)` | GameState.level.tileMap |
| **UI: Tuning Panel** (existing) | Live constant adjustment. Uses eval() to read/write top-level `let` constants. | `buildTuningPanel()` | Top-level constants (eval) |
| **UI: Debug Panel** (existing) | Real-time diagnostics overlay. | `updateDebugPanel()` | GameState (read-only) |

---

## Data Flow

### Player Input → Game Response (Existing — No Change)

```
1. Keyboard event fires (keydown/keyup) → keysDown[key] set
2. processInput() reads keysDown → sets input.left/right/jumpPressed/jumpHeld
3. updatePhysics(state, dt) reads input → applies acceleration/friction/gravity
4. resolveHorizontal() / resolveVertical() → tile collision
5. updateJump() → jump execution (coyote, buffer)
6. updateDrift(state, dt) → position nudge + velocity residue (sanity-based)
7. updateJuice(dt) → squash/stretch lerp, particle update, shake decay
8. render(state) → canvas draw with shake, zombie, particles, vignette
```

### Civilian Eat Flow (New)

```
1. Player moves zombie into civilian AABB overlap
2. checkEatCollision(state) detects overlap → civilian found
3. Civilian marked as dead: civilian.alive = false
4. Sanity restored: state.sanity.value += SANITY_PER_EAT (capped at max)
5. Stats updated: state.stats.civiliansEaten++
6. broadcastScream(state, civilian.x, civilian.y) called directly:
   a. Iterate state.entities.guards
   b. For each guard: distance check from scream position
   c. If within SCREAM_ALERT_RANGE:
      - Set guard.state = 'RESPONDING_TO_SCREAM'
      - Set guard.alertTarget = { x, y }
      - Set guard.alertTimer = SCREAM_ALERT_DURATION
7. Visual feedback: scream indicator, civilian disappears
8. Guard reassignment: guard whose civilian died → assign nearest remaining civilian
```

### Guard Damage Flow (New)

```
1. checkGuardCollision(state) runs AABB overlap: zombie vs each guard
2. If overlap AND zombie.invincibleTimer <= 0:
   a. zombie.hp -= GUARD_DAMAGE
   b. zombie.invincibleTimer = INVINCIBILITY_DURATION
   c. Visual feedback: flash, shake
3. updateHealth(state) checks zombie.hp:
   a. If zombie.hp <= 0: state.phase = 'GAME_OVER_HP'
   b. HP check runs BEFORE sanity check (edge case #3 from CORE_LOOP_SPEC)
```

### Game Phase Transition Flow (New)

```
updateGamePhase(state) runs each frame:

  TITLE:
    - Render title overlay
    - Wait for Enter key → state.phase = 'PLAYING', reset all game state

  PLAYING:
    - All update systems run (physics, sanity, civilians, guards, etc.)
    - After updates, check end conditions:
      1. zombie.hp <= 0 → state.phase = 'GAME_OVER_HP'     (checked first)
      2. state.sanity.value <= 0 → state.phase = 'GAME_OVER_SANITY'
      3. zombie overlaps exit zone → state.phase = 'LEVEL_COMPLETE'

  GAME_OVER_HP:
    - Render "GAME OVER" overlay
    - Wait for R key → state.phase = 'PLAYING', reset state

  GAME_OVER_SANITY:
    - Render "MIND LOST" overlay (absorbs existing Gone overlay)
    - Wait for R key → state.phase = 'PLAYING', reset state

  LEVEL_COMPLETE:
    - Render "LEVEL COMPLETE" overlay + stats
    - Wait for Enter key → load next level or restart

Note: game loop keeps running in ALL phases. Only update systems are gated.
Rendering continues for overlays, and input is checked for phase transitions.
```

### Save/Load Flow (Future — Strategy Only)

```
SAVE:
1. Player triggers save (level checkpoint or manual)
2. getSerializableState() extracts JSON-safe subset from GameState:
   - phase, zombie (position, hp, sanity), level number, stats
   - Excludes: input, camera, particles, keysDown, canvas, ctx
3. JSON.stringify() → localStorage.setItem('bfb_save', json)
4. Version field included for future migration

LOAD:
1. localStorage.getItem('bfb_save') → JSON.parse()
2. Version check → apply migrations if needed
3. Apply to GameState: zombie position, hp, sanity, level number
4. Reinitialize runtime-only state: input, camera, particles
5. Load correct level, resume play
```

---

## State Management Approach

### GameState Object — The Core Structural Change

Consolidate all scattered globals into one centralized object. This is the single most important architectural change — it makes state ownership explicit, enables save/load, and makes system boundaries enforceable.

```javascript
let GameState = {
    phase: 'TITLE',    // TITLE | PLAYING | LEVEL_COMPLETE | GAME_OVER_HP | GAME_OVER_SANITY

    zombie: {
        x: 0, y: 0,
        width: 32, height: 48,
        vx: 0, vy: 0,
        grounded: false,
        facingRight: true,
        hp: 5,
        maxHP: 5,
        invincibleTimer: 0,
        // existing fields preserved:
        coyoteTimer: 0,
        jumpBufferTimer: 0,
        jumpCut: false,
        prevVxSign: 0,
        scaleX: 1, scaleY: 1,
        landVy: 0
    },

    sanity: {
        value: 12,
        maxValue: 12,
        drainRate: 0.15,      // per second — configurable via tuning panel
        drainEnabled: true    // toggle for toy mode vs game mode
    },

    entities: {
        civilians: [],        // flat data objects from createCivilian()
        guards: []           // flat data objects from createGuard()
    },

    level: {
        tileMap: null,        // 2D boolean grid (existing)
        cols: 40,
        rows: 25,
        entrance: { x: 0, y: 0 },
        exit: { x: 0, y: 0, width: 32, height: 64 },
        seed: 0
    },

    stats: {
        timeElapsed: 0,
        civiliansEaten: 0,
        damagesTaken: 0
    }
};
```

**Runtime-only state (NOT in GameState — never serialized):**

```javascript
// These are re-created on init/reset, don't persist between sessions
let input = { left: false, right: false, jumpPressed: false, jumpHeld: false };
let drift = { timer: 0, impulse: 0, direction: 0, flashTimer: 0, ... };
let particles = [];
let camera = { shakeX: 0, shakeY: 0, shakeIntensity: 0, ... };
let keysDown = {};
let canvas, ctx;
```

**Why this split:**
- `GameState` contains everything needed to describe "what's happening in the game" — position, health, sanity, entities, level, stats. This is what save/load serializes.
- Runtime-only state is ephemeral: input state resets every frame, particles are cosmetic, camera shake decays, and canvas/ctx are DOM references. Saving these would be meaningless.

### Save/Load Litmus Test

```javascript
function getSerializableState() {
    return {
        version: 1,
        phase: GameState.phase,
        zombie: {
            x: GameState.zombie.x,
            y: GameState.zombie.y,
            hp: GameState.zombie.hp,
            maxHP: GameState.zombie.maxHP
        },
        sanity: {
            value: GameState.sanity.value,
            maxValue: GameState.sanity.maxValue
        },
        level: {
            seed: GameState.level.seed
            // tileMap regenerated from seed, not stored
        },
        stats: { ...GameState.stats }
    };
}
```

**Minimal save for v1:** Level reached + HP. Sanity resets per level. The level can be regenerated from its seed. Entities are re-placed on level load. This keeps saves tiny (< 1KB) and avoids serializing complex entity state.

### What Lives Where — Ownership Table

| Data | Owner | Modified By | Read By |
|------|-------|------------|---------|
| `GameState.phase` | Game Phase Machine | `updateGamePhase()` | All systems (gating), render |
| `GameState.zombie.*` | Zombie Controller | `updatePhysics()`, `updateJump()`, `updateDrift()`, `damageZombie()` | Civilian AI (flee range), guardAI (detection), render |
| `GameState.sanity.*` | Sanity System | `updateSanityDrain()`, `checkEatCollision()` | Movement helpers (`getSanityT()`), drift, render |
| `GameState.entities.civilians[]` | Civilian System | `updateCivilians()`, `checkEatCollision()` | Guard AI (guard assignment), render |
| `GameState.entities.guards[]` | Guard System | `updateGuards()`, `broadcastScream()` | Zombie collision, render |
| `GameState.level.*` | Level System | `applyLevel()`, `generateLevel()` | Collision detection, render, entity placement |
| `GameState.stats.*` | Stats Tracking | Various (on events) | Level Complete overlay, render |
| `input` | Input System | `processInput()`, keydown/keyup | Physics, jump |
| `drift` | Drift System | `updateDrift()` | Render (flash, vignette) |
| `particles[]` | Juice System | `emitDust()`, `updateJuice()` | `renderParticles()` |
| `camera` | Juice System | `triggerShake()`, `updateJuice()` | `render()` (shake offset) |

---

## Module Pattern Within Single File

### Convention: Comment-Banner Sections (Keep Existing)

The existing comment-banner convention is already clear and well-maintained (18 sections). Keep it. New sections to add:

| New Section | Position (after) | Contains |
|-------------|-----------------|----------|
| **GAME STATE** (expand existing) | Tile Map | GameState object definition, `createGameState()`, `resetGameState()` |
| **GAME PHASE MACHINE** | Game State | `updateGamePhase(state)`, phase transition logic |
| **SANITY DRAIN** | Sanity Tier Helpers | `updateSanityDrain(state, dt)` |
| **HEALTH** | Sanity Drain | `damageZombie(state, amount)`, `updateHealth(state)` |
| **CIVILIAN SYSTEM** | Health | `createCivilian(x, y)`, `updateCivilians(state, dt)` |
| **EAT + DEATH SCREAM** | Civilian System | `checkEatCollision(state)`, `broadcastScream(state, x, y)` |
| **GUARD SYSTEM** | Eat + Death Scream | `createGuard(x, y)`, `updateGuards(state, dt)`, `checkGuardCollision(state)` |
| **LEVEL ZONES** | Level Generator | `checkExitZone(state)`, entrance/exit rendering |

### Convention: State as Parameter

New system update functions take `state` as their first parameter:

```javascript
// NEW convention — explicit state dependency
function updateSanityDrain(state, dt) {
    if (!state.sanity.drainEnabled) return;
    state.sanity.value = Math.max(0, state.sanity.value - state.sanity.drainRate * dt);
}

function updateCivilians(state, dt) {
    for (const civ of state.entities.civilians) {
        if (!civ.alive) continue;
        // AI logic using state.zombie for flee distance, state.entities.guards for seek
    }
}
```

Existing functions (`updatePhysics`, `updateDrift`, `updateJump`, etc.) will be migrated to accept `state` in the final migration pass (step 11). During intermediate steps, they continue reading globals — the Strangler Fig approach allows this coexistence.

### Convention: Constants Stay as Top-Level `let`

The tuning panel uses `eval()` to read/write constants by name:

```javascript
// This pattern is locked — tuning panel depends on it
eval(name)           // read
eval(name + ' = ' + value)  // write
```

Constants MUST remain as top-level `let` variables. New game system constants (e.g., `SANITY_DRAIN_RATE`, `GUARD_SPEED`, `CIVILIAN_FLEE_SPEED`) follow the same convention and are added to the tuning panel.

### What We Are NOT Doing

- **No IIFE modules** — adds indirection with zero benefit in a single-file context
- **No ES6 classes** — Rule of Three not met (zombie, civilian, and guardhave very different field sets; a shared Entity base class would be premature abstraction)
- **No inheritance** — flat data objects with factory functions instead
- **No event bus** — only one event type exists (death scream). Direct function call is simpler and traceable. Upgrade to event bus IF/WHEN a second event type is needed (Rule of Three)
- **No module bundler/build step** — the single-file constraint is a feature for distribution and portability

---

## Entity System Design

### Flat Data Objects

Entities are plain JavaScript objects. No shared base class, no prototype chain, no methods. This is intentional — the zombie has 15+ unique fields (coyoteTimer, jumpCut, scaleX, etc.) that no other entity shares. A shared Entity base would be premature abstraction.

### Factory Functions

```javascript
function createCivilian(x, y) {
    return {
        x: x, y: y,
        width: 24, height: 24,
        vx: 0,
        alive: true,
        state: 'WANDER',   // WANDER | FLEE | SEEK
        guardedBy: null     // reference to guard guarding this civilian (or null)
    };
}

function createGuard(x, y) {
    return {
        x: x, y: y,
        width: 32, height: 32,
        vx: 0,
        state: 'GUARD_PATROL',  // GUARD_PATROL | CHASE | RESPONDING_TO_SCREAM
        guardedCivilian: null,  // reference to civilian being guarded
        patrolCenter: x,        // center of patrol range
        alertTimer: 0,
        alertTarget: { x: 0, y: 0 },
        facingRight: true
    };
}
```

**Why flat objects:**
- Simple to create, simple to serialize (JSON.stringify just works)
- No hidden state in prototype chains
- Easy to iterate (`for (const civ of state.entities.civilians)`)
- Easy to inspect in browser devtools
- Matches the existing `zombie` object pattern

### Shared Utility: Entity-Entity Collision

```javascript
function aabbOverlap(a, b) {
    return a.x < b.x + b.width &&
           a.x + a.width > b.x &&
           a.y < b.y + b.height &&
           a.y + a.height > b.y;
}
```

Used for: zombie↔civilian (eat), zombie↔guard (damage), zombie↔exit zone (level complete).

### Simplified Tile Collision for NPCs

Civilians and guardsneed basic tile collision but much simpler than the zombie's system:
- **Ground check ahead:** Is there a solid tile below the next step position? If not, reverse direction (don't walk off ledges).
- **Wall check ahead:** Is there a solid tile at head height in the movement direction? If yes, reverse direction.
- No gravity (entities stay on their platform — they don't jump or fall).
- No X-before-Y resolution (entities only move horizontally).

```javascript
function hasGroundAhead(entity, direction, tileMap, cols, rows) {
    const checkX = direction > 0 ? entity.x + entity.width : entity.x - 1;
    const col = Math.floor(checkX / TILE_SIZE);
    const footRow = Math.floor((entity.y + entity.height) / TILE_SIZE);
    return col >= 0 && col < cols && footRow < rows && isSolid(col, footRow);
}

function hasWallAhead(entity, direction, tileMap, cols) {
    const checkX = direction > 0 ? entity.x + entity.width : entity.x - 1;
    const col = Math.floor(checkX / TILE_SIZE);
    const headRow = Math.floor(entity.y / TILE_SIZE);
    return col >= 0 && col < cols && isSolid(col, headRow);
}
```

---

## Event System for Death Scream

### Direct Function Call — Not an Event Bus

Only one event type exists in v1: the death scream. Using an event bus for a single event adds indirection (subscribe, emit, listen) without benefit. A direct function call is simpler, traceable (grep for `broadcastScream`), and debuggable (set a breakpoint on one function).

```javascript
function broadcastScream(state, x, y) {
    for (const guard of state.entities.guards) {
        const dx = guard.x - x;
        const dy = guard.y - y;
        const dist = Math.sqrt(dx * dx + dy * dy);
        if (dist <= SCREAM_ALERT_RANGE) {
            guard.state = 'RESPONDING_TO_SCREAM';
            guard.alertTarget = { x: x, y: y };
            guard.alertTimer = SCREAM_ALERT_DURATION;
        }
    }
}
```

**Called from:** `checkEatCollision(state)` immediately after a civilian is consumed.

**Upgrade path:** If a second event type is needed (e.g., zombie stomp shockwave, environmental alarm), extract an event bus at that point. Rule of Three: don't abstract until you have three cases.

---

## Game Phase Machine

### String-Based State on GameState.phase

```javascript
GameState.phase = 'TITLE';  // one of 5 values
```

5 phases:
- **TITLE** — Title screen overlay. Game loop runs but no update systems. Wait for Enter.
- **PLAYING** — All update systems active. The main gameplay state.
- **LEVEL_COMPLETE** — Overlay with stats. Game loop runs (for rendering). Wait for Enter/R.
- **GAME_OVER_HP** — "GAME OVER" overlay. Zombie died from guarddamage. Wait for R.
- **GAME_OVER_SANITY** — "MIND LOST" overlay. Sanity hit 0. Absorbs existing Gone overlay. Wait for R.

### Phase Update Function

```javascript
function updateGamePhase(state) {
    switch (state.phase) {
        case 'TITLE':
            // Input: Enter → transition to PLAYING
            // No game systems update
            break;

        case 'PLAYING':
            // All systems run (called separately in game loop)
            // End-condition checks (in this order — HP before sanity per edge case #3):
            if (state.zombie.hp <= 0) { state.phase = 'GAME_OVER_HP'; break; }
            if (state.sanity.value <= 0) { state.phase = 'GAME_OVER_SANITY'; break; }
            if (checkExitZone(state)) { state.phase = 'LEVEL_COMPLETE'; break; }
            break;

        case 'LEVEL_COMPLETE':
            // Input: Enter → load next level, R → restart
            break;

        case 'GAME_OVER_HP':
            // Input: R → reset state, phase = 'PLAYING'
            break;

        case 'GAME_OVER_SANITY':
            // Input: R → reset state, phase = 'PLAYING'
            break;
    }
}
```

### Game Loop Integration

```javascript
function gameLoop(currentTime) {
    const dt = Math.min((currentTime - lastTime) / 1000, 0.05);
    lastTime = currentTime;

    if (GameState.phase === 'PLAYING') {
        processInput();
        updateSanityDrain(GameState, dt);
        updatePhysics(GameState, dt);
        // ... collision, jump, drift, juice ...
        updateCivilians(GameState, dt);
        updateGuards(GameState, dt);
        checkEatCollision(GameState);
        checkGuardCollision(GameState);
    }

    updateGamePhase(GameState);  // always runs — handles transitions + input in non-PLAYING phases
    render(GameState);           // always runs — draws overlays for non-PLAYING phases

    requestAnimationFrame(gameLoop);
}
```

**Key design decision:** The game loop keeps running in ALL phases. Only update systems are gated behind `phase === 'PLAYING'`. Rendering always runs (for overlays, backgrounds, and the frozen game state beneath overlays). This avoids the complexity of pausing/resuming the loop and keeps the render path unified.

---

## What NOT to Refactor

These systems are proven, tuned, and working. They get only the minimal changes needed to read from `GameState` instead of bare globals. No structural changes, no decomposition, no pattern application.

| System | Lines | Change | Rationale |
|--------|-------|--------|-----------|
| **Level Generator** | ~600 lines (358-955) | `tileMap` → `GameState.level.tileMap` reference | Generator is complex, proven, and self-contained. Touching it risks regression with zero benefit. |
| **Collision Detection** | ~80 lines (1257-1363) | `tileMap` → `GameState.level.tileMap` in `isSolid()` | Tile collision is correct and battle-tested. The X-before-Y resolution order is critical. |
| **Jump System** | ~90 lines (1363-1468) | `zombie` → `GameState.zombie` references | Variable jump height, coyote time, and jump buffer are tuned. One-cut fix is delicate. |
| **Input Handling** | ~50 lines (1107-1158) | No change | `keysDown` and `input` stay as runtime-only globals. No benefit to putting them in GameState. |
| **Tuning Panel** | ~130 lines (1957-2117) | Add new constant sliders | `eval()` pattern requires top-level `let`. Adding sliders for new constants follows existing pattern exactly. |
| **Juice Systems** | ~180 lines (1564-1758) | `zombie` → `GameState.zombie` references | Squash/stretch, particles, shake, vignette — all cosmetic, all working. |
| **Physics Update** | ~110 lines (1149-1257) | `zombie` → `GameState.zombie`, `sanity` → `GameState.sanity.value` | Core movement feel. Frame-rate independent. Proven through multiple playtest passes. |

**Rule:** If it works and isn't blocking a new system, leave it alone. Reference changes only.

---

## Migration Plan (Strangler Fig — 12 Atomic Steps)

Each step is atomic — the game works after every step. No "big bang" rewrite. The Strangler Fig approach wraps new patterns around existing code, then gradually replaces the old code with calls to the new patterns.

### Step 0: Create GameState Shell

**Create the GameState object that initially just references existing globals.**

```javascript
let GameState = {
    phase: 'PLAYING',        // start with PLAYING (no title screen yet)
    zombie: zombie,           // reference to existing zombie object
    sanity: {
        value: sanity,        // initially a copy; will be the source of truth later
        maxValue: 12,
        drainRate: 0.15,
        drainEnabled: false   // off by default so toy behavior is preserved
    },
    entities: {
        civilians: [],
        guards: []
    },
    level: {
        tileMap: tileMap,     // reference to existing global
        cols: MAP_COLS,
        rows: MAP_ROWS,
        entrance: { x: 64, y: 0 },
        exit: { x: 0, y: 0, width: 32, height: 64 },
        seed: 0
    },
    stats: {
        timeElapsed: 0,
        civiliansEaten: 0,
        damagesTaken: 0
    }
};
```

**Verification:** Game works exactly as before. GameState exists but nothing reads it yet.

### Step 1: Add Game Phase Machine

- Add `updateGamePhase(state)` with switch on `state.phase`
- Initially only two phases: `PLAYING` (existing behavior) and `GAME_OVER_SANITY` (absorb existing Gone overlay)
- Gate the existing Gone overlay behind `state.phase === 'GAME_OVER_SANITY'`
- Game loop calls `updateGamePhase(GameState)` after updates

**Verification:** Sanity 0 triggers "MIND LOST" via the new phase system. Press R to restart. Existing behavior preserved.

### Step 2: Add Sanity Drain System

- Add `updateSanityDrain(state, dt)` that decreases `state.sanity.value`
- Add "Auto Drain" toggle to UI (next to sanity slider)
- When drain is on, sanity ticks down; slider still works for manual override
- Sync: sanity slider reads/writes `state.sanity.value` instead of bare `sanity` global

**Verification:** Toggle drain on → sanity decreases over time. Toggle off → original slider-only behavior. Hitting 0 triggers GAME_OVER_SANITY phase.

### Step 3: Add Health System

- Add `zombie.hp`, `zombie.maxHP`, `zombie.invincibleTimer` to zombie object
- Add `damageZombie(state, amount)` function
- Add `updateHealth(state)` — decrements invincibleTimer, checks hp <= 0
- No damage sources yet — test via console: `damageZombie(GameState, 1)`

**Verification:** `damageZombie(GameState, 5)` in console → zombie.hp reaches 0. Health system ready for damage sources.

### Step 4: Add GAME_OVER_HP Phase

- Add 'GAME_OVER_HP' case to `updateGamePhase()`
- When `state.zombie.hp <= 0`, set `state.phase = 'GAME_OVER_HP'`
- Render "GAME OVER" overlay (distinct from "MIND LOST" — different text, different color)
- HP death check runs BEFORE sanity check (edge case #3 from CORE_LOOP_SPEC)
- Press R → reset HP, sanity, position, phase = 'PLAYING'

**Verification:** Console: `damageZombie(GameState, 5)` → "GAME OVER" overlay. Press R → restart. Sanity 0 still shows "MIND LOST" (different overlay).

### Step 5: Add Civilian Entities + AI

- Add `createCivilian(x, y)` factory function
- Add `updateCivilians(state, dt)` with AI: WANDER, FLEE, SEEK
- Add simplified tile collision for civilians (ground-check-ahead, wall-check-ahead)
- Add edge avoidance (reverse at platform edges)
- Render civilians as blue/cyan rectangles with "C" label
- Place 4-5 test civilians on the existing Gauntlet map
- Add civilian constants to tuning panel: `CIVILIAN_FLEE_SPEED`, `CIVILIAN_FLEE_RANGE`, `CIVILIAN_SEEK_RANGE`, `CIVILIAN_WANDER_SPEED`

**Verification:** Civilians wander on platforms. Approach a civilian → it flees. Civilians reverse at edges. No eat mechanic yet — civilians are just visual.

### Step 6: Add Eat + Death Scream

- Add `checkEatCollision(state)` — AABB overlap zombie ↔ civilians
- On eat: civilian.alive = false, state.sanity.value += SANITY_PER_EAT (capped), state.stats.civiliansEaten++
- Add `broadcastScream(state, x, y)` — iterates guards, distance-checks, sets alert state
- (Scream fires but no guardsexist yet to respond — that's fine)
- Add `SANITY_PER_EAT`, `SCREAM_ALERT_RANGE`, `SCREAM_ALERT_DURATION` to tuning panel

**Verification:** Walk into civilian → civilian disappears, sanity increases (visible on slider). Scream fires (log or visual indicator). Eating at sanity 12 still consumes civilian (waste case).

### Step 7: Add Guard Entities + Guard Patrol

- Add `createGuard(x, y)` factory function
- Add `updateGuards(state, dt)` with GUARD_PATROL state
- Guard patrol: guard patrols within `GUARD_WATCH_RANGE` of assigned civilian
- Guard assignment at level load: each guard→ nearest same-platform civilian
- Add simplified tile collision for guards(ground-check-ahead, wall-check-ahead)
- Render guardsas red rectangles with "!" label
- Place 3 test guardson the Gauntlet map
- Add guardconstants to tuning panel

**Verification:** Red rectangles patrol near blue civilian rectangles. Guards reverse at edges and walls. No chase or damage yet.

### Step 8: Add Guard Chase + Scream Response

- Add CHASE state: when zombie within `GUARD_DETECTION_RANGE` AND within `GUARD_LEASH_DISTANCE`, chase at `GUARD_CHASE_SPEED`
- Add RESPONDING_TO_SCREAM state: move to scream location, timer-based
- Chase breaks when zombie exceeds leash distance → return to guard
- Scream response overrides chase
- Guard reassignment: when guarded civilian is eaten, guardreassigns to nearest remaining civilian

**Verification:** Approach a guard→ it chases. Run away far enough → it returns to guard. Eat a civilian near a guard→ guardresponds to scream, moves to eat location. After alert timer → returns to guard.

### Step 9: Add Guard Damage Collision

- Add `checkGuardCollision(state)` — AABB overlap zombie ↔ guards
- On overlap + not invincible: call `damageZombie(state, GUARD_DAMAGE)`
- Invincibility frames: zombie flashes during `invincibleTimer > 0`
- Multiple overlapping guardsduring iframes → no additional damage

**Verification:** Touch a guard→ HP decreases, zombie flashes. Touch again during iframes → no damage. HP reaches 0 → "GAME OVER" overlay.

### Step 10: Add Level Entrance/Exit

- Add `TITLE` and `LEVEL_COMPLETE` phases to `updateGamePhase()`
- Render exit zone as gold/yellow rectangle with "EXIT" label
- Zombie spawns at entrance position
- Zombie overlaps exit → `state.phase = 'LEVEL_COMPLETE'`
- Level Complete overlay shows stats (time, civilians eaten, HP remaining)
- Add title screen overlay ("BRAINS FOR BREAKFAST" + "Press ENTER")

**Verification:** Game starts at title screen. Press Enter → game begins. Reach exit → "LEVEL COMPLETE" with stats. Press R → restart. All 5 phases work.

### Step 11: Final State Migration Pass

- Replace all remaining bare `zombie` references with `GameState.zombie`
- Replace `sanity` (scalar) reads with `GameState.sanity.value`
- Replace `tileMap` references in collision with `GameState.level.tileMap`
- Replace `MAP_COLS`/`MAP_ROWS` references with `GameState.level.cols`/`GameState.level.rows`
- Remove the old `zombie`, `sanity`, `tileMap` globals (they now live in GameState)
- Keep `input`, `drift`, `particles`, `camera`, `keysDown` as runtime-only globals
- Add `getSerializableState()` function
- Verify save/load litmus test: `JSON.parse(JSON.stringify(getSerializableState()))` produces valid state

**Verification:** Full playtest. All systems work. No bare globals remain except runtime-only state and constants. `getSerializableState()` returns a clean JSON object.

### Migration Step Summary

| Step | What | New Code | Risk |
|------|------|----------|------|
| 0 | GameState shell | ~30 lines | None — shell only |
| 1 | Phase machine (PLAYING + GAME_OVER_SANITY) | ~40 lines | Low — absorbs existing overlay |
| 2 | Sanity drain | ~20 lines | Low — additive |
| 3 | Health system | ~30 lines | Low — no damage sources |
| 4 | GAME_OVER_HP phase | ~20 lines | Low — extends step 1 |
| 5 | Civilian entities + AI | ~150 lines | Medium — new AI code |
| 6 | Eat + death scream | ~40 lines | Low — collision + function call |
| 7 | Guard entities + guard patrol | ~150 lines | Medium — new AI code |
| 8 | Chase + scream response | ~80 lines | Medium — AI state transitions |
| 9 | Guard damage collision | ~30 lines | Low — uses existing health system |
| 10 | Level entrance/exit + title | ~60 lines | Low — UI overlays |
| 11 | Final global migration | ~0 new (refactor) | Medium — many reference changes |

**Estimated total new code:** ~650 lines (bringing index.html to ~2,800 lines). With new constants and tuning panel additions, final file may reach ~3,200-3,500 lines.

---

## Save/Load Strategy

**Status:** Nice to Have per SCOPE_V1.md. Strategy defined here; implementation deferred.

### Minimal Save Format

```json
{
    "version": 1,
    "levelReached": 1,
    "zombie": {
        "hp": 4,
        "maxHP": 5
    }
}
```

**What's saved:** Level number + HP (carries across levels per design decision). That's it.

**What's NOT saved:** Sanity (resets each level), entity positions (re-placed on level load), mid-level state (levels are 2-5 minutes; losing progress is acceptable).

### Storage

`localStorage.setItem('bfb_save', JSON.stringify(saveData))`

### Version Migration

Version field enables future format changes:
- v1 → v2: add field with default value
- Unknown version: reject, start fresh

### When to Implement

Only if levels exceed 5 minutes each (per SCOPE_V1.md). With 3 short levels, session length is ~15-20 minutes — no save needed.

---

## Performance & Conventions

### Performance

**Not a concern.** The target entity count is ~8 (1 zombie + 5 civilians + 3 guards) on a 40x25 tile grid. Canvas 2D rendering at 60fps handles this trivially. No spatial partitioning needed. No object pooling needed. Linear iteration over entity arrays is fine.

**When it might become a concern:** If entity count exceeds ~50 (e.g., swarm mode), or if particle count exceeds ~200. Neither is planned for v1.

### Naming Conventions (Codify Existing)

| Category | Convention | Examples |
|----------|-----------|----------|
| Constants | `UPPER_SNAKE_CASE` | `BASE_MAX_SPEED`, `GUARD_CHASE_SPEED`, `SANITY_PER_EAT` |
| Functions | `camelCase`, verb-first | `updatePhysics()`, `createCivilian()`, `checkEatCollision()`, `broadcastScream()` |
| Variables | `camelCase` | `zombie`, `sanity`, `tileMap`, `alertTimer` |
| Entity states | `UPPER_SNAKE_CASE` strings | `'GUARD_PATROL'`, `'CHASE'`, `'RESPONDING_TO_SCREAM'`, `'WANDER'`, `'FLEE'` |
| Game phases | `UPPER_SNAKE_CASE` strings | `'TITLE'`, `'PLAYING'`, `'LEVEL_COMPLETE'`, `'GAME_OVER_HP'` |
| Factory functions | `create` + noun | `createCivilian()`, `createGuard()` |
| Update functions | `update` + system | `updatePhysics()`, `updateCivilians()`, `updateGuards()` |
| Check functions | `check` + what | `checkEatCollision()`, `checkGuardCollision()`, `checkExitZone()` |
| Boolean queries | `is`/`has` prefix | `isSolid()`, `hasGroundAhead()`, `hasWallAhead()` |

### Comment Convention

- Section banners: `// ============ SECTION NAME ============` (existing pattern)
- Function purpose: one-line comment before function if name isn't self-explanatory
- No JSDoc, no type annotations (prototype code — speed over ceremony)
- TODOs: `// TODO: description` (no name tag for solo dev)

---

## Revision History

| Date | Version | Change | Reason |
|------|---------|--------|--------|
| Feb 2026 | 1.0 | Initial refactor plan | Pre-Sprint 1 architecture evaluation. Defines GameState consolidation, entity system design, game phase machine, and 12-step migration plan for supporting core loop, civilian ecology, and guardsystems. |

---

*Produced using the gamedev-code-architecture skill (4-step process: Map → Choose Patterns → Define Boundaries → Implement+Refactor). Patterns applied: State (game phases), Observer (deferred — Rule of Three), Strangler Fig (migration). Anti-patterns avoided: God Object (centralize state, not behavior), Premature Abstraction (flat objects, not Entity base class), Architecture Astronautics (minimal patterns for prototype phase).*
