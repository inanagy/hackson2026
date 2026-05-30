# Phase 1 Data Model: AIWS Toy Story Room

This is a real-time scene, not a persisted data app. "Entities" here are runtime objects and state, not database tables.

## Global State Machine

```
            ┌─────────────────────────────────────────────┐
            │                 PERCEPTUAL                    │
            │  (alive = true)                               │
            │  uPhase drifts: 0 → +1 (giant) → 0 →          │
            │                 −1 (tiny) → 0 → … (loop)      │
            └───────────────┬───────────────▲──────────────┘
        human-presence timer│               │presence cue subsides
            fires            ▼               │
            ┌─────────────────────────────────────────────┐
            │               HUMAN_PRESENT                   │
            │  (alive = false)                              │
            │  uPhase, uIntensity → 0 (normal, undistorted) │
            │  entities lerp to rest pose and hold          │
            │  shadow sweep + door light + (audio) play     │
            └─────────────────────────────────────────────┘
```

- **alive: boolean** — global flag. `true` in PERCEPTUAL, `false` in HUMAN_PRESENT. Every animate entity reads it each frame.
- **presenceTimer** — schedules the next HUMAN_PRESENT; randomized interval (vary by frame count / index, since `Math.random` semantics are fine in the browser at runtime).
- Transition timings: freeze snap is fast (≤ ~0.5s per SC-004); revive is gentle (~1–2s per SC-005).

## Perceptual State (drives the distortion)

| Field | Range | Meaning |
|-------|-------|---------|
| `uPhase` | −1 … +1 | −1 micropsia (tiny), 0 normal, +1 macropsia (giant) |
| `uIntensity` | 0 … 1 | overall distortion strength; 0 during freeze |
| `uTime` | seconds | drives curl-noise animation |
| camera `fov` | derived | `65 − uPhase·22` (narrow when giant) |
| camera eye height | derived | `base + uPhase·1.0` (high when giant) |

The cycle (giant → normal → tiny → normal) loops on a fixed schedule with eased (sine) envelopes between stages (FR-002, FR-004).

## Ripple (pointer-driven fluid impulse)

A small fixed-size pool (e.g. 8) of decaying impulses (FR-002a).

| Field | Meaning |
|-------|---------|
| `pos` (vec2, UV) | where the pointer was |
| `strength` | current magnitude, decays each frame |
| `age` | seconds since spawn; impulse retired after ~1–2s |

Pointer/touch move → seed/refresh an impulse at that UV. The shader sums active impulses into extra UV displacement modulated by curl noise.

## Animate Entity (base concept)

Two concrete kinds — **Toy** and **Trash** — share this shape:

| Field | Meaning |
|-------|---------|
| `object3D` | the mesh/group in the scene |
| `restPose` | stored position/rotation captured at init (the "lifeless" pose) |
| `idle(t)` | function producing live motion when `alive` |
| `kind` | `"toy"` \| `"trash"` (trash must be visibly junk, FR-007) |

Behavior per frame:
- `alive === true` → apply `idle(t)` (breathing sway, hop, small walk).
- `alive === false` → lerp object3D toward `restPose` quickly, then hold (play dead).

### Toy instances (FR-006)
- Building blocks (several), a ball, a cowboy-ish figure, a spaceman-ish figure (original homages, Assumptions).

### Trash instances (FR-007)
- At least one of: crumpled paper scrap, a can, a candy wrapper — given exaggerated, slightly "wrong" idle motion (twitch/hop) so its aliveness reads clearly.

## Human Presence Cue (visual + audio)

| Field | Meaning |
|-------|---------|
| `shadow` | a large dark plane/gradient that sweeps across the room |
| `doorLight` | a brightening wash from one side (door opening) |
| `audio` | optional footstep/door clip, gated behind first user gesture (FR-008a) |
| `progress` | 0→1 over the event; drives sweep position, light, and audio timing |

Visual cues are authoritative; audio is enhancement only.

## On-Screen Indicator (FR-005)

Reads `uPhase` and the state machine to display the current perceptual label (e.g. 巨人症 / normal / 小人症) and a scale bar. During HUMAN_PRESENT it reflects the return-to-normal (and may show a "freeze" hint).
