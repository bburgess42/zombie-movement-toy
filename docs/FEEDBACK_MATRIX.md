# Player Feedback Matrix: Zombie Movement Toy

**Last Updated:** Feb 2026
**Skill Applied:** gamedev-ui-ux (Feedback Matrix framework)
**References:** Swink's Game Feel (input → simulation → feedback), Vlambeer's Art of Screenshake, Jonasson & Purho's "Juice It or Lose It"
**Platform:** HTML5 Canvas — no haptic channel available. Feedback limited to Visual, Audio, Camera, Timing, UI.

---

## Current Feedback Audit

### Available Channels (HTML5 Canvas)

| Channel | Available? | Current Implementation |
|---------|-----------|----------------------|
| Visual (Sprite/Animation) | YES | Rectangle color changes by tier. "Z" label. Direction arrow. Drift white flash. |
| Visual (Screen Effect) | YES | Gone overlay ("MIND LOST"). Nothing else. |
| Audio (SFX) | NO | Zero audio in the prototype. |
| Audio (Music) | NO | Zero audio. |
| Haptic | NO | Browser — not available. |
| UI | YES | Sanity slider, debug panel, tuning panel. All designer-facing, not player-facing. |
| Camera | PARTIAL | No camera system. Canvas is fixed. Could add canvas-level offset for shake. |
| Timing | YES | Not implemented. requestAnimationFrame loop could support hit stop / slow-mo. |

### Diagnosis

The movement toy has **1 feedback layer** across all player actions: mechanical change (position/velocity update) is rendered to canvas. The only juice in the entire 1615-line codebase is the **drift flash** (white flash when drift impulse fires, `index.html:1338-1353`). Every other action — walk, jump, land, fall, change direction, reach top speed — produces zero feedback beyond the sprite moving.

Per Swink's model, the input → simulation chain is excellent (tuned through multiple playtests). The simulation → feedback chain is almost entirely missing. This is why the toy feels "correct" but not "alive."

Per the Feedback Intensity Guidelines: even Intensity Level 1 (Subtle) actions like footsteps should have some response. Currently everything is at Intensity 0 (invisible).

---

## Movement Actions — Current State vs. Recommended

### Walk / Run

| Channel | Current | Gap? | Recommended | Parameters |
|---------|---------|------|-------------|------------|
| Visual (Sprite) | Rectangle slides. Direction arrow at |vx| > 10. | YES | **Squash/stretch:** Compress zombie width by 8% and extend height by 8% while accelerating. Return to normal when at constant speed. Subtly communicates acceleration vs. cruising. | `squashX: 0.92, stretchY: 1.08` when `|accel| > 0`, lerp back at rate 8/s |
| Visual (Particles) | None | YES | **Dust puffs on direction change:** When zombie reverses horizontal direction (vx crosses zero while input held), emit 2-3 small dust particles at feet. Sells the "skidding" of momentum-based movement. | 2-3 particles, size 3-5px, color `#888`, lifetime 0.3s, velocity opposite to new direction, gravity 100px/s² |
| Visual (Particles) | None | YES | **Speed lines at high speed:** When |vx| > 80% of current max speed, draw 1-2 faint horizontal lines trailing behind the zombie. Communicates "going fast." | Lines at zombie.y ± random(5,15), length 15-30px, opacity 0.3, color matches tier |
| Audio (SFX) | None | YES | **Footstep sounds:** Soft thud on a timer while grounded and |vx| > 50. Interval shortens with speed. Grounded-only — no footsteps in air. | Interval: `0.3 - 0.15 * (|vx| / maxSpeed)` seconds. Volume 0.2-0.4. Pitch slight random ±5%. |
| Camera | None | MINOR | **Slight look-ahead:** Offset the canvas rendering by a few pixels in the direction of movement. The zombie's "center" shifts toward where they're heading. Gives the feeling of peering ahead. | Offset: `vx * 0.02` pixels, lerped at rate 5/s. Max offset ±15px. |
| **Total channels active** | **1** (sprite position) | | **Target: 3-4** | |

**Feedback intensity: Level 1-2 (Subtle to Light).** Walking is the most frequent action — keep feedback low-key. It should be felt subconsciously, not noticed consciously.

### Jump (Launch)

