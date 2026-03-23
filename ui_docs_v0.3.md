# Ironbound — Interface Design Document

**Version:** 1.1-draft
**Compatible with GDD version:** 4.0-draft
**Target URL:** `https://dx-games-1339.github.io/strategy`
**Renderer:** HTML/CSS for all UI panels and menus; HTML5 Canvas 2D for the global map and POI zone graph

---

## Compatibility Notice

This document was written against **GDD v4.0-draft**. If the main Game Design Document has been updated to a newer version since this document was last revised, the two documents should be reconciled before implementation proceeds. Key GDD sections that drive interface requirements: §3 (Layouts), §6 (Organizations), §7 (Groups), §8 (POIs), §9 (Economy), §12 (UI Layout), §14 (Dev Mode).

---

## Changelog

| Version | Changes |
|---|---|
| 1.1-draft | Section 6 fully specified: camera model, draw layer order, terrain colours, marker shape grammar, fixed-size icon rules, image system, all concrete pixel values from map rendering tests |
| 1.0-draft | Initial document structure |

---

## 1. Principles

- All menus, panels, and overlays are rendered as standard HTML/CSS DOM elements layered over the canvas.
- The Canvas element is used exclusively for spatial rendering: the global map terrain/tokens and the POI zone graph.
- Every interactive element must be reachable by mouse click. Keyboard shortcuts are secondary and optional for v1.
- No modal dialogs block the entire screen except the Launch Menu, World Options, and Test Scenario selector — which are pre-game screens where no world is active.
- The UI must remain legible at viewport widths from 1024px to 4K.

---

## 2. Screens and Navigation Flow

```
Browser loads
      │
      ▼
┌─────────────────┐
│  Launch Menu    │──── Load Game ────► (game screen)
└─────────────────┘
      │ Start New Game
      ▼
┌─────────────────┐
│  World Options  │──── Back ─────────► Launch Menu
└─────────────────┘
      │ Begin
      ▼
┌──────────────────────────────────────┐
│            GAME SCREEN               │
│  Toolbar (always visible)            │
│  ┌──────────────────┬─────────────┐  │
│  │                  │             │  │
│  │   Active View    │   (fixed)   │  │
│  │  (canvas or UI)  │             │  │
│  └──────────────────┴─────────────┘  │
│  Status bar (always visible)         │
└──────────────────────────────────────┘
        │
        ├── Click city    ► City Details Popup, then Enter ► City Layout
        ├── Click POI     ► POI Details Popup, then Enter ► POI Layout
        └── Click group   ► Group Panel Popup
```

Navigation between views is explicit — the player clicks to enter or exit. Only one view is active at a time. The Toolbar and Status Bar remain visible across all views.

---

## 3. Global Layout

*To be defined.*

---

## 4. Toolbar

*To be defined.*

---

## 5. Sidebar

*To be defined.*

---

## 6. Active View — Global Map

The Global Map renders on a full-viewport HTML5 Canvas element. It is pannable (mouse drag and touch drag) and zoomable (scroll wheel and pinch-to-zoom). The map is always the default active view when the game is open.

The canvas is sized to `window.innerWidth × window.innerHeight` scaled by `devicePixelRatio` to stay sharp on high-DPI screens. All draw calls use logical pixels (divided by DPR before drawing) so coordinates are expressed in CSS pixels throughout.

---

### 6.1 Camera Model

```
cam = { x, y, zoom }
// x, y   — world offset in tile units (top-left visible world coordinate)
// zoom   — scale multiplier, applied on top of BASE_TS

BASE_TS = 14            // base tile size in CSS pixels at zoom = 1
ts()    = BASE_TS × zoom  // effective tile size in CSS pixels

worldToScreen(wx, wy):
  sx = (wx − cam.x) × ts()
  sy = (wy − cam.y) × ts()

screenToWorld(sx, sy):
  wx = sx / ts() + cam.x
  wy = sy / ts() + cam.y
```

**Zoom range:** 0.25 – 20. Zoom-in step ×1.19, zoom-out step ×0.84 per scroll tick.

**Initial camera:** on game start the camera is centred on the world midpoint:
```
cam.x = MAP/2 − viewportWidth  / (BASE_TS × 2)
cam.y = MAP/2 − viewportHeight / (BASE_TS × 2)
```

**Zoom pivot:** zoom always keeps the cursor position fixed in world space. When zooming, the world coordinate under the cursor is calculated before the zoom change and the camera offset is recomputed after so that same world point maps back to the same screen point.

---

### 6.2 Draw Layer Order

All layers are drawn each frame in this fixed order. Later layers overdraw earlier ones.

| # | Layer | Notes |
|---|---|---|
| 1 | Background fill | `#04060c` — covers the entire canvas before anything else |
| 2 | Terrain tiles | Only tiles within the visible frustum are drawn |
| 3 | Grid lines | Drawn only when `ts() > 20` |
| 4 | Roads | Dashed lines connecting each city pair |
| 5 | POI markers | All POIs, discovered and undiscovered |
| 6 | City markers | Drawn after POIs so cities always appear on top |
| 7 | Vignette | Radial gradient overlay, purely cosmetic |

