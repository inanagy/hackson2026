# Implementation Plan: 出鱈目ジェンガ＆クラッシュ — Toy Story Room

**Branch**: `002-jenga-crash-toystory` | **Date**: 2026-05-30 | **Spec**: [spec.md](./spec.md)

**Input**: Feature specification from `/specs/002-jenga-crash-toystory/spec.md`

## Summary

A single-page WebGL experience set in a Toy Story–style bedroom whose entire loop is **build tension → release catharsis**: while the viewer holds the pointer still, animate toys auto-stack blocks/teacups/mini-cars into a precarious central tower; when the viewer swings the pointer fast (an invisible "wrecking ball") or clicks to fire a toy cannon, the tower collapses in a real rigid-body physics simulation; then the scene self-resets and rebuilds, looping forever with no score. A **Ctrl/⌘+click** triggers **Chaos Mode** — gravity lerps to ~0 and the toys swirl in a zero-gravity parade while a hidden **Treasure** appears among them, vanishing the moment the viewer makes a deliberate pointer move and gravity returns (crashing is disabled while gravity is off). A decorative **wandering human** roams near the tower and averts its gaze while Chaos Mode is active. **Audio is implemented**: two ambient loops crossfade by pointer state ("Chaotic Toy Parade" active / "When Humans Look Away" idle), started on first gesture, plus an optional impact-scaled crash SFX (`assets/crash.mp3`, silently skipped if absent). Built as one static `index.html` using **Three.js (render) + cannon-es (physics), both vendored locally** under `vendor/` — no CDN, no build step — so it runs by opening the file and deploys to Vercel static unchanged. Desktop only.

## Technical Context

**Language/Version**: JavaScript (ES modules), WebGL2 via Three.js. No transpilation, no bundler.

**Primary Dependencies**:
- **Three.js `0.169.0`** — rendering, scene graph, `Raycaster` for cannon-aiming/picking. Already vendored at `vendor/three/three.module.js` (+ `addons/`).
- **cannon-es `0.20.0`** — rigid-body physics (gravity, collisions, the stack-and-collapse). Pure JavaScript, **no WASM**. To be vendored at `vendor/cannon-es/cannon-es.js`.
- Both resolved via an in-page **import map** pointing at the local `vendor/` files (no network fetch at runtime).

**Storage**: N/A (stateless; no persistence).

**Testing**: Manual visual verification in-browser against the spec's acceptance scenarios and success criteria (single-file art demo; no automated test framework).

**Target Platform**: Modern **desktop** browsers with WebGL2 + ES-module import maps (Chrome/Safari/Firefox/Edge current). Desktop only — mobile/touch/portrait is out of scope.

**Project Type**: Single-file static web app (client-only).

**Performance Goals**: ~60 fps on a current mid-range laptop, sustained even during a multi-piece collapse and during the zero-g chaos parade (SC-006). First paint + animation start within 5s (loading from local vendor files is effectively instant). (Desktop only — no mobile target.)

**Constraints**:
- No build step; everything in one `index.html` servable as a static file from repo root (FR-013, FR-015).
- **Dependencies vendored locally, not CDN** — CDN loading was observed to hang; local files load reliably and cannon-es being pure-JS avoids any WASM fetch (the reason cannon-es was chosen over Rapier).
- Physics body count must stay bounded so collapse stays smooth on mid-range hardware (cap pieces; sleep settled bodies; pool/recycle on reset).
- Audio subject to browser autoplay policy — ambient loops + crash SFX start only on the first user gesture; the visual experience must stand alone (FR-010). The crash SFX file (`assets/crash.mp3`) may be absent and is then silently skipped.
- Chaos Mode is triggered by Ctrl/⌘+click (the sole trigger; pointer stillness does not enter chaos) and toggles world gravity (lerp to ~0 on entry, back to 1 on the deliberate pointer move that exits it); the crash trigger is disabled while gravity is off so chaos cannot be smashed (FR-019). The Treasure is only active during chaos (FR-020).

**Scale/Scope**: One room scene, one central tower of ~15–25 rigid pieces at peak, a handful of builder toys, one decorative wandering human, one hidden treasure, one physics world with a toggleable gravity (normal ↔ ~0 Chaos Mode), one build→crash→reset state machine plus a Ctrl/⌘+click-triggered chaos override, two crossfaded ambient audio loops + one optional crash SFX, pointer-velocity + click + Ctrl/⌘+click input. Single viewer, no backend, desktop only.

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

The project constitution (`.specify/memory/constitution.md`) is still the unfilled template — no ratified principles to gate against. No violations possible. **PASS (vacuous).**

Self-imposed guardrails adopted for this feature (in lieu of a constitution):
- **Single-file simplicity**: keep it one `index.html`; no build toolchain for a demo this size (YAGNI).
- **Local vendored deps only**: no runtime CDN/network dependency; Three.js + cannon-es load from `vendor/`.
- **Procedural / self-contained assets**: generate room textures and toy geometry in code so the page loads instantly; an optional background image may be added later if the user supplies one (see research).
- **Original homage only**: no licensed Toy Story characters/assets.
- **Fresh build**: feature 002 is implemented from scratch; the feature-001 `index.html` is not reused.

## Project Structure

### Documentation (this feature)

```text
specs/002-jenga-crash-toystory/
├── plan.md              # This file
├── research.md          # Phase 0 output (stack/physics decisions)
├── data-model.md        # Phase 1 output (entities + state machine)
├── quickstart.md        # Phase 1 output (vendor + run locally → Vercel)
├── contracts/
│   └── physics-interface.md  # Phase 1 output (the physics/render sync "interface")
└── tasks.md             # Phase 2 output (/speckit-tasks)
```

### Source Code (repository root)

```text
index.html              # The entire experience: scene, physics world, builder AI, input, crash, loop, UI
vendor/
├── three/
│   ├── three.module.js
│   └── addons/…         # (present; Raycaster is in core, addons optional here)
└── cannon-es/
    └── cannon-es.js     # ESM build, vendored locally (Phase 1/setup task)
vercel.json             # Static-hosting config (already present)
```

**Structure Decision**: Single-file client-only app. All logic — Three.js scene graph, the cannon-es physics world, the builder-toy stacking behavior, the pointer-velocity "wrecking ball" + click "cannon" input, the rigid-body crash, the build→crash→reset state machine, and the minimal control-hint UI — lives inline in `index.html`. Dependencies are local files under `vendor/` referenced by an import map. This matches FR-013/FR-015 (open-a-page, static-deployable) and the user's local-first, no-CDN-hang requirement. A build system or module split would add ceremony without value at this scope.

## Complexity Tracking

No constitution violations. Not applicable.
