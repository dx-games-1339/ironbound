# Ironbound — Interface Design Document

**Version:** 1.0-draft
**Compatible with GDD version:** 4.0-draft
**Target URL:** `https://dx-games-1339.github.io/strategy`
**Renderer:** HTML/CSS for all UI panels and menus; HTML5 Canvas 2D for the global map and POI zone graph

---

## Compatibility Notice

This document was written against **GDD v4.0-draft**. If the main Game Design Document has been updated to a newer version since this document was last revised, the two documents should be reconciled before implementation proceeds. Key GDD sections that drive interface requirements: §3 (Layouts), §6 (Organizations), §7 (Groups), §8 (POIs), §9 (Economy), §12 (UI Layout), §14 (Dev Mode).

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

## 6. Active View — Global Map

The Global Map renders on the HTML5 Canvas element. It is pannable (mouse drag) and zoomable (scroll wheel). The map is always the default active view when the game is open.

### 6.1 Canvas contents (drawn by `drawMap()`)

| Layer | Description |
|---|---|
| Terrain tiles | Coloured squares per tile type (water, plains, forest, hills). Drawn first, entire visible region. |
| Grid lines | Subtle grid at zoom > 20 tile-pixels. |
| Roads | Dashed lines between city pairs. |
| POI markers | Discovered: filled rotated square (diamond shape) in gold. Undiscovered: outlined rotated square in muted blue-grey with a "?" label at sufficient zoom. Selected POI has a glow effect. |
| City markers | Filled circle in blue. HQ city has an additional outer ring glow. |
| Group tokens | Rhombus portrait grids (not yet implemented — placeholder). |
| Labels | City names and (at sufficient zoom) POI names drawn via Canvas 2D text. |

### 6.2 Interaction

| Action | Result |
|---|---|
| Click discovered POI | Opens POI Details Popup |
| Click undiscovered POI | Opens POI Details Popup (no Enter button) |
| Click city | Opens City Details Popup (same flow as POI) |
| Click empty space | Closes any open Details Popup |
| Drag | Pans the map |
| Scroll wheel | Zooms in/out, keeping the cursor position fixed in world space |

### 6.3 Camera model

```
cam = { ox, oy, zoom, ts }
// ox, oy: pixel offset of the world origin from the canvas origin
// zoom:   scale multiplier applied on top of base tile size ts
// ts:     base tile size in pixels (set at init to fit world in viewport)

worldToScreen(wx, wy) = [ wx * ts * zoom - ox,  wy * ts * zoom - oy ]
screenToWorld(sx, sy) = [ (sx + ox) / (ts * zoom),  (sy + oy) / (ts * zoom) ]
```

Zoom range: 0.2 – 16. Default at game start: world fits within the map container with ~20px margin.

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

*End of document — ui-v1.0-draft*
*Compatible with GDD version: 4.0-draft*
