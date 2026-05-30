# Feature Specification: 出鱈目ジェンガ＆クラッシュ — Toy Story Room

**Feature Branch**: `002-jenga-crash-toystory`

**Created**: 2026-05-30

**Status**: Draft

**Input**: User description: トイストーリー風の子供部屋を舞台にしたブラウザWebGLデモ。「積み上げたものを一気に壊す」根源的なスッキリ感（カタルシス）を体験させる、爽快・全ぶっ壊し系のインタラクティブ体験。ターンA：出鱈目ジェンガ＆クラッシュ（息を潜める→積み上げ→豪快クラッシュ→ループ）。

## Overview

A browser-based WebGL demo set in a Toy Story–style child's bedroom built entirely around one primal pleasure: **stacking things sky-high and then smashing them all down at once**. When the viewer holds still, the room's toys come alive and busily pile blocks, teacups, and mini-cars into an absurd, physics-defying tower in the center of the view. The tension builds as the tower grows taller and wobblier — until the viewer makes their move: a vigorous mouse swing sends an invisible "wrecking ball" through the tower, or a click fires a toy cannon at it. With a loud, satisfying *crash-clatter-clatter*, the haphazard tower collapses and scatters in a real physics simulation. The reward is the stress-melting exhilaration of "**I** caused that magnificent crash." Then the toys quietly begin rebuilding, and the loop repeats.

There is no score and no failure state; the value is the felt cycle of **anticipation → release → catharsis**, endlessly repeatable.

## Clarifications

### Session 2026-05-30

- Q: What is the core sensation this piece delivers? → A: Cathartic destruction — the satisfaction of toppling a tall, precariously stacked tower that was built up while you waited.
- Q: How does the build phase start/advance? → A: Self-playing while the viewer is idle. When the pointer is held still, toys autonomously stack objects ever higher; pointer stillness is the "holding your breath" cue that lets the tower grow.
- Q: How does the viewer trigger the crash? → A: Two ways, both available: (a) swing the mouse fast so the cursor acts as an invisible wrecking ball that knocks into the tower, and (b) click to fire a toy cannon projectile at the tower.
- Q: What makes the crash satisfying? → A: A real rigid-body physics collapse — pieces tumble, bounce, and scatter individually; the destruction must look physically convincing, not a canned animation. (Audio reinforces it but the collapse stands alone on visuals — see below.)
- Q: What happens after the crash? → A: The toys automatically clear/reset and begin stacking a fresh tower; the experience loops indefinitely with no score or end state.
- Q: Should on-screen title/branding text be shown? → A: No. Do not display title/branding strings. A minimal control hint (how to crash) is allowed; the room and the stacking tower carry the piece.
- Q: Should sound be implemented in this iteration? → A: Originally deferred; now implemented. Two looping ambient tracks crossfade by pointer state — active/moving → "Chaotic Toy Parade", idle/still → "When Humans Look Away" — started on the first user gesture (autoplay policy). A crash SFX (`assets/crash.mp3`, volume scaled by impact strength) plays on collapse when the file is present, and is silently skipped if absent (the crash sound file is not yet provided).
- Q: What is the room's background? → A: A supplied pre-rendered worldview image (`assets/ChatGPT_Image_2026530_12_52_55.png`, a richly detailed fantastical toy room) is used as the far backdrop; the 3D physics tower is built and smashed in the foreground in front of it.
- Q: What happens when the viewer holds the pointer perfectly still (the "humans look away" cue)? → A: The toys keep doing what they do in User Story 1 — they continue stacking the precarious tower — and the ambient music crossfades to the idle track "When Humans Look Away". Pointer stillness does NOT enter Chaos Mode; it is only the "holding your breath" build cue plus the idle music.
- Q: How is Chaos Mode entered and exited? → A: Chaos Mode is entered **exclusively** by Ctrl+click (or Cmd+click) — that is the sole trigger. There is no idle/stillness path into chaos. On Ctrl/⌘+click, gravity drops to ~0 and the toys break into a crazy zero-gravity "parade," swirling and floating chaotically around the center; a hidden **Treasure** object appears, mixed among the floating toys, ONLY during this chaos, and the wandering human averts its gaze. Chaos ends when the viewer makes a deliberate pointer move (pointer speed over a threshold) — "humans look back" — at which point gravity returns, the treasure vanishes, the pieces fall, and the normal build/crash loop resumes.
- Q: Is mobile/touch supported? → A: No — this is desktop only. Mobile and portrait support are explicitly out of scope.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Watch the tower rise while holding still (Priority: P1)

