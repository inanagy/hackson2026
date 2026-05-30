# Tasks: 出鱈目ジェンガ＆クラッシュ — Toy Story Room

**Input**: Design documents from `/specs/002-jenga-crash-toystory/`

**Prerequisites**: [plan.md](./plan.md), [spec.md](./spec.md), [research.md](./research.md), [data-model.md](./data-model.md), [contracts/physics-interface.md](./contracts/physics-interface.md), [quickstart.md](./quickstart.md)

**Tests**: Not requested — single-file art demo verified by manual visual inspection. No automated test tasks are generated.

**Organization**: Tasks are grouped by user story (US1/US2/US3/US4 from spec.md) so each can be implemented and visually verified as an independent increment.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel (different files / independent regions, no dependencies). NOTE: the app is a single `index.html`, so most source tasks edit the same file and are therefore **not** [P]; [P] is reserved for genuinely independent setup/config/asset steps.
- **[Story]**: US1, US2, US3, US4 — maps to spec user stories.
- All source tasks target the single file `index.html` at the repository root; descriptions name the specific code region to add. Built from scratch — feature 001's `index.html` is NOT reused.

---

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Get dependencies vendored and a runnable skeleton open in the browser.

- [X] T001 [P] Vendor cannon-es `0.20.0` locally into `vendor/cannon-es/cannon-es.js` per [quickstart.md](./quickstart.md) (download the ESM build once at setup time; verify it contains `export` and is not an error page). Three.js is already at `vendor/three/`.
- [X] T002 [P] Confirm the backdrop asset `assets/ChatGPT_Image_2026530_12_52_55.png` is present (1672×941 PNG).
- [ ] T003 Create a fresh `index.html` at repo root: import map pointing `three`, `three/addons/`, and `cannon-es` at the local `vendor/` files (per [contracts/physics-interface.md](./contracts/physics-interface.md)); a full-screen `WebGLRenderer` with capped pixel ratio, a `PerspectiveCamera`, an empty scene, and resize handling. Add a small visible error box so module-load failures are obvious.
- [ ] T004 Verify local run via `python3 -m http.server 8000` → `http://localhost:8000/` (modules + import map load from `vendor/` with no CDN/network and no console errors).

**Checkpoint**: A full-screen WebGL canvas renders locally from vendored deps only.

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Physics world, backdrop, floor, camera framing, and the state-machine scaffold that every user story depends on. ⚠️ Must complete before US1–US3.

- [ ] T005 In `index.html`, create the cannon-es `World` with gravity and a fixed-timestep stepping helper using an accumulator and `dt` clamp (per [contracts/physics-interface.md](./contracts/physics-interface.md)); allow body sleep.
- [ ] T006 In `index.html`, load the backdrop image via `THREE.TextureLoader` from `./assets/ChatGPT_Image_2026530_12_52_55.png` and render it as a static far backdrop (large textured plane / background quad), cover-fit to the frustum (research Decision 8, FR-001).
- [ ] T007 In `index.html`, add an invisible `mass:0` physics ground plane plus a soft contact-shadow catcher, and tune camera pitch/height/position so the foreground floor aligns with the depicted wooden floor in the backdrop and the central tower zone is framed (research Decision 3 & 8).
- [ ] T008 In `index.html`, add scene lighting matched to the backdrop's warm key light so foreground 3D pieces sit naturally in the image.
- [ ] T009 In `index.html`, implement the global state machine (`Phase = BUILDING|READY|CRASHING|SETTLING|RESET`) with a single `setPhase()` transition point and the `buildTimer`/`idleTime`/`settleTimer` timers ([data-model.md](./data-model.md)); wire it into the animate loop (calling `stepPhysics`) even before behaviors exist.
- [ ] T010 In `index.html`, define the Piece record + pool (`{mesh, body, kind, home, inTower}`, cap `PIECE_POOL_MAX`) and the per-frame sync `mesh.position/quaternion ← body.position/quaternion` (contracts, data-model).

**Checkpoint**: Backdrop + floor render, a test piece dropped into the world falls and rests on the floor, state machine ticks.

