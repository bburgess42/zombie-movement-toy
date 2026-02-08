# Brains for Breakfast: Complete Development Roadmap

From validated movement toy to shippable game. Each phase has specific prompts that leverage the gamedev skills framework.

---

## PHASE 0: PROJECT RECOVERY & SCOPE LOCK
**Goal:** Assess where you are, define what "done" means, cut everything that isn't essential.
**Skill:** gamedev-production

You have a working movement toy with physics, sanity tiers, and a level generator. Before building forward, lock scope.

### Step 0.1: Scope Audit
**Prompt:**
> "Read the zombie-movement-toy codebase and any existing design docs. Using the production skill, perform a mid-project reassessment. Inventory every feature and system that currently exists (movement, sanity, level generation, enemies, pickups, etc.) and categorize each as Must Have / Should Have / Nice to Have / Won't Have for a shippable v1. The essential experience is: 'A gentleman zombie navigating a decaying city, where losing your mind makes you more powerful but less controllable.' Apply the cutting framework — for each Nice to Have and Won't Have, answer the four questions. Produce a scope document with a clear feature list for v1."

### Step 0.2: Milestone Plan
**Prompt:**
> "Using the production skill, take the v1 scope document and break it into milestones. Each milestone should end with something playable and testable. Use relative sizing (S/M/L) not hour estimates. Define exit criteria for each milestone — what must be true before moving to the next. Include a pre-production exit gate: 'Two complete, playable levels with all core systems working.' Produce a milestone plan document."

### Step 0.3: Sprint Kickoff
**Prompt:**
> "Using the production skill, take Milestone 1 from the plan and break it into sprint-sized tasks (1-3 day chunks). For each task, note which skill domain it falls in (code, design, art, audio, etc.) and any dependencies. Produce a sprint backlog I can track in a markdown file."

---

## PHASE 1: LOCK THE TOY
**Goal:** Make the core movement loop undeniably fun before adding any game systems.
**Skills:** gamedev-systems-balance, gamedev-ui-ux, gamedev-level-design

The movement toy exists. Now make it feel perfect.

### Step 1.1: Movement Feel Pass
**Prompt:**
> "Read the zombie-movement-toy code, focusing on the physics and movement system. Using the ui-ux skill's feedback matrix framework, evaluate every player action (walk, run, jump, land, change direction, fall) and map what visual/audio/screen feedback currently exists for each. Identify feedback gaps — actions that feel 'dead' because nothing responds. For each gap, suggest a specific juice technique (screen shake, squash/stretch, particle burst, camera lerp, hit pause) with concrete parameter values. Produce a feedback matrix table."

### Step 1.2: Sanity-Movement Tuning
**Prompt:**
> "Using the systems-balance skill, analyze the sanity system's effect on movement. The design intent is: lower sanity = more powerful movement but less precise control. The risk/reward ratio should make players WANT to flirt with low sanity, not avoid it. Run a diagnostic: Is there a dominant strategy? Is there a sanity level that's always optimal? Model the tradeoff curve — what does the player gain vs. lose at each sanity tier? If the curve is broken, propose specific parameter changes. Use the Monte Carlo simulation template if helpful."

### Step 1.3: Level Generator Pacing Upgrade
**Prompt:**
> "Using the level-design skill, implement the pacing-aware section sequencer recommended in the previous evaluation. Replace random archetype selection with a deliberate sequence: hook → rising action → valley → climax → resolution. Add a per-section difficulty parameter that scales gap widths, platform sizes, and jump precision requirements. Add transition zones between sections. Test with 5 different seeds and evaluate each generated level against the interest curve template."

### Step 1.4: Toy Validation Playtest
**Prompt:**
> "Using the qa-testing skill, design a playtest session for the movement toy. The question we're answering is: 'Is the movement fun to play with even without game objectives?' Create a playtest plan with: 5 specific tasks for the tester (navigate to the end, try to reach the highest platform, play at minimum sanity, etc.), observation notes template, and a post-play questionnaire based on the shared playtest questionnaire template. Include the Lens of the Toy (#15): 'Is this fun to play with, separate from any game built on top of it?'"

---

## PHASE 2: CORE GAME LOOP
**Goal:** Add the minimum systems that turn the toy into a game — objectives, threats, rewards.
**Skills:** gamedev-design-doc, gamedev-systems-balance, gamedev-code-architecture

