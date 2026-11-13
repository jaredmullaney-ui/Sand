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
| Garden | Opens a small menu of zen garden items: river stones, a stone lantern, a moss mound and a cherry blossom tree. Drag one onto the sand (or tap it to drop it in an open spot). Setting an item down leaves no marks. Drag a placed item to move it and it plows a furrow through the sand like a real object. Items can't be resized. Raked and sliding sand piles up against them and flows around them. Tilt the tray steeply or shake it hard and items can slide off the edge; they are gone until you add them again. **Clear** in the menu removes them all. |
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

- **Garden items:** each item is generated once on the GPU as a top-down relief (albedo, height, material) and lit live by the same lamp as the sand, so highlights and long shadows swing as you tilt. A coarse map of each item's solid spans lets a lantern roof or a tree canopy shade the sand while light still passes beneath it, and the item's base acts as a wall in the sand physics.

Add `#debug` to the URL for an FPS / mass overlay.

## Analytics and the Clear A/B test

The Clear button runs an A/B test. Each device is assigned once, at random, to one of two variants:

- `undo`: Clear removes the items at once and shows an **Undo** button for 6 seconds.
- `confirm`: Clear first asks "Remove all N items?" with Cancel / Remove.

Add `#clear=undo` or `#clear=confirm` to the URL to force a variant while testing.

Events go to [PostHog](https://posthog.com) (free tier). Analytics are off until a project key is set: paste it into `ANALYTICS.key` near the top of the script in `index.html` (and switch `host` to `https://eu.i.posthog.com` for an EU project). No cookies are used: PostHog runs with in-memory persistence, and the app keeps only an anonymous random id and the variant in localStorage. Browsers sending Do Not Track or Global Privacy Control are not tracked.

| Event | When |
| --- | --- |
| `app_open`, `app_hidden`, `app_resumed` | Session start, leaving the app (with session length and interaction count), coming back |
| `garden_item_placed`, `garden_item_moved`, `garden_item_lost` | Item added (by drag or tap), moved, or slid off the tray (tilt or shake) |
| `garden_clear_tapped`, `garden_clear_cancelled`, `garden_clear_done`, `garden_clear_undone` | The Clear flow |
| `garden_clear_followup` | One minute after a clear, or as soon as the user leaves the app: `kept_playing`, `interactions_60s`, `first_interaction_s`, `left_app`, `left_after_s`, `items_placed_after`, `undone` |

Every event carries `clear_variant`. To compare the variants in PostHog, make a Trends insight on `garden_clear_followup`, broken down by `clear_variant`, and compare the share with `kept_playing = true` (and `left_app = true`). Keep whichever variant leaves fewer people disengaged.
