# Feature Specification: Alice in Wonderland Syndrome — Toy Story Room

**Feature Branch**: `001-aiws-toystory-room`

**Created**: 2026-05-30

**Status**: Draft

**Input**: User description: トイストーリーの部屋を舞台にした、不思議のアリス症候群（巨人症/小人症）を体験するブラウザWebGLデモ。おもちゃとゴミに命が宿り、人間が入ってくると全員死んだふりでフリーズする。

## Overview

A browser-based, full-screen experiential demo that places the viewer inside a Toy Story–style child's bedroom and simulates **Alice in Wonderland Syndrome (AIWS)** — a perceptual disorder in which one's own body and surroundings feel abnormally large (macropsia / 巨人症) or small (micropsia / 小人症). The viewer's sense of scale drifts continuously between giant and tiny, with the room visually warping to match. The room is alive: toys and even discarded trash move on their own — until a human's presence is sensed, at which point everything instantly freezes into lifeless objects, then resumes once the coast is clear.

The piece is atmospheric and self-playing in its core arc (no game objective), but responds to pointer movement so the viewer can stir the world; its value is the felt experience and the tension of the "freeze" moment.

## Clarifications

### Session 2026-05-30

- Q: Is the experience interactive or purely self-playing? → A: Mouse-linked + auto-progressing — the perceptual cycle and freeze events run automatically, while moving the pointer/touch creates fluid-distortion ripples at that location (preserving the three-fluid-fx feel).
- Q: How is the "a human is coming" presence expressed? → A: All three cues together — a large shadow sweeping across the room, light spilling from an opening door, and footstep/door audio.
- Q: During the freeze ("play dead"), what happens to the perceptual (giant/tiny) warp? → A: The warp also stills and returns to normal — everything holds its breath as an ordinary, undistorted room; tension peaks in the stillness.
- Q: Motion-intensity / nausea-mitigation option? → A: Deferred — not a concern for now; revisit later.
- Q: Should on-screen title/branding text be shown? → A: No. Do not display title/branding strings (e.g. "不思議のアリス症候群", "Toy Story Room"). Keep the arrow-key control guide (bottom-right) and the functional perception-scale indicator. The room itself, not text, carries the piece.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Experiencing the perceptual scale shift (Priority: P1)

A visitor opens the page and is placed in a first-person view inside the bedroom. Without any input, their perceived scale slowly drifts: at times they feel enormous (the room shrinks, walls bend inward, ceiling feels low), at times tiny (the room yawns vast and tall around them), passing through a "normal" baseline between extremes. An on-screen indicator communicates the current perceptual state.

**Why this priority**: This is the core sensation the whole piece exists to deliver. Without it there is no experience. It is also the foundation already prototyped.

**Independent Test**: Open the page and watch for at least one full cycle (giant → normal → tiny → normal). The visual warp, the change in apparent room size, and the state indicator must all be observable without any user input.

**Acceptance Scenarios**:

1. **Given** the page has loaded, **When** the viewer does nothing, **Then** the perceived scale automatically transitions through giant, normal, and tiny states on a continuous loop.
2. **Given** the macropsia (giant) state is active, **When** it peaks, **Then** the room appears smaller/closer and the image distorts with a fisheye/barrel character.
3. **Given** the micropsia (tiny) state is active, **When** it peaks, **Then** the room appears larger/taller and the image distorts with a pincushion character.
4. **Given** any state, **When** it is active, **Then** an on-screen indicator shows the current perceptual scale (e.g. 巨人症 / normal / 小人症).

---

### User Story 2 - The room is alive (toys and trash) (Priority: P2)

The viewer notices that the room's inhabitants are not static. Toys — building blocks, a ball, character figures — sway, shuffle, and move as if quietly alive. Beyond the obvious toys, discarded trash on the floor (paper scraps, a can, candy wrappers) is *also* alive, twitching and hopping, reinforcing the dream-logic that in this world even garbage has a soul.

**Why this priority**: This establishes the Toy Story world and the surreal mood that makes the freeze moment (Story 3) meaningful. It is additive to the core sensation rather than required for it.

**Independent Test**: Observe the scene at rest (no human present, normal perceptual state) and confirm that multiple toys AND at least one piece of trash exhibit independent, ongoing motion.

**Acceptance Scenarios**:

1. **Given** no human presence is active, **When** the scene is at rest, **Then** multiple toys exhibit independent idle motion (breathing sway, small hops, walking, etc.).
2. **Given** no human presence is active, **When** the scene is at rest, **Then** at least one trash/junk object also exhibits independent motion, visibly distinct from inert scenery.

---

### User Story 3 - "Play dead" when a human approaches (Priority: P2)

Periodically, the presence of a human is sensed — telegraphed by sensory cues (footsteps, a door sound, a large shadow sweeping across the room, a shift in light). The instant this happens, every living thing — toys and trash alike — snaps to a frozen, lifeless pose, indistinguishable from ordinary objects, exactly as in Toy Story. The viewer holds in this tense stillness until the human presence passes, after which the room comes back to life.