### Step 2.1: Core Loop Mechanic Spec
**Prompt:**
> "Using the design-doc skill, write a mechanic spec for the core game loop of Brains for Breakfast. The loop is: explore level → find civilians → eat civilians (restore sanity, trigger death scream) → avoid/evade threats → reach level exit. The gentleman zombie twist is that the player WANTS to maintain composure (high sanity = more control) but NEEDS low sanity for certain traversal challenges. Civilians flee the zombie and seek threats for protection, creating pursuit and risk. Spec out each element of the loop using the mechanic-spec template. Apply Lens #33 (Meaningful Choices) to every player decision point."

### Step 2.2: Civilian-Threat Ecology Design
**Prompt:**
> "Using the systems-balance skill, design the civilian-threat ecology system. Living civilians replace static brain pickups — the zombie must catch and eat civilians to restore sanity. Civilians flee the zombie and seek nearby threats for protection. Threats guard assigned civilians and patrol near them (leash behavior). When the zombie eats a civilian, a death scream alerts threats within 128px for 3 seconds. As civilians are eaten, threats reassign to remaining civilians, consolidating defense and creating natural difficulty escalation. Model the faction interaction matrix (zombie, civilians, threats). Use the economy-spec template to map sanity, HP, civilian population, and threat attention as interconnected resources. Identify emergent behaviors (eat order, approach angle, scream management). Run edge cases: what if all civilians are eaten? What if a civilian flees off a platform? NO starvation mechanic — sanity drains at a constant rate regardless of civilian population."

### Step 2.3: Civilian Variant Design (Stretch)
**Prompt:**
> "Using the design-doc skill, spec out civilian variants. Different civilian types grant different sanity restore amounts or temporary effects when eaten. For each variant, define: what it does mechanically, how it interacts with sanity, how long any temporary effect lasts, and what the player sacrifices by choosing to eat it (e.g., a guarded civilian is harder to reach but worth more sanity). Apply Lens #34 (Triangularity) — every variant should involve a risk/reward tradeoff, not just a bigger reward. Use the mechanic-spec template. Then hand off to systems-balance: are the variants balanced? Is any type dominant?"

### Step 2.4: Architecture Refactor
**Prompt:**
> "Using the code-architecture skill, evaluate the current codebase and plan a refactor to support the new game systems (civilian AI, threat AI, death scream events, sanity effects, level progression). Map the major systems and their communication needs. Recommend a component or module pattern that keeps things clean in a single-file HTML5 context. Specifically: design the state management so that save/load is possible (all game state serializable to JSON). Produce an architecture document with a migration plan — incremental extraction from the current code, not a rewrite."

---

## PHASE 3: THREAT AI & CIVILIAN ECOLOGY
**Goal:** Implement threat guard/chase/scream AI and tune the civilian-threat ecology.
**Skills:** gamedev-code-architecture, gamedev-systems-balance, gamedev-level-design

### Step 3.1: Threat AI Implementation
**Prompt:**
> "Using the code-architecture skill, implement the threat AI state machine. States: GUARD_PATROL (patrol within guard range of assigned civilian), CHASE (pursue zombie within leash distance of guarded civilian), RESPOND_TO_SCREAM (move to death scream location at chase speed). Threats should guard assigned civilians, break chase when zombie exceeds leash distance, and respond to death screams with higher priority than chase. When a guarded civilian is eaten, reassign to nearest remaining civilian. Produce the state diagram first, then implement. Reference the design-patterns reference for appropriate AI patterns."

### Step 3.2: Civilian AI Implementation
**Prompt:**
> "Using the code-architecture skill, implement the civilian AI state machine. States: WANDER (slow random patrol at 30 px/s), FLEE (run from zombie at 60 px/s when within 96px), SEEK (drift toward nearest threat within 128px for protection). Civilians should reverse at platform edges (never fall off). Civilians are eaten on zombie overlap (AABB collision), restoring sanity and triggering a death scream. Produce the state machine, implement, then use the qa-testing skill to write edge case tests (what happens when a civilian is cornered? What if multiple civilians cluster near one threat?)."

### Step 3.3: Ecology Balance Tuning
**Prompt:**
> "Using the systems-balance skill, civilians and threats are now in the game. Run a comprehensive balance pass. Test configurations: 5 civilians with 3 threats (default), 3 civilians with 2 threats (sparse), 7 civilians with 4 threats (dense). For each, observe: Does the player have meaningful eat-order choices? Does threat consolidation create natural difficulty escalation? Is the death scream range creating tension? Does the ecology self-regulate (later eats harder due to guard consolidation)? Produce a balance report with tuning recommendations for civilian count, threat count, guard range, leash distance, and scream range."

