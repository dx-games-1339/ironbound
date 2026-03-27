# Ironbound — POI Zone Layout Documentation

**Prototype file:** `poi-zone-v2.html`  
**Type:** Single-file HTML prototype (HTML + CSS + JS, no external dependencies except Google Fonts)  
**Color scheme:** Cold — steel blue-grey, silver accents, cool borders

---

## 1. Layout Overview

The POI Zone Layout is the primary gameplay view where players manage characters, assign tasks, and interact with zone objects inside a POI. It is a three-panel resizable layout with a fixed toolbar at the top.

### Panel Structure

```
┌─────────────────── TOOLBAR ───────────────────┐
│ Turn 47 │ Gold 3,840 │ Roster 3    [POI][HQ]… │
├──────┬──────────┬─────────────────────────────┤
│ Task │  Groups  │       Zone Objects          │
│Queue │ in Zone  │  ┌──────────────────────┐   │
│ ~10% │  ~28%    │  │ Located              │   │
│      │          │  ├──────────────────────┤   │
│      │          │  │ Known                │   │
│      │          │  ├──────────────────────┤   │
│      │          │  │ Undiscovered         │   │
│      │          │  └──────────────────────┘   │
└──────┴──────────┴─────────────────────────────┘
```

**Column widths:** Task Queue ~10%, Groups ~28%, Zone Objects fills remaining space (flex:1). Columns are resizable via drag handles between them.

**Row sections** inside Zone Objects: Located, Known, and Undiscovered are vertically resizable via row handles.

---

## 2. Toolbar

Fixed 44px height bar at the top. Displays:

- **Turn counter** — current turn number (e.g. "Turn 47")
- **Gold balance** — guild gold (e.g. "Gold 3,840")
- **Roster count** — total recruits
- **Navigation buttons** — POI Zones (placeholder), HQ (placeholder), Groups (placeholder)
- **End Turn button** — styled with accent colors; does NOT deselect the currently selected character

---

## 3. Task Queue Panel

Left panel (~10% width). Contains:

### 3.1 Action Buttons

Six zone-wide action buttons at the top:

| Button | Condition |
|---|---|
| **Move to…** | Requires player group |
| **Scout zone** | Requires selected player character |
| **Rest** | Requires selected player character |
| **Eat** | Requires selected player character |
| **Exit POI** | Requires entry zone + player group |
| **Return to HQ** | Always enabled (placeholder) |

**Move to** opens a zone picker popup listing 4 adjacent zones with their states (Scouted/Unknown). Selecting a zone creates per-member tasks.

Enemy characters (Wolf) cannot be assigned any zone-wide tasks.

### 3.2 Task Cards

Each task card displays:

- **Drag handle** (⠿) — for drag-to-reorder
- **Task label** — action name, truncated with ellipsis
- **Cancel button** (✕) — removes the task
- **Slot display** — diamond-shaped character portrait + arrow + square target icon (colored by type: green=tree, grey=rock, blue=character, red=enemy)
- **Progress bar** — horizontal bar showing execution progress

**Task flash animation** — when a new task is added, it animates with a build-up curve: 4→7→10→13→15→13→10 over 1.2 seconds, where 10 corresponds to the standard highlight level. The animation ends at exactly the highlight-strong value so there is no visual discontinuity when the `hl` class takes over.

**Drag-to-reorder** — tasks can be reordered by dragging. Drag guards prevent DOM rebuilds during drag operations.

### 3.3 Highlight Interactions

- Hovering a task card highlights its assigned character (gold glow on diamond) and its target object (inset glow on zone object row)
- Hovering a character diamond highlights related tasks
- Hovering a zone object highlights related tasks

---

## 4. Groups Panel

Middle panel (~28% width). Displays all groups present in the current zone.

### 4.1 Group Card

Each group card shows:

- **Header** — group name + faction tag (Player=blue, Hostile=red)
- **Character diamonds** — rhombus-shaped portraits in a row. Click to select a character. Selected characters get an animated gold glow. Highlighted characters get a dimmer pulsing glow.
- **Empty slots** — dashed diamond outlines for unfilled group slots
- **Buttons** — Character, Log, Config, Inventory
- **Inventory** — expandable list of group-level items

**Character button** opens a Character Unified View for the currently selected character in that group.

**Enemy groups** have a red left border and cannot be selected for task assignment.

### 4.2 Character Label Hover Preview

Hovering over a character's name label (below the diamond portrait) shows a tooltip-style preview popup with the character's full details. See Section 8 for the preview layout.

---

## 5. Zone Objects Panel

Right panel (flex:1). Three vertically resizable sections:

### 5.1 Located Objects

Objects with visibility ≥70% of ceiling. Full multi-column layout.

