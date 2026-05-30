# Contract: AIWS Distortion Pass

The single "interface" this client-only app exposes is the uniform contract between the JS state machine and the GLSL post-processing pass. Keeping it stable lets the scene logic and the shader evolve independently.

## Uniforms (JS → GLSL)

| Uniform | Type | Range | Set by | Meaning |
|---------|------|-------|--------|---------|
| `tDiffuse` | sampler2D | — | EffectComposer | rendered scene (auto-bound) |
| `uTime` | float | ≥ 0 | clock | seconds; animates curl noise |
| `uPhase` | float | −1 … +1 | perceptual cycle / freeze | −1 tiny, 0 normal, +1 giant. Forced to 0 during HUMAN_PRESENT (FR-009a) |
| `uIntensity` | float | 0 … 1 | perceptual cycle / freeze | overall distortion strength; 0 during freeze (FR-010) |
| `uResolution` | vec2 | px | resize handler | viewport size for aspect-correct warp |
| `uRipples` | vec3[ N ] | — | pointer handler | per-impulse `(uvx, uvy, strength)`; strength 0 = inactive (FR-002a) |

## Shader behavior contract

1. **Lens warp**: sign of `uPhase` selects barrel (giant) vs pincushion (tiny); magnitude × `uIntensity` sets curvature (FR-003).
2. **Fluid layer**: sum active `uRipples` into UV displacement, modulated by curl noise driven by `uTime`; each ripple fades as its strength decays (FR-002a).
3. **Normal state**: when `uPhase == 0` and `uIntensity == 0` and no active ripples, output MUST equal the source image (so the freeze looks like an ordinary room — FR-009a).
4. **Chromatic aberration / vignette**: optional stylistic layers scaled by `uIntensity`; MUST vanish at `uIntensity == 0`.

## Invariants

- The pass never reads or writes outside `[0,1]` UV destructively (clamp/mirror at edges).
- All effects are continuous in `uPhase`/`uIntensity` so freeze/revive lerps read smoothly (FR-004).
- Disabling everything (all zero) is a valid, identity state — this is what the freeze relies on.