---

## PHASE 4: LEVEL DESIGN & CONTENT
**Goal:** Build real levels with authored pacing, not just generated noise.
**Skills:** gamedev-level-design, gamedev-narrative

### Step 4.1: Level Progression Design
**Prompt:**
> "Using the level-design skill, design the level progression for v1. How many levels? What does each level introduce (new mechanic, new threat behavior, new civilian placement pattern)? Map the difficulty curve across the entire game using the pacing curve template. Each level should teach something before testing it. Apply Lens #60 (Interest Curves) to the full game arc, not just individual levels. Produce a level progression document."

### Step 4.2: Environmental Narrative
**Prompt:**
> "Using the narrative skill, design the environmental storytelling for Brains for Breakfast. There may be minimal or no dialogue — the story is told through the city itself. What happened to cause the outbreak? Why is the player different? How does the player discover the answers through exploration? Use the environmental narrative framework — for each story beat, define: what the player sees, what it implies, what question it raises, where the answer is found. Create a Player Knowledge Map showing what the player learns and when."

### Step 4.3: Level Generator Enhancement
**Prompt:**
> "Using the level-design skill and code-architecture skill, enhance the procedural generator to support the level progression design. The generator should accept parameters from the level progression document: difficulty target, civilian count and placement zones (free vs. guarded), threat count and guard assignments, which environmental narrative elements to inject. Each generated level should feel like a designed experience, not random output. Implement, then generate 3 levels at different progression points and evaluate each against the interest curve template."

### Step 4.4: Authored Set Pieces
**Prompt:**
> "Using the level-design skill, identify 3-5 moments in the game that should be hand-authored rather than generated. These are the memorable moments: the first unguarded civilian (safe eat tutorial), the first guarded civilian with death scream consequence, the climax level with all threats consolidated around the last civilians. For each, sketch the layout, define the pacing beat by beat, place civilian and threat spawns deliberately, and mark the intended player emotional arc. These hand-crafted sections can be stitched into generated levels as 'event sections.'"

---

## PHASE 5: ART & VISUAL IDENTITY
**Goal:** Establish a consistent visual style and produce all necessary art assets.
**Skills:** gamedev-art-direction

### Step 5.1: Style Guide
**Prompt:**
> "Using the art-direction skill, establish the visual identity for Brains for Breakfast. Define 3-5 visual pillars. The tone is: dark comedy — a decaying city viewed through the eyes of a gentleman who happens to be undead. Think Grim Fandango meets Limbo. Constraints: solo dev, no hand-drawing skills, will use AI generation + procedural techniques + geometric styles. Define: color palette (base/accent/mood), resolution/grid standard, outline rules, perspective, animation frame counts. Produce a style guide document and a consistency checklist."

### Step 5.2: Character Art Pipeline
**Prompt:**
> "Using the art-direction skill, create the character art production pipeline. Characters needed: gentleman zombie (player, multiple sanity states), civilians (3-4 variants — different appearances for visual variety), threats (2-3 variants for different threat types in later milestones). For each character, create an AI image generation prompt using the ai-image-prompt template, including negative prompts for consistency. Define the post-processing pipeline (resize, palette reduction, outline addition, animation frame extraction). Produce a character asset list with status tracking."

### Step 5.3: Environment Art Pipeline
**Prompt:**
> "Using the art-direction skill, create the environment tileset. The city needs: ground tiles (sidewalk, road, rubble), building tiles (facades, interiors, rooftops), atmospheric elements (fog, rain, streetlights), and interactive objects (doors, barricades, civilian spawn points). Define the tileset grid, create AI generation prompts for each category, and establish the post-processing pipeline. Produce an environment asset list."

### Step 5.4: Visual Audit
**Prompt:**
> "Using the art-direction skill, perform a visual audit of all generated assets. Check every asset against the style guide: correct resolution? Correct palette? Consistent outline weight? Consistent perspective? Run the squint test — do characters read clearly against backgrounds at gameplay zoom? Run the gameplay readability test — can you instantly tell civilians from threats from the player? Document any inconsistencies and fix them."

---

