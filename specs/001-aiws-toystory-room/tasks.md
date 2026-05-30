# Tasks: Alice in Wonderland Syndrome — Toy Story Room

**Input**: Design documents from `/specs/001-aiws-toystory-room/`

**Prerequisites**: [plan.md](./plan.md), [spec.md](./spec.md), [research.md](./research.md), [data-model.md](./data-model.md), [contracts/shader-uniforms.md](./contracts/shader-uniforms.md)

**Tests**: Not requested in the spec — this is a single-file art demo verified by manual visual inspection. No automated test tasks are generated.

**Organization**: Tasks are grouped by user story (US1/US2/US3 from spec.md) so each can be implemented and visually verified as an independent increment.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel (different files / independent regions, no dependencies). NOTE: This app is a single `index.html`, so most tasks edit the same file and are therefore **not** [P]; [P] is reserved for genuinely independent docs/config.
- **[Story]**: US1, US2, US3 — maps to spec user stories.
- All source tasks target the single file `index.html` at the repository root; the descriptions name the specific code region/section to add.

---

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Get a runnable skeleton open in the browser.

- [X] T001 Confirm `index.html` at repo root loads Three.js + `EffectComposer`/`RenderPass`/`ShaderPass` via the CDN import map and renders a blank/full-screen canvas with resize handling (the existing prototype already does this — verify it still opens cleanly).
- [X] T002 Verify local run works via `python3 -m http.server 8000` → `http://localhost:8000/` per [quickstart.md](./quickstart.md); note any `file://` module/CORS issues.

**Checkpoint**: A full-screen WebGL canvas renders locally.

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Core scene + the central state machine that every user story depends on. ⚠️ Must complete before US1–US3.

- [X] T003 In `index.html`, build the Toy Story–style room (FR-001): procedural cloud-wallpaper walls, wooden floor, ceiling, lighting, shadows. (Prototype already has this — keep/clean it.)
- [X] T004 In `index.html`, add the global state machine described in [data-model.md](./data-model.md): an `alive` flag plus `PERCEPTUAL` ↔ `HUMAN_PRESENT` states with a `presenceTimer`, exposing eased transition helpers (fast freeze ≤0.5s, gentle revive ~1–2s).
- [X] T005 In `index.html`, define the AIWS distortion `ShaderPass` honoring the uniform contract in [contracts/shader-uniforms.md](./contracts/shader-uniforms.md): `uTime`, `uPhase` (−1..+1), `uIntensity` (0..1), `uResolution`, `uRipples[]`. Guarantee the identity state (all zero → source image unchanged).

**Checkpoint**: Room renders, distortion pass is wired into the composer, state machine ticks (even if effects are still flat).

---

## Phase 3: User Story 1 — Perceptual scale shift (Priority: P1) 🎯 MVP

**Goal**: The viewer's perceived scale drifts giant → normal → tiny → normal on a loop, room warps accordingly, indicator shows the state.

**Independent Test**: Open the page, do nothing, watch one full cycle; confirm barrel(giant)/pincushion(tiny) warp, apparent room-size change, and the indicator updating (SC-001, SC-002).

- [X] T006 [US1] In `index.html`, implement the perceptual cycle driving `uPhase`/`uIntensity` through giant→normal→tiny→normal with sine-eased envelopes (FR-002, FR-004).
- [X] T007 [US1] In the distortion shader (`index.html`), implement the signed lens warp: barrel/fisheye when `uPhase>0` (giant), pincushion when `uPhase<0` (tiny), scaled by `uIntensity` (FR-003).
- [X] T008 [US1] In `index.html`, couple camera FOV and eye height to `uPhase` (narrow+high when giant, wide+low when tiny) per research Decision 3 (FR-003).
- [X] T009 [US1] In `index.html`, add the on-screen perceptual-state indicator (label 巨人症 / normal / 小人症 + scale bar) bound to `uPhase`/state (FR-005).
- [X] T010 [US1] Manual verify US1 against SC-001/SC-002 in the browser; tune timing/strength.

**Checkpoint**: MVP — the core AIWS sensation works on its own.

---

## Phase 4: User Story 2 — The room is alive (toys & trash) (Priority: P2)

**Goal**: Toys and at least one piece of trash move on their own when no human is present.

**Independent Test**: At rest (normal state, no human), point to ≥3 independently moving entities, ≥1 of which is trash (SC-003).

- [X] T011 [US2] In `index.html`, define the animate-entity concept from [data-model.md](./data-model.md): each entity stores a `restPose`, an `idle(t)` motion fn, and a `kind` ("toy"/"trash"); per-frame it applies `idle` when `alive`, else lerps to `restPose`.
- [X] T012 [US2] In `index.html`, add Toy entities with idle motion (blocks sway, ball bob, cowboy-ish & spaceman-ish figures shuffle) — original homages only (FR-006).
- [X] T013 [US2] In `index.html`, add Trash entities (crumpled paper, can, wrapper) with exaggerated twitch/hop idle so aliveness reads clearly (FR-007).
- [X] T014 [US2] Manual verify US2 against SC-003 in the browser.

**Checkpoint**: The world feels alive; US1 still works.

---

## Phase 5: User Story 3 — "Play dead" when a human approaches (Priority: P2)

**Goal**: A human-presence event makes everything freeze (and the warp return to normal), then life resumes.

**Independent Test**: Trigger/wait for a presence event; confirm all motion stills ≤~0.5s with a perceptible human cue, holds, then revives (SC-004, SC-005).