**Column order:** Icon → Name/Type/Size → Materials → Status Effects → Content → Tags

| Column | Width | Description |
|---|---|---|
| **Icon** | 33×33px | Square for non-living objects, diamond for living characters/groups. Color-coded by type. |
| **Name/Type/Size** | 120px | Object name (13px), type label below in smaller mono font (e.g. "Tree", "Rock", "Group"), size value. Groups also show member diamonds. |
| **Materials** | 110px | Pile-based item icons. Items extractable via Harvest action. |
| **Status Effects** | 92px | Circular dots with initial letter. Color-coded: green=buff, rose=debuff, steel=neutral. JS-positioned tooltips on hover showing name, charge bar, description. |
| **Content** | flex | Darker background (`rgba(8,12,18,0.5)`), full row height via `align-self:stretch`. At 100% visibility: pile-based item icons. Below 100%: silver visibility progress bar with eye icon (no percentage text; tooltip shows actual % on hover). |
| **Tags** | 120px | Pill-shaped labels. Hover shows JS-positioned tooltip with tag description. |

**Row height:** min-height 54px, allowing two rows of pile items in Content.

**Object at 100% visibility** — content column shows items directly. No visibility bar.  
**Object below 100% visibility** — content column shows eye icon + silver progress bar (scaled 40%→100%) + "Revealed at 100%" placeholder text.

### 5.2 Known Objects

Objects with visibility ≥34% but <70% of ceiling. Simplified layout with aligned columns matching Located structure. Empty Materials, Status Effects, Tags columns reserve space. Content column shows the visibility bar in the same position as Located objects.

### 5.3 Undiscovered Section

Static text message: "There might be unknown objects in this zone."

---

## 6. Group Zone Objects

Groups appear in the Zone Objects panel as single objects per GDD 4.6. Individual characters cannot be targeted from outside the group.

### 6.1 Faction-Colored Frame

Each group object is wrapped in a `.grp-frame` container with a colored 2px border:
- Player groups: blue (`#3a6090`)
- Enemy groups: red (`#904040`)

### 6.2 Expand/Collapse

Click the group header bar to toggle between states.

**Collapsed state:**
- Group diamond icon with member portrait initials in a compact row
- Summed materials (all characters combined)
- Combined content (all character inventories merged)
- Single group-level visibility
- No status effects on group level
- No tags on group level

**Expanded state:**
Each character displayed as a separate row within the faction outline:
- Individual character icon (diamond, faction-colored)
- Character name + "Character" type label
- Per-character materials
- Per-character status effects (wounds, hunger, buffs, debuffs)
- Per-character content/inventory
- Per-character tags

Clicking any character row opens the action dropdown for the **whole group** — individual characters cannot be targeted for object interactions.

### 6.3 Interaction Rules

- A character cannot interact with the group they belong to ("Cannot interact with own group" warning)
- Enemy units cannot be commanded by the player ("Cannot command enemy units" warning)
- Actions require a selected player character

---

## 7. Pile-Based Item Display

All items (materials and content) use a pile decomposition system. No text names or quantities are shown — icons only.

### 7.1 Pile Types

| Pile | Quantity | Border Color | Visual |
|---|---|---|---|
| Single | 1 | Default (`--border`) | Plain icon |
| Pile of 3 | 3 | Silver (`#a0a0b0`) | Silver-outlined icon |
| Pile of 10 | 10 | Purple (`#8848b0`) | Purple-outlined icon |

### 7.2 Decomposition Algorithm

Given a quantity, decompose greedily: pile-10s first, then pile-3s, then singles.

Example: 25 Meat → 2× pile-10 (purple) + 1× pile-3 (silver) + 2× single

### 7.3 Item Spacing

Items have `gap:0` between pile icons. Moving the mouse between adjacent items does not create dead space — the tooltip updates seamlessly without disappearing.

### 7.4 Item Tooltip

Hovering a pile item shows a tooltip with:
- Larger 32px icon with appropriate pile outline
- Item name and pile label (e.g. "Wood — Pile of 10")
- Quantity as number (1, 3, or 10)
- Item type (Resource, Ore, Food, Weapon, etc.)
- Gold value range (e.g. "8–12 gold") or "No value"

Clicking a pile item is a no-op in this version.

---

## 8. Character Preview Popup

Appears on hovering a character name label in the Groups panel. Positioned below the label, clamped to viewport.

**Layout:** Two-column, 420px wide, pointer-events:none (tooltip-style).

### 8.1 Left Column (140px)

