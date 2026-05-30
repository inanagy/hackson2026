# Contract: Physics ↔ Render ↔ Input Interface

This is the internal "interface" of the single-page app — the shapes and seams that the scene, physics world, and input handlers agree on. Keeping these stable lets each region (build, crash, reset, input, render) be implemented and tuned independently.

## Module imports (import map → local vendor)

```html
<script type="importmap">
{ "imports": {
    "three": "./vendor/three/three.module.js",
    "three/addons/": "./vendor/three/addons/",
    "cannon-es": "./vendor/cannon-es/cannon-es.js"
} }
</script>
```
Runtime MUST NOT fetch these from any CDN. Files MUST exist locally before load.

## Piece record

```js
// one per stackable toy; mesh mirrors body every frame
{ mesh: THREE.Mesh, body: CANNON.Body, kind: 'block'|'teacup'|'minicar', home: THREE.Vector3, inTower: boolean }
```
- `body.mass > 0` (dynamic). Shape MUST match mesh extents (Box ↔ BoxGeometry, Cylinder ↔ Cylinder/teacup).
- Invariant each frame after `world.step`: `mesh.position.copy(body.position)`, `mesh.quaternion.copy(body.quaternion)`.

## Physics world stepping

```js
const FIXED_DT = 1/60;
function stepPhysics(dt) {
  // accumulator pattern; clamp dt to avoid spiral-of-death / tab-hidden blowup
  world.step(FIXED_DT, Math.min(dt, 0.1), 3);
}
```
- Gravity: `world.gravity.set(0, -G, 0)` (G ~ 9.82, may be scaled for toy feel).
- Settled bodies MAY sleep (`body.allowSleep = true`).

## State machine surface

```js
const Phase = { BUILDING, READY, CRASHING, SETTLING, RESET };
let phase = Phase.BUILDING;
// timers: buildTimer, idleTime, settleTimer
function setPhase(next) { /* single transition point; resets relevant timers */ }
```
Transitions per [data-model.md](../data-model.md). A crash trigger is accepted in BUILDING and READY (when ≥ MIN_PIECES_TO_CRASH pieces exist), else ignored.

## Input contract

```js
// pointer velocity (wrecking cursor) — FR-005, FR-007
pointer = { x, y, prevX, prevY, speed /* px/s */ };
const SWING_THRESHOLD = /* px/s */;        // below → no crash; tower keeps building
const wreckBall = { body: CANNON.Body /* sphere */, active: bool };
// active = pointer.speed > SWING_THRESHOLD; impulse imparted on contact ∝ ball velocity

// toy cannon — FR-006
function fireCannon(screenX, screenY) {
  // Raycaster from camera through pointer → aim dir; spawn pooled projectile body w/ velocity
}
window.addEventListener('pointermove', updatePointerVelocity);
window.addEventListener('pointerdown', e => fireCannon(e.clientX, e.clientY)); // click/tap
```
- Idle accounting: `idleTime` accrues only while `pointer.speed ≈ 0`. A fast swing both triggers a crash (if tower exists) and resets idle.

## Crash trigger + reserved audio hook

```js
function triggerCrash(by /* 'cursor'|'cannon' */, impulse, point) {
  onImpact(impulse, point);   // RESERVED no-op now; audio added later (FR-010 / FE-003)
  setPhase(Phase.CRASHING);
}
function onImpact(impulse, point) { /* intentionally empty this iteration */ }
```

## Backdrop contract

```js
// FR-001 — supplied worldview image as far backdrop (local, no CDN)
new THREE.TextureLoader().load('./assets/ChatGPT_Image_2026530_12_52_55.png', tex => { /* backdrop plane / bg quad */ });
```
- Backdrop is static (no physics). An invisible `mass:0` ground plane + soft contact-shadow catcher aligns foreground pieces to the depicted floor.

## Reset / pool contract

```js
const PIECE_POOL_MAX = /* e.g. 25 */;       // hard cap on dynamic bodies (perf / SC-006)
function recyclePieces() { /* reset body pos→home, zero velocity, sleep; clear tower */ }
```
- No score/timer/win-lose state is ever created (FR-012).

## Resize contract

On `resize`: update camera aspect + `renderer.setSize`; keep the tower centered and the backdrop cover-fit so the central action is never pushed off-screen (FR-014).