- [X] T015 [US3] In `index.html`, implement the human-presence cue from [data-model.md](./data-model.md): a large shadow plane sweeping across the room + a door-light wash brightening from one side, driven by an event `progress` 0→1 (FR-008, visuals authoritative per FR-008a).
- [X] T016 [US3] In `index.html`, add optional footstep/door audio gated behind the first user gesture; ensure absence of audio does not break the beat (FR-008a).
- [X] T017 [US3] In `index.html`, wire the freeze: on `HUMAN_PRESENT`, set `alive=false` so all entities snap to `restPose` and hold, and lerp `uPhase`/`uIntensity` to 0 so the room looks ordinary and undistorted (FR-009, FR-009a, FR-010).
- [X] T018 [US3] In `index.html`, wire the revive: when the presence cue subsides, set `alive=true` and resume the perceptual cycle (FR-011).
- [X] T019 [US3] Manual verify US3 against SC-004/SC-005 in the browser; tune freeze snap vs revive timing.

**Checkpoint**: All three stories work; the signature freeze beat lands.

---

## Phase 6: Pointer-driven fluid ripples (cross-cutting, FR-002a)

**Goal**: Pointer/touch movement stirs fluid ripples (the three-fluid-fx feel), layered over the perceptual cycle.

**Independent Test**: Move mouse / drag on touch and see ripples follow the pointer; ripples are suppressed during freeze.

- [X] T020 In `index.html`, add a small decaying ripple pool (pos/strength/age) seeded by pointer & touch move events per [data-model.md](./data-model.md).
- [X] T021 In the distortion shader (`index.html`), sum active `uRipples` into curl-noise-modulated UV displacement; ensure ripples fade to nothing and are zeroed during freeze (FR-002a, FR-010).

**Checkpoint**: The world is touchable.

---

## Phase 7: User Story 4 — Walk through the room in first person (Priority: P3)

**Goal**: Arrow keys move the viewer through the room with a walking feel, coexisting with cursor ripples; no-input still self-plays.

**Independent Test**: Hold arrow keys → camera walks/turns with head bob; cursor still ripples at the same time; perceptual warp and freeze still apply during movement (US4 acceptance scenarios; FR-016, FR-017).

- [X] T022 [US4] In `index.html`, add arrow-key first-person locomotion: forward/back + turn (or strafe), constrained within the room bounds, layered on top of the existing automatic camera sway/FOV/height (FR-016).
- [X] T023 [US4] In `index.html`, add a subtle walking feel (head bob / step cadence) while moving, and ensure it composes with the perceptual FOV/height coupling and the freeze (movement still allowed during freeze, but the world stays frozen) (FR-016).
- [X] T024 [US4] In `index.html`, confirm arrow-key movement and pointer/touch ripples are handled by independent input handlers so both work simultaneously, and no-input remains fully self-playing (FR-017).
- [X] T025 [US4] Manual verify US4 acceptance scenarios in the browser; tune walk speed, turn rate, and bob amplitude so it feels natural and isn't nausea-inducing.

**Checkpoint**: The room is walkable and stirrable at once; self-play intact.

---

## Phase 8: Polish & Cross-Cutting Concerns

- [ ] T026 [P] Mobile/portrait check and capped pixel ratio for performance (SC-006); adjust if a real phone struggles.
- [ ] T027 Tune overall art direction (colors, lighting, timing) so giant/tiny/freeze beats read clearly to a first-time viewer.
- [ ] T028 Run the full [quickstart.md](./quickstart.md) checklist end-to-end locally and confirm every SC (SC-001…SC-007 except the deferred FR-015) passes.
- [ ] T029 [P] After local sign-off, deploy to Vercel (`npx vercel` / dashboard) per quickstart — **only after the user approves the local version**.

---

## Dependencies & Execution Order

- **Setup (T001–T002)** → no deps.
- **Foundational (T003–T005)** → after Setup; **blocks all user stories** (the state machine + shader pass are shared).
- **US1 (T006–T010)** → after Foundational. This is the MVP; deliverable on its own.
- **US2 (T011–T014)** → after Foundational. Independent of US1, but US3 depends on US2 existing (things must be alive to freeze).
- **US3 (T015–T019)** → after US2 (needs animate entities to freeze) and the state machine.
- **Ripples (T020–T021)** → after the shader pass (T005) and US1 distortion (T007); independent of US2/US3.
- **US4 / arrow-key walking (T022–T025)** → after Foundational (needs the camera + state machine); independent of US2/US3 and of ripples, though it must compose with both. Pressing nothing must still self-play (FR-017).
- **Polish (T026–T029)** → after the stories you intend to ship; T029 (Vercel) is gated on explicit user approval of the local build.

### Reality check on parallelism

Because the entire app is one `index.html`, source tasks **edit the same file** and should be done sequentially to avoid conflicts — they are intentionally **not** marked [P]. Only T026 and T029 (and the docs) are [P]-eligible as they touch config/external steps.

---

## Implementation Strategy

### MVP first
1. Phase 1 Setup → 2 Foundational → 3 US1. **STOP and validate** the core AIWS sensation. This alone is a demoable MVP.

### Incremental delivery
2. Add US2 (alive room) → validate.
3. Add US3 (freeze beat) → validate — this is the emotional payoff.
4. Add Phase 6 ripples for tactility.
5. Polish, then (only after approval) deploy to Vercel.

---

## Notes
- Single-file demo: commit after each phase/logical group.
- Verify visually at each checkpoint before moving on.
- FR-015 (motion-intensity/nausea option) is **deferred** — not in this task set.
- Push to GitHub / Vercel only when the user explicitly asks.