1. **Portrait** — 60px diamond, faction-colored, initial letter
2. **Name** — centered below portrait
3. **Equipment grid:**
   - Row 1: Head (Hd), Garment (Arm), Backpack (Bk), Trinket 1 (T1), Trinket 2 (T2)
   - Row 2: Weapon 1 (W1), Weapon 2 (W2), Weapon 3 (W3)
   - Filled slots show item icon; empty slots are dimmed; Trinket 2 is locked (dashed border) unless character has the `Provident` trait
4. **Stats:**
   - LVL — white progress bar, 0=empty, 11=full, capped at 100% for levels >11
   - HP — dark red track (`#301010`), bright red fill. Current health / base max health
   - STR — numeric value
   - DEX — numeric value
   - PRC — numeric value

### 8.2 Right Column (flex)

**Status Effects** — each effect displays:
- Circular colored dot (wound=red, hunger=yellow, cold=light-blue, buff=green, debuff=rose)
- Effect name
- Direction indicator: `»` (increasing), `«` (decreasing), `⟨⟩` (random)
- Color-coded progress bar (charges/maxCharges)

### 8.3 Bottom (full width)

**Inventory** — carrying capacity as `used/max` (e.g. "6/15"), pile-based items

---

## 9. Character Unified View

A persistent, draggable window opened via the "Character" button in the Groups panel. Opens for the currently selected character.

### 9.1 Window Properties

- **Draggable** — drag by header bar to reposition freely
- **Multiple instances** — can open separate windows for different characters simultaneously (for item transfer workflows). Cannot open duplicate windows for the same character.
- **Bring-to-front** — clicking inside a window raises it above other windows (z-index counter)
- **Close** — ✕ button in the header
- **Staggered positioning** — each new window offsets by 24px from previous

### 9.2 Layout

**Header:** Character name (title font) + "Currently in group [Name]" + ✕ close button

**Body (two-column):**
- **Left (140px):** Portrait, equipment grid, stats — identical to preview popup
- **Right (flex):**
  - **Status Effects area** — same as preview but interactive: hovering an effect shows a detailed tooltip (name, charge bar, charge count, description). Tooltips render at z-index 9999, above all other UI.
  - **Inventory area** — dark background matching the zone objects content area. Minimum height even when empty. Shows pile-based items with tooltips.

---

## 10. Tooltip System

All tooltips use a unified JS-driven fixed-position system. Tooltips are never CSS-based (no `position:absolute` inside clipped containers).

### 10.1 Functions

| Function | Purpose |
|---|---|
| `showTip(anchor, desc, title)` | Show tooltip above anchor element |
| `showTipRaw(anchor, html)` | Show tooltip with custom HTML above anchor |
| `showTipAtEvent(ev, html)` | Show tooltip at mouse cursor position |
| `hideTip()` | Remove active tooltip |

### 10.2 Positioning

- Centered horizontally above the anchor element
- If no room above, flips below
- Clamped to viewport edges (4px margin)
- Visibility bar tooltips use `showTipAtEvent` for cursor-aligned positioning

### 10.3 Z-Index

Tooltips use `z-index: 9999`, ensuring they render above character view windows (z-index 700+), dropdowns (z-index 500), and all other UI elements.

---

## 11. Action Dropdown

Right-clicking (actually left-clicking) a zone object opens a context-style dropdown menu.

### 11.1 Actions by Object Type

| Object Type | State | Available Actions |
|---|---|---|
| Tree (Located) | located | Examine, Cut Down, Gather Resources, Harvest |
| Rock (Located) | located | Examine, Gather Resources, Harvest, Move |
| Group (Located, enemy) | located | Attack, Examine |
| Group (Located, player) | located | Examine, Loot |
| Any (Known) | known | Search for |

### 11.2 Conditions

- Actions are disabled if no character is selected ("Select a character first")
- Actions are disabled for enemy characters ("Cannot command enemy units")
- Actions are disabled for own group ("Cannot interact with own group")

---

## 12. Color Scheme — Cold

All colors are defined as CSS custom properties in `:root`.

### 12.1 Background Palette

| Variable | Value | Usage |
|---|---|---|
| `--bg` | `#141618` | Page background |
| `--bg-warm` | `#181b1e` | Panel backgrounds |
| `--bg-card` | `#1e2226` | Card/item backgrounds |
| `--bg-card-hover` | `#252a2f` | Hover state |
| `--bg-inset` | `#12151a` | Inset elements (slots, inputs) |
| `--bg-deep` | `#0e1014` | Deep backgrounds (headers, bars) |

### 12.2 Text Palette

| Variable | Value | Usage |
|---|---|---|
| `--text` | `#c0c8d4` | Primary text |
| `--text-bright` | `#e0e6ee` | Emphasized text |
| `--text-mid` | `#8898a8` | Secondary text |
| `--text-dim` | `#506070` | Dimmed labels |
| `--text-faint` | `#384450` | Barely visible text |