---

## Phase 3: User Story 1 — Watch the tower rise while holding still (Priority: P1) 🎯 MVP

**Goal**: While the pointer is still, toys auto-stack a precarious, growing tower at center.

**Independent Test**: Open the page, keep the pointer still, watch a tower of multiple pieces grow taller and look unstable, with no input (SC-001, US1 scenarios).

- [ ] T011 [US1] In `index.html`, build the piece factory for `kind` ∈ {block, teacup, minicar}: procedural Three.js geometry + warm colors and a matching cannon-es shape (Box/Cylinder) per piece (data-model Piece, FR-002).
- [ ] T012 [US1] In `index.html`, implement the BUILDING behavior: on `buildTimer`, take a pooled piece and spawn it at the current tower top with a deliberate random horizontal offset + random yaw so the stack leans/"出鱈目" (research Decision 4, FR-002, FR-003); track `tower.pieces` and `topY`.
- [ ] T013 [US1] In `index.html`, enforce the tower bound: stop adding at `targetCount`/`maxHeight` and transition BUILDING → READY; allow accumulated lean to topple naturally at the cap (FR-004, edge case "viewer never acts").
- [ ] T014 [US1] In `index.html`, add decorative BuilderToy figures (cowboy-ish, spaceman-ish — original homages) with idle motion + a carry/place gesture near the spawn point (FR-017, data-model BuilderToy).
- [ ] T015 [US1] In `index.html`, add pointer-stillness ("holding breath") detection that accrues `idleTime` only while pointer speed ≈ 0 and gates the build cadence on it (US1 scenario 1/3).
- [ ] T016 [US1] Manual verify US1 against SC-001 in the browser; tune spawn cadence, offsets, and cap so the tower clearly grows and reads as precarious.

**Checkpoint**: MVP — a wobbly tower builds itself while you hold still.

---

## Phase 4: User Story 2 — Smash the tower and feel the catharsis (Priority: P1)

**Goal**: A fast pointer swing (invisible wrecking ball) or a click (toy cannon) collapses the tower via real physics; slow movement does not.

**Independent Test**: With a tower present, swing the pointer fast through it and (separately) click to fire the cannon → per-piece physics collapse; a slow drift does NOT crash it (SC-002, SC-003, FR-007).

- [ ] T017 [US2] In `index.html`, track pointer position/velocity each frame (`speed` px/s) and project the cursor onto a vertical plane through the tower center to get a world target (contracts Input, research Decision 5).
- [ ] T018 [US2] In `index.html`, add the WreckingCursor: a sphere body lerped toward the cursor world target; mark it `active` only when `speed > SWING_THRESHOLD`, and on contact while active impart impulse ∝ its velocity (FR-005, FR-007).
- [ ] T019 [US2] In `index.html`, add the toy cannon: on `pointerdown`, `Raycaster` from camera through the pointer gives the aim direction; spawn a pooled projectile body with high velocity toward the tower (FR-006, data-model CannonProjectile).
- [ ] T020 [US2] In `index.html`, implement `triggerCrash(by, impulse, point)`: accept a crash only when ≥ `MIN_PIECES_TO_CRASH` pieces exist (BUILDING/READY) AND gravity is on (ignore during Chaos Mode, FR-019), call the `onImpact(impulse, point)` hook (plays the impact-scaled crash SFX, FR-010 — see T034), and `setPhase(CRASHING)` (contracts, edge case "crash with no tower").
- [ ] T021 [US2] In `index.html`, tune collision impulse/restitution/friction so the collapse is convincing and cathartic — pieces topple, bounce, and scatter individually across the floor (FR-008, FR-009).
- [ ] T022 [US2] Manual verify US2 against SC-002/SC-003 and FR-007 in the browser (fast swing crashes, click crashes, slow drift does not).

**Checkpoint**: The tower can be smashed two ways; the collapse feels great with no sound.

---

## Phase 5: User Story 3 — The cycle loops (Priority: P2)

**Goal**: After a crash settles, the scene auto-resets and rebuilds, looping forever with no score.

