# Quickstart: AIWS Toy Story Room

## Run locally (do this first)

The experience is a single `index.html` at the repo root. Running it through a tiny static server is the most reliable way (ES-module import maps + CDN fetches behave best over `http://`).

```bash
cd /Users/toueiaoi/Git/hackson2026
python3 -m http.server 8000
# then open http://localhost:8000/ in your browser
```

Fallback: you can also just double-click `index.html` to open it via `file://`, but some browsers restrict module loading there — prefer the server above.

### What you should see (maps to spec acceptance scenarios)

- The view sits inside a Toy Story–style bedroom. (FR-001)
- Your perceived scale drifts on its own: giant → normal → tiny → normal, looping, with the room bending accordingly and an on-screen indicator naming the state. (US1 / FR-002, FR-003, FR-005)
- Moving the mouse / dragging on touch stirs fluid ripples at the pointer. (FR-002a)
- Toys and at least one piece of trash move on their own. (US2 / FR-006, FR-007)
- Every so often a human approaches — a shadow sweeps, light spills from a door, (audio if enabled) — and everything freezes into a plain, undistorted room, then comes back to life. (US3 / FR-008…FR-011)

## Deploy to Vercel (only after local is approved)

The repo already has `vercel.json` for static hosting and `index.html` at root.

```bash
npx vercel        # link + deploy preview
npx vercel --prod # promote to production
```

Or connect the GitHub repo in the Vercel dashboard for auto-deploy on push.

> Per project preference: get it working locally and approved **before** going to Vercel.