A visitor opens the page into a first-person-ish view of a Toy Story–style bedroom. When they do nothing — pointer at rest — the toys spring to life and start carrying blocks, teacups, and mini-cars to the center of the room, stacking them into a tall, wobbly, physically unstable tower. The longer the viewer "holds their breath" (stays still), the higher and more precarious the tower climbs, building palpable tension.

**Why this priority**: This is the anticipation half of the core loop; without a tower rising on its own there is nothing to destroy and no tension to release. It also establishes the living-toy world and the physics that the crash depends on.

**Independent Test**: Open the page, keep the pointer still, and confirm that multiple toy objects are stacked into a growing tower that visibly gains height and looks increasingly unstable over time, with no input required.

**Acceptance Scenarios**:

1. **Given** the page has loaded and the pointer is at rest, **When** the viewer waits, **Then** toys autonomously move objects to the center and stack them into a tower that grows taller over time.
2. **Given** the tower is being built, **When** it gains height, **Then** the stack looks visibly precarious/unstable (leaning, irregular, "出鱈目" balance) rather than a neat, safe stack.
3. **Given** the stacking is in progress, **When** the viewer continues to do nothing, **Then** the tower keeps rising until it reaches a "ready to topple" height (a clear visual peak of tension).

---

### User Story 2 - Smash the tower and feel the catharsis (Priority: P1)

Once the tower is tall enough, the viewer acts: they either swing the mouse quickly so the cursor — an invisible wrecking ball — plows through the stack, or they click to fire a toy cannon at it. Either way the tower collapses in a convincing physics simulation: pieces topple, bounce, and scatter individually across the floor, accompanied by a loud, gratifying crash-and-clatter. The viewer feels they personally caused a big, messy, satisfying collapse.

**Why this priority**: This is the release/payoff half of the core loop and the entire reason the piece exists. The stacking (US1) only has meaning because this destruction pays it off.

**Independent Test**: With a tower present, swing the pointer fast through it (and separately, click to fire the cannon) and confirm the tower collapses via per-piece physics with a crash sound, and that the collapse reads as caused by the viewer's action.

**Acceptance Scenarios**:

1. **Given** a tall tower exists, **When** the viewer swings the pointer rapidly into/through it, **Then** the impacted pieces are knocked loose and the tower collapses with individual pieces tumbling and scattering.
2. **Given** a tall tower exists, **When** the viewer clicks, **Then** a toy-cannon projectile is fired toward the tower and, on impact, knocks it down with the same physics collapse.
3. **Given** a collapse is triggered, **When** pieces fall, **Then** the collapse reads as physically convincing on visuals alone — pieces tumble, bounce, and scatter — with no hard dependence on sound (audio reinforces but is not required, see FR-010).
4. **Given** a slow, gentle pointer drift, **When** it merely grazes the tower, **Then** it does NOT trigger a full crash (only a fast swing or a cannon shot does), preserving the "hold still to build" tension.

---

### User Story 3 - The cycle loops (Priority: P2)

After a crash, the scattered pieces settle, the toys tidy up, and they begin building a fresh tower. The viewer can sit back and repeat the satisfying build-and-smash cycle as many times as they like, with no score, timer, or end state.

**Why this priority**: Looping turns a one-shot gag into a repeatable stress-relief toy, which is the product's actual value. It depends on US1 and US2 existing, so it is sequenced after them.

**Independent Test**: Trigger a crash, then wait, and confirm the scene resets itself and a new tower begins stacking without any reload or manual reset.

**Acceptance Scenarios**:

1. **Given** a tower has just collapsed, **When** the pieces come to rest, **Then** the scene clears the debris (or recycles the pieces) and the toys begin stacking a new tower.
2. **Given** the loop has run several times, **When** the viewer keeps watching, **Then** the build→smash→reset cycle continues indefinitely with no score or failure state shown.

---

### User Story 4 - Chaos Mode: zero-gravity parade & hidden treasure (Priority: P2)

When the viewer Ctrl/⌘+clicks, the room slips into **Chaos Mode**: gravity fades to almost nothing and the toys, instead of falling, break into a crazy zero-gravity "parade," swirling and drifting chaotically around the center. The wandering human averts its gaze (turns away / covers eyes) for the duration. While this chaos lasts, a hidden **Treasure** appears, mixed in among the floating toys — visible only when no one is "watching." The instant the viewer makes a deliberate pointer move (pointer speed over a threshold) — "humans look back" — gravity snaps back, the treasure vanishes, the pieces drop, and the normal build/crash loop resumes. (Note: merely holding the pointer still does NOT enter chaos — that keeps building the tower as in User Story 1; Ctrl/⌘+click is the sole chaos trigger.)

