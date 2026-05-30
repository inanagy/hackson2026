# Quickstart: 出鱈目ジェンガ＆クラッシュ

How to vendor dependencies, run locally, and (later) deploy to Vercel. No build step.

## 1. Vendor dependencies (one-time)

Three.js is already vendored at `vendor/three/`. Add cannon-es locally (pure JS, no WASM):

```sh
mkdir -p vendor/cannon-es
# fetch the ESM build of cannon-es 0.20.0 into vendor/ (any of: curl from a package mirror,
# or copy from an npm install's dist/cannon-es.js). The file must be a self-contained ES module.
curl -L -o vendor/cannon-es/cannon-es.js \
  https://unpkg.com/cannon-es@0.20.0/dist/cannon-es.js
```
> The download happens **once at setup time**, not at runtime. After this, the page never touches the network for dependencies — avoiding the CDN load-hang and working offline.

Verify the file is a real ES module (contains `export`), not an HTML error page:
```sh
head -c 200 vendor/cannon-es/cannon-es.js
```

## 2. Confirm assets

The worldview backdrop image must be present:
```sh
ls -la assets/ChatGPT_Image_2026530_12_52_55.png   # 1672×941 PNG
```

## 3. Run locally (serve over http)

ES-module import maps need an http origin (some browsers block modules over `file://`):

```sh
python3 -m http.server 8000
# then open http://localhost:8000/
```

## 4. Manual verification checklist (against spec Success Criteria)

- [ ] **SC-001 / US1**: Within ~15s of load with no input, toys are stacking a tower that visibly grows taller and looks precarious.
- [ ] **SC-002 / US2**: A fast pointer swing through the tower collapses it within a fraction of a second; a click fires the toy cannon and also collapses it.
- [ ] **FR-007**: Slow/gentle pointer movement does NOT collapse the tower (it keeps building).
- [ ] **SC-003**: On collapse, several distinct pieces tumble and scatter independently (real physics, not a canned animation).
- [ ] **SC-005 / US3**: After the crash settles, the scene auto-resets and starts a new tower within a couple of seconds; loops forever, no score/end.
- [ ] **FR-001**: The supplied worldview image shows as the room backdrop; pieces appear to rest on the depicted floor.
- [ ] **FR-010**: The crash is satisfying with NO sound (audio deferred).
- [ ] **FR-014 / SC-006**: Resize + portrait keep the tower framed; motion stays smooth during a multi-piece collapse on a mid-range laptop/phone.
- [ ] **FR-016**: No title/branding text; only a minimal control hint.

## 5. Deploy to Vercel (only after local sign-off)

The repo is already static-host ready (`vercel.json` present, single `index.html` + `vendor/` + `assets/` at root):

```sh
npx vercel        # or push to a Vercel-connected git repo
```
Because deps are vendored locally, the deployed site loads identically to local with no CDN dependency.

> Do NOT deploy until the user approves the local build.