**Why this priority**: This is the signature dramatic beat and the most recognizable Toy Story rule. It depends on Story 2 (things must be alive to convincingly play dead), so it shares P2 but is sequenced after it.

**Independent Test**: Trigger or wait for a human-presence event and confirm that all moving entities freeze near-instantly, remain frozen for the duration of the presence, and resume motion afterward.

**Acceptance Scenarios**:

1. **Given** toys and trash are moving, **When** a human-presence event begins, **Then** all moving entities freeze within a fraction of a second into static poses.
2. **Given** a human-presence event is active, **When** the viewer observes the scene, **Then** there are clear sensory cues of the human (e.g. shadow/light change and/or sound) and nothing in the room moves.
3. **Given** a human-presence event ends, **When** the presence cue subsides, **Then** the toys and trash resume their independent motion.

---

### User Story 4 - Walk through the room in first person (Priority: P3)

The viewer can move through the bedroom in first person using the arrow keys, with a gentle walking feel (head bob / footstep cadence), while the perceptual scale shift and freeze events continue around them. Cursor/touch fluid ripples remain available at the same time, so the viewer can both *walk* (arrow keys) and *stir* (pointer) simultaneously. A viewer who presses nothing still gets the full self-playing experience.

**Why this priority**: This turns the piece from a thing-you-watch into a space-you-inhabit, deepening the AIWS embodiment — but the core sensation (US1) and the Toy Story world (US2/US3) already deliver value without it, so it is P3.

**Independent Test**: Press the arrow keys and confirm the camera walks forward/back and turns/strafes through the room with a walking feel, that the perceptual warp and freeze still operate during movement, and that moving the cursor still produces ripples at the same time.

**Acceptance Scenarios**:

1. **Given** the experience is running, **When** the viewer holds an arrow key, **Then** the first-person camera moves accordingly (forward/back and turn or strafe) with a subtle walking motion.
2. **Given** the viewer is walking, **When** they also move the cursor/touch, **Then** fluid ripples are produced at the pointer at the same time as the movement.
3. **Given** the viewer is walking, **When** the perceptual cycle reaches a giant or tiny state or a human-presence freeze begins, **Then** those effects still apply correctly relative to the viewer's current position.
4. **Given** the viewer presses no keys, **When** they simply watch, **Then** the full self-playing experience (US1–US3) proceeds unchanged.

---

### Edge Cases

- **Reduced motion preference**: For viewers who have requested reduced motion, intense full-screen warping can cause discomfort or nausea; the experience should offer a gentler mode or honor the system preference.
- **Small / portrait screens**: On a phone the room and indicator must remain legible and the warp must not push critical UI off-screen.
- **Low-end GPU**: If the device cannot sustain a smooth frame rate, the experience should degrade gracefully (reduced effect intensity or simpler scene) rather than stutter badly.
- **Overlap of states**: A human-presence freeze can begin in the middle of a giant or tiny perceptual state; per FR-009a the warp eases back to normal as the entities freeze, so the freeze always presents an ordinary undistorted room regardless of which perceptual state it interrupted.
- **Tab backgrounded**: When the page is not visible, animation should pause/throttle so it does not waste resources or jump on return.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: The experience MUST present a first-person view positioned inside a Toy Story–style child's bedroom (recognizable cues such as cloud wallpaper, wooden floor, scattered toys).
- **FR-002**: The experience MUST automatically and continuously cycle the viewer's perceived scale through at least three states: macropsia (giant), a normal baseline, and micropsia (tiny), without requiring user input.
- **FR-002a**: The experience MUST respond to pointer movement (mouse or touch) by generating fluid-distortion ripples at the pointer location, layered on top of the automatic perceptual cycle, so the viewer can stir the world without controlling its arc.
- **FR-003**: The macropsia state MUST visually convey "the viewer is huge / the room is small" and the micropsia state MUST visually convey "the viewer is tiny / the room is vast," using distinct and opposite visual distortion characters.
- **FR-004**: Transitions between perceptual states MUST be smooth (eased), not abrupt, to read as a drifting sensation rather than a flicker.
- **FR-005**: The experience MUST display an on-screen indicator of the current perceptual state that updates as the state changes.
- **FR-006**: Multiple toy entities MUST exhibit independent, ongoing idle motion while no human presence is active.
- **FR-007**: At least one trash/junk entity MUST also exhibit independent motion, visually establishing that trash is "alive" too.
- **FR-008**: The experience MUST periodically generate a "human presence" event, communicated through all of the following cues together: a large shadow sweeping across the room, light spilling from an opening door, and footstep/door audio.
- **FR-008a**: Because the human-presence audio may be blocked by browser autoplay policies, the visual cues (sweeping shadow and door light) MUST be sufficient on their own to convey the event; audio is an enhancement and may require a one-time user gesture to enable.
- **FR-009**: When a human-presence event begins, all living entities (toys and trash) MUST freeze into static, lifeless poses near-instantly.
- **FR-009a**: When a human-presence event begins, the perceptual (giant/tiny) warp MUST also still and return to a normal, undistorted view, so the room reads as an ordinary, lifeless room during the freeze.
- **FR-010**: While a human-presence event is active, no living entity may move and the perceptual warp MUST remain at normal.
- **FR-011**: When a human-presence event ends, living entities MUST resume their independent motion and the automatic perceptual cycle MUST resume.
- **FR-012**: The experience MUST run in a current desktop or mobile web browser by opening a single page, with no installation or build step required of the viewer.
- **FR-013**: The experience MUST adapt to the browser viewport on load and on resize, remaining usable in both landscape and portrait orientations.
- **FR-014**: The experience MUST be publicly deployable as a static site (suitable for Vercel static hosting) served from the repository root.
- **FR-015**: (Deferred) Motion-intensity / nausea-mitigation handling (e.g. honoring reduced-motion preference, offering an intensity toggle) is out of scope for now and may be revisited later.
- **FR-016**: The experience MUST support first-person locomotion via the arrow keys (move forward/back and turn or strafe) with a subtle walking feel (e.g. head bob), while the perceptual cycle and freeze events continue to operate.
- **FR-017**: Arrow-key locomotion (FR-016) and pointer/touch fluid ripples (FR-002a) MUST work simultaneously, and the experience MUST remain fully self-playing for a viewer who provides no input.