**Why this priority**: Chaos Mode is the secret, playful payoff layered on top of the core build→smash loop — it rewards a deliberate Ctrl/⌘+click with a surreal spectacle and a glimpse of treasure, reinforcing the "when humans look away, toys come alive" fantasy. It builds on the existing physics world (US1) and the idle ambient track, so it is sequenced after the core loop (P2).

**Independent Test**: Ctrl/⌘+click and confirm the human averts its gaze, gravity drops, the toys begin a floating/swirling zero-g parade, and a treasure object appears among them; then make a deliberate pointer move and confirm gravity returns, the treasure disappears, and the pieces fall back into the normal loop.

**Acceptance Scenarios**:

1. **Given** the scene is in the normal loop, **When** the viewer Ctrl/⌘+clicks, **Then** gravity lerps toward ~0, the wandering human averts its gaze, and the toys begin a chaotic zero-gravity parade (swirling/floating around the center) rather than stacking or falling.
2. **Given** Chaos Mode is active, **When** the viewer looks at the floating toys, **Then** a single hidden Treasure object is visible mixed among them, present ONLY during chaos.
3. **Given** Chaos Mode is active, **When** the viewer makes a deliberate pointer move (speed over the threshold), **Then** gravity is restored, the Treasure vanishes, the pieces fall, and the scene settles back into the normal build/crash loop.
4. **Given** the pointer is held perfectly still, **When** the idle threshold is reached, **Then** the toys keep stacking the tower (as in User Story 1) and the ambient music crossfades to "When Humans Look Away" — Chaos Mode is NOT entered.
5. **Given** Chaos Mode (zero-g) is active, **When** the viewer swings the cursor or fires the cannon, **Then** no crash/collapse is triggered (the crash mechanic is disabled while gravity is off) — the chaos parade is not interrupted into a normal collapse.

---

### Edge Cases