| Channel | Current | Gap? | Recommended | Parameters |
|---------|---------|------|-------------|------------|
| Visual (Sprite) | Zombie moves upward (vy set). No visual change on the launch frame. | **CRITICAL GAP** | **Squash on launch:** On the frame jump fires, compress zombie vertically by 20% and stretch horizontally by 15%. This is the single most impactful juice technique for platformer jumps (Celeste, Hollow Knight, Dead Cells all do this). Lerp back to normal over 0.1s. | `squashY: 0.80, stretchX: 1.15` on jump frame, lerp to 1.0 over 0.1s |
| Visual (Particles) | None | YES | **Dust burst at feet:** 3-5 particles at zombie's foot position on jump frame. Downward velocity. Communicates "pushed off the ground." | 3-5 particles, size 4-6px, color `#aaa`, lifetime 0.25s, vy: 50-100px/s down, spread: ±30px horizontal |
| Audio (SFX) | None | YES | **Jump sound:** Short, airy upward whoosh. Pitch shifts slightly with jump velocity (higher pitch at Feral = more explosive launch). | Duration 0.1s. Base pitch 400Hz sweep to 600Hz. Volume 0.3. At Feral: pitch +20%. |
| Camera | None | MINOR | **Tiny upward nudge:** Offset canvas 2-3px upward on jump frame, settle back over 0.15s. Subtly sells the launch force. | Offset: -3px Y, decay over 0.15s |
| **Total channels active** | **0** (only position change) | | **Target: 3-4** | |

**Feedback intensity: Level 3 (Light).** Jumping is frequent but each jump is a deliberate decision. Feedback should be satisfying but not distracting.

### Jump (Landing)

