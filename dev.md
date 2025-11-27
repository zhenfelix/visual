**Summary of work (2025-11-27)**

This document summarizes the key concepts reviewed and the code changes made in `index.html` today. It is intended as a developer-facing log to help future edits, tuning, and performance work.

**Key Concepts Learned**
- **CSS animations & timing:** `@keyframes` defines named animation sequences; `animation-delay` staggers starts; shorthand `animation:` controls name, duration, timing, delay, iteration and fill-mode. Use `transform` and `opacity` for best performance.
- **Conic gradients for effects:** `conic-gradient(...)` is an easy way to create a rotating radar wedge; rotating the element gives a sweeping beam when combined with `@keyframes` that rotates the element.
- **Three.js fundamentals:** scene, camera, renderer, meshes, materials, BufferGeometry, and the animation loop (`requestAnimationFrame`). Use `BufferGeometry` attributes and set `needsUpdate = true` when mutating vertex buffers.
- **Direct vertex manipulation:** modifying `geometry.attributes.position.array` is efficient when done in-place. Remember to keep a snapshot of original positions if you need reference values.
- **Particle systems (Points):** `THREE.Points` + `PointsMaterial` is a cheap way to render many stars. Per-particle updates can be done by mutating the `position` attribute array.
- **Performance tradeoffs:** high `gridSegments`, high particle counts, `antialias: true`, and large `devicePixelRatio` increase GPU/CPU workload. Prefer smaller segment counts, capped DPR, and reuse arrays to reduce GC pressure.

**Developments / Code Changes Made**
- File modified: `index.html` (multiple edits in the Three.js section)
  - Replaced uniform star generation with polar sampling biased to the center so star density is higher near the black hole:
    - Parameter: `radialBias` (default 2.5) and `maxStarRadius` (default 600).
  - Created per-star state arrays: `starAngles`, `starRadii`, `starYs` to hold polar coordinates and vertical offsets.
  - Switched animation loop to use real delta time (`performance.now()`), making motion frame-rate independent.
  - Implemented per-star orbital motion with angular speed that increases when radius decreases:
    - `angularSpeed = baseSpeed + clamp(30.0 / safeR, 0, 6.0)` (tunable constants).
  - Updated star positions each frame by computing x,z from angle and radius and computing Y from the same gravity+wave formula used for the grid so stars vertically follow the gravity well.
  - Added `starsGeo.attributes.position.needsUpdate = true` after per-frame edits.

**Where to tune (parameters)**
- `radialBias` (in star generation): larger => stronger concentration near center. Try values 1.0 (uniform) → 4.0 (tight). 
- `maxStarRadius`: outer extent of starfield.
- `starCount`: total points to render (2000 by default).
- `baseSpeed`, `invContribution` constants inside per-star update: control orbital speed and how strongly proximity affects speed.
- `safeDist` minimums used for gravity and star orbital math to avoid singularities (currently 5.0 or 10.0). Adjust carefully.
- `gridSegments`: reduces/increases vertex detail of the gravity grid (currently high in file; lower improves performance).
- `renderer.setPixelRatio(...)`: consider capping at `Math.min(window.devicePixelRatio, 2)` to balance quality & perf.

**Next recommended steps**
- Enable the accretion disk rendering by uncommenting `scene.add(disk);` so the rotating ring appears. Consider using an emissive material or post-process bloom for glow.
- Add `renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));` and optionally reduce `gridSegments` for smoother performance on low-end devices.
- Optionally color/scale stars based on radius to visually emphasize stars closer to the hole.
- If more realistic lensing or event-horizon effects are required, add a post-processing pass or custom shader.

**How to preview**
- Serve the folder and open `index.html` in a browser. Example (from repo root):

```
python3 -m http.server 8000
# then open http://localhost:8000/index.html
```

**Quick file pointers**
- `index.html`: main file edited — Three.js scene, star generation, and animation loop.

If you want, I can create a small commit with these changes and add a short demo README with tuning knobs exposed as top-level constants in `index.html`.