- **Viewer never acts**: If the viewer holds still indefinitely, the tower should reach a maximum height and either topple on its own (gravity wins) or hold at a capped peak, so the scene never grows without bound or off-screen.
- **Crash with no tower present**: A swing or click during the reset/early-build phase (few or no pieces) should harmlessly do nothing dramatic rather than error.
- **Audio blocked by autoplay policy**: The crash must remain visually satisfying on its own; ambient tracks and the crash SFX are enhancements that start only after the first user gesture. If the crash sound file (`assets/crash.mp3`) is absent, the crash SFX is silently skipped (no error).
- **Crash trigger during Chaos Mode**: While gravity is off (zero-g chaos), a fast pointer swing or cannon click MUST NOT trigger a normal collapse — the crash mechanic is disabled until gravity is restored, so chaos cannot be "smashed."
- **Treasure outside chaos**: The Treasure MUST be hidden/inactive whenever Chaos Mode is not active; it never appears during normal BUILDING/READY/CRASHING/SETTLING/RESET play and never persists after gravity returns.
- **Window resize**: The tower and the room must stay framed and legible as the desktop browser window is resized; the central stack must not be pushed off-screen. (Desktop only — mobile/portrait layout is out of scope.)
- **Low-end GPU / many pieces**: With many rigid bodies in flight, the simulation must degrade gracefully (fewer pieces or simpler shapes) rather than stutter badly.
- **Tab backgrounded**: When the page is not visible, the simulation should pause/throttle so it does not waste resources or "explode" the physics on return.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: The experience MUST present a Toy Story–style child's bedroom scene as the setting for the stacking and crashing, using the supplied worldview image (`assets/ChatGPT_Image_2026530_12_52_55.png`) as the far backdrop, with the interactive 3D tower in the foreground in front of it.
- **FR-002**: While the pointer is at rest (the viewer is "holding their breath"), animate toys MUST autonomously gather objects (blocks, teacups, mini-cars, etc.) and stack them into a tower at a central location, with no input required.
- **FR-003**: The stacked tower MUST grow taller over time and MUST read as precarious / haphazardly balanced ("出鱈目"), conveying mounting instability and tension rather than a safe, orderly stack.
- **FR-004**: The tower MUST grow toward a clear peak height that signals "ready to topple," and MUST be bounded so it cannot grow off-screen or without limit (per the edge case, it caps or self-topples at the maximum).
- **FR-005**: The viewer MUST be able to trigger a crash by swinging the mouse pointer rapidly so the cursor acts as an invisible "wrecking ball" that strikes the tower.
- **FR-006**: The viewer MUST be able to trigger a crash by clicking to fire a "toy cannon" projectile toward the tower, which knocks it down on impact.
- **FR-007**: A slow or gentle pointer movement MUST NOT trigger a full crash — only a sufficiently fast swing or a cannon shot does — so that staying calm/still is what lets the tower keep building.
- **FR-008**: When a crash is triggered, the tower MUST collapse via a rigid-body physics simulation in which individual pieces topple, bounce, and scatter independently (not a single canned/pre-baked animation).
- **FR-009**: The collapse MUST feel physically convincing and "爽快" (cathartic) — pieces respond to the impact direction/force and tumble believably across the floor.
- **FR-010**: The crash MUST be visually satisfying and cathartic on its own, with NO hard dependence on audio. Audio is now layered in as an enhancement: two looping ambient tracks crossfade by pointer state (active → "Chaotic Toy Parade", idle/still → "When Humans Look Away"), started on the first user gesture (autoplay policy). A crash SFX (`assets/crash.mp3`, volume scaled by impact strength) plays on collapse when the file is present and is silently skipped if absent.
- **FR-018**: A human figure MUST roam randomly on foot near the central tower (wander to random nearby targets, pause, look around, repeat) to add life to the scene; it is decorative and does not affect the physics.
- **FR-019**: When the viewer Ctrl+clicks (or Cmd+clicks), the experience MUST enter **Chaos Mode**: world gravity MUST lerp toward ~0 and the toy pieces MUST break into a chaotic zero-gravity "parade" (swirling/floating around the center). Ctrl/⌘+click is the sole trigger for Chaos Mode — pointer stillness MUST NOT enter chaos. When the viewer makes a deliberate pointer move (pointer speed over a threshold), gravity MUST be restored, the pieces MUST fall, and the normal build/crash loop MUST resume. While gravity is off, the crash trigger MUST be disabled (FR-005/FR-006 do not fire during chaos).
- **FR-020**: A single hidden **Treasure** object MUST appear, mixed among the floating toys, ONLY while Chaos Mode is active, and MUST be hidden/inactive at all other times (it vanishes the moment gravity is restored).
- **FR-021**: While Chaos Mode is active (entered via the Ctrl/⌘+click trigger of FR-019), the wandering human figure MUST avert its gaze (turn away / cover eyes) for the duration of chaos, and return to its normal wandering once chaos ends (on the deliberate pointer move).
- **FR-011**: After a collapse, the scene MUST automatically settle, clear/recycle the scattered pieces, and begin stacking a fresh tower, so the build→smash→reset cycle loops indefinitely.
- **FR-012**: The experience MUST run with no score, timer, win/lose, or end state; it is a self-resetting, endlessly repeatable toy.
- **FR-013**: The experience MUST run in a current desktop web browser by opening a single page, with no installation or build step required of the viewer. (Desktop only — mobile is out of scope.)
- **FR-014**: The experience MUST adapt to the desktop browser window on load and on resize, keeping the central tower framed. (Mobile/portrait support is out of scope — desktop only.)
- **FR-015**: The experience MUST be publicly deployable as a static site (suitable for Vercel static hosting) served from the repository root, unchanged from the local version.
- **FR-016**: The experience MUST NOT display title/branding strings; a minimal control hint explaining how to trigger the crash is permitted, but the scene itself carries the piece.
- **FR-017**: Toy figures MUST be original, generic homages (e.g. a cowboy-ish figure, a spaceman-ish figure) — no licensed characters, exact likenesses, or copyrighted assets.

### Key Entities

