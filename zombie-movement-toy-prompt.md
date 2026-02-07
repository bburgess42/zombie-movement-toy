# Claude Code Prompt: Zombie Movement Toy

## Purpose

Build a minimal real-time movement prototype to test whether the zombie feels right to control. This is NOT a game — it's a toy. No enemies, no combat, no win conditions. Just a zombie that runs, jumps, and changes feel based on sanity level.

The goal is to answer one question: **Does this feel like the right foundation for a real-time zombie platformer?**

---

## Technical Requirements

- Single HTML file with embedded CSS and JavaScript
- HTML5 Canvas for rendering
- 60 FPS game loop using `requestAnimationFrame`
- Keyboard input (arrow keys or WASD for movement, space for jump)
- No external dependencies, no build tools

---

## The Zombie

A rectangle (for now — no sprites needed). Render it as a 32x48 pixel green rectangle with a small "Z" label.

### Core Movement Variables

All of these should be tunable constants at the top of the file:

```javascript
// Base values (Lucid tier)
const BASE_MAX_SPEED = 300;        // pixels per second
const BASE_ACCELERATION = 1800;    // pixels per second squared
const BASE_DECELERATION = 2400;    // friction when no input
const BASE_AIR_CONTROL = 0.8;      // multiplier on acceleration while airborne (1.0 = full, 0 = none)

// Jump
const JUMP_VELOCITY = -500;        // initial upward velocity (negative = up)
const GRAVITY = 1400;              // pixels per second squared
const COYOTE_TIME = 0.1;           // seconds after leaving platform where jump still allowed
const JUMP_BUFFER_TIME = 0.1;      // seconds before landing where jump input is remembered

// Sanity tier multipliers
const SLIPPING_SPEED_MULT = 1.25;
const SLIPPING_ACCEL_MULT = 1.3;
const SLIPPING_AIR_CONTROL_MULT = 0.85;

const FERAL_SPEED_MULT = 1.5;
const FERAL_ACCEL_MULT = 1.6;
const FERAL_AIR_CONTROL_MULT = 0.5;
```

### Movement Feel

The zombie should feel **weighty but responsive**. Not floaty, not sluggish. Key behaviors:

1. **Acceleration-based movement** — Pressing left/right accelerates toward max speed, not instant velocity. Releasing input decelerates via friction. This creates momentum.

2. **Air control** — While airborne, the player can still influence horizontal velocity, but with reduced effectiveness (controlled by `AIR_CONTROL` multiplier). This allows course correction mid-jump without feeling floaty.

3. **Coyote time** — If the zombie walks off a platform, they can still jump for a brief window (`COYOTE_TIME`). This is essential for responsive-feeling platformers.

4. **Jump buffering** — If the player presses jump slightly before landing, the jump executes on landing. Prevents frustrating "I pressed jump!" moments.

5. **Variable jump height** — If the player releases the jump button early, upward velocity is reduced (multiplied by 0.5). Holding jump = full height, tapping = short hop.

---

## Sanity System (Movement Effects Only)

Sanity is a float from 0 to 12. Tiers:

| Tier | Sanity Range | Movement Effects |
|------|-------------|------------------|
| **Lucid** | 7–12 | Base values. Crisp, precise control. |
| **Slipping** | 4–6.99 | Faster speed, snappier acceleration, slightly reduced air control. Occasional input drift (see below). |
| **Feral** | 0.01–3.99 | Fast speed, high acceleration, poor air control. Significant input drift. Momentum commits you to directions. |
| **Gone** | 0 | Zombie collapses. No movement. (Just stop responding to input and display "MIND LOST" on screen.) |

### Input Drift (Slipping and Feral only)

At lower sanity tiers, the zombie's movement becomes slightly erratic:

**Slipping:** Every 0.5–1.5 seconds (random interval), apply a small random horizontal impulse (±50 pixels/second). Subtle — the player should feel like the controls are slightly "off" without being unplayable.

**Feral:** Every 0.3–0.8 seconds, apply a larger random impulse (±150 pixels/second). Additionally, direction changes have a 0.05 second input delay (pressing right while moving left doesn't instantly start accelerating right — there's a brief unresponsive moment).

The drift should feel like the zombie's body is fighting the player's intentions. Not random teleportation — just a sense that you're not fully in control.

---

## The Test Level

Hardcode a simple test environment. No procedural generation. The level should let you test:

- Running on flat ground
- Jumping onto platforms of various heights (1 tile, 2 tiles, 3 tiles up)
- Jumping across gaps of various widths (2 tiles, 4 tiles, 6 tiles)
- Falling from height
- Navigating a vertical shaft (jumping up between alternating platforms)
- A long flat runway for testing top speed

Suggested layout (ASCII representation, 40 tiles wide × 15 tiles tall):

```
........................................
........................................
........................................
....====............................====
........................................
..====....====..........====............
................====....................
........................................
====........====........====....====....
........................................
....====................................
........................................
====....====....====....====....====....
........................................
========================================
```

