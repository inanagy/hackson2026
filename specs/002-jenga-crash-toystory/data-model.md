# Phase 1 Data Model: 出鱈目ジェンガ＆クラッシュ

Entities are runtime objects in a single client-side scene — no persistence. This captures their fields, relationships, and the state machine that drives the loop.

## Entities

### PhysicsWorld
The cannon-es `World`. One per page.
- `gravity` (Vec3, ~ -9.82 on Y, possibly scaled for "toy" feel)
- `gravityScale` (0..1) — multiplier lerped toward 0 in Chaos Mode and back to 1 when chaos ends (see ChaosMode)
- `fixedTimeStep` (1/60), `accumulator` (for fixed stepping)
- `bodies[]` — all rigid bodies (static colliders + dynamic pieces + impactors)
- Relationship: owns the simulation; every dynamic Piece references one of its bodies.

### Piece (Stackable Object)
A toy unit (block / teacup / mini-car) — the raw material of tower and debris.
- `mesh` (THREE.Mesh) — visual, with procedural geometry + color
- `body` (CANNON.Body) — dynamic, `mass > 0`, shape matching the mesh (Box/Cylinder)
- `kind` ("block" | "teacup" | "minicar") — drives shape/look
- `home` (Vec3) — its resting spot in the source pile (used by builder + reset)
- `inTower` (bool) — currently part of the active tower
- Per-frame: `mesh.position/quaternion ← body.position/quaternion`
- Pooled and recycled across loop iterations (Decision 7).

### Tower
The transient central stack being built.
- `pieces[]` — ordered references to Pieces currently stacked
- `topY` — current top height (drives where the next piece spawns)
- `targetCount` / `maxHeight` — bound for FR-004
- `isReadyToTopple` (bool) — peak reached
- Derived, not a separate physics object — it is "the set of Pieces with `inTower=true`".

### BuilderToy
Decorative animate figure that visually carries/places pieces during build.
- `group` (THREE.Group) — cowboy-ish / spaceman-ish original homage
- `idle(t)` motion + a "carry/place" gesture near the spawn point
- Visual only — does not physically move Pieces (the spawn does); keeps it simple/robust.

### WreckingCursor
Pointer reinterpreted as an invisible heavy ball.
- `body` (CANNON.Body) — sphere following the cursor's projected world position
- `screenPos` (Vec2), `prevScreenPos` (Vec2) → `speed` (px/s)
- `active` (bool) — true only when `speed > SWING_THRESHOLD` (FR-007)
- On contact while active: imparts impulse ∝ its velocity → starts a Crash.

### CannonProjectile (Toy Cannon)
- `mesh` + `body` (dynamic sphere/ball)
- Spawned on click with velocity along the Raycaster aim direction
- On impact with the tower: imparts impulse → starts a Crash.
- Pooled (a small ring of projectiles) to bound count.

### CrashEvent
The triggered collapse moment.
- `triggeredBy` ("cursor" | "cannon")
- `impactPoint` (Vec3), `impactImpulse` (Vec3)
- Effects: marks all tower Pieces free (already dynamic — just perturbed), calls `onImpact(impulse, point)` which plays the crash SFX at a volume scaled by impact strength (skipped if `assets/crash.mp3` is absent), transitions state to CRASHING.
- Disabled while Chaos Mode is active (gravity off) — a trigger during chaos is ignored (FR-019).

### ChaosMode
Override state entered by Ctrl/⌘+click (the sole trigger — pointer stillness does not enter chaos).
- `active` (bool) — chaos currently on
- `gravityScale` (0..1) — what `PhysicsWorld.gravityScale` is lerped toward (→0 entering, →1 exiting)
- `treasure` (ref → Treasure) — the treasure made active only for the duration of chaos
- Behavior: while active, applies swirling/wandering forces to Pieces for the zero-g "parade", makes the Human avert its gaze, and disables the crash trigger; exits when the viewer makes a deliberate pointer move (pointer speed over a threshold), at which point it restores gravity, deactivates the Treasure, and lets Pieces fall back into the normal loop (FR-019, FR-020, FR-021).

### Treasure
A special object that exists only during Chaos Mode.
- `mesh` (THREE.Mesh) — distinctive "treasure" look, mixed among floating toys
- `body` (CANNON.Body) — joins the zero-g swirl while active
- `visible`/`active` (bool) — true ONLY while `ChaosMode.active`; hidden + removed from the sim otherwise (FR-020).