**Independent Test**: Trigger a crash, wait → debris settles, scene resets, a new tower starts, no reload (SC-005, US3 scenarios).

- [ ] T023 [US3] In `index.html`, implement CRASHING → SETTLING → RESET: detect settle when total kinetic energy < ε or a max-settle timeout elapses ([data-model.md](./data-model.md)).
- [ ] T024 [US3] In `index.html`, implement `recyclePieces()`: reset pooled piece bodies to `home`, zero velocity, sleep, clear `tower`, then RESET → BUILDING to restart the loop (FR-011, FR-012, research Decision 7).
- [ ] T025 [US3] Manual verify US3 against SC-005 in the browser; tune settle threshold/timeout and reset pacing so the loop feels natural (rebuild within a couple of seconds).

**Checkpoint**: All three stories work; build→smash→reset loops indefinitely.

---

## Phase 6: Polish & Cross-Cutting Concerns

- [ ] T026 In `index.html`, add the minimal control hint UI (how to crash: swing / click) and confirm NO title/branding strings appear (FR-016).
- [ ] ~~T027~~ **REMOVED — out of scope (desktop only).** Mobile/portrait support is no longer required (FR-014, SC-006). Desktop resize handling (cover-fit backdrop, keep the tower centered/framed, cap pixel ratio) is covered by the Setup/Foundational resize handling (T003) and the perf pass (T029).
- [ ] T028 In `index.html`, pause/throttle physics + rAF when the tab is hidden (visibility change) so the sim doesn't blow up on return (edge case "tab backgrounded").
- [ ] T029 In `index.html`, performance tune the body cap / shapes so a full multi-piece collapse stays smooth on a mid-range laptop and phone (SC-006).
- [ ] T030 Run the full [quickstart.md](./quickstart.md) verification checklist end-to-end locally and confirm SC-001…SC-007 pass (audio plays per FR-010; the crash SFX is silently skipped until `assets/crash.mp3` is provided).
- [ ] T031 [P] After local sign-off, deploy to Vercel (`npx vercel` / dashboard) per quickstart — **only after the user approves the local version**.

---

## Phase 7: Chaos Mode, Wandering Human & Audio (Priority: P2 + cross-cutting)

**Purpose**: Layer the secret zero-gravity Chaos Mode + hidden treasure (US4) on top of the core loop, plus the decorative wandering human and the now-implemented audio. Builds on the physics world (T005) and state machine (T009); the audio crossfade also uses the pointer-state/idle detection (T015). Chaos is triggered by Ctrl/⌘+click, not by idle.

**Independent Test**: Ctrl/⌘+click → gravity drops, the human averts its gaze, toys swirl in a zero-g parade, a treasure appears among them; make a deliberate pointer move → gravity returns, treasure vanishes, pieces fall back into the normal loop. Holding the pointer still keeps building the tower (US1) + crossfades to idle music and does NOT enter chaos.

