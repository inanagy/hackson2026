# Phase 0 Research: AIWS Toy Story Room

No open `NEEDS CLARIFICATION` markers remained after `/speckit-clarify`. This document records the key technical decisions and the reasoning behind them.

## Decision 1: Delivery as a single static `index.html` via CDN import map

- **Decision**: Ship the whole experience in one `index.html`, importing Three.js + addons from jsDelivr through an ES-module import map. No bundler, no npm install.
- **Rationale**: FR-012/FR-014 require open-a-page + static hosting; the user wants to run locally first. A single file is the fastest path to "see it working," and Vercel serves it unchanged.
- **Alternatives considered**: Vite/npm project (rejected — build ceremony unjustified at this scope); vendoring Three.js locally (rejected for now — CDN is simpler; revisit only if offline/file:// CORS becomes an issue, see Decision 6).

## Decision 2: AIWS distortion as a single GLSL post-processing pass

- **Decision**: One `ShaderPass` after the scene render. Macropsia uses barrel (fisheye) lens warp; micropsia uses pincushion. A signed `uPhase` (−1 tiny … 0 normal … +1 giant) drives the lens sign; `uIntensity` scales overall strength.
- **Rationale**: A single signed parameter makes the giant↔tiny continuum trivial to animate and to force back to 0 during a freeze (FR-009a). Matches the prototype already built.
- **Alternatives considered**: Per-state separate shaders (rejected — harder to cross-fade); geometry-level scaling of the room (rejected — less dreamlike, more expensive, doesn't read as *perceptual* distortion).

## Decision 3: Camera reinforcement of scale (FOV + height)

- **Decision**: Couple camera FOV and eye height to `uPhase`: giant → narrower FOV + higher eye; tiny → wider FOV + lower eye.
- **Rationale**: Lens warp alone is screen-space; pairing it with FOV/height gives a bodily sense of being big/small (FR-003).
- **Alternatives considered**: Distortion-only (rejected — weaker illusion).

## Decision 4: Pointer-driven fluid ripples (three-fluid-fx feel) without a full fluid sim

- **Decision**: Maintain a small set of decaying "ripple" impulses seeded at pointer/touch positions; feed them into the distortion shader as additional UV displacement (curl-noise modulated), each fading over ~1–2s.
- **Rationale**: FR-002a wants the reference's tactile feel, but a full Navier–Stokes GPU fluid sim is overkill for a one-file demo. Decaying splat impulses + curl noise reproduce the momentum-smear look cheaply.
- **Alternatives considered**: Full ping-pong FBO fluid solver (rejected for v1 — large code, perf risk on mobile); no interaction (rejected — violates FR-002a).

## Decision 5: Freeze via a global state machine + per-entity behavior flag

- **Decision**: A top-level state machine cycles `PERCEPTUAL` (giant/normal/tiny drift) and interrupts with `HUMAN_PRESENT`. Each animate entity reads a global `alive` flag; when false it lerps to its stored rest pose and stops. On freeze, `uPhase`/`uIntensity` also lerp to 0 (FR-009a/FR-010); on release they resume (FR-011).
- **Rationale**: Centralizing the "human present" signal keeps toys, trash, warp, shadow, door-light, and audio in sync. Storing each entity's rest pose lets the freeze look like a real "play dead" snap-and-hold.
- **Alternatives considered**: Per-entity timers (rejected — desync risk, FR-009 wants near-instant simultaneous freeze).

## Decision 6: Human-presence cues — shadow + door light + audio, visuals authoritative

- **Decision**: Implement all three (FR-008): a large dark plane / shadow that sweeps across the scene, a brightening "door light" wash from one side, and an optional footstep/door audio clip. Audio is gated behind the first user gesture; if never granted, visuals alone carry the beat (FR-008a).
- **Rationale**: Browser autoplay policy blocks unprompted sound; spec already designates visuals as sufficient.
- **Alternatives considered**: Audio-first (rejected — unreliable autoplay).

## Decision 7: Local-run method (file:// vs static server)

- **Decision**: Recommend running via a tiny static server (`python3 -m http.server`) rather than `file://`, because ES-module import maps + cross-origin CDN fetches are more reliable over `http://`. Document both in quickstart.
- **Rationale**: Some browsers restrict module loading and CORS under `file://`. A one-line Python server avoids surprises and mirrors how Vercel will serve it.
- **Alternatives considered**: Pure `file://` (works in some browsers but flaky — documented as fallback).

## Open risks / follow-ups

- Mobile GPU performance under full-screen post-processing — mitigate with capped pixel ratio (already in prototype); revisit if SC-006 fails on a real phone.
- Motion-intensity/nausea option is **deferred** (FR-015) per clarification; note for a future iteration.