Group tokens (not yet implemented) will be inserted between layers 6 and 7 when added.

---

### 6.3 Terrain

Terrain is a square tile grid of size `MAP × MAP`. Each tile has one of the following types, drawn as a filled rectangle at the tile's screen position with size `ts() + 0.5` (the +0.5 prevents sub-pixel gaps between tiles).

| Tile type | Colour |
|---|---|
| `water` | `#071828` |
| `shallows` | `#0a1e2e` |
| `plains` | `#0d1a0e` |
| `forest` | `#071309` |
| `hills` | `#111418` |

Only tiles within the visible screen region are drawn each frame. The visible range is computed from `cam.x`, `cam.y`, `ts()`, and the canvas dimensions, then clamped to `[0, MAP)`.

**Grid lines** are drawn at tile boundaries when `ts() > 20` using `rgba(255,255,255,.04)` at `lineWidth = 0.5`. Below that zoom threshold the grid is omitted entirely.

---

### 6.4 Roads

Roads are drawn as dashed lines connecting every city pair. Roads scale with zoom — they are part of the world, not fixed UI.

| Property | Value |
|---|---|
| Colour | `rgba(130,100,40,.3)` |
| Line width | `max(1, ts() × 0.16)` |
| Dash pattern | `[ts() × 0.4, ts() × 0.28]` |

---

### 6.5 Marker Shape Grammar — Fixed Rule

**Shape is semantically meaningful.** The outline shape of a map marker encodes the category of the object it represents. This is a hard rule that must be followed consistently throughout the entire game:

| Shape | Category | Examples |
|---|---|---|
| **Circle** | Static locations — cities and POIs | City markers, POI markers |
| **Rhombus (diamond outline)** | Living characters and groups | Recruit tokens, group tokens |

No exceptions. A POI must never use a diamond outline. A character token must never use a circle outline.

---

### 6.6 Fixed-Size Markers — Core Principle

City and POI markers are rendered at a **fixed screen size that does not change with zoom level**. Their world anchor point moves with zoom and pan as normal, but the drawn pixel radius is constant.

This was established during map rendering tests and is the required behaviour for all static location markers. The practical effect is that cities and POIs are always legible and clickable regardless of how far the player has zoomed out.

Fixed sizes (CSS pixels):

| Marker | Constant | Value |
|---|---|---|
| City circle radius | `CITY_R` | `38 px` |
| POI circle radius | `POI_R` | `18 px` |

Culling uses these fixed pixel sizes, not the zoom-scaled tile size:
- POIs are culled when their screen position is more than 80 px outside the viewport bounds.
- Cities are culled when their screen position is more than 120 px outside the viewport bounds.

---

### 6.7 City Markers

Cities are drawn as circular markers, always at `CITY_R = 38 px` radius.

**Draw order for a single city marker:**

1. **HQ glow rings** (HQ city only) — two concentric strokes outside the circle:
   - Inner ring: radius `CITY_R + 14`, `rgba(180,130,10,.25)`, width `8 px`
   - Outer ring: radius `CITY_R + 22`, `rgba(180,130,10,.10)`, width `4 px`

2. **Dark backing disc** — radius `CITY_R + 2`, fill `rgba(4,6,14,.85)`. Creates a clean separation between the icon and terrain.

3. **Image clip** — the city icon image is drawn inside a circular clip of radius `CITY_R`. A dark overlay `rgba(0,0,0,.22)` is applied on top to improve integration with the dark terrain. If the image is unavailable the circle is filled with `#3a80c0` as a fallback.

4. **Circle border** — stroked after the clip is released:
   - HQ city: `#c8900e` (gold), width `2.5 px`
   - Standard city: `#4a70a0` (blue), width `1.8 px`

5. **Name label** — always visible, fixed size, positioned `CITY_R + 5 px` below centre:
   - Font: `600 12px Palatino, serif`
   - Colour: `#8ac4e0`
   - Text shadow: `rgba(0,0,0,.98)` blur `5`

6. **`[HQ]` sub-label** (HQ city only) — positioned `CITY_R + 19 px` below centre:
   - Font: `300 10px monospace`
   - Colour: `#c89010`

All city text uses `textAlign = center`, `textBaseline = top`.

---

### 6.8 POI Markers

POIs are drawn as circular markers, always at `POI_R = 18 px` radius. There is no diamond or rotated-square framing — the circle is the complete shape. The POI has two display states:

**Discovered POI**

1. **Dark backing disc** — radius `POI_R + 2`, fill `rgba(4,6,14,.85)`.
2. **Image clip** — the appropriate size-variant icon is drawn inside a circular clip of radius `POI_R`, followed by a `rgba(0,0,0,.2)` overlay.
3. **Circular border** — `#c8900e` (gold), solid, width `2 px`.
4. **Name label** — always visible, positioned `POI_R + 6 px` below centre:
   - Font: `400 11px Georgia, serif`
   - Colour: `rgba(200,144,14,.9)`
   - Text shadow: `rgba(0,0,0,.98)` blur `4`