| Channel | Current | Gap? | Recommended | Parameters |
|---------|---------|------|-------------|------------|
| Visual (Sprite) | Zombie snaps to ground surface. No visual change. | **CRITICAL GAP** | **Stretch on land:** On the frame grounded becomes true (and wasn't true last frame), stretch zombie vertically by 15% and compress horizontally by 10%. Lerp back over 0.12s. Intensity scales with fall distance (higher vy at moment of landing = more stretch). | `stretchY: 1.0 + min(0.2, |vy_at_land| / 3000)`, `squashX: 1.0 - min(0.12, |vy_at_land| / 5000)`, lerp back over 0.12s |
| Visual (Particles) | None | YES | **Landing dust:** Bilateral dust puff at feet on landing. More particles for higher falls. | 2-6 particles (scales with vy_at_land), size 3-6px, color `#999`, spread ±20px horizontal, vy: -30 to -80px/s, lifetime 0.3s |
| Audio (SFX) | None | YES | **Landing thud:** Volume and pitch scale with fall distance. Short fall = quiet tap. Long fall = heavy thump. | Duration 0.05-0.1s. Volume: `0.15 + min(0.4, |vy_at_land| / 2000)`. Pitch: lower for harder landings. |
| Camera | None | YES | **Landing shake:** For falls greater than 4 tiles, add a small screen shake. Proportional to fall distance. Sells impact weight (Swink: simulation → feedback). | Trigger when `vy_at_land > 400`. Shake: `min(4, vy_at_land / 300)` px, duration 0.08s, decay exponential. |
| Timing | None | MINOR | **Landing pause (heavy falls only):** For falls greater than 6 tiles, freeze for 1-2 frames. The zombie's landing was so heavy it stopped time. Use sparingly. | Trigger when `vy_at_land > 600`. Freeze: 2 frames (33ms). |
| **Total channels active** | **0** | | **Target: 3-5** (scales with fall distance) | |

**Feedback intensity: Level 2-5 (Light to Medium), scaling with fall distance.** Short hops get subtle feedback. Long falls get dramatic feedback. This dynamic scaling is key — it makes the landing proportional to the fall.

### Change Direction (Reversal)

| Channel | Current | Gap? | Recommended | Parameters |
|---------|---------|------|-------------|------------|
| Visual (Sprite) | Direction arrow flips side. No other change. | MINOR | **Brief horizontal squash:** When vx crosses zero while decelerating, compress width by 10% for 0.05s. Sells the "braking" moment. | `squashX: 0.90` for 0.05s when `vx` sign flips, lerp back over 0.08s |
| Visual (Particles) | None | YES | **Skid dust** (same as walk reversal dust above). Only fires if |vx| was > 150 at the moment of reversal. No dust for gentle direction changes. | See Walk dust puffs above. Only trigger when speed at reversal > 50% max. |
| Audio (SFX) | None | MINOR | **Skid sound:** Only for high-speed reversals. Short scrape. | Trigger when |vx| > 200 at reversal. Duration 0.08s. Volume 0.2. |
| **Total channels active** | **1** (arrow flip) | | **Target: 2-3** | |

**Feedback intensity: Level 1-2 (Subtle).** Direction changes are very frequent. Only high-speed reversals get noticeable feedback.

### Fall (Airborne, Descending)

| Channel | Current | Gap? | Recommended | Parameters |
|---------|---------|------|-------------|------------|
| Visual (Sprite) | Zombie rectangle moves downward. Indistinguishable from any other state. | YES | **Vertical stretch while falling:** Stretch zombie vertically by up to 12% proportional to downward velocity. Communicates speed and creates contrast with the landing squash. | `stretchY: 1.0 + min(0.12, vy / 4000)` when `vy > 100 && !grounded` |
| Visual (Screen) | None | MINOR | **Subtle vignette at terminal velocity:** If falling for more than 0.5s, add a faint dark vignette. Communicates "this is a big fall." | Opacity: `min(0.15, (fallTime - 0.5) * 0.1)`. Only when `vy > 500` and `fallTime > 0.5s`. |
| Audio (SFX) | None | MINOR | **Wind rush for long falls:** Looping whoosh that fades in after 0.3s of falling, volume scales with vy. | Volume: `min(0.3, (vy - 200) / 2000)`. Fade in over 0.2s. Fade out on landing. |
| **Total channels active** | **0** | | **Target: 1-2** | |

**Feedback intensity: Level 1 (Subtle).** Falling is passive — the player isn't doing anything. Feedback should be ambient, not demanding attention.

---

## Sanity System — Current State vs. Recommended

### Sanity Tier Transition

| Channel | Current | Gap? | Recommended | Parameters |
|---------|---------|------|-------------|------------|
| Visual (Sprite) | Zombie color changes to tier color (green/yellow/orange/red). | Partial | Color change is good. **Add a brief flash** on the transition frame — the zombie pulses bright in the new tier color then settles. Makes the transition a noticeable event. | Flash: tier color at 100% opacity for 0.1s, decay over 0.2s |
| Visual (Screen) | None | **CRITICAL GAP** | **Screen vignette by sanity:** Permanent vignette that intensifies as sanity drops. At Lucid: invisible. At Slipping: faint yellow-tinted edges. At Feral: strong orange/red vignette. At Gone: full red overlay (exists as "MIND LOST"). This is the primary "losing your mind" feedback — the screen itself becomes less reliable. | Vignette radius: `1.0 - getSanityT() * 0.35` (1.0 = no vignette, 0.65 = significant). Color: interpolate tier color. Opacity: `getSanityT() * 0.25` |
| Visual (Screen) | None | YES | **Subtle screen distortion at Feral:** At Feral tier, add a very slight wobble to the canvas rendering — a 1-2px sinusoidal offset that makes the world feel "wrong." The player should feel subtly nauseous, not seasick. | Amplitude: `1 + getSanityT()` px, only when tier is Feral. Frequency: 2Hz horizontal, 1.5Hz vertical. Disable-able. |
| Audio (SFX) | None | YES | **Tier transition sting:** A brief audio cue when crossing a tier boundary. Lucid→Slipping: subtle dissonant tone. Slipping→Feral: louder, more distorted. Feral→Gone: final descending tone. | Duration: 0.5s. Volume: 0.3-0.5. Pitch: descends with tier. |
| Audio (Music) | None | YES (future) | **Music layer shift:** When music exists, cross-fade between calm and tense layers based on sanity. Not implementable until music exists (Milestone 3). | Crossfade duration: 2s. Trigger on tier boundary crossing. |
| UI | Sanity label color changes. Slider updates. | Partial | Current implementation is designer-facing. For a player-facing HUD, the sanity bar should **pulse** when crossing a tier boundary. | Scale bounce: 1.0 → 1.15 → 1.0 over 0.3s on tier crossing. |
| **Total channels active** | **2** (sprite color + UI label) | | **Target: 4-5** | |

**Feedback intensity: Level 5 (Medium).** Tier transitions are significant, infrequent events. The player should FEEL the shift. This is the Thrill of Danger form of fun — the world should change around you as sanity drops.

### Drift Impulse

| Channel | Current | Gap? | Recommended | Parameters |
|---------|---------|------|-------------|------------|
| Visual (Sprite) | White flash with tier-colored outline. 0.15s duration. | GOOD | Current implementation is solid. This is the only juice in the prototype and it works well. **No changes needed.** | Existing: `drift.flashTimer = 0.15`, white fill with tier outline |
| Visual (Screen) | None | MINOR | **Brief directional vignette on Feral drift:** When a Feral-tier drift fires, flash a brief vignette on the side the drift pushed TOWARD. Communicates direction of the interference. | One-sided vignette, drift direction side, opacity 0.2, duration 0.1s. Only at Feral tier. |
| Audio (SFX) | None | YES | **Drift sound:** Involuntary grunt/stumble sound. Slipping: subtle shuffle. Feral: louder, more distressed. Communicates "that wasn't me, that was the zombie's body." | Slipping: volume 0.15, short shuffle. Feral: volume 0.3, grunt + momentum sound. |
| Camera | None | YES | **Directional micro-shake on Feral drift:** When Feral drift fires, shake the canvas 2-3px in the drift direction. The world lurches with the zombie. | Shake: 3px toward drift direction, decay over 0.08s. Only Feral tier. |
| **Total channels active** | **1** (sprite flash) | | **Target: 2-3** | |

**Feedback intensity: Level 3 (Light) for Slipping, Level 5 (Medium) for Feral.** Drift is THE signature mechanic. Slipping drift should be noticed but not alarming. Feral drift should be scary — it should feel like the game itself hiccupped.

### Feral Input Delay (Direction Reversal Block)

| Channel | Current | Gap? | Recommended | Parameters |
|---------|---------|------|-------------|------------|
| Visual (Sprite) | None. The delay is invisible — the player presses a key and nothing happens for 0.03s. | **CRITICAL GAP** | **Zombie "resists" animation:** During the input delay, tint the zombie slightly red/darker and add a 1px jitter. Communicates "the zombie's body is fighting you." Without this, the delay feels like input lag, not a game mechanic. | Tint: darken by 20% during delay. Jitter: ±1px random offset per frame. |
| Audio (SFX) | None | YES | **Resistance grunt:** Very short, strained sound during the delay. Sells the "fighting your own body" fantasy. | Duration: 0.03s (matches delay). Volume 0.2. Low, guttural. |
| UI | None | MINOR | **Debug panel already shows "Input Delay" when active.** For player-facing: no UI needed. The sprite feedback is sufficient. | N/A |
| **Total channels active** | **0** | | **Target: 2** | |

**Feedback intensity: Level 2 (Light).** The delay is only 0.03s — feedback must be instant and brief. The key insight: without visual feedback, a 30ms input delay feels like a bug. With feedback, it feels like a mechanic.

---

## Game State Events — Current State vs. Recommended

### Gone State (Sanity = 0)

| Channel | Current | Gap? | Recommended | Parameters |
|---------|---------|------|-------------|------------|
| Visual (Screen) | "MIND LOST" overlay — red text with glow on dark background. | GOOD | Current implementation is dramatic and clear. **Add:** brief screen desaturation leading into the overlay (0.3s fade to gray, then overlay appears). Sells "the color is draining from the world." | Desaturation: 0% → 100% over 0.3s before overlay. |
| Audio | None | YES | **Mind Lost sound:** Descending dissonant chord that fades to silence. Should feel final, not jumpscary. | Duration: 1.5s. Descending pitch. Reverb tail. Volume 0.5. |
| Timing | None | MINOR | **Slow-mo into Gone:** When sanity hits 0, slow the game to 30% speed for 0.5s before freezing. The world winds down as the mind goes. | `timeScale` from 1.0 → 0.3 over 0.5s, then freeze + overlay. |
| **Total channels active** | **1** (overlay) | | **Target: 3** | |

---

## Feedback Gap Summary (Priority Ranked)

Actions ranked by how "dead" they currently feel vs. how important they are to the core experience.

| Priority | Action | Current Channels | Target | Gap | Impact if Fixed |
|----------|--------|-----------------|--------|-----|-----------------|
| **CRITICAL** | Jump (launch) | 0 | 3-4 | Squash, dust, sound | Jumping is the core verb. Zero feedback = zero satisfaction. Fixing this single action transforms the feel more than anything else. |
| **CRITICAL** | Jump (land) | 0 | 3-5 | Stretch, dust, thud, shake | Landing completes the jump arc. Without landing feedback, every jump feels like it ends with a thud that nobody heard. |
| **CRITICAL** | Sanity tier transition | 2 | 4-5 | Screen vignette, transition flash, sting | The sanity system is the core tension mechanic. If tier transitions don't FEEL different, the slider is just changing numbers. |
| **HIGH** | Feral input delay | 0 | 2 | Sprite resistance, grunt | A 30ms unresponsive window with zero feedback feels like a bug. With feedback, it's a terrifying mechanic. |
| **HIGH** | Feral drift impulse (upgrade) | 1 | 2-3 | Directional vignette, camera shake | Drift flash is good but one-channel. Adding camera shake makes drift events feel like the world lurched, not just the sprite. |
| **MEDIUM** | Walk/run at speed | 1 | 3-4 | Speed lines, footsteps, look-ahead | Movement is constant. Subtle feedback creates a baseline "alive" feeling across the entire session. |
| **MEDIUM** | Direction reversal | 1 | 2-3 | Skid dust, squash, skid sound | High-speed reversals should feel satisfying (momentum fighting inertia). Currently invisible. |
| **LOW** | Fall (descending) | 0 | 1-2 | Vertical stretch, wind rush | Falling is passive. Ambient feedback is nice but not essential. |
| **LOW** | Walk dust | 0 | 1 | Dust puffs | Very subtle. Nice atmospheric touch but low priority. |

---

## Implementation Approach

### Phase A: Critical Gaps (do first — biggest feel impact)

1. **Squash/stretch system** — Add `zombie.scaleX` and `zombie.scaleY` to state. Render using `ctx.save()`, `ctx.translate()`, `ctx.scale()`, `ctx.restore()`. Lerp scales toward 1.0 each frame. Set scales on jump launch, landing, and direction reversal.

2. **Particle system** — Lightweight array of `{ x, y, vx, vy, life, maxLife, size, color }`. Update all particles each frame (position += velocity * dt, life -= dt). Render as small filled circles. Remove when life <= 0. Emit on jump, land, and reversal.

3. **Screen shake system** — Add `camera.shakeX`, `camera.shakeY`, `camera.shakeDuration`, `camera.shakeIntensity` to state. On shake trigger: set intensity and duration. Each frame: if shaking, offset all rendering by random ±intensity pixels (decaying). Apply via `ctx.translate()` before rendering, undo after.

4. **Sanity vignette** — Draw a radial gradient overlay after all game rendering. Center: transparent. Edges: tier-colored at `getSanityT() * 0.25` opacity. Cheap, always-on, scales with sanity.

### Phase B: High Priority (do second)

5. **Feral input delay visual** — During `drift.inputDelayTimer > 0`, darken zombie fill by 20% and add ±1px random offset to zombie render position.

6. **Feral drift camera shake** — When drift fires at Feral tier, trigger a directional screen shake (3px toward drift direction).

### Phase C: Medium Priority (do if time)

7. **Speed lines** — When |vx| > 80% max speed, draw 1-2 faint horizontal lines behind the zombie.

8. **Landing intensity scaling** — Track `vy` at moment of landing. Scale landing stretch, dust count, and shake intensity proportionally.

### Phase D: Audio (deferred to Milestone 3)

9. All audio feedback is deferred until audio system exists. Document the timing and emotional quality for each sound event for handoff to gamedev-audio.

---

## Feedback Budget (Avoiding Overload)

Per the anti-pattern "Feedback Overload": when everything is emphasized, nothing is. The hierarchy for this game:

| Intensity | Events |
|-----------|--------|
| **7-9 (Dramatic)** | Gone state, player death, tier transition to Feral |
| **5 (Medium)** | Feral drift impulse, heavy landing (>4 tiles), tier transition (any) |
| **3 (Light)** | Jump launch, normal landing, brain pickup, take damage |
| **1-2 (Subtle)** | Walk footsteps, direction reversal, speed lines, fall stretch |

Never let Level 1-2 events compete with Level 5+ events. If a Feral drift fires during a landing, the drift shake takes priority over the landing shake (use the larger, don't stack).

---

*Produced using the gamedev-ui-ux skill (Feedback Matrix framework). References: Swink's input → simulation → feedback model, Vlambeer's layering order, Jonasson & Purho's juice principles. Lenses applied: #56 (Feedback — does every action give appropriate response?), #57 (Juiciness — does feedback feel satisfying and alive?), #55 (Transparency — does the interface disappear during play?).*