## PHASE 6: AUDIO & MUSIC
**Goal:** Design and implement the sonic identity.
**Skills:** gamedev-audio

### Step 6.1: Sonic Identity
**Prompt:**
> "Using the audio skill, establish the sonic identity for Brains for Breakfast. Define 3 adjectives for how the game should sound. The gentleman zombie concept suggests a contrast: elegant/refined sounds mixed with grotesque/organic undertones. Classical music meets wet squelching. Define what sounds should be ABSENT — what silence means in this world. Produce a sonic identity document."

### Step 6.2: SFX Mapping
**Prompt:**
> "Using the audio skill, map every game event to an audio event. Walk through every system: movement (footsteps per surface, jump, land, wall-slide), eating (civilian catch, death scream, sanity restore chime), combat (hit, damage taken, death), sanity (level change transitions, low-sanity audio distortion, high-sanity clarity), civilians (fleeing footsteps, idle ambient), threats (patrol footsteps, chase growl, scream response alert), UI (menu select, pause, level complete). Categorize as Critical/Important/Polish. Produce an SFX list with the sfx-list template. Note which sounds can be procedural/parameterized vs. which need fixed assets."

### Step 6.3: Music Design
**Prompt:**
> "Using the audio skill, design the music system. Should music be adaptive (responding to game state) or fixed (per-level tracks)? If adaptive: what parameters drive the music (sanity level, faction proximity, pacing moment)? Design the layering system — what instruments/tracks fade in and out based on game state. The gentleman zombie concept suggests: at high sanity, refined chamber music; at low sanity, the same melody distorted and primal. Produce a music spec using the music-spec template."

### Step 6.4: Audio Implementation
**Prompt:**
> "Using the audio skill and code-architecture skill, implement the audio system. Map the audio event system to the game's existing architecture. Use Web Audio API for spatial audio and dynamic mixing. Implement: footstep surface detection, sanity-driven audio processing (reverb, distortion, pitch shift at low sanity), adaptive music layers, and SFX triggers for all Critical-priority events. Test with audio ON vs OFF to confirm audio adds meaningful information."

---

## PHASE 7: UI & GAME FEEL POLISH
**Goal:** Make every interaction feel responsive, clear, and satisfying.
**Skills:** gamedev-ui-ux

### Step 7.1: HUD Design
**Prompt:**
> "Using the ui-ux skill, design the HUD for Brains for Breakfast. Run the information architecture audit: what must be always visible (sanity meter, health), what's on-demand (inventory, map), what's contextual (interaction prompts, ability timer), what's never shown (exact enemy HP). The gentleman zombie theme means the HUD itself should reflect the character — perhaps an elegant, hand-drawn journal style that distorts as sanity drops. Produce an HUD spec using the hud-spec template."

### Step 7.2: Feedback & Juice Pass
**Prompt:**
> "Using the ui-ux skill, do a comprehensive juice pass on the entire game. For every player action and game event, verify the feedback matrix is complete. Add: screen shake on impact (0.1-0.3s), hit pause on civilian consumption (2-4 frames), death scream visual pulse, camera lerp on fast movement, vignette pulse on sanity change, squash/stretch on landing. Apply Lens #57 (Juiciness) — does every action feel satisfying? Apply Lens #56 (Feedback) — does the player always know what just happened?"

### Step 7.3: Menu Flow
**Prompt:**
> "Using the ui-ux skill, design the menu system. Map the complete flow: title screen → new game / continue / settings → gameplay → pause menu → level complete → game over. Design each screen's layout. For a browser-based game, keep it minimal — no complex options trees. Ensure save/load works from the pause menu. Produce a menu flow document using the menu-flow template."

---

## PHASE 8: TESTING & BALANCE
**Goal:** Find and fix everything broken before anyone else plays it.
**Skills:** gamedev-qa-testing, gamedev-systems-balance

### Step 8.1: Test Plan
**Prompt:**
> "Using the qa-testing skill, create a comprehensive test plan for Brains for Breakfast v1. Use the risk/testability matrix to prioritize. Identify: automated tests (physics edge cases, save/load integrity, civilian population math, threat guard assignment), manual tests (movement feel across all sanity levels, civilian flee AI, threat guard/chase/scream behavior, every level from generator), and playtest sessions (full game run-through, first-time player observation). For the procedural generator specifically: define property-based tests (invariants that must hold for ANY generated level — all platforms reachable, civilians and threats can spawn, no softlocks). Produce a test plan document."