Where `=` is solid platform and `.` is empty space. Tile size: 32×32 pixels. Total canvas: 1280×480 pixels (or scale to fit window).

Place the zombie at the bottom left on game start.

---

## UI Elements

### Sanity Slider

A horizontal slider at the top of the screen:
- Range: 0 to 12
- Default: 12 (Lucid)
- Draggable in real-time while playing
- Display current value and tier name: "Sanity: 8.5 (Lucid)"

The slider lets the designer instantly feel the difference between tiers without waiting for sanity to drain naturally. This is the primary testing tool.

### Debug Info Panel

Display in the corner (monospace font, semi-transparent background):

```
Sanity: 8.5 (Lucid)
Velocity: (245, 0)
Grounded: true
Coyote: 0.00
Jump Buffer: 0.00
Air Control: 0.80
Drift Timer: 0.32
```

Update every frame. This helps diagnose why movement feels wrong when tuning.

### Tuning Panel (Optional but Recommended)

A collapsible panel with sliders for all the movement constants:
- Base max speed (100–600)
- Base acceleration (500–3000)
- Base deceleration (500–3000)
- Air control (0–1)
- Jump velocity (−300 to −800)
- Gravity (800–2000)
- Coyote time (0–0.3)
- Jump buffer time (0–0.3)
- Tier multipliers

Changes apply instantly. This allows rapid iteration without editing code.

---

## What Success Looks Like

After building this, the designer should be able to:

1. Push the zombie around and say "yes, this feels like a zombie" or "no, this feels wrong" within 30 seconds
2. Slide sanity from Lucid to Feral and *feel* the difference — Lucid should feel safe and precise, Feral should feel fast but scary
3. Attempt a precision jump at Feral tier and experience the reduced air control making it harder
4. Notice the input drift at low sanity and feel slightly out of control without feeling cheated
5. Tune the constants until the movement feels right, then record those values for the full game

---

## Implementation Notes

### Game Loop Structure

```javascript
let lastTime = 0;

function gameLoop(currentTime) {
    const deltaTime = (currentTime - lastTime) / 1000; // convert to seconds
    lastTime = currentTime;
    
    // Cap delta time to prevent spiral of death on lag spikes
    const dt = Math.min(deltaTime, 0.05);
    
    processInput();
    updatePhysics(dt);
    updateDrift(dt);
    render();
    
    requestAnimationFrame(gameLoop);
}
```

### Physics Update

```javascript
function updatePhysics(dt) {
    // Apply gravity
    zombie.vy += GRAVITY * dt;
    
    // Apply horizontal acceleration based on input
    const accel = getAcceleration(); // accounts for sanity tier and air control
    if (inputLeft) zombie.vx -= accel * dt;
    if (inputRight) zombie.vx += accel * dt;
    
    // Apply friction if no input
    if (!inputLeft && !inputRight) {
        // Decelerate toward zero
        const friction = getDeceleration() * dt;
        if (zombie.vx > 0) zombie.vx = Math.max(0, zombie.vx - friction);
        else if (zombie.vx < 0) zombie.vx = Math.min(0, zombie.vx + friction);
    }
    
    // Clamp to max speed
    const maxSpeed = getMaxSpeed(); // accounts for sanity tier
    zombie.vx = Math.max(-maxSpeed, Math.min(maxSpeed, zombie.vx));
    
    // Update position
    zombie.x += zombie.vx * dt;
    zombie.y += zombie.vy * dt;
    
    // Collision detection and resolution
    resolveCollisions();
    
    // Update coyote time and jump buffer
    updateJumpTimers(dt);
}
```

### Collision Detection

Simple AABB collision against the tile grid. When the zombie collides with a platform:
- If falling and hitting the top of a platform, stop vertical velocity and place zombie on top
- If rising and hitting the bottom of a platform, stop upward velocity
- If moving horizontally into a wall, stop horizontal velocity and place zombie adjacent

The zombie should never clip through platforms. Err on the side of stopping movement rather than allowing phasing.

---

## Do NOT Build

- Enemies of any kind
- Combat or attacking
- The eating mechanic
- AI
- Civilians, hunters, or monsters
- Sanity drain over time (use the slider instead)
- Win or lose conditions
- Procedural level generation
- Multiple levels
- Sound
- Sprites or animations (rectangles are fine)
- Save/load
- Any game systems beyond movement and sanity-affected feel

If you find yourself building any of these, stop. The prototype is growing beyond its purpose. Movement feel is the ONLY question we're answering.

---

## After Building

Play with the toy for at least 15 minutes. Adjust the constants. Try to answer:

1. Does Lucid feel safe and precise?
2. Does Feral feel fast but dangerous?
3. Is the jump satisfying? Does it have the right arc?
4. Does the momentum feel right? Too slidey? Too sticky?
5. Is the input drift annoying or interestingly scary?
6. What values feel best? Write them down.

Then we build the next layer: a single civilian to chase and eat.
