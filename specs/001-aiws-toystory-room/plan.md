# Implementation Plan: Alice in Wonderland Syndrome — Toy Story Room

**Branch**: `001-aiws-toystory-room` | **Date**: 2026-05-30 | **Spec**: [spec.md](./spec.md)

**Input**: Feature specification from `/specs/001-aiws-toystory-room/spec.md`

## Summary

A single-page WebGL experience that puts the viewer inside a Toy Story–style bedroom and simulates Alice in Wonderland Syndrome: the perceived scale drifts automatically between macropsia (giant) and micropsia (tiny) via a GLSL post-processing distortion, while pointer movement stirs fluid ripples. Toys and trash are alive with idle motion; periodically a human-presence event (sweeping shadow + door light + footstep/door audio) makes everything freeze and the warp return to normal, then life resumes. Built as one static `index.html` using Three.js via CDN import map — runnable locally by just opening the file, and later deployable to Vercel static hosting unchanged.

## Technical Context

**Language/Version**: JavaScript (ES modules), GLSL (WebGL2 via Three.js). No transpilation.

**Primary Dependencies**: Three.js `0.169.0` (CDN, jsDelivr) + its `EffectComposer`/`RenderPass`/`ShaderPass` addons. No npm/build step.

**Storage**: N/A (no persistence; stateless demo).

**Testing**: Manual visual verification in-browser against the spec's acceptance scenarios and success criteria (no automated test framework — appropriate for a single-file art demo).

**Target Platform**: Modern desktop and mobile browsers with WebGL2 (Chrome/Safari/Firefox/Edge current).

**Project Type**: Single-file static web app (client-only).

**Performance Goals**: Smooth visible motion (target ~60 fps desktop, ~30 fps mid-range mobile); first paint + animation start within 5s on broadband (SC-006).

**Constraints**: No build step; everything in one `index.html` servable as a static file from repo root (FR-012, FR-014). Must run by opening locally first (file:// or a simple static server), Vercel later. Audio subject to browser autoplay policy — visuals must stand alone (FR-008a).

**Scale/Scope**: One scene, ~a dozen animate entities, one post-processing distortion pass, one cyclic state machine. Single viewer, no backend.

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

The project constitution (`.specify/memory/constitution.md`) is still the unfilled template — no ratified principles to gate against. No violations possible. **PASS (vacuous).**

Self-imposed guardrails adopted for this feature (in lieu of a constitution):
- **Single-file simplicity**: keep it one `index.html`; do not introduce a build toolchain for a demo of this size (YAGNI).
- **No external assets**: generate textures/geometry procedurally so the file is self-contained and loads instantly.
- **Original homage only**: no licensed Toy Story characters/assets.

## Project Structure

### Documentation (this feature)

```text
specs/001-aiws-toystory-room/
├── plan.md              # This file
├── research.md          # Phase 0 output
├── data-model.md        # Phase 1 output (entity/state model)
├── quickstart.md        # Phase 1 output (how to run locally → Vercel)
├── contracts/
│   └── shader-uniforms.md   # Phase 1 output (the distortion pass "interface")
└── tasks.md             # Phase 2 output (/speckit-tasks)
```

### Source Code (repository root)

```text
index.html        # The entire experience: scene, shaders, state machine, interaction, UI
vercel.json       # Static-hosting config (already present) for the later Vercel step
```

**Structure Decision**: Single-file client-only app. All logic (Three.js scene graph, the AIWS GLSL distortion pass, the perceptual-cycle + freeze state machine, pointer-driven fluid ripples, entity idle/freeze behaviors, and the on-screen indicator) lives inline in `index.html`. This matches FR-012/FR-014 (open-a-page, static-deployable) and the user's local-first workflow. A build system or module split would add ceremony without value at this scope.

## Complexity Tracking

No constitution violations. Not applicable.
