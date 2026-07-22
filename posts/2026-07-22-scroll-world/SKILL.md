---
name: scroll-flight
description: Build the scroll-scrubbed flight UX for any subject as one self-contained HTML file. Scrolling drives a continuous camera along a path through an ordered sequence of scenes with no cuts; each scene has a copy card, a rail dot, and optional HUD readouts. The scene content can be anything renderable. Use when the user wants a scroll cinematic, a fly-through page, a scroll world, or a scroll-driven story in any visual style.
---

# scroll-flight

The UX: the page is one long scroll, and scrolling does not move the page, it moves a camera. The visitor flies through an ordered sequence of scenes as one continuous shot. Copy cards surface per scene, a rail shows where you are, and the whole thing is a single HTML file.

This skill is the frame. It says nothing about what the scenes contain; that is a per-project decision, and any renderer works behind it (three.js, canvas 2D, CSS transforms, pre-rendered frames).

## The five pieces

Every build is exactly these, wired the same way.

1. **Scroll track.** An empty div sets the page height, roughly `N * 200vh` for N scenes. More height per scene means slower, heavier flight. The visible content is a fixed, full-viewport stage behind it.

2. **One path.** A single continuous camera path through the scenes, parameterized by `t = scrollY / (scrollHeight - innerHeight)` in [0,1]. Hand-place waypoints so each scene gets an approach, a hold, and an exit. The hold is where the visitor reads; give it 25 to 40 percent of the scene's scroll budget. One continuous path is the whole point: no cuts means no seams, and the transitions between scenes are as designed as the scenes themselves.

3. **Smoothed camera, raw UI.** The camera follows the scroll target through a lerp (`cur += (target - cur) * 0.08`) so the flight has weight. Every UI element (cards, dots, counters, HUD) reads the RAW scroll fraction instead. The camera is allowed to lag; the UI is never allowed to, because UI driven by the smoothed value breaks at low frame rates and feels unresponsive at high ones.

4. **Copy cards and rail.** One card per scene: eyebrow, headline, one short paragraph, 0 to 3 stat blocks. A card is on while the raw fraction sits inside that scene's hold band; it fades with a 0.4s transition and a small translateY. The rail is one dot per scene, active by band, click scrolls to the band center. Cards live in fixed-position DOM, not inside the renderer, so text stays crisp and selectable.

5. **Furniture.** A loader that counts assets and fades out when ready. A "scroll" hint that disappears on first scroll. Optional HUD readouts that track the flight continuously (progress, a running total, whatever the subject offers). A footer citing sources if the content carries data.

## Feel

- Ease the sub-parameters. Raw linear t inside a scene reads mechanical; shape approach and exit with smoothstep.
- If the camera moves through orders of magnitude of distance, interpolate that dimension in log space. Linear zoom always reads wrong.
- The camera should never reverse direction along the path. A pull-back that retraces the approach reads as scrubbing a video backwards, which kills the flight illusion.
- Direction changes get their own travel time while the camera is far from any scene. Turning while close to content reads as disorientation.
- One idle animation per scene at most, something that moves without scroll, so a stopped page still feels alive.

## Hardening

- `100dvh` for the stage, safe-area insets for all fixed UI, `viewport-fit=cover`.
- On touch devices, ignore resizes where only the height changed; that is the URL bar, and re-laying out on it makes the page jump. Keep the width-change and orientation paths.
- `prefers-reduced-motion`: drop the camera lerp (snap to target), kill the hint animation and card transitions. The page must still fully work.
- Keyboard: rail dots are real buttons with focus styles and labels.
- The file is self-contained: assets inlined, no runtime fetches beyond at most one CDN script tag.

## Verify headless

Headless browsers run rendering slowly, so the smoothed camera lags scroll and early screenshots look broken when nothing is wrong. Two rules:

- Assert UI state (card class, dot class, HUD text) immediately after each scripted scroll. It reads the raw fraction, so it must be correct at any frame rate. Walk every band; all of them fire or the build fails.
- Judge visuals only after a long settle (15 to 30 seconds), and prefer pixel statistics of the frame center over eyeballing when the environment cannot display images.

Any number the page computes from its inputs (a running total, a final count) gets checked against the independently computed answer.

## Ship checklist

- [ ] One file, opens from disk with no network beyond the CDN tag
- [ ] Every card band and rail dot asserted in headless
- [ ] Reduced motion, phone viewport, and URL-bar scroll all clean
- [ ] Camera never reverses along the path
- [ ] Sources cited if the content carries data