- [X] T032 In `index.html`, add the decorative wandering Human figure (original homage) that roams on foot near the tower: wander to a random nearby `target`, walk facing `yaw`, pause/look around, repeat; does not affect physics (FR-018, data-model Human).
- [X] T033 In `index.html`, implement the two crossfading ambient audio loops gated on pointer state — active/moving → "Chaotic Toy Parade", idle/still → "When Humans Look Away" — started on the first user gesture to satisfy autoplay policy (FR-010, Clarifications).
- [X] T034 In `index.html`, wire `onImpact(impulse, point)` to play the crash SFX from `assets/crash.mp3` at a volume scaled by impact strength; load defensively so a missing file is silently skipped (the crash sound file is not yet provided) (FR-010, edge case "audio blocked / file absent").
- [ ] T035 [US4] In `index.html`, add the Chaos Mode gravity toggle + Ctrl/⌘+click trigger (the sole trigger — pointer stillness must NOT enter chaos): on a Ctrl/⌘+click, lerp `PhysicsWorld.gravityScale` toward 0 and set `chaos.active`; when the viewer makes a deliberate pointer move (speed over a threshold), lerp gravity back to 1 and clear chaos so pieces fall and the normal loop resumes (FR-019, data-model ChaosMode + state-machine override).
- [ ] T036 [US4] In `index.html`, apply the zero-g swirl/parade forces to Pieces while chaos is active (swirling/wandering forces around the center) so the toys float chaotically instead of falling, and disable the crash trigger while gravity is off (FR-019).
- [ ] T037 [US4] In `index.html`, add the hidden Treasure object (mesh + body) that becomes visible/active ONLY while Chaos Mode is on, mixed among the floating toys, and is hidden/removed from the sim the instant gravity is restored (FR-020, data-model Treasure).
- [ ] T038 [US4] In `index.html`, implement the avert-gaze visual tied to Chaos Mode: while `chaos.active` (entered via the Ctrl/⌘+click trigger of T035), set `Human.avertGaze` so the human turns away / covers eyes for the duration of chaos, and clear it when chaos ends on the deliberate pointer move (FR-021).
- [ ] T039 [US4] Manual verify US4 in the browser: Ctrl/⌘+click → zero-g parade + treasure appears + human averts gaze; deliberate pointer move → gravity returns, treasure vanishes, pieces fall; swing/click during chaos does NOT crash; holding the pointer still keeps building the tower + idle music (no chaos). Tune thresholds and swirl forces.

**Checkpoint**: Stillness yields a surreal zero-g parade with a hidden treasure; audio crossfades by pointer state; a human wanders the room.

---

## Dependencies & Execution Order

- **Setup (T001–T004)** → no deps. T001/T002 are [P] (vendor + asset are independent).
- **Foundational (T005–T010)** → after Setup; **blocks all user stories** (physics world + backdrop + floor + state machine + piece pool are shared).
- **US1 (T011–T016)** → after Foundational. This is the MVP; deliverable on its own.
- **US2 (T017–T022)** → after Foundational; needs a tower to exist (US1) to be meaningfully testable, though the input/cannon code is independent.
- **US3 (T023–T025)** → after US2 (needs a crash to recover from) and the state machine.
- **Polish (T026–T031)** → after the stories you intend to ship; T031 (Vercel) is gated on explicit user approval of the local build. (T027 removed — desktop only.)
- **Phase 7 — Chaos/Human/Audio (T032–T039)** → after Foundational (physics world T005, state machine T009); audio also uses the pointer-state/idle detection from US1 (T015) for the music crossfade. T032 (human) and T033/T034 (audio) are cross-cutting and already implemented [X]; chaos tasks (T035–T039, US4) are P2 (triggered by Ctrl/⌘+click) and implemented now.

### Reality check on parallelism

Because the app is one `index.html`, source tasks **edit the same file** and should be done sequentially to avoid conflicts — they are intentionally **not** marked [P]. Only T001, T002, and T031 (vendor/asset/deploy) are [P]-eligible. (T027 is removed — desktop only.)

---

## Implementation Strategy

### MVP first
1. Phase 1 Setup → 2 Foundational → 3 US1. **STOP and validate** the self-building wobbly tower. This is a demoable MVP (anticipation half).

### Incremental delivery
2. Add US2 (smash + physics collapse) → validate — this is the catharsis payoff.
3. Add US3 (auto-reset loop) → validate — makes it a repeatable toy.
4. Add US4 (Chaos Mode + treasure, Phase 7) → validate — the secret zero-g parade payoff; also layer in the wandering human and audio crossfade/crash SFX.
5. Polish (control hint, tab-hidden, desktop perf), then (only after approval) deploy to Vercel.

---

## Notes
- Single-file demo: commit after each phase/logical group.
- Verify visually at each checkpoint before moving on.
- **Audio is implemented** (FR-010, T033/T034): two crossfading ambient loops gated on pointer state + an impact-scaled crash SFX via `onImpact()`. The crash sound file (`assets/crash.mp3`) is not yet provided, so the crash SFX is silently skipped until it is added.
- Dependencies are vendored locally (no CDN) — keep it that way to avoid load hangs.
- Push to GitHub / Vercel only when the user explicitly asks.