### Human (Wandering Human)
Decorative figure that roams on foot near the tower; does not affect physics.
- `group` (THREE.Group) — original homage human figure
- `pos` (Vec3), `target` (Vec3) — current and next wander destination near the tower
- `yaw` (number) — facing direction (walks toward `target`)
- `pauseTimer` (s) — dwell time at a target before picking the next (wander → pause → look around)
- `avertGaze` (bool) — true while Chaos Mode is active (entered via Ctrl/⌘+click): turns away / covers eyes for the duration of chaos (FR-021).

### AppState (state machine)
Single enum + timers governing the loop:
- `phase` ∈ { BUILDING, READY, CRASHING, SETTLING, RESET }
- `chaos` (ChaosMode) — an override that can be active on top of any `phase`
- timers: `buildTimer` (spawn cadence), `idleTime` (pointer-still duration — drives the idle ambient-music crossfade and the "holding breath" build cue; does NOT trigger chaos), `settleTimer`

## State Machine

```
        ┌──────────────────────────────────────────────────┐
        v                                                   │
   ┌──────────┐  tower reaches    ┌────────┐  crash       ┌──────────┐
   │ BUILDING │ ───────────────►  │ READY  │ ───trigger──►│ CRASHING │
   └──────────┘  targetCount      └────────┘  (cursor/    └──────────┘
        ▲  ▲          │                │       cannon)          │
        │  │ spawn    │ crash trigger  │ crash trigger          │ pieces
        │  └ piece    │ (can also fire │ (fast swing/click)     │ in motion
        │    on timer │  during BUILD) │                        ▼
        │             ▼                ▼                   ┌──────────┐
        │       (same CRASHING path — crash allowed        │ SETTLING │
        │        any time a tower exists, FR-005/006)       └──────────┘
        │                                                        │ KE < ε
   ┌──────────┐                                                  │ or timeout
   │  RESET   │ ◄────────────────────────────────────────────────┘
   └──────────┘  recycle pieces → home, clear tower, then → BUILDING
```

Transitions:
- **BUILDING → READY**: `tower.pieces.length ≥ targetCount` (or `topY ≥ maxHeight`). In READY the tower holds (or slowly leans) awaiting input; if the viewer never acts it may self-topple → CRASHING (FR-004 edge case).
- **BUILDING/READY → CRASHING**: a crash trigger fires while ≥ a few pieces are stacked (FR-005, FR-006). A trigger with no/near-empty tower is a no-op (edge case).
- **CRASHING → SETTLING**: impulse applied; pieces are tumbling.
- **SETTLING → RESET**: total kinetic energy below threshold, or a max-settle timeout elapses.
- **RESET → BUILDING**: recycle Piece bodies to their `home` (reset position/velocity/sleep), clear `tower`, restart `buildTimer` → loop (FR-011, FR-012).

### Chaos Mode override

Chaos Mode is **not** a `phase` value — it is an override that can be layered on top of any phase (most naturally entered from BUILDING/READY):

- **Enter**: a Ctrl/⌘+click — the sole trigger. There is no idle/stillness path into chaos. On entry, `chaos.active` is set, `Human.avertGaze` is set, `PhysicsWorld.gravityScale` lerps toward 0, the Treasure is activated/shown, the crash trigger is disabled, and swirling/wandering forces drive the zero-g parade (FR-019, FR-020, FR-021).
- **While active**: gravity ≈ 0; Pieces (and the Treasure) float and swirl chaotically around the center; the Human keeps its gaze averted; FR-005/FR-006 crash triggers are ignored.
- **Exit**: the viewer makes a deliberate pointer move (pointer speed over a threshold — "humans look back"). On exit, `gravityScale` lerps back to 1, `Human.avertGaze` clears, the Treasure is deactivated/hidden, and the Pieces fall under restored gravity. The underlying `phase` then settles (typically via SETTLING → RESET → BUILDING) back into the normal loop (FR-019).

## Validation / Rules

- `idleTime` only accrues while pointer speed ≈ 0; any fast swing both resets idle accounting and (if a tower exists and gravity is on) triggers a crash (ties US1 "hold still to build" to US2 "swing to smash"). Sustained stillness keeps building the tower (US1) and crossfades the ambient music to the idle track — it does NOT enter Chaos Mode. Chaos Mode is entered only by Ctrl/⌘+click (US4); a deliberate pointer move exits chaos and restores gravity.
- Dynamic body count is capped (pool size) so the sim stays smooth (perf constraint / SC-006).
- Settled bodies are allowed to `sleep` to save CPU between events.
- On tab-hidden, stepping pauses (clamped accumulator) to avoid a physics blow-up on return (edge case).
- No score, timer, or win/lose state is ever created (FR-012).
