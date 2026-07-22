---
name: scroll-world-realtime
description: Build a scroll-scrubbed camera flight through any subject as one self-contained HTML file. Scrolling drives a continuous camera along a path through a sequence of scenes, with a copy card, rail dot, and HUD per scene. Scenes can be built geometry, real terrain, satellite city slabs, or a full globe with dives; the engine is the same. No API keys, no video generation, no runtime fetches. Use when the user wants a scroll cinematic, a fly-through landing page, a scroll world, or a scroll-driven data story over real places or data.
---

# scroll-world-realtime

One HTML file where scroll drives a camera. The visitor scrolls and the camera flies a continuous path through N scenes with no cuts. Each scene gets a copy card with real stats, a dot on a navigation rail, and optionally a HUD readout that tracks the flight. The original scroll-world skill renders the flight as AI-generated video scrubbed by scroll; this version renders it live in three.js, which removes the asset generation, the credit spend, and the seam problem entirely, because there are no seams in one continuous real-time camera.

## The invariant frame

Every build is the same five pieces. Only the scene content changes.

1. A tall scroll track (`height: N * 200vh` or more) over a fixed full-viewport canvas.
2. A camera path: one `CatmullRomCurve3` through hand-placed waypoints (or a parametric path like orbit-dive-orbit), sampled at `t = scrollY / maxScroll`.
3. Smoothed camera, raw UI. The camera follows `curT += (target - curT) * 0.08` for weight; every UI element (copy bands, rail dots, counters, HUD) reads the RAW scroll fraction. The camera may lag, the UI must never.
4. Per-scene copy cards: eyebrow, headline, one paragraph, stat blocks with real numbers. A card is on when the raw fraction is inside its band. Rail dots scroll you to band centers on click.
5. A loader that counts inlined assets, a scroll hint that disappears on first scroll, and a footer citing every data and imagery source.

## Scene content, pick per subject

- Built geometry: low-poly scenes from boxes, cylinders, cones on floating islands. Zero data fetches. Good for brands and stories.
- Real terrain: AWS Terrarium elevation tiles (keyless) decoded to a displaced plane, textured with stitched Esri World Imagery. True vertical scale sells it.
- Satellite slabs: one stitched Esri patch per place as a flat textured plane, camera does a low pass over each. Simplest hyperreal option.
- Globe with dives: NASA Blue Marble sphere, camera dives exponentially from orbit to about 27 km per point, hi-res patch fades in at the bottom. The Google Earth feel comes from the exponential altitude curve, never linear.

## Keyless asset endpoints

```
elevation  https://s3.amazonaws.com/elevation-tiles-prod/terrarium/{z}/{x}/{y}.png
imagery    https://server.arcgisonline.com/ArcGIS/rest/services/World_Imagery/MapServer/tile/{z}/{y}/{x}
globe      https://eoimages.gsfc.nasa.gov/images/imagerecords/73000/73909/world.topo.bathy.200412.3x5400x2700.jpg
```

Terrarium decode: `elev_m = R*256 + G + B/256 - 32768`. Fetch tiles with a thread pool and always run a second pass re-requesting any file under 500 bytes; a few tiles fail per batch. Stitch, resize, encode jpg quality 70 to 80, inline as base64. Base64 adds 37% to raw bytes; keep the final HTML under about 6 MB.

## Engine rules learned the hard way

- Unlit materials for anything textured with real imagery. Sunlight is baked into satellite photos, and three r155+ physically correct lighting renders `MeshLambertMaterial` plus ambient dark. Use `MeshBasicMaterial`.
- Scale discipline: pick a unit (1 unit = 100 m for terrain, R = 100 for a globe so 1 unit = 63.7 km) and derive every size from it. Real proportions are what make it read as real.
- Globe dives need `logarithmicDepthBuffer: true` plus a dynamic near plane (`alt * 0.05`) or the surface z-fights at the bottom.
- Terrain waypoints are placed as height-above-ground: sample the heightmap at (x, z) and add clearance, never hardcode y.
- Stats never drive geometry they do not describe. Population fills cards and counters; it sizes nothing.
- Phone hardening: `100dvh`, safe-area insets, ignore height-only resizes on touch (the URL bar), reduced-motion drops the smoothing and the hint animation.
- No em or en dashes anywhere, including copy strings inside the JS.

## Verify headless, trust numbers not eyes

Headless chromium runs software WebGL at a few fps, so the smoothed camera lags scroll badly and screenshots taken too early look broken when nothing is wrong. Two rules:

- Assert UI state (card class, dot class, counter text) immediately after scrolling; it reads the raw fraction and must be correct at any frame rate.
- Judge framing by pixel statistics after a 15 to 30 second settle: center-region mean above 90 and std above 30 means a textured scene fills the frame; a dark or flat read means the camera had not arrived. Also verify any computed total (a running population counter must end at the exact sum of the inputs).

## Ship checklist

- [ ] One file, all assets inlined, only external reference is the three.js CDN script tag
- [ ] Every copy band and rail dot asserted in headless
- [ ] Sources cited in the footer
- [ ] `grep -c` for em and en dashes returns 0 (append `; true`, it exits 1 on zero matches)
- [ ] Loads on a phone viewport with nothing overlapping