### Key Entities

- **Perceptual State**: The viewer's current sense of scale along a continuum from tiny (micropsia) through normal to giant (macropsia); drives the room's visual distortion, apparent size, and the on-screen indicator.
- **Toy**: An animate inhabitant of the room (block, ball, character figure) with an idle motion behavior and a frozen "play dead" pose.
- **Trash**: An animate piece of discarded junk (paper scrap, can, wrapper) with its own idle motion and frozen pose; conceptually distinct from Toy to make "even trash is alive" legible.
- **Human Presence Event**: A timed intrusion representing an approaching human; carries sensory cues (shadow/light, sound) and commands all animate entities to freeze for its duration.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: Within 30 seconds of opening the page (and with no input), a first-time viewer observes at least one complete perceptual cycle spanning both the giant and tiny extremes.
- **SC-002**: A viewer can correctly identify, at any moment, whether they are currently in a giant, normal, or tiny perceptual state by reading the on-screen indicator.
- **SC-003**: At rest, a viewer can point to at least three independently moving entities, at least one of which is trash rather than a toy.
- **SC-004**: When a human-presence event occurs, every visible moving entity becomes still within roughly half a second, and a viewer can perceive the human cue (shadow/light and/or sound) that triggered it.
- **SC-005**: After a human-presence event ends, the room visibly returns to life within a couple of seconds.
- **SC-006**: The experience loads and begins animating within 5 seconds on a typical broadband connection and sustains smooth visible motion on a current mid-range laptop and a current mid-range phone.
- **SC-007**: The experience is reachable at a public URL and renders correctly without the viewer installing or configuring anything.

## Assumptions

- The piece is **self-playing in its arc but interactive**: the perceptual cycle and freeze events run on their own, while the viewer can simultaneously stir fluid ripples with the pointer/touch (FR-002a) and walk through the room in first person with the arrow keys (FR-016/FR-017). The viewer does not control the narrative arc, and pressing nothing still yields the full experience.
- The first delivery target is **running locally** (open the page in a browser on the developer's machine); public Vercel deployment comes only after the local version works and is approved.
- "Toy Story room" refers to the *aesthetic and rules* (Andy's-bedroom look, the play-dead convention) as homage; no licensed characters, exact likenesses, or copyrighted assets are used. Figures are original, generic homages (a cowboy-ish figure, a spaceman-ish figure).
- Audio cues for the human-presence event are desirable but optional; if browser autoplay restrictions block sound, visual cues (shadow/light) alone satisfy the requirement.
- Target viewers are general audiences viewing a hackathon/art demo on a personal device; no accounts, persistence, or multi-user concerns apply.
- The existing prototype (room + AIWS distortion in a single page) is the starting point and will be extended, not replaced.
- Deployment targets static hosting on Vercel with the single page at the repository root; no server-side logic is required.

## Future Enhancements (Out of scope for this iteration)

These are explicitly **not** part of the current spec/tasks but are recorded for a later iteration:

- **FE-001 — Footstep / movement SFX with random variation**: The arrow-key walking (now in scope as US4 / FR-016) plays footstep sound effects. There are several SFX variants and the played sound is chosen at random each step, so footsteps don't sound mechanically repetitive. (Subject to the same browser-autoplay/user-gesture constraints as other audio.) Deferred only because the silent walking experience is shippable first; the random-SFX layer is the next addition.
