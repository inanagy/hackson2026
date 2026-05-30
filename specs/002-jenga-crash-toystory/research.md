# Phase 0 Research: 出鱈目ジェンガ＆クラッシュ

All Technical Context unknowns resolved below. No outstanding NEEDS CLARIFICATION.

## Decision 1 — Physics engine: cannon-es (vendored local)

**Decision**: Use **cannon-es `0.20.0`** as the rigid-body physics engine, vendored as a local ESM file at `vendor/cannon-es/cannon-es.js`.

**Rationale**:
- **Pure JavaScript, no WASM** → nothing extra to fetch/instantiate; avoids the load-hang seen with CDN/WASM dependencies. Loads instantly from a local file.
- Gives real gravity, collision response, and impulse application — exactly the "pieces topple, bounce, scatter individually" the crash requires (FR-008, FR-009).
- Mature, well-documented, small, ESM-native (drops straight into an import map).

**Alternatives considered**:
- **Rapier** — faster/higher-fidelity but WASM-based; reintroduces the load-fetch risk we are explicitly avoiding. Rejected.
- **React Three Fiber + Rapier** — requires a Vite/build environment; breaks the single-file/no-build rule. Rejected.
- **Babylon.js (built-in physics)** — would mean abandoning Three.js; no reason to switch engines. Rejected.
- **Hand-rolled rigid-body math** — would have to reimplement stacking + collision response; not worth it when cannon-es is tiny and local. Rejected.

## Decision 2 — Local vendoring via import map (no CDN)

**Decision**: Reference both deps from local `vendor/` files through an in-page import map:
```json
{ "imports": {
  "three": "./vendor/three/three.module.js",
  "three/addons/": "./vendor/three/addons/",
  "cannon-es": "./vendor/cannon-es/cannon-es.js"
}}
```

**Rationale**: No runtime network dependency → no CDN hang, works offline, and the exact same files deploy to Vercel static. Three.js is already vendored; cannon-es is added by a setup task (download the `0.20.0` ESM build into `vendor/cannon-es/`).

**Note on `file://`**: ES-module import maps work when served over `http://` (e.g. `python3 -m http.server`); some browsers restrict module loading over `file://`. Quickstart documents serving locally over http to be safe.

**Alternatives considered**: CDN import map (jsDelivr/unpkg) — the previous approach; rejected due to observed loading hangs and the offline/local-first requirement.

## Decision 3 — Render ↔ physics sync model

**Decision**: cannon-es owns the simulation; Three.js mirrors it. Each dynamic piece is a pair `{ mesh (THREE.Mesh), body (CANNON.Body) }`. Each frame: step the physics world with a fixed timestep + accumulator, then copy `body.position`/`body.quaternion` into the mesh. Static colliders (floor, walls) are `mass: 0` bodies with matching invisible/visible geometry.

**Rationale**: Standard, robust Three+cannon integration; fixed-timestep stepping keeps the sim stable and frame-rate-independent (avoids "explosion" when the tab is backgrounded and `dt` spikes — clamp/accumulate, FR edge case).

## Decision 4 — Builder behavior & the "出鱈目" (haphazard) stack

**Decision**: During the build phase, spawn pieces one at a time on a timer and place each at the current tower top with a **deliberate random offset + random yaw** so the stack leans and looks precarious (FR-003). Pieces are added as dynamic bodies that briefly settle, so cannon-es itself produces the wobble. Builder toys are decorative animate figures that visually "carry/place" near the spawn — the tower's physical truth is the cannon-es bodies.

**Rationale**: Letting real physics hold the imperfect stack makes the instability genuine (a nudge can topple it) rather than faked, which pays off the crash. Random offsets create the "出鱈目" look cheaply.

**Tower height bound (FR-004)**: Cap piece count / target height; when reached, either stop adding (hold at peak) or let accumulated lean topple it naturally. Either satisfies "ready to topple / bounded."

## Decision 5 — Crash triggers: pointer velocity + click cannon

**Decision**:
- **Wrecking cursor (FR-005, FR-007)**: Track pointer position per frame; compute pointer speed (px/s). Maintain an invisible kinematic/dynamic "ball" body following the cursor's world-projected position. Only when pointer speed exceeds a threshold does the ball become an active impactor (apply its velocity as impulse on contact). Slow drift → speed below threshold → no crash, preserving build tension.
- **Toy cannon (FR-006)**: On click/tap, `Raycaster` from the camera through the pointer gives an aim direction; spawn a projectile body with high velocity toward the tower; its impact knocks pieces loose.

**Rationale**: Two independent, simultaneously available triggers per the spec. The velocity gate on the cursor is what enforces FR-007 ("only a fast swing crashes").

**Cursor→world projection**: Raycast the pointer onto a vertical plane through the tower center to get the ball's target world position; lerp the ball body toward it so its motion carries momentum.

## Decision 6 — Audio: deferred this iteration

**Decision**: **No audio is implemented now.** Per the user, the crash "ガシャン", a mouse-stop/idle cue, and any movement SFX will be added later by them. This iteration ships silent and the catharsis lands on visuals alone (FR-010).

**Implementation hook (reserved, not built)**: Leave a single clearly-marked seam where a future sound trigger can hook in — i.e. the CrashEvent / hard-collision callback can later call an (currently no-op) `onImpact(impulse, point)`. Do not wire WebAudio now.

**Rationale**: Keeps scope tight and avoids autoplay-policy work the user will handle later; the reserved hook means adding sound later is a small, localized change.

## Decision 7 — Reset / loop

**Decision**: After a crash, wait until bodies' kinetic energy falls below a threshold (settled) or a max timeout, then **recycle the existing bodies** (reset positions/velocities) rather than destroy/recreate, and restart the build phase. Recycling avoids GC churn and keeps the body pool bounded (FR-011, FR-012, perf).

**Rationale**: Object pooling keeps memory/perf stable across infinite loops.

## Decision 8 — Background: supplied worldview image as backdrop

**Decision**: Use the user-supplied image `assets/ChatGPT_Image_2026530_12_52_55.png` (1672×941, a richly detailed fantastical toy room with warm wooden floor) as the **far backdrop**, and build the interactive 3D tower/floor in the foreground in front of it. The room is therefore the image, not procedural walls (FR-001).

**Implementation approach**:
- Load via `THREE.TextureLoader` from the local path (no CDN).
- Render it as a full-frame backdrop — simplest robust option is a large textured plane far behind the action, sized to fill the camera frustum, or a fixed full-screen background quad rendered first. Keep it static (it does not need to react to physics).
- Position an **invisible physics ground plane** + a thin visible/soft contact-shadow catcher aligned so stacked pieces appear to rest on the depicted wooden floor. Tune camera pitch/height so the foreground floor line matches the image's floor.
- Foreground 3D pieces (blocks/teacups/mini-cars) are lit to roughly match the image's warm key light so they sit in the scene.

**Open tuning items (handled during implementation, not blocking)**: exact camera framing to align the floor plane with the image's perspective; whether to crop/letterbox the image on extreme aspect ratios (FR-014) — likely cover-fit with the tower kept centered.

**Rationale**: The supplied art is far richer than anything procedural and instantly sets the "出鱈目な玩具部屋" mood; using it as a backplate lets all 3D/physics effort go into the tower and the crash. Local file → no CDN hang.