**Undiscovered POI**

1. **Dark backing disc** — radius `POI_R + 2`, fill `rgba(4,6,14,.85)`.
2. **Image clip** — the undiscovered icon drawn at `globalAlpha = 0.55`, followed by a `rgba(4,6,20,.45)` overlay to mute it further.
3. **Circular border** — `#2a3a60` (muted blue), dashed `[4, 3]`, width `1.5 px`.
4. **No name label.** Undiscovered POIs show no text.

**Image key selection:**

| Condition | Image key |
|---|---|
| Discovered, size `small` | `map-poi-small-disc` |
| Discovered, size `medium` | `map-poi-medium-disc` |
| Discovered, size `large` | `map-poi-large-disc` |
| Undiscovered (any size) | `map-poi-undisc` |

If an image fails to load, the fallback is a filled circle in a gold tone appropriate to size (large: `#c0900e`, medium: `#a07008`, small: `#806005`) for discovered POIs, and `#1c2a40` for undiscovered.

---

### 6.9 Image System

All map images are loaded at startup from the `./images/` directory. Every image is accessed through a central registry by key. No image path is hardcoded outside the registry. Missing images never crash the renderer — every draw call has a procedural fallback.

**File naming convention:**

| Pattern | Purpose |
|---|---|
| `map-city-{size}.png` | City icon, clipped to circle on map (400×400 px, transparent background) |
| `map-poi-{size}-disc.png` | Discovered POI icon, clipped to circle on map (400×400 px, transparent background) |
| `map-poi-undisc.png` | Undiscovered POI icon, clipped to circle on map (400×400 px, transparent background) |
| `portrait-{id}.png` | Character portrait, clipped to rhombus for group tokens |
| `poi-bg-{type}.png` | POI side panel header background (440×140 px) |
| `zone-{name}.png` | Zone background image in zone graph and zone detail header (440×140 px) |
| `item-{id}.png` | Item icon (64×64 px) |
| `obj-{id}.png` | Zone object icon (32×32 px) |
| `bg-{id}.png` | Screen background image |

**Loading pattern:**

```js
const IMG = {};
function loadImg(key, file) {
  const img = new Image();
  img.onload  = () => { IMG[key] = img; draw(); };  // triggers redraw on load
  img.onerror = () => { IMG[key] = null; };          // null = use fallback
  img.src = './images/' + file;
}
```

Images are loaded asynchronously at startup. The map redraws when each image arrives, so icons progressively improve from fallback shapes to real images as assets load. `IMG[key] === null` explicitly means the file was requested but failed; `IMG[key] === undefined` means the key was never requested.

---

### 6.10 Interaction

| Action | Result |
|---|---|
| Mouse drag | Pans the map; cursor changes to `grabbing` |
| Scroll wheel | Zooms in/out with cursor as pivot |
| Touch drag (1 finger) | Pans the map |
| Pinch (2 fingers) | Zooms in/out with midpoint as pivot |
| Click city | Opens City Details Popup |
| Click discovered POI | Opens POI Details Popup |
| Click undiscovered POI | Opens POI Details Popup (no Enter button) |
| Click empty space | Closes any open popup |

---

### 6.11 Vignette

A radial gradient from transparent at the centre to `rgba(4,6,12,.6)` at the edges is drawn over the entire canvas as the final layer. Inner stop radius: `H × 0.38`. Outer stop radius: `H × 0.78`. This is purely cosmetic and does not affect interaction.

---

## 7. Active View — POI Layout

*To be defined.*

---

## 8. Active View — City Layout

*To be defined.*

---

## 9. Launch Menu

Full-screen overlay shown before any world is initialised. Replaces the game screen entirely. No back button — navigating away requires a page refresh.

```
┌─────────────────────────────────────┐
│                                     │
│           I R O N B O U N D         │
│      MERCENARY GUILD MANAGEMENT     │
│   ─────────────────────────────     │
│      [ ▶  START NEW GAME ]          │
│      [ ⟳  LOAD GAME      ]          │
│      [ ⚙  TEST SCENARIO  ]          │
│                                     │
│                        v0.1 draft   │
└─────────────────────────────────────┘
```

- Start New Game → World Options screen
- Load Game → browser file picker, accepts `.json`; invalid file shows toast error
- Test Scenario → Test Scenario selector (not yet implemented; shows toast)

---

## 10. Toast Notifications

A non-blocking message strip that fades in at the bottom-centre of the screen and auto-dismisses after ~2.75 seconds. Used for confirmations (save, load) and informational messages. Never used for critical decisions that require acknowledgement.

---

## 11. Dev Panel

Visible only when `?dev` is in the URL. A 260px panel anchored to the bottom of the screen, toggled by a badge in the bottom-right corner. Contains a validation log, scenario buttons, and quick state injection helpers. Full specification in GDD §17.

---

*End of document — ui-v1.1-draft*
*Compatible with GDD version: 4.0-draft*
