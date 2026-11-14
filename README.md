# Sandtray

A real-time sand tray for phones and tablets, in the browser. Draw in it with your fingers, tilt or shake the device, and the sand slides, piles up and avalanches like the real thing.

**Open it:** https://jaredmullaney-ui.github.io/Sand/ (on iPhone, tap the screen once and allow motion access when asked).

## Two ways to play

Every visit opens on a start screen with two modes. Each one starts on a fresh tray. The button in the top-left corner brings the start screen back (switching modes starts a fresh tray; **Back to my tray** keeps yours).

- **Zen Garden:** the sand tools, the classic one-tap castle, and the Garden items below.
- **Castle Builder:** the sand tools, plus a ribbon of castle pieces you build with piece by piece. The tray starts with deeper sand to build with.

Add `#mode=zen` or `#mode=castle` to the URL to skip the start screen.

### Castle Builder

Tap **Castle** to open the ribbon of pieces, pick one, then touch the sand. Every piece is packed from the tray's own sand, so the sand around it gets shallower.

| Piece | How to place it |
| --- | --- |
| Tower | Touch to place a bucket-moulded tower. Drop one on another to build it taller. Pinch with a second finger to resize. |
| Wall | Drag along the sand. A new wall piece is added for every stretch you drag, so you can lay a long, straight or curving wall in one stroke. |
| Level | A flat block to build on: a new level for your castle. Twist with a second finger to turn it, pinch to resize. |
| Spire | A drip-castle spire of runny wet sand. |
| Stairs | Touch where the stairs start and drag toward the top. |
| Castle | The whole classic castle in one go. Twist to turn it. |
| Flag | Touch the top of a tower or wall to plant a toothpick flag. It flutters in the breeze and falls over when the sand under it gives way, or when you shake the tray. |
| Moat | Draw a channel around your castle. When you lift your finger, water runs in from where you started and fills it. The banks are damp and hold their shape. The water is real: tilt the tray and it runs to the low side, spills over low banks and pours off the edge of the tray. Shake the banks down and it spreads out and soaks into the sand, which stays dark until it dries. |

There is no snapping: pieces land exactly where you put them. What happens next is up to the physics:

- A piece rests on the highest ground under it. Short overhangs and narrow gaps are held by the damp sand.
- Where a piece reaches further out over a small drop, it **sags** down onto the ground below.
- Where it hangs out over a big drop, that part **breaks off** and crumbles into loose sand.
- If its centre of mass isn't over its support, the whole piece **topples** and crumbles.
- Swiping through it, tilting steeply or shaking knocks everything down, just like the classic castle.

**Undo** in the ribbon takes back the last piece, flag or moat (up to six steps). On a computer: click to place, drag for walls and stairs, scroll to turn a piece, Shift+scroll to resize it, R to rotate, Esc to put the piece down.

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

- **Castle pieces:** each piece is a height profile rasterised into the sand grid with 3×3 supersampling and packed (cohesive) sand. When it is placed, the column heights under it decide where it rests (on the highest 30% of the ground), which cells are supported, how far each unsupported cell is from support (damp sand bridges about one sand-depth), and whether its centre of mass lies inside the convex hull of its support. Unsupported parts get a temporary visual drop: sagging slabs bend down over 0.7 s and broken chunks fall under gravity, then become loose sand that the physics spreads out.
- **Moat water:** a shallow-water layer simulated on the GPU with the "virtual pipes" method. Each cell keeps a water depth and four outflows that are pushed by the difference in water surface height plus the pull of the tilt. The tray's edges are open, so water that reaches them runs off. Water that reaches dry sand soaks some of it in, and the sand stays wet until it dries. Digging a moat pours water in where you started, enough to fill the channel to a little below the sand around it. The dug channel is also kept as a mask so freshly dug banks hold their shape. The water is rendered over the wet sand with depth-dependent absorption, scattering from silt, ripple caustics, a room reflection and lamp glints.
- **Flags:** small cloth meshes built each frame, waved by a breeze, lit by the lamp and casting shadows on the sand. The sand height under each flag is read back from the GPU a few times a second so flags fall when the castle under them crumbles.

- **Garden items:** each item is generated once on the GPU as a top-down relief (albedo, height, material) and lit live by the same lamp as the sand, so highlights and long shadows swing as you tilt. A coarse map of each item's solid spans lets a lantern roof or a tree canopy shade the sand while light still passes beneath it, and the item's base acts as a wall in the sand physics.

Add `#debug` to the URL for an FPS / mass overlay.

The version from before the zen garden and Castle Builder (sand plus the classic castle only) is kept on the [`classic-castles`](https://github.com/jaredmullaney-ui/Sand/tree/classic-castles) branch.

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
| `mode_selected` | A mode was picked on the start screen (`mode`, `from`) |
| `castle_ribbon_opened`, `castle_piece_placed`, `castle_flag_planted`, `castle_moat_dug`, `castle_undo` | Castle Builder use. `castle_piece_placed` says which piece and how many pieces stood, sagged, broke or toppled |

Every event carries `clear_variant`. To compare the variants in PostHog, make a Trends insight on `garden_clear_followup`, broken down by `clear_variant`, and compare the share with `kept_playing = true` (and `left_app = true`). Keep whichever variant leaves fewer people disengaged.