- **Stackable Object**: A physical toy piece (building block, teacup, mini-car, etc.) with shape, mass, and a rigid body; can be carried, stacked, struck, and scattered. The raw material of both the tower and the debris.
- **Tower**: The transient central stack of Stackable Objects being built up during the idle phase; characterized by its current height and instability, and the target of the crash.
- **Builder Toy**: An animate toy (cowboy-ish, spaceman-ish, etc.) that carries Stackable Objects to the center and places them on the Tower during the build phase.
- **Wrecking Cursor**: The pointer reinterpreted as an invisible heavy ball; when moved fast enough it imparts impact force to the Tower's pieces.
- **Toy Cannon Projectile**: A physical projectile spawned on click that flies toward the Tower and transfers impact force on collision.
- **Crash Event**: The moment of triggered collapse; converts the Tower into free-falling, scattering pieces and cues the subsequent reset. Triggers the crash SFX (`assets/crash.mp3`) at a volume scaled by impact strength when the file is present; disabled while Chaos Mode (zero-g) is active.
- **Chaos Mode**: An override state entered by Ctrl/⌘+click in which world gravity scales toward ~0 and pieces swirl/float in a zero-gravity parade. Holds a reference to the active Treasure and a flag; exits when the pointer makes a deliberate move (speed over a threshold), restoring gravity and resuming the normal loop.
- **Treasure**: A special hidden object (mesh + body) that becomes visible/active only while Chaos Mode is on, mixed among the floating toys; hidden and inactive at all other times.
- **Wandering Human**: A decorative human figure that roams on foot near the tower (wander → pause → look around) and does not affect physics; while Chaos Mode is active (Ctrl/⌘+click trigger) it averts its gaze (turns away / covers eyes), returning to normal wandering when chaos ends.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: Within 15 seconds of opening the page (and with no input), a first-time viewer sees toys actively stacking a tower that has grown visibly taller.
- **SC-002**: A viewer can topple the tower on demand: a fast pointer swing through it, or a click to fire the cannon, causes a collapse within a fraction of a second of the action.
- **SC-003**: When the tower collapses, a viewer perceives individual pieces tumbling and scattering (at least several distinct pieces moving independently), not a single rigid animation.
- **SC-004**: A first-time viewer recognizes the experience as satisfying or stress-relieving ("scratched the itch to knock it down") — the anticipation→release loop lands.
- **SC-005**: After a crash, the scene begins building a new tower within a couple of seconds, with no reload or manual reset.
- **SC-006**: The experience loads and begins animating within 5 seconds on a typical broadband connection and sustains smooth visible motion (including during a multi-piece collapse) on a current mid-range laptop. (Desktop only.)
- **SC-007**: The experience is reachable at a public URL and renders correctly without the viewer installing or configuring anything.

## Assumptions

- The piece is **self-playing in its build phase but interactive in its payoff**: the tower stacks itself while the viewer is idle, and the viewer chooses when to unleash the crash (fast swing or cannon click). Doing nothing still produces a continually building (and eventually self-toppling/capped) tower.
- The first delivery target is **running locally** (open the page in a browser on the developer's machine); public Vercel deployment comes only after the local version works and is approved.
- "Toy Story room" refers to the *aesthetic and rules* (Andy's-bedroom look) as homage; no licensed characters, exact likenesses, or copyrighted assets are used.
- Audio is now **implemented**: two looping ambient tracks crossfade by pointer state ("Chaotic Toy Parade" when active, "When Humans Look Away" when idle/still), started on the first user gesture (autoplay policy). A crash SFX (`assets/crash.mp3`, volume scaled by impact strength) plays on collapse when present and is silently skipped if absent — the crash sound file is not yet provided. The visual collapse alone still delivers the catharsis without sound.
- A worldview backdrop image is supplied at `assets/ChatGPT_Image_2026530_12_52_55.png` and is used as the far background; the interactive 3D tower/floor sits in front of it.
- Target viewers are general audiences viewing a hackathon/art demo on a personal device; no accounts, persistence, or multi-user concerns apply.
- Deployment targets static hosting on Vercel with the single page at the repository root; no server-side logic is required.
- This feature supersedes the earlier AIWS perceptual-scale concept (feature 001); the Toy Story room aesthetic is reused, but the core mechanic is now stacking and cathartic destruction.

## Future Enhancements (Out of scope for this iteration)

These are explicitly **not** part of the current spec/tasks but are recorded for a later iteration:

- **FE-001 — Cursor weight & resistance (ネバネバ沼の重み)**: Give the cursor a sense of mass and friction so it lags behind the actual pointer — moving the mouse feels like dragging through a sticky swamp, adding a tactile heaviness to the "wrecking ball." Deferred because the core build-and-smash loop ships first; the weighted-cursor feel is a polish layer on top.
- **FE-002 — Cursor inertia / sliding feel (慣性・氷の上の滑り)**: Even after the pointer stops, on-screen elements (the cursor influence, or light objects it was pushing) keep gliding a moment before settling, as if on ice — adding a momentum/slide sensation distinct from FE-001's drag-resistance. Deferred for the same reason.
- **FE-003 — Crash & ambient audio**: ~~Deferred~~ **Now implemented** (see FR-010 / Clarifications): two crossfading ambient loops ("Chaotic Toy Parade" / "When Humans Look Away") gated on pointer state and started on first gesture, plus an impact-scaled crash SFX (`assets/crash.mp3`) that is silently skipped when the file is absent (it is not yet provided). The visual experience remains complete without sound. Remaining nice-to-haves (e.g. randomized builder footstep/movement SFX) may still be added later.