### 12.3 Accent Colors

| Variable | Value | Usage |
|---|---|---|
| `--gold` | `#90a8c0` | Primary accent (silver-blue) |
| `--gold-dim` | `#607888` | Dimmed accent |
| `--red` | `#a04848` | Enemy/danger |
| `--green` | `#4a8870` | Silver-green |
| `--blue` | `#5888c0` | Player/info |

### 12.4 Status Effect Colors

| Effect Type | Dot Background | Bar Fill |
|---|---|---|
| Wound | Red gradient (`#602020`→`#903030`) | `#d03030` |
| Hunger | Yellow gradient (`#605020`→`#907830`) | `#c0a030` |
| Cold | Blue gradient (`#204860`→`#306888`) | `#70a8d0` |
| Buff | Green gradient (`#204838`→`#306850`) | `var(--green)` |
| Debuff | Rose gradient (`#402830`→`#603840`) | `#c06060` |

---

## 13. Typography

| Variable | Font | Usage |
|---|---|---|
| `--font-title` | Almendra (serif) | Headers, group names, character names |
| `--font-ui` | Alegreya Sans (sans-serif) | Body text, descriptions, tooltips |
| `--font-mono` | JetBrains Mono (monospace) | Stats, values, labels, tags |

---

## 14. Interaction Model

### 14.1 Selection

- Click a character diamond → select that character. Sets `selCharId` and `selGroupId`.
- Click End Turn → character stays selected (toolbar excluded from deselection handler)
- Click empty space → deselect all

### 14.2 Hover Highlighting

All hover interactions use class-toggle functions (not DOM rebuilds) to prevent mouseenter/mouseleave desync:

| Hover Target | Effect |
|---|---|
| Task card | Highlights assigned character + target object |
| Character diamond | Highlights related tasks + related zone objects |
| Zone object | Highlights related tasks + related characters |
| Group card | Shows group's tasks in task queue |

### 14.3 Global Click Handler

The global click listener deselects all state and closes dropdowns unless the click target is inside one of the following containers: `.char-gem`, `.obj-row`, `#dropdown`, `.grp-btn`, `.grp-card`, `.task-card`, `#toolbar`, `.grp-frame`, `.grp-frame-header`, `.char-view`.

---

## 15. Test Scenario Data

### 15.1 Characters

| Character | Faction | LVL | HP | STR | DEX | PRC | Status Effects | Equipment |
|---|---|---|---|---|---|---|---|---|
| Bob | Player | 6 | 105/120 | 10 | 11 | 9 | Well Fed (buff), Hunger (45/120) | Leather Armor, Backpack, Torch, Sword |
| Alice | Player | 8 | 88/110 | 9 | 10 | 11 | Scratch (wound), Alert (buff), Hunger (80/120) | Metal Helmet, Chainmail, Large Backpack, Lamp, Bow, Knife. Provident trait → Trinket 2 unlocked |
| Wolf | Enemy | 3 | 65/70 | 12 | 8 | 10 | Aggressive (debuff), Hungry (hunger), Frost (cold) | None |

### 15.2 Groups

| Group | Faction | Members | Summed Materials |
|---|---|---|---|
| Test 1 | Player | Bob, Alice | Meat ×8 |
| Enemy Test 2 | Enemy | Wolf | Meat ×6 |

### 15.3 Zone Objects

- 36 random trees/rocks with randomized materials, tags (~20%), status effects (~25%), content (40%)
- **Glowing Willow** — 100% vis, Light Source tag, Glowing buff, Wood + Herbs content
- **Berry Bush** — 100% vis, Food(420) tag, 25 Meat content
- Player group "Test 1" and enemy group "Enemy Test 2" as group zone objects
- Current zone: Collapsed Gallery (not entry zone → Exit POI disabled)
- Adjacent zones: Entrance Hall (Scouted), Flooded Passage (Scouted), Deep Cavern (Unknown), Fungal Grove (Scouted)

### 15.4 Initial Tasks

| Task | Action | Target | Assignee | Progress |
|---|---|---|---|---|
| t1 | Gather Resources | First located tree/rock | Bob | 62% |
| t2 | Scout | None (zone-wide) | Unassigned | 15% |

---

## 16. Known Issues / Pending Items

- Drag reorder has a potential off-by-one in upward drag case
- Dropdown opens even without a selected character (shows disabled actions with warning)
- No keyboard support (Escape for dropdowns/popups)
- Dropdown positioning doesn't account for actual rendered height
- Column resize doesn't constrain right panel minimum width
- Character Unified View does not update dynamically if character data changes during the session