### Step 8.2: Balance Sweep
**Prompt:**
> "Using the systems-balance skill, run a final balance pass across all interconnected systems. Check: sanity risk/reward curve (is low sanity tempting enough?), civilian eat-order decisions (is there a dominant strategy?), threat guard consolidation (does difficulty escalate naturally as civilians are eaten?), death scream tension (does eating feel risky near threats?), difficulty progression (does the game get harder at the right rate?), level generator output distribution (are generated levels consistently fair and fun?). For each system, state whether it passes or needs tuning, and provide specific parameter changes for anything that fails."

### Step 8.3: Playtest Sessions
**Prompt:**
> "Using the qa-testing skill, run 3 structured playtest sessions. Session 1: First-time player — observe without guidance, note where they get confused or stuck. Session 2: Experienced player — push the systems hard, try to break things, find dominant strategies. Session 3: Full playthrough — does the game hold up from start to finish? Is the pacing right? Does the ending land? Use the playtest session template for each. Compile findings into a single bug/improvement list prioritized by severity."

### Step 8.4: Bug Fix Sprint
**Prompt:**
> "Using the qa-testing skill and production skill, triage all bugs and issues from playtesting. Categorize: Critical (crashes, softlocks, game-breaking balance issues), Major (confusing UX, unclear feedback, frustrating difficulty spikes), Minor (visual glitches, audio pops, polish items). Fix all Critical and Major issues. Document all Minor issues for post-launch. Verify each fix with a regression test."

---

## PHASE 9: POLISH & SHIP
**Goal:** Final polish pass, build, and release.
**Skills:** gamedev-production, gamedev-art-direction, gamedev-ui-ux

### Step 9.1: Polish Checklist
**Prompt:**
> "Using the production skill and the shared design-review-checklist, run a final quality check on the complete game. Walk through Schell's Eight Filters: Does it feel right? Will the audience like it? Is it well-designed? Is it novel? Will it sell? Is it technically solid? Does it meet community goals? Do playtesters enjoy it? For any filter that doesn't pass, identify the specific issue and whether it's fixable in the time remaining or should be accepted as a known limitation."

### Step 9.2: Performance & Platform Testing
**Prompt:**
> "Using the qa-testing skill, test the game across target platforms. For an HTML5 browser game: test Chrome, Firefox, Safari, Edge. Test at multiple viewport sizes. Test on a low-end machine. Profile for performance bottlenecks — is the frame rate stable at 60fps? Does the procedural generator cause hitches? Does audio loading cause delays? Fix any platform-specific issues."

### Step 9.3: Store Page & Release Materials
**Prompt:**
> "The game is ready to ship. Write: a store page description (itch.io format), 5 screenshot captions highlighting key features, a one-paragraph press kit summary, and a launch post for social media. The tone should match the game: darkly comedic, sophisticated, slightly absurd. Emphasize the unique angle — you play AS the zombie, and you're a gentleman about it."

### Step 9.4: Build & Deploy
**Prompt:**
> "Using the production skill, prepare the final build. Checklist: all debug tools removed, console logging disabled, save system verified, all assets loading correctly, build size optimized, deployed to itch.io (or target platform). Create a v1.0 git tag. Write a brief post-mortem noting: what went well, what you'd do differently, and what's on the backlog for v1.1."

---

## POST-LAUNCH: V1.1 BACKLOG
**Prompt:**
> "Using the production skill, compile the v1.1 backlog from: Minor bugs deferred during the bug fix sprint, Nice to Have features from the original scope audit, new ideas that emerged during development, and player feedback from launch. Prioritize using MoSCoW. Plan the first post-launch sprint."

---

## Quick Reference: Phase → Primary Skills

| Phase | Primary Skill | Supporting Skills |
|-------|--------------|-------------------|
| 0. Scope Lock | production | design-doc |
| 1. Lock the Toy | ui-ux, systems-balance | level-design, qa-testing |
| 2. Core Loop | design-doc, systems-balance | code-architecture |
| 3. Threat AI & Ecology | code-architecture | systems-balance, qa-testing |
| 4. Levels & Content | level-design, narrative | code-architecture |
| 5. Art | art-direction | — |
| 6. Audio | audio | code-architecture |
| 7. UI Polish | ui-ux | — |
| 8. Testing | qa-testing, systems-balance | production |
| 9. Ship | production | qa-testing, art-direction |
