# Sandtray

A real-time sand tray for phones and tablets, in the browser. Draw in it with your fingers, tilt or shake the device, and the sand slides, piles up and avalanches like the real thing.

**Open it:** https://jaredmullaney-ui.github.io/Sand/ (on iPhone, tap the screen once and allow motion access when asked).

## What you can do

| Tool | What it does |
| --- | --- |
| Finger | Drag to dig trenches. Sand is pushed aside into berms and collapses at its angle of repose. Several fingers work at once. |
| Rake | Combs five parallel grooves, zen-garden style. |
| Smooth | Loosens the sand under your finger so it settles flat. |
| Pour | Hold to pour a stream of sand that builds a cone. |
| Castle | Tap Castle, then tap the sand to build a classic sand castle. It is packed from the tray's own sand, so the area around it gets shallower. Swiping through it, tilting steeply or shaking knocks it down into loose sand again. When the sand runs low, shake to free some up. |
| Tilt | Uses the phone's motion sensors. Small tilts do nothing (static friction); steep tilts make the sand avalanche and slide downhill with momentum; a quick shake levels it. |

On a computer, drag with the mouse to draw, use the arrow keys or the on-screen tilt pad to tilt the tray, and press Space to shake it.

## How it works

Everything runs on the GPU in a single `index.html` with WebGL 2 and no dependencies.

- **Physics:** a heightfield of sand depth with velocities on a staggered grid. Each step applies gravity from the device sensors, a surface-pressure term, and Coulomb friction (static and dynamic, angle of repose about 33°). Only a thin surface layer flows, which gives layered avalanches. Face fluxes are exactly mass-conserving, so sand is never created or lost except when you pour.
- **Touch:** each finger is a swept capsule that locks the sand beneath it and bulldozes the rest outward and ahead of the motion.
- **Grains:** a procedurally generated 1024² tile of individual mineral grains (quartz, feldspar, dark heavy minerals). Grain coordinates are advected with the flow, so grains travel with the sand when it slides.
- **Lighting:** the lamp is fixed in the room, so as you tilt the device the shadows swing and quartz glints twinkle. Soft heightfield shadows, multi-scale ambient occlusion, per-grain contact shadows, ACES tone mapping.
- **Castles:** each column also stores how much of it is packed sand. Packed sand doesn't flow; it gives way where a finger cuts it, where it loses support, and on its downhill faces when the tray is shaken or tilted steeply, and the freed sand becomes ordinary loose sand.
- **Sound and haptics:** a granular crunch/hiss synthesized with Web Audio, driven by the measured sand flow; vibration ticks on Android.
- **Adaptive quality:** render resolution and physics substeps drop automatically if the device can't hold the frame rate.

Add `#debug` to the URL for an FPS / mass overlay.
