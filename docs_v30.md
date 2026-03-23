# Ironbound — Game Design Document

**Version:** 4.0-draft  
**Target URL:** `https://dx-games-1339.github.io/strategy`  
**Renderer:** WebGL 2D, single-page application  
**Hosting:** GitHub Pages (static, no backend)

---

## 1. Concept Overview

The player controls a mercenary guild — a for-profit organization operating in a medieval fantasy kingdom. The core loop is resource allocation under uncertainty: identify profitable opportunities on the world map, deploy groups of recruits to exploit them, manage costs and risks, and keep the guild solvent and staffed.

There is no real-time action. All gameplay happens through menus and turn-based decisions. The WebGL canvas renders the world map and POI layout views as navigable visual surfaces; all actions are taken through UI panels overlaid on those views.

The game has no defined win condition. The player survives as long as the guild remains operational — solvent, staffed, and with a functional headquarters.

---

## 2. Time System

| Unit | Duration |
|---|---|
| 1 turn | 4 in-game hours |
| 1 day | 6 turns |
| 1 week | 42 turns |
| 1 month | 180 turns (30 days) |

The player advances time by pressing **End Turn**. All scheduled events, travel, task progress, and POI timers resolve turn by turn. The player may end multiple turns rapidly if no attention is required, but the game pauses automatically when events occur that require a decision.

A month is defined as exactly 30 in-game days. Weeks do not divide evenly into months — a month contains 4 full weeks (168 turns) plus 2 additional days (12 turns).

---

## 3. Layouts

The game has four distinct layouts (views). Only one is active at a time. Navigation between layouts is explicit — the player clicks to enter or exit a view.

### 3.1 Global Map

The primary view. A scrollable 2D map rendered in WebGL.

**Contents:**
- Static cities, always visible, never despawn
- All active POIs, each in either a Discovered or Undiscovered state
- Guild group tokens showing the current location of each deployed group (see Group Token Display below)
- Travel routes shown as lines when a group is in transit

**Interactions:**
- Click a city → open City Details Panel (same flow as POI — see POI visibility below). From the City Details Panel the player can click Enter City to open the City Layout.
- Click a visible POI → open POI Details Panel. From the Details Panel the player can click Enter POI to open the POI Layout.
- Click a group token → open Group Panel.
- Pan and zoom the map

**Coordinates:** Every city and POI on the global map has a fixed pair of coordinates. Distance between any two points is calculated from these coordinates using standard Euclidean distance. Coordinates are used internally for travel time calculations and are not displayed to the player as numbers.

**Distance measurement:** The player can measure the distance between any two points on the global map directly in the UI. Selecting a group and hovering over or clicking a destination displays the distance to that destination and the estimated number of turns required for the selected group to travel there.

**Travel time formula:**

> **turns = distance / group_speed**

Where:
- **distance** is the Euclidean distance between the two coordinate pairs in map units.
- **group_speed** is the movement speed of the group's slowest member in map units per turn.
- Movement speed is a 1:1 relationship — a character with speed 1 travels 1 map unit per turn; a character with speed 14 travels 14 map units per turn.

A character with the baseline speed of 10 travels 10 map units per active turn. With 5 active turns per day cycle (the 6th turn reserved for rest), the average group covers 50 map units per day. Wound penalties affecting movement speed are factored into group_speed at the time of calculation.

**Group token display:**

Each group deployed on the global map is represented by a token rendered at the group's current coordinates. The token is a rhombus-shaped grid of character portraits belonging to the members of that group, following these layout rules:

- **1–4 members** — portraits are arranged in a 2×2 rhombus grid. Empty grid positions are left blank if the group has fewer than 4 members.
- **5–9 members** — portraits expand to a 3×3 rhombus grid. The group can contain up to 9 members, which fills the grid exactly.
- **Center portrait** — the portrait occupying the center position of the grid is displayed at a larger size than the portraits at the edge positions. This creates a visual focal point and helps distinguish the token at a glance on a busy map.

All portraits within the token use the rhombus icon shape consistent with the living character display rules (see Section 8.4.5). Clicking the token opens the Group Panel for that group.

**Movement constraints:** The player cannot send groups to arbitrary locations on the map. A group may only be assigned to travel to a known city or to a Discovered POI. The global map is not a free-movement surface — it is a visual overview of available destinations.

**POI visibility:** There is a fixed number of POIs active in the world at any given time. All active POIs are visible on the global map — their location and existence are always shown. However POIs exist in one of two discovery states:

- **Discovered** — the player can interact with this POI. Groups can be assigned to travel to it and enter it.
- **Undiscovered** — the POI's location is visible on the map and the player knows it exists, but the player cannot interact with it or send groups to it until it is discovered.

Discovery is determined by two mechanisms:

- **Intelligence network** — the guild maintains agents at HQ assigned to the *Gather Rumors* task (see Section 3.2). The number of agents and their capabilities determine how many POIs spawn as Discovered. In v1 this mechanic is a placeholder and has no actual effect.
- **References** — during the exploration of a POI, a group may find a reference that directly discovers a specific other POI. This is the only way to discover POIs beyond the baseline public count.

An Undiscovered POI reverts to nothing if its despawn timer expires before it is discovered — the opportunity is lost.

### 3.2 City Layout

A menu-driven view representing a city and the guild's headquarters when it is based there.

**Navigation:** Clicking a city on the Global Map opens a City Details Panel — identical in structure to the POI Details Panel. The panel displays the city's name, size, and a "never" despawn value (cities never despawn). An **Enter City** button navigates into the City Layout. Cities are always in the Discovered state and the Enter City button is always enabled.

Each city has a fixed pair of **coordinates** on the global map. These are used to calculate distances and travel times in the same way as POI coordinates.

Each city also has a **size** property (small, medium, or large) assigned at world generation. Size determines the range and quantity of goods available in the city's Market — larger cities stock more items.

**City panels:**
- **Market** — purchase food, equipment, and weapons. Items are divided into three stock categories (see Section 9.3). The player stages items for purchase in the For Purchase box and confirms the transaction from this panel (see Section 9.5).
- **Tavern / Recruitment** — hire new recruits. Each recruit has a name and a hiring cost. The pool refreshes at the start of each new in-game week.

**Headquarters panels (visible only if HQ is in this city):**
- **Roster** — full list of all guild recruits, their conditions, equipment, and assignment status
- **Storeroom** — inventory of all items stored at HQ. The Storeroom has no capacity limit. Gold is tracked separately as the guild's gold balance (see Section 9.1), not as a stored item. The Storeroom contains a dedicated **For Sale** section where the player can stage items for sale (see Section 9.4).
- **Ledger** — gold income and expenditure log per turn, running balance
- **Group Manager** — create, disband, and configure groups; assign recruits to groups; assign groups to tasks
- **Staff Assignments** — assign recruits to passive HQ tasks (see below)

**HQ staff tasks:**

Recruits not assigned to a field group can be assigned to one of the following passive tasks at HQ. Each task has a capacity — more recruits produce better output. Assignments can be changed at any time.

| Task | Effect |
|---|---|
| Gather Rumors | Determines how many newly spawned POIs are Discovered beyond the public baseline. In v1 this task produces no effect beyond the baseline set by the *Target publicly known POIs* world option — all discovery beyond the baseline requires finding references in the field. The task slot and assignment UI must be present in v1 so agents can be assigned in preparation for when the full mechanic is activated in a future update. |
| Develop Trading Connections | Increases the number of items available in the city Market and raises the chance of rare items appearing in the rotation. |
| Train Recruits | Slowly improves unassigned recruits over time. |
| Treat Wounded | Reduces recovery time for injured recruits. More agents and more capable agents speed up recovery further. |

A recruit assigned to an HQ task is unavailable for field deployment until reassigned. Task output is calculated each turn based on the number of assigned recruits and their capabilities.

In V1 HQ staff tasks are a placeholder and has no real effect or game logic behind it.

### 3.3 POI Layout

A view representing the internal structure of a single POI.

A POI is composed of **zones** connected by passages. The layout is a node graph: zones are nodes, passages are edges. The graph is rendered on the WebGL canvas with zones as clickable shapes and passages as lines between them. Each zone has a **size** characteristic that determines how large the area is. Size is reflected visually in the POI graph and governs the cost and yield of exploration actions inside the zone (see Section 4.1).

<u>POI Zones are where most of the gameplay actions happen. Players are supposed to spend the most of their playing time in these layouts.<u>

**Zone states:**

Zone states are a coarse, zone-level summary visible in the POI graph view. They are distinct from object-level awareness states (Located, Known, Undiscovered), which are finer-grained and only visible inside the Zone Detail Panel. A zone being Scouted does not mean every object inside it has been Located — it means the zone has been entered and observed at least once.

- **Unknown** — zone exists but has not been entered or scouted. Displayed as a dim node with a question mark.
- **Scouted** — the zone has been entered at least once. Displayed with a summary icon indicating what has been found (hostile characters present, resources available, artifact detected, etc.). Individual objects within the zone may still be Unknown or Known at the object-awareness level.

**Zone contents (examples):**
- Resource deposits (ore, pelts, herbs)
- Artifacts (high-value single-loot objects)
- Trapped or incapacitated friendly groups (rescue targets)
- Living characters and animals

**Group tokens** appear inside zones showing which groups are present, using the same rhombus portrait grid layout as on the global map (see Section 3.1). Clicking a zone opens the Zone Detail Panel.

**POI metadata** shown in a sidebar:
- POI type and name
- Turns remaining before despawn

### 3.4 Zone Detail Panel

A panel opened by clicking a zone inside the POI Layout. The Zone Detail Panel is where all granular gameplay decisions are made. Full documentation of zone mechanics — conditions, object awareness, factions, and actions — is in **Section 4. Zone Mechanics**.

**Displays:**
- Zone name and active conditions (light level, weather)
- List of Located objects in this zone, from the perspective of the player faction
- List of Known objects in this zone (existence confirmed, location unknown)
- List of Located characters and groups from other factions visible to the player faction, including their visible status effects (see Section 5 for status transparency rules). Living characters are displayed with a rhombus portrait icon; dead bodies are displayed with a square dead body icon (see Section 8.4.5)
- Available actions for each player-controlled group or individual recruit in this zone

**Character status display:**

Every Located character — whether player-controlled, an animal, or dead — displays the following when selected or hovered in the zone panel:

- Current health (effective maximum after all wound penalties)
- All active wounds with name, stage, current charges, and max charges displayed as a progress bar
- Hunger stage with current charges and max charges
- All other active status effects with their current state
- Loyalty (player-controlled characters only)

This applies equally to animals. Once an animal is in the Located state for the player's faction, its full status — wounds, hunger, charges — is visible. Dead characters in the Located state display all status effects frozen at the moment of death.

---

## 4. Zone Mechanics

The zone is the primary arena of gameplay. This section documents how zones work, what they contain, and what actions can be taken within them.

### 4.1 Zone Properties

Each zone has a set of static and dynamic properties that govern how it behaves.

**Static properties:**

- **Size** — a fixed numeric value representing the physical area of the zone. Size does not change during a playthrough. It affects the cost and yield of awareness actions: a larger zone takes more turns of Scouting to cover and each individual Scout action reveals fewer objects relative to the total number present. Larger zones can also contain more objects overall. Size is not displayed to the player as a number but is reflected in the zone's visual scale in the POI graph and in how quickly scouting progresses.

**Dynamic properties (zone conditions):**

- **Light level** — a value from 0% (absolute darkness) to 100% (full daylight). Light level affects what characters can perceive and how quickly they detect objects. Diurnal characters perform well at high light levels; nocturnal creatures may be comfortable around 30% but are also impaired in absolute darkness.
- **Weather effects** — conditions such as rain or sandstorm that can be applied to individual zones. Weather affects all objects and characters in the zone and may restrict certain actions or alter how quickly characters are detected.
- **Hazard conditions** — passive conditions that apply damage probabilistically when characters perform certain actions in the zone. Each hazard condition specifies which actions trigger it, the probability of each damage event, and the damage value. Damage is rolled per action attempt, not per full action execution. See Section 4.9.2 for the full hazard condition model.

### 4.2 Living Characters and Animals

A zone may contain any number of living characters or animals alongside static objects. These are full objects in the zone's object list and follow the same awareness mechanics as everything else — they may be Undiscovered, Known, or Located depending on the faction observing them.

Living characters and animals each have a **disposition** toward every other group or faction present in the zone. Disposition is not binary. A character or group may be hostile toward one faction, neutral toward another, and indifferent to a third simultaneously. The following dispositions are possible:

- **Hostile** — actively seeks to harm the other group when aware of them.
- **Neutral** — neither allied nor hostile; will not initiate conflict unprovoked but may respond if threatened.
- **Friendly** — cooperative; shares awareness information and will not attack unless provoked.
- **Indifferent** — unaware of or uninterested in the other group; takes no action based on their presence.

Disposition is independent for each pairing. It is entirely possible to enter a zone and find two groups of animals already engaged in conflict that has nothing to do with the player's presence. The player may choose to intervene, avoid, observe, or exploit the situation. In future releases, NPC groups will also occupy zones and operate under the same disposition system.

### 4.3 Object Awareness

A zone may contain a large number of objects — potentially hundreds. Each faction present in a zone maintains an independent numeric **faction visibility** value toward every object. This value accumulates over time through character actions and determines whether the object is Undiscovered, Known, or Located from that faction's perspective.

- **Located** — the faction knows this object exists and knows where it is. Characters can interact with it directly.
- **Known** — the faction knows this object exists somewhere in the zone but has not pinpointed its location. Interaction is not yet possible.
- **Undiscovered** — the object is present in the zone but the faction has no knowledge of it. This state exists in the game data but is never shown to the player.

State transitions are driven entirely by numeric thresholds on the faction visibility value. The full generation rules, threshold formulas, and scouting mechanics are documented in Sections 4.3.1 through 4.3.3.

Characters also have their own visibility value, which is not fixed. A character can actively modify their visibility by performing certain actions — for example by hiding, which raises the threshold another faction must meet in order to locate them. A sufficiently capable scout or a group that searches a zone long enough can still locate a hiding character, but doing so requires greater effort than detecting one who is not concealed.

#### 4.3.1 Object Visibility Generation

When a POI is generated — either at world generation or when a new POI spawns dynamically — each zone object is assigned a **discoverability–visibility classification**: a pair of integer values `_discover_value` and `_visibility_value`, each in the range 0–4.

- `_discover_value` — how much total effort is required to advance the object toward the Located state. 0 = easy to find; 4 = very hard to find.
- `_visibility_value` — the object's starting position on the discoverability spectrum. A high value means the object begins closer to Located; a low value means it starts deep in the Undiscovered range.

These two values are assigned randomly per object instance from ranges defined by the object's type. From them, three instance-specific visibility variables are derived:

**Step 1 — Compute the faction visibility ceiling:**

> **faction_visibility_max = 2 ^ _discover_value × 300**

This is the numeric ceiling of the faction visibility scale for this object. Values range from 300 (discover value 0) to 4800 (discover value 4).

**Step 2 — Compute the initial visibility anchor:**

> **initial_objects_visibility = (faction_visibility_max / 5) × _visibility_value**

This sets the midpoint of the range from which the object's starting visibility is drawn.

**Step 3 — Roll the object's starting visibility:**

A random value is rolled in the range:

> **[0.5 × initial_objects_visibility, initial_objects_visibility]**

The result is assigned as the object's `visibility` characteristic. This is the faction visibility value at which a new faction starts when they first enter the zone — their initial awareness of this object before any scouting occurs.

**Step 4 — Roll the upper bound:**

A random value is rolled in the range:

> **[0.8 × faction_visibility_max, 1.2 × faction_visibility_max]**

The result is assigned as the object's `upper_bound_visibility`. All faction visibility values are bounded by this ceiling — no faction's visibility toward this object can exceed it.

**Living objects — runtime mutability**

For static zone objects (rocks, trees, containers, etc.) `visibility` and `upper_bound_visibility` are spawn-time constants. They do not change unless a character explicitly acts on them (e.g. via the Conceal action).

For **living objects** — characters and character groups present in a zone — `visibility` and `upper_bound_visibility` are **runtime values** that can change at any time through actions such as Hide. When either value changes on a living object, the awareness states of all factions that have a visibility entry toward that object are re-evaluated immediately against the new thresholds. A faction that had Located a living object may lose that state and revert to Known or Undiscovered if the object's `upper_bound_visibility` increases sufficiently, and similarly a faction that had not yet Located an object may gain it if `upper_bound_visibility` decreases. Awareness state changes on living objects are applied in the same turn they occur.

#### 4.3.2 Faction Visibility Thresholds

Each faction's current visibility value toward an object determines the object's awareness state for that faction:

| Condition | Awareness state |
|---|---|
| faction_visibility < 40% of upper_bound_visibility | Undiscovered |
| faction_visibility ≥ 40% of upper_bound_visibility | Known |
| faction_visibility > 90% of upper_bound_visibility | Located |

When a new faction enters a zone for the first time, their visibility toward each object in that zone is initialised to the object's `visibility` characteristic (the value rolled in Section 4.3.1 Step 3). The awareness state is then evaluated immediately from this starting value — objects with a high starting visibility may be Known or even Located the moment a faction arrives, while low-visibility objects begin Undiscovered and require active scouting to surface.

Faction visibility values are stored per object per faction and increase only through character actions. They never decrease naturally.

#### 4.3.3 Scout Zone — Awareness Mechanics

The Scout Zone action interacts with the awareness system in two distinct ways: a **passive per-attempt tick** applied during every action point investment, and an **active discovery pulse** applied once per full execution.

**Passive per-attempt tick**

Every time a character invests the action point cost (5 AP) toward a Scout Zone attempt — regardless of whether the action fully executes that turn — the faction visibility of **all objects** in the zone is increased by:

> **passive_increase = ⌈0.25 × character_perception⌉**

This applies to every object in the zone without exception, including objects already in the Located state (though further increase beyond the upper bound has no effect).

**Active discovery pulse (on full execution)**

When a Scout Zone action fully executes — that is, when the accumulated progress reaches the progress threshold and the action fires — the following steps are applied:

1. **Calculate the character's Discovery Value:**

> **discovery_value = 3 × level + 11 × perception + 0.5 × dexterity + 1 × movement_speed**

2. **Roll the number of objects affected:**

A random integer is rolled in the range [2, 6] inclusive. This determines how many objects in the zone receive the full discovery pulse that execution.

3. **Apply the discovery pulse:**

The `discovery_value` is added to the faction visibility of each of the selected objects. Objects are selected randomly **without replacement** from the full set of objects in the zone — each object can be selected at most once per discovery pulse. Any object — regardless of its current awareness state — can be selected. If the zone contains fewer objects than the rolled count, all objects in the zone receive the pulse. Objects whose faction visibility would exceed `upper_bound_visibility` after the addition are capped at `upper_bound_visibility`.

After each discovery pulse, awareness states are re-evaluated for all affected objects and transitions to Known or Located are applied immediately if the thresholds are crossed.

### 4.4 Factions

Awareness is tracked per faction, not per individual character or group. A faction is a set of groups that share complete knowledge of each other and of all objects they have collectively discovered within a zone. Groups belonging to the same controller (e.g. all player-controlled groups) always form a single faction within a POI and share awareness state instantly.

Animal groups belong to their own factions. It is possible for two factions to occupy the same zone with no awareness of each other — neither appears in the other's Located list. Conflict between factions only becomes possible once at least one group in each faction has located at least one group in the other. In future releases, NPC character groups will also form their own factions and operate under this same system.

### 4.5 Zone-wide Actions

Zone-wide actions have no specific target. They are assigned to a group and apply broadly to the zone or to all eligible objects within it.

- **Scout** — the character actively searches the zone. Each action point investment (5 AP per attempt) applies a passive visibility increase to all objects in the zone. Each full execution fires a discovery pulse that applies a larger visibility increase to a random subset of objects. Both effects advance object awareness toward Known and Located for the scouting character's faction. Full mechanics are documented in Section 4.3.3.
- **Hunt** — the group passively engages all huntable animals present in the zone without needing to locate them individually. No specific target is required; the action resolves against any huntable animal in the zone.
- **Hide** — the group actively conceals itself, increasing how difficult it is for other factions to detect them. Because characters and groups are living objects, their `visibility` and `upper_bound_visibility` are runtime values (see Section 4.3.1). Each completed Hide execution raises the group's `upper_bound_visibility`, pushing the Known and Located thresholds higher and forcing other factions to accumulate more visibility to maintain or achieve the same awareness state. A faction that had Located the hiding group may lose that state and revert to Known or Undiscovered if the group's `upper_bound_visibility` rises above the faction's current visibility relative to the new thresholds. A sufficiently capable opposing scout or sustained search effort can still locate a hiding group by continuing to accumulate faction visibility past the new higher thresholds.
- **Rest** — the character takes the full turn off to recover. Costs 46 action points. This cost is intentionally set to 46 rather than 50: since most zone actions cost 5 AP, the remaining 4 AP after resting are insufficient to perform any standard action, ensuring rest is a full-turn commitment. However, lightweight group management actions (such as arranging inventory or transferring an item between characters) will cost only 1 AP and are designed to be performed alongside rest — these do not apply changes to the zone and only affect the internal state of the group. Such actions are planned for a future update. The Rest action is subject to the 50 AP action cost cap — even if a head wound applies an action cost increase, Rest will never cost more than 50 AP and remains executable by any living character. During rest the character consumes food from their inventory until hunger charges are reduced to 20 or below in the Hunger stage — eating through Starvation and Famine if necessary. See Section 5.6.2 for the full Rest eating mechanic and loyalty bonuses.
- **Move** — assign a group or individual to travel to an adjacent zone. The number of turns required depends on the distance between zones and the movement speed of the slowest group member. No target object required; destination zone is the parameter.
- **Retreat** — a special case of Move directing the group back toward the POI entry zone and out of the POI.
- **Wait** — hold position for a specified number of turns (e.g. awaiting a supply group).

### 4.6 Object Interactions

Object interactions require a specific target. A target must be in the Located state before any object interaction can be assigned, with one exception: a group may be assigned to **Search for** a Known object, which applies a focused visibility increase to that specific object on each execution and a smaller secondary increase to a random set of other objects in the zone. See the Search for entry below for the full numeric mechanic.

Object interactions are divided into four subcategories:

**Common actions** — applicable to most or all objects:
- **Take** — pick up the object and place it in a character's inventory. Requires sufficient free inventory capacity. Cannot be applied to objects too large to carry.
- **Gather resources** — extract usable material from the object (e.g. harvesting from a plant, stripping a carcass). Can be performed without tools but is significantly faster if a compatible tool is present.
- **Examine** — inspect the object closely to learn more about its properties without interacting with it further.
- **Conceal** — deliberately make a Located object harder to find. Each completed Conceal execution applies two effects:
  - **Base visibility reduction** — the object's `visibility` characteristic (the value used to initialise faction visibility for new factions entering the zone) is reduced by 10% of its current value: `object.visibility = ⌊object.visibility × 0.9⌋`. This makes the object harder to discover for any faction that has not yet entered the zone.
  - **Faction visibility reduction** — for every faction that has already Located the object, their faction visibility toward it is reduced by 10% of its current value: `factionAwareness[factionId][objectId] = ⌊factionAwareness[factionId][objectId] × 0.9⌋`. The faction that performed the Conceal action is exempt from this reduction. After the reduction, awareness states are re-evaluated — a faction whose visibility falls back below the Located threshold (≤90% of `upper_bound_visibility`) will lose the Located state and revert to Known. Factions that have not yet Located the object are unaffected by this component.
- **Search for** (Known objects only) — direct the group's effort toward locating a specific Known object. The Search for action produces no passive per-attempt tick — action point investments during progress accumulation apply no visibility increase to any object. On full execution the following steps are applied:

  1. **Calculate the character's Discovery Value** using the same formula as Scout Zone (see Section 4.3.3):

     > **discovery_value = 3 × level + 11 × perception + 0.5 × dexterity + 1 × movement_speed**

  2. **Apply the focused increase to the target object** — a random value is rolled in the range [0.8 × discovery_value, 1.75 × discovery_value] and added to the performing faction's visibility toward the target object.

  3. **Apply the secondary increase to other objects** — a random integer is rolled in the range [1, 5] inclusive to determine how many other objects in the zone are affected. That many objects are selected at random from all objects in the zone excluding the target, and each receives a visibility increase of 0.8 × discovery_value for the performing faction.

  All visibility increases are capped at `upper_bound_visibility`. Awareness states are re-evaluated for all affected objects immediately after the execution and transitions to Known or Located are applied if thresholds are crossed.

**Inventory actions** — operate on items in a character's inventory rather than on zone objects:
- **Drop** — removes a selected item from a character's inventory and places it into the zone as a new object. The dropped item is immediately Located by the dropping character's faction. Other factions must discover it through normal awareness mechanics.

**Construction actions** — introduce new permanent objects into the zone. Constructed objects enter the Located state for the builder's faction immediately:
- **Build camp** — establishes a camp structure providing a base for resting, treating injuries, and storing supplies. Requires materials and time.
- **Build container** — constructs a basic storage box or crate. Can store items, cache supplies, or serve as bait. Additional buildable structures may be defined per game content.

**Object-specific actions** — available only for particular object types, surfaced contextually in the UI:
- Examples: **Open** (chest, door), **Unlock** (locked container), **Cut down** (tree), **Rescue** (incapacitated friendly character), **Attack** (living characters and animals only).
- Object-specific actions are not enumerated exhaustively here. They appear in the UI only when the relevant object is selected and the action is applicable.

### 4.7 Tool Requirements and Compatibility

Actions fall into three categories with respect to tools:

- **No tool required** — the action can always be performed regardless of inventory.
- **Tool optional** — the action can be performed without a tool but progress is multiplied by the tool multiplier if one is present (see Section 4.8).
- **Tool required** — the action cannot be performed unless the character has at least one item meeting the minimum compatibility threshold. If no qualifying item is in inventory, the action is unavailable.

Tool requirements are expressed as a named **condition** (e.g. "sharp tool", "key") paired with a minimum **compatibility threshold** (e.g. ≥ 70%). Each item has its own compatibility rating for each condition it belongs to. An axe, for example, might have 80% compatibility with "sharp tool" — satisfying an action that requires sharp tool ≥ 70%, but not one that requires sharp tool ≥ 95%.

Key rules:
- One item can satisfy multiple conditions simultaneously, each at its own rating.
- Only the best qualifying item in the character's inventory is used when checking a requirement.
- When a tool is optional, the highest-rated qualifying item in inventory provides the tool multiplier applied to progress gain.

### 4.8 Action Execution Cycle

All actions performed inside a POI zone follow a unified execution cycle based on **action points** and **action progress**.

#### 4.8.1 Action Points

Each character has **50 action points** per turn by default. Action points represent the total effort a character can invest in activities during one turn. They are spent in discrete amounts each time a character attempts to advance an action. Unspent action points at the end of a turn are lost — they do not carry over.

#### 4.8.2 Action Properties

Every action has the following properties:

| Property | Description |
|---|---|
| **Action point cost** | The number of action points spent per attempt. The base cost is defined per action. Certain status effects (e.g. head wounds) apply an **action cost increase** expressed as a percentage — the base cost is multiplied by `(1 + increase%)` and rounded up to the nearest integer. The resulting cost is capped at 50 under all circumstances; no effect can raise the cost of any action above 50. |
| **Contribution factors** | A list of `(characteristic, multiplier)` pairs. Each attempt gains progress equal to the sum of `characteristic value × multiplier` across all factors. |
| **Progress threshold** | The total progress that must be accumulated for the action to execute and apply its effect. Most actions use a threshold of 100. |
| **Tool requirement** | A named condition and minimum compatibility threshold. If not met, the action cannot be attempted at all. |
| **Tool multiplier** | If the character has a qualifying tool in inventory, all progress gained per attempt is multiplied by this value. Applied on top of the contribution factors. |

#### 4.8.3 Per-attempt Progress Calculation

Each time a character invests the action point cost into an action, progress is calculated as:

> **progress gained = Σ (characteristic × multiplier) × tool\_multiplier**

If no tool is present and the action has no tool requirement, the tool multiplier is treated as 1 (no effect).

Progress accumulates across attempts within the same turn and carries over between turns. When accumulated progress reaches the progress threshold, the action executes — its effect is applied to the zone — and the progress counter resets to zero. Any surplus progress above the threshold is discarded.

A character may execute the same action multiple times within a single turn if they have enough action points to fund the required number of attempts.

#### 4.8.4 Progress Persistence and Reset

A character can only hold progress toward a single action at a time. The following rules govern when progress is preserved and when it is lost:

- **Progress persists** when the character continues performing the same action across turns. If a character accumulates partial progress toward an action but does not complete a full execution within a turn — because their action points were exhausted — that progress carries into the next turn and the character resumes from where they left off.
- **Progress resets to zero** when a character is assigned a new action, regardless of the reason. This includes manual reassignment by the player, a new task being assigned by the automation system, or an event forcing a different action. Switching to a new action always discards all accumulated progress on the previous one.

A character cannot hold partial progress on two different actions simultaneously. Assigning any new action clears the existing progress immediately.

#### 4.8.5 Worked Example — Scout Zone

Action stats:

| Property | Value |
|---|---|
| Action point cost | 5 |
| Progress threshold | 100 |
| Contribution factors | Level × 1.0; Perception × 2.5; Movement speed × 0.1 |
| Tool requirement | None |
| Tool multiplier | None |

Character: level 7, perception 8, movement speed 10, 50 action points.

Progress per attempt: `(7 × 1.0) + (8 × 2.5) + (10 × 0.1) = 7 + 20 + 1 = 28`

Attempts to reach threshold: `⌈100 ÷ 28⌉ = 4 attempts` (at 4 attempts: 112 progress → threshold met, 12 surplus discarded)

Action point cost for one execution: `4 × 5 = 20 action points`

With 50 action points available the character can execute Scout Zone twice in the same turn (40 action points spent across 8 attempts). The remaining 10 action points fund 2 more attempts, accumulating 56 progress toward the third execution. That 56 progress persists into the next turn, where the character needs only 2 more attempts (56 progress gained, threshold met at 112 — surplus discarded) to complete the third execution.

---

## 4.9 Damage

### 4.9.1 Overview

Damage is a flat numeric value applied to a character. It does not directly reduce health — instead it is measured against the character's current effective health and, based on that relationship, may apply wounds of varying severity or render the character Incapacitated. The full formula is documented inline in Section 4.9.4.

### 4.9.2 Sources of Damage

Damage can come from multiple sources:

**Combat actions** — the primary source of damage in the full game. Not implemented in v1. The damage pipeline (section 4.9.3) must be built from v1 onward so combat can be activated without structural changes.

**Zone hazard conditions** — zones may carry passive conditions that apply damage probabilistically during the execution of certain actions. Each hazard condition defines:

- Which zone actions trigger a damage roll
- The trigger point — damage is rolled each time a character invests action points in one attempt of the relevant action (not once per full execution)
- One or more damage events, each with a probability and a damage value range

Example — *Dangerous exploration* condition (found in zones such as mountain terrain):

| Trigger | Probability | Damage |
|---|---|---|
| Each Scout Zone attempt (per 5 AP invested) | X% | Y1 damage |
| Each Scout Zone attempt (per 5 AP invested) | Z% | Y2 damage |

Multiple damage events on the same condition are rolled independently. A single attempt may trigger none, one, or all of them.

### 4.9.3 Damage Application Pipeline

When a character receives damage the following steps are applied in order:

1. **Incapacitation check** — only evaluated if `current_health > 0`. If `raw_damage ≥ current_health`, the character becomes Incapacitated. If `current_health ≤ 0` the character is already dead and this check is skipped. Initial charges are calculated as described in Section 4.9.7.
2. **Wound probability calculation** — the damage ratio R is computed and used to select a wound severity tier.
3. **Wound slot assignment** — the specific wound is selected from the character's available wound slots for that severity. If the slot is already occupied, charges are added to the existing wound instead (which may trigger progression).

### 4.9.4 Damage Ratio and Wound Probability Formula

All probability calculations are driven by a single normalised value:

> **R = raw_damage / character_current_health**

Where `character_current_health` is the effective maximum health after all active wound penalties.

**Incapacitation:** if raw_damage ≥ character_current_health the character becomes Incapacitated regardless of wound outcome. Both can apply on the same hit. See Section 4.9.7 for charge calculation.

**Wound severity zones** — R maps to five tiers with overlapping soft boundaries:

| Zone | R range | Active tiers |
|---|---|---|
| Graze | 0.00 – 0.15 | Generic only (Scratch, Bruise) |
| Light | 0.10 – 0.25 | Generic fading, Light entering |
| Medium | 0.25 – 0.70 | Light + Medium |
| Severe | 0.55 – 2.00 | Medium + Severe |
| Lethal | 2.00+ | Medium + Severe + Lethal |

**Sigmoid gate function:**

```
gate(x, center, steepness) = 1 / (1 + e ^ (−steepness × (x − center)))
```

**Raw weights per tier:**

Below R = 0.25, generic and light weights are controlled by a two-segment exponential ratio:

```
// segment constants
a1 = 4.790,  b1 = 36.917   // R <= 0.10
a2 = 2.197,  b2 = 10.986   // 0.10 < R < 0.25

if R < 0.25:
    ratio = exp(a1 − b1×R)       if R <= 0.10
    ratio = max(0, exp(a2 − b2×R))  otherwise
    w_generic = ratio
    w_light   = 1.0              // ratio controls the split
else:
    w_generic = 0
    w_light   = gate(R, 0.10, 40) × gate(0.72 − R, 0, 9)

w_medium  = R >= 0.25 ? gate(R − 0.25, 0.05, 30) × gate(1.7 − R, 0, 4) : 0
w_severe  = gate(R, 0.65, 13) × gate(10.0 − R, 0, 4)
w_lethal  = R >= 2.0 ? (R − 2.0) × 0.5 : 0
```

**Normalise:**

```
total = w_generic + w_light + w_medium + w_severe + w_lethal
p_X   = w_X / total
```

**Probability reference table:**

| R | Generic | Light | Medium | Severe | Lethal |
|---|---|---|---|---|---|
| 0.05 | 95% | 5% | 0% | 0% | 0% |
| 0.10 | 75% | 25% | 0% | 0% | 0% |
| 0.20 | 50% | 50% | 0% | 0% | 0% |
| 0.25 | 0% | 84% | 16% | 0% | 0% |
| 0.40 | 0% | 49% | 49% | 2% | 0% |
| 0.60 | 0% | 36% | 48% | 17% | 0% |
| 0.80 | 0% | 15% | 45% | 40% | 0% |
| 1.00 | 0% | 4% | 47% | 49% | 0% |
| 2.00 | 0% | 0% | 19% | 81% | 0% |
| 2.50 | 0% | 0% | 3% | 78% | 19% |
| 3.00 | 0% | 0% | 0% | 66% | 34% |
| 3.50 | 0% | 0% | 0% | 57% | 43% |
| 4.00 | 0% | 0% | 0% | 50% | 50% |
| 5.00 | 0% | 0% | 0% | 40% | 60% |

Lethal wound probability does not exceed 35% at R = 3.0 and continues rising naturally for higher damage values. Medium remains present throughout the entire range including lethal values.

### 4.9.5 Wound Selection

Once a severity tier is selected:

1. Build the list of available wound slots for that character's type at that severity.
2. **Unoccupied slot** — assign the wound normally with its defined starting charges.
3. **Occupied slot** — inject +200 charges into the existing wound. This may trigger progression to the next stage or render the character **Incapacitated** (see Section 5.3.8) if the wound is already at its final stage.
4. Distribute probability equally across available slots unless body part weights are defined.

**Generic wound selection (Scratch / Bruise):**

Generic wounds are stackable — no slot conflict. A secondary roll determines the type based on hit type:

| Hit type | More likely |
|---|---|
| Bladed / sharp | Scratch |
| Blunt | Bruise |
| Piercing | Scratch |

**Starting charges on application:**

| Severity | Starting charges |
|---|---|
| Generic (Scratch, Bruise) | 50 |
| Light | 230 |
| Medium | 400 |
| Severe | 600 |

When a wound progresses via charge injection (repeat hit to an occupied slot), the wound advances to the next stage and its charge counter resets to that stage's starting value.

### 4.9.6 Function Signature

```
applyDamage(character, rawDamage, hitType)
  → { incapacitated: bool, woundApplied: WoundInstance | null }
```

| Parameter | Type | Description |
|---|---|---|
| `character` | Object | Full character state — current health, active wound slots, type |
| `rawDamage` | Number | Rolled damage value |
| `hitType` | String | `"bladed"`, `"blunt"`, `"piercing"` |

### 4.9.7 Incapacitated Status Effect

Incapacitation is a charge-based status effect applied when a character receives a blow exceeding their current effective health. It is not a wound and does not progress into any other state — it simply depletes and removes itself.

**Initial charge calculation:**

> **initial_charges = min(200, 40 + 20 × (max_health / current_health))**

Where:
- `current_health` — the character's current effective health at the moment of the hit (base health minus all active wound penalties). Must be greater than zero — a character with `current_health ≤ 0` is already dead (see Section 5.3.1) and the incapacitation check does not fire. The incapacitation check is only evaluated when `current_health > 0`.
- `max_health` — the character's base health (unmodified by wounds)
- The result is capped at 200 — any excess above 200 is discarded

**Depletion:**

- Charges decrease by 20 per turn automatically
- The character is unconscious and cannot perform any actions while the status is active
- When charges reach 0 the status effect is removed and the character regains consciousness

**Properties:**

| Property | Value |
|---|---|
| Starting charges | 40 + 20 × (max_health / current_health), capped at 200 |
| Max charges | 200 |
| Charge change per turn | −20 (fixed depletion) |
| Progression | None — removed when charges reach 0 |
| Effects | Character cannot perform any actions |

**Example:** A character with base health 100 and current effective health 60 (due to wounds) is hit for 70 damage — exceeding their 60 effective health. Initial charges = 40 + 20 × (100 / 60) = 40 + 33.3 = 73.3, rounded to 73. The character will regain consciousness after 4 turns (73 ÷ 20, rounded up). A more heavily wounded character would produce a higher ratio and therefore more incapacitation charges, reflecting that a weakened body takes longer to recover from a severe blow.

---

## 5. Recruits

Recruits are the guild's core asset and its main operating cost.

### 5.1 States

A recruit's state is a combination of concurrent conditions rather than a single exclusive value. The following conditions can apply simultaneously:

- **Available** — at HQ, ready to be assigned. Mutually exclusive with Deployed.
- **Deployed** — assigned to a group currently on the map or inside a POI. Mutually exclusive with Available.
- **Wounded** — carries one or more active wounds. May still be deployed depending on severity; some wounds do not prevent field activity while others restrict actions or reduce capability significantly. Can co-exist with Incapacitated.
- **Incapacitated** — the character is temporarily unconscious and cannot perform any actions. Incapacitation is a charge-based status effect that depletes automatically each turn until the character regains consciousness. It is triggered either by receiving a blow exceeding the character's current effective health, or by a progressive wound being forced to advance beyond its final non-lethal stage. In both cases the initial charge count is determined by the standard incapacitation formula. A character can be both Wounded and Incapacitated at the same time — the Incapacitated condition takes precedence for UI display and task scheduling purposes. See Section 4.9.7 for the full mechanic.
- **Dead** — the character has died. The body remains in the zone where death occurred as a persistent object (see Section 5.4). All other conditions are frozen at the moment of death.

### 5.2 Growth and Change

Recruits change over time through events rather than through a conventional XP system:

- **Traits** — positive or negative permanent modifiers that alter a character's characteristics, capabilities, or behaviour. Each character may have up to two traits simultaneously. See Section 5.5 for the full trait system.
- **Wounds and status effects** — see Section 5.3 for the full wound and lifecycle system
- **Death** — any recruit can die permanently from lethal wounds, critical wound progression, or starvation. The player is warned when a recruit is at high risk but cannot always prevent it. Dead characters remain in the zone as inspectable bodies (see Section 5.4).

### 5.3 Character Lifecycle and Wound System

#### 5.3.1 Health

Each character has a **health** characteristic representing their physical resilience. The valid range for health is defined per race (see Section 5.8). Health is a static value — it does not naturally increase or decrease over time. Instead, health is affected indirectly by wounds and status effects that impose a **maximum health penalty**, reducing the effective ceiling of the character's health pool.

A character **dies instantly** when their effective maximum health (base health minus all active maximum health penalties) reaches zero or below. A character also dies immediately when a lethal wound is applied (see Lethal Wounds below).

#### 5.3.2 Character Characteristics

Each character has the following persistent characteristics. Unless otherwise noted, characteristics are fixed values that do not change on their own — they are checked or compared when actions are performed, and may be modified by equipment, wounds, or status effects.

**Level**

A single value ranging from 1 to 10 representing overall experience and capability. Most recruits available for hire start at level 5 or 6. Level does not directly govern other characteristics but serves as a general indicator of a character's competence and is likely to influence action outcomes and event resolution.

**Combat and action characteristics**

- **Strength** — measured when performing actions that require physical force, such as melee attacks, moving heavy objects, or breaking through obstructions.
- **Dexterity** — measured when performing actions that require precision or speed, such as ranged attacks, picking locks, or performing treatment on wounds.
- **Perception** — measured when performing awareness-related actions such as scouting, detecting hidden objects, or identifying threats at a distance.

These three characteristics do nothing passively. They are only relevant when an action that checks them is performed.

**Survival characteristics**

- **Regeneration** — determines how effectively the character recovers from wounds over time. Influences the rate at which wound charges accumulate each turn and how quickly bandaged wounds heal.
- **Movement speed** — determines how quickly the character travels. Used to calculate the number of turns required to move between zones inside a POI and to travel between locations on the global map. A character's effective movement speed can be reduced by wound effects (e.g. leg wounds). When a character's movement speed is reduced to zero they cannot move independently.
- **Carrying capacity** — each character has a base carrying capacity ranging between 5 and 10 units for human-like characters. This is the character's intrinsic capacity before any backpack bonus is applied. Total effective capacity is higher when a backpack is equipped. See Section 5.3.4 for the full carrying capacity system.

**Loyalty**

Loyalty represents how willing the character is to remain with their current controller. It ranges from 0 to 30.

Loyalty decreases over time through events such as unpaid upkeep, witnessing the death of companions, or being left in dangerous situations without support. It may increase through positive events such as successful missions, receiving good equipment, or resting in safe conditions.

When a character's loyalty reaches 0, two outcomes are possible depending on their location:

- **Inside a POI zone** — the character immediately leaves their group and becomes an independent object within the zone. They retain all characteristics, wounds, and equipment. They are no longer under player control but remain present as an interactable and attackable object in the zone.
- **Outside a POI zone** (travelling or at HQ) — the character deserts. They are removed from the player's roster and group without returning equipment.

#### 5.3.3 Movement Speed

Each character has a **movement speed** characteristic that determines how quickly they can travel. Movement speed affects two contexts:

- **Global map travel** — the number of turns required for a group to travel from its current location to a destination city or POI.
- **Zone movement** — the number of turns required for a character to move between zones inside a POI.

A group always moves at the speed of its slowest member. If one character in a group has a movement speed penalty from a wound (e.g. a leg wound), the entire group's travel time is calculated using that character's reduced speed. This creates a meaningful trade-off when deciding whether to keep a wounded character in a group or leave them behind.

**Typical values**

A standard day cycle consists of 6 turns. Of these, 1 turn is assumed to be spent resting, leaving 5 turns of active travel per day. A healthy group is expected to cover approximately 50 map units per day, giving a baseline travel rate of 10 units per active turn. From this the reference values follow:

| Movement speed | Description |
|---|---|
| 12–14 | Fast — lightly equipped scouts |
| 10 | Average — a typical recruit at full health with standard equipment |
| 7–9 | Slowed — carrying heavy equipment |
| 4–6 | Impaired — light leg wound or heavily loaded |
| 1–3 | Severely impaired — medium leg wound or equivalent penalty |
| 0 | Immobile — severe leg wound; character cannot move independently |

#### 5.3.4 Carrying Capacity and Equipment

**Carrying capacity**

Each character has an individual carrying capacity measured in abstract units. Items stored in a character's unequipped inventory each consume a defined number of units. A character cannot pick up or be assigned an item that would cause their total carried units to exceed their available capacity.

Capacity is composed of two parts:

- **Base capacity** — an intrinsic characteristic of the character, ranging between 5 and 10 units for human-like characters.
- **Backpack bonus** — if the character has a backpack equipped in the backpack slot, it adds additional capacity. A standard backpack adds approximately 10 units; a large backpack adds approximately 20 units.

Equipped items do not consume inventory capacity. An item worn in an equipment slot is not counted against the character's carrying capacity, regardless of its size. This makes equipping items important not just for their effect but for freeing up carry space.

Characters in a group do not share carrying capacity. Each character holds their own items independently.

**Equipment slots**

Each character has four equipment slots. Items placed in these slots are worn or carried on the body and do not consume inventory capacity.

| Slot | Name | Description |
|---|---|---|
| Body | **Garment** | The character's primary worn layer — cloth, leather armour, chainmail, plated armour, travelling outfit, and similar. Provides the bulk of physical protection. |
| Head | **Head** | A helmet, hat, hood, or similar item worn on the head. |
| Back | **Backpack** | A backpack, quiver, or other item worn on the back. Backpacks extend carrying capacity. Most weapons and large tools can be worn in this slot when no backpack is equipped, keeping them accessible without consuming inventory space. |
| Accessory | **Trinket** | A catch-all slot for items that do not fit the above categories. Any item with a size value below 5 can be worn in the trinket slot. Wearing a small item here rather than storing it in inventory saves that amount of carrying capacity. |

A character may only have one item equipped per slot at a time. Swapping an equipped item places the removed item into the character's unequipped inventory, where it consumes capacity normally.

#### 5.3.5 Status Effects and Wounds

Characters can carry any number of active **status effects**. Wounds are a category of status effect. Each wound applies one or more passive modifiers to the character for as long as it remains active — typically a reduction to maximum health, and sometimes additional restrictions on actions or capabilities (e.g. inability to use two-handed weapons, reduced carrying capacity).

#### 5.3.6 Wound Categories

Wounds fall into three structural categories that determine how they are applied and whether they can stack:

- **Stackable wounds** — can be applied to a character multiple times simultaneously, with each instance tracked independently. Each application adds its modifier separately. Example: a *Scratch* wound applies −3 max health per instance. A character can accumulate many scratches at once, each one counting individually toward their health penalty.

- **Progressive wounds** — cannot stack. Each progressive wound occupies a single slot per body location. If a character already has a progressive wound on a given location and that same location is damaged again, the existing wound advances to the next severity stage rather than a new instance being added. Progressive wounds move through a fixed sequence of stages (e.g. Light → Medium → Severe). When the final stage is reached and further progression is triggered, the character becomes **Incapacitated** (see Section 4.9.7) instead of advancing further. For wounds on vital locations (head, chest) the final progression stage is Lethal, which causes immediate death.

- **Special wounds** — applied as discrete status effects outside the progressive or stackable systems. They behave like stackable wounds in that multiple instances can be active simultaneously, but they represent specific injury types tied to particular damage sources. Example: *Animal Bite* (−20 max health), applied by animal attacks. Each bite is a separate instance. A human-like character can sustain 5–6 simultaneous animal bites before their effective maximum health reaches zero.

#### 5.3.7 Lethal Wounds

Certain wound types cause **immediate death** regardless of the character's current health or any other status. These are classified as **Lethal** and bypass all other checks when applied. For human-like characters, the lethal wound types are:

- Lethal head wound
- Lethal chest wound

Lethal wounds are the terminal stage of progressive wound sequences on vital body locations. They can be applied directly by high-damage hits (see Section 4.9.4) or triggered when a severe progressive wound on a vital location is forced to advance beyond its final stage.

#### 5.3.8 Wound-Triggered Incapacitation

When a progressive wound is at its final non-lethal stage and damage forces it to progress further, the character becomes **Incapacitated**. The initial charge count is determined by the standard incapacitation formula (see Section 4.9.7), using the character's current effective health and base health at the moment the wound progression is triggered. If the character's current effective health is zero or below at the moment of triggering — meaning wound penalties have already reduced them to a lethal state — the character dies instead of becoming Incapacitated, and the formula is not evaluated. Incapacitation depletes automatically each turn and is removed when charges reach zero — the character then regains consciousness. Other characters can still interact with an Incapacitated character (e.g. to move them or administer treatment) while the status is active.

The wound that triggered Incapacitation remains active and continues accumulating charges while the character is unconscious. If the wound is not treated before its charges reach the progression threshold again, Incapacitation is applied a second time immediately after the character regains consciousness — or sooner, if the threshold is crossed while the character is still unconscious. This creates a recurring collapse cycle for neglected final-stage wounds and provides a strong mechanical incentive to treat the wound during the window when the character is incapacitated and accessible.

#### 5.3.9 Wound Examples

The full wound table for human characters — including all progressive, stackable, and special wound types — is defined in **Section 5.8 (Race)**. The wound table for a character's race is the authoritative reference for which wound slots exist, what penalties each stage applies, and which stages are lethal. Characters of other races will have their own wound tables defined when those races are introduced.

#### 5.3.10 Wound Charges

Each active wound accumulates **charges** from 0 up to a maximum defined by the wound type. When a wound reaches its maximum charge threshold it progresses to the next stage — or renders the character **Incapacitated** (see Section 5.3.8) if already at its final stage.

Every wound instance tracks the following values:

- **Current charges** — starts at a defined initial value when the wound is first applied
- **Max charges** — the threshold at which the wound progresses
- **Degeneration value** — a base rate at which charges accumulate per turn (positive = worsening)
- **Regeneration contribution** — derived from the character's regeneration characteristic; reduces charge accumulation each turn

**Each turn**, the following calculation is performed for each active wound:

1. A random value is rolled within the wound's degeneration range
2. A random value is rolled within the character's regeneration range
3. The net result `(degeneration roll − regeneration roll)` is added to the current charges
4. If charges reach the max threshold, the wound progresses

*Example:* A Light arm wound is applied with 230 starting charges and a progression threshold of 1000 (the charge value at which it advances to Medium). Its degeneration produces rolls in the range 10–20. The character's regeneration produces rolls in the range 5–10. On a given turn: degeneration rolls 15, regeneration rolls 7. Net: +8. Charges advance from 230 to 238. This continues each turn until charges reach 1000, at which point the wound advances to a Medium arm wound — which is applied at its own starting charges (400, as defined in Section 4.9.5), not at zero. The progression threshold and the starting charges of the next stage are independent values.

#### 5.3.11 Wound Treatment

Wounds can be treated by characters performing the **Treat** action (requires a compatible medical tool). Treating a wound does not remove it — it transforms it into a **bandaged** variant that heals passively over time.

Each wound has a corresponding bandaged form:

- *Light arm wound (left)* → *Bandaged arm wound (left)*
- *Severe head wound* → *Bandaged severe head wound*
- And so on for all progressive and stackable wound types.

Bandaged wounds retain a modified set of the original wound's effects (reduced penalties) and have a **negative degeneration value**, meaning their charges decrease each turn rather than increasing. The character's regeneration characteristic accelerates this decrease. Once a bandaged wound's charges reach zero, the wound is fully healed and removed.

Lethal wounds cannot be treated — they cause immediate death upon application.

### 5.4 Dead Characters and Body Persistence

When a character dies, their body is not removed from the game. The body remains in the zone of the POI where the character died as a persistent object in the zone's object list. It follows the same awareness and visibility mechanics as any other object — it can be Undiscovered, Known, or Located depending on the observing faction.

**Body state:**

- All status effects, wounds, and hunger stage active at the moment of death are preserved on the body permanently.
- Wound degeneration and charge accumulation stop immediately upon death. No status effect on a dead character progresses further.
- The body retains the character's full equipment and inventory.

**Body persistence:**

A body remains in the zone until the POI despawns, at which point all objects including bodies are removed.

**Inspecting bodies:**

A Located body can be examined by any character using the Examine action. Inspection reveals:

- The character's name, level, and faction at time of death.
- All wounds and status effects present on the body at time of death, including their charge levels.
- The body's full equipment and inventory contents, which can be looted using the Take or Gather resources actions.

Animal bodies are subject to the same inspection rules. If an animal body is in the Located state for the player's faction, its information is fully transparent — including wounds.

**Roster impact:**

A dead player-controlled character is removed from the guild's active roster immediately upon death. They no longer count toward upkeep. Their body remains in the POI zone as described above and is independent of the roster system.

### 5.5 Traits

#### 5.5.1 Overview

Each character may carry up to **two traits** simultaneously. A trait is a permanent modifier to one or more of the character's characteristics, capabilities, or behaviour. Traits can be positive, negative, or a mixture of both.

Traits are assigned once, when a character is first initialised — either when they appear in the Tavern recruitment pool or when they are placed into a POI zone for the first time. Once assigned, traits are permanent: they cannot be added to or removed from an existing character by any means.

The following rules apply to all traits:

- A character cannot hold two instances of the same trait.
- Certain traits are **mutually exclusive** — a character cannot hold both traits in an exclusive pair at the same time (e.g. Fast and Slow).
- Percentage modifiers to characteristics are applied **after** all other additive effects have been calculated, unless stated otherwise.
- Traits that modify loyalty interact with the loyalty system (see Section 5.3.2) but do not override the desertion threshold — a character with modified loyalty still deserts at 0.

#### 5.5.2 Trait List

| Trait | Effect | Notes |
|---|---|---|
| **Loyal** | Loyalty is permanently fixed at 30. Cannot be raised or lowered by any effect. | Mutually exclusive with Disloyal |
| **Disloyal** | Loses 5 loyalty on the first turn of the last day of each in-game month. | Mutually exclusive with Loyal |
| **Provident** | Character has two Trinket slots instead of one. | — |
| **Strong** | Strength +10% (applied after all other modifiers); maximum health +2. | Mutually exclusive with Weak |
| **Weak** | Strength −15%; maximum health −2. | Mutually exclusive with Strong |
| **Dexterous** | Dexterity +10% (applied after all other modifiers). | Mutually exclusive with Clumsy |
| **Clumsy** | Dexterity −15%. | Mutually exclusive with Dexterous |
| **Intelligent** | Level +1; perception +5%. | — |
| **Watchful** | Perception +15% (applied after all other modifiers). Perception benefits more from this trait than physical characteristics do from their equivalents. | Mutually exclusive with Dumb |
| **Dumb** | Perception −30%. | Mutually exclusive with Watchful |
| **Fast** | Movement speed +20%; maximum health −10%. | Mutually exclusive with Slow |
| **Slow** | Movement speed −10%; maximum health +5%. | Mutually exclusive with Fast |
| **Resilient** | Maximum health +10. | Mutually exclusive with Fragile |
| **Fragile** | Maximum health −20. | Mutually exclusive with Resilient |
| **Greedy** | Upkeep cost +10%. | — |

#### 5.5.3 Mutual Exclusivity Reference

| Exclusive pair |
|---|
| Loyal / Disloyal |
| Strong / Weak |
| Dexterous / Clumsy |
| Watchful / Dumb |
| Fast / Slow |
| Resilient / Fragile |

### 5.6 Hunger

Hunger is a progressive status effect that all characters carry at all times. It tracks how long a character has gone without food and escalates through three active stages (Hunger, Starvation, Famine) before triggering the Starved to Death status effect, which causes immediate death.

Hunger is driven by the same charge accumulation system as wounds (see Section 5.3.10). Each stage has a defined starting charge, maximum charge threshold, and charge gain per turn. When a stage reaches its maximum charge it progresses to the next stage automatically.

Consuming food reduces a character's hunger charges directly rather than resetting the hunger chain. Food can reverse progression — a character in Starvation or Famine can eat their way back to an earlier stage.

#### 5.6.1 Hunger Stages

**Hunger**

The baseline state. All characters begin here.

| Property | Value |
|---|---|
| Starting charges | 0 |
| Max charges | 120 |
| Charge gain per turn | 20 |
| Effects | None |

Hunger applies no penalties. It exists to represent the natural passage of time without food and will progress into Starvation if the character is not fed.

---

**Starvation**

| Property | Value |
|---|---|
| Starting charges | 0 |
| Max charges | 300 |
| Charge gain per turn | 20 |
| Effects | Strength −20%; dexterity −20%; perception −20%; loyalty −1 per turn |

The character is going without food. Loyalty erodes each turn as morale suffers. Physical and cognitive capability is meaningfully reduced. Progresses into Famine if left untreated.

---

**Famine**

| Property | Value |
|---|---|
| Starting charges | 0 |
| Max charges | 100 |
| Charge gain per turn | 20 |
| Effects | Strength −80%; dexterity −80%; perception −80%; loyalty −2 per turn |

The character is in critical condition. All action-relevant characteristics are severely impaired. Loyalty collapses rapidly. Famine progresses to Starved to Death upon reaching 100 charges.

---

**Starved to death**

| Property | Value |
|---|---|
| Starting charges | 1 |
| Max charges | 1 |
| Charge gain per turn | 0 |
| Effects | Immediate death on application |

Starved to Death is the terminal stage of the hunger progression chain. When Famine reaches its maximum charge threshold the character's HungerState transitions to stage StarvedToDeath with charges set to 1 — and the character dies immediately. The StarvedToDeath stage is never cleared and persists on the body after death, so that any character examining the body can identify starvation as the cause of death.

#### 5.6.2 Food Items and the Food Tag

Every edible object or item carries a **Food (value)** tag where value is a positive integer representing how many hunger charges are removed when one unit of that item is consumed. Zone objects with a sufficiently small size can be picked up and become items in a character's inventory (see Section 5.3.4), at which point their Food tag applies in the same way as any other food item.

**Consumption mechanic:**

When a character eats one unit of a food item, the item's Food value is subtracted from the character's current hunger stage charges. If the subtraction reduces the current stage's charges to zero or below, the character regresses to the previous hunger stage at its maximum charge value, with any remaining reduction applied to that stage's charges. This can chain across multiple stages in a single eating action.

Examples:
- A character in Hunger at 90 charges eats a Food (40) item — charges drop to 50. Still in Hunger.
- A character in Starvation at 30 charges eats a Food (80) item — charges drop to 0, regressing to Hunger at 120 charges, with 50 remaining reduction applied: Hunger charges become 70.
- A character in Famine can eat their way back through Starvation and into Hunger if they consume enough food value in one eating action.

**The Eat action:**

Inside a POI zone, eating is performed as an explicit action:

- **Action point cost:** 5
- **Effect:** consumes one unit of the highest Food-value item in the character's inventory and applies its Food value to the hunger charge reduction.
- A character must have at least one food item in their inventory to perform this action.

**Automatic eating — self-assigned sub-goal:**

When a character's Hunger stage charges exceed 80 (out of 120 maximum) and the group's **Allow self-management** setting is enabled (see Section 13.5), the character automatically inserts a **Feed Self** sub-goal into the group task list, placed immediately below the character's currently active task, and assigns themselves as the responsible character.

Feed Self is an abstract sub-goal, not a single action. It decomposes automatically into sub-tasks based on the character's current situation:

1. **If food is available in the character's inventory** — the character performs Eat actions until hunger charges are reduced to 20 or below in the Hunger stage.
2. **If no food is in the character's inventory but food exists in another group member's inventory** — a redistribution sub-task is generated to transfer food to the hungry character, followed by Eat actions.
3. **If no food is available in the group at all** — the character attempts to find food within the current zone or adjacent zones, generating sub-tasks to Scout for edible objects, Gather from any Located food sources, and then Eat. If no food sources can be found within the give-up turn count, the sub-goal expires and hunger continues to accumulate.

If Allow self-management is disabled for the group, the character does not insert this sub-goal. Hunger accumulation is then handled entirely by the maintenance lifecycle (see Section 13.6) or by the player manually.

**Rest eating and loyalty:**

When a character performs the Rest action they consume food from their inventory, eating items one at a time until their hunger charges are at 20 or below in the Hunger stage (regressing through Starvation and Famine if necessary). If any food was consumed during rest:

- The character receives **+1 loyalty** regardless of what was eaten.
- If a consumed item also carries a **Delicious (X)** tag, X is added to loyalty on top of the base +1, giving a total of **1 + X loyalty** for that item.
- Multiple delicious items consumed during the same rest each contribute their X value separately.

The Delicious (X) tag and its loyalty bonus only apply during the Rest action. Consuming food via the Eat action outside of rest reduces hunger charges normally but grants no loyalty.

When selecting which food items to consume during rest, the character prioritises items with the highest Delicious value first, then falls back to any remaining food items.

### 5.7 Upkeep

Each character requires a gold payment on the first turn of the last day of each in-game month. This is their upkeep — the combined cost of pay, food, and basic supplies needed to keep them in the guild.

Upkeep is defined as a **per-level rate** for each character, determined when the character is initialised. The actual monthly payment is calculated as:

> **upkeep payment = base rate × current level**

For example, a character with a base rate of 10 gold/month at level 6 costs 60 gold that month. If that character's level increases to 7 before the first turn of the last day of the month, their payment for that month becomes 70 gold. The calculation always uses the character's level at the time the payment is processed.

On the first turn of the last day of each month the total upkeep for all characters currently on the guild's roster is deducted from the guild's gold balance in a single transaction.

The Greedy trait increases a character's base rate by 10%, applied before the level multiplier.

Food and supplies consumed during expeditions are tracked separately from upkeep — a group on a long expedition requires adequate food packed at departure or a resupply group sent after them (see Section 9.2). Running out of food in the field causes hunger to progress through increasingly severe stages, applying mounting penalties to strength, dexterity, perception, and loyalty, and ultimately resulting in death if left unresolved (see Section 5.6). This does not interact with the monthly upkeep payment directly.


### 5.8 Race

Every character has a **race** characteristic assigned at initialisation and fixed for the character's lifetime. Race determines which wound types a character can receive and which other standard status effects apply to them. Characters of the same race share an identical wound table and status effect table.

Race is a data-driven definition — each race specifies its full set of progressive wound sequences, stackable wound types, special wound types, and any race-specific status effects. The damage pipeline (Section 4.9) selects wounds from the character's race wound table when applying damage.

**Human race — wound table:**

The following wound sets are defined for human characters. This is the reference wound table for all human recruits in v1.

*Progressive wounds — left arm:*

| Stage | Max health penalty | Additional effects |
|---|---|---|
| Light arm wound (left) | −15 | — |
| Medium arm wound (left) | −35 | Cannot use two-handed weapons |
| Severe arm wound (left) | −45 | Cannot use two-handed weapons; carrying capacity −10% |
| *(Incapacitated on further damage — see Section 5.3.8)* | | |

Identical stages and penalties to left arm. Tracked independently.

*Progressive wounds — left leg:*

| Stage | Max health penalty | Additional effects |
|---|---|---|
| Light leg wound (left) | −20 | Movement speed −20% |
| Medium leg wound (left) | −40 | Movement speed −40%; carrying capacity −15% |
| Severe leg wound (left) | −60 | Cannot move |
| *(Incapacitated on further damage — see Section 5.3.8)* | | |

Identical stages and penalties to left leg. Tracked independently.

*Progressive wounds — head:*

| Stage | Max health penalty | Additional effects |
|---|---|---|
| Light head wound | −20 | — |
| Medium head wound | −40 | Action cost increase +20% (base 5 AP actions cost 6 AP); scouting effectiveness reduced |
| Severe head wound | −70 | Action cost increase +40% (base 5 AP actions cost 7 AP); scouting effectiveness greatly reduced |
| Lethal head wound | — | Immediate death |

*Progressive wounds — chest:*

| Stage | Max health penalty | Additional effects |
|---|---|---|
| Light chest wound | −30 | — |
| Medium chest wound | −45 | Strength, dexterity, and perception −30% |
| Severe chest wound | −60 | Strength, dexterity, and perception −75% |
| Lethal chest wound | — | Immediate death |

*Stackable wounds:*

| Wound | Max health penalty | Notes |
|---|---|---|
| Scratch | −3 | Applied by bladed or sharp damage; stacks indefinitely |
| Bruise | −2 | Applied by blunt damage; stacks indefinitely; slower degeneration than Scratch |

*Special wounds:*

| Wound | Max health penalty | Notes |
|---|---|---|
| Animal bite | −20 | Applied by animal attacks; stackable; 5–6 simultaneous instances lethal |
| Burn | −10 per instance | Applied by fire or heat; stackable; high degeneration rate |
| Poisoned | −5 per instance | Applied by toxic sources; stackable; degeneration accelerates over time |

**Human race — base characteristics:**

| Characteristic | Range |
|---|---|
| Health | 90 – 130 |
| Base carrying capacity | 5 – 10 |

Additional races may be introduced in future updates. Each new race is defined as an independent wound table and characteristic range without requiring changes to the damage pipeline.

---

## 6. Organizations

### 6.1 Player Guild

The player controls a single guild throughout a playthrough. The guild has:

- A **name** (set at game start)
- A **headquarters city** — assigned at world generation as one of the 3 cities on the map, selected at random. Can be relocated to a different city for a gold cost after the game begins.
- A **gold balance** — the primary resource and the loss condition (if gold reaches zero and no recruits are deployed, the guild dissolves)
- A **roster** of recruits
- An **inventory** of supplies and equipment stored at HQ
- A **criminal record** — a log of offences committed by guild members. See Section 6.3 for full design. *Not implemented in v1 — infrastructure to be prepared for future update.*

### 6.2 Rival Organizations (Planned — v2)

Future releases will introduce rival guilds operating autonomously on the same world map. They will compete for POIs and hire from the same recruit pool. The data model must be designed from v1 to accommodate multiple organizations without refactoring — specifically, group ownership, HQ location, and gold balance must be stored per-organization, not as global player state.

### 6.3 Criminal Records (Planned — future update)

Criminal records are not implemented in the v1 release. The data structures described here must be prepared in the codebase to support future implementation without requiring a refactor.

**Overview:**

A criminal record is a per-guild log of offences committed by guild members. Records are zone-specific — each offence is tied to the zone in which it occurred, not recorded globally. At the end of each month, outstanding offences are reviewed and fines deducted from the guild's gold balance (see Section 13.6 turn order).

**Zone ownership:**

Zones within POIs may carry an **ownership** condition — a global zone property indicating that the zone belongs to a particular kingdom or faction. Cities are treated as kingdoms in their own right. When a POI spawns near a city, its zones have a higher probability of being assigned ownership by that city's kingdom. Zone ownership is a static property assigned at POI spawn and does not change during the POI's lifetime.

**What constitutes a crime:**

In the initial criminal record implementation, an offence is recorded when a player-controlled character performs a violent action against an NPC whose faction name matches the kingdom name of the zone in which the action takes place. The faction name match is a direct string comparison — if the zone is owned by kingdom "Valdenmoor" and the NPC belongs to faction "Valdenmoor", any violent action against that NPC in that zone is recorded as a crime.

**Codebase requirements:**

The following must be in place before criminal records are activated:
- Zone ownership as a named property on every zone object (nullable — zones without an owner have no criminal implications)
- NPC faction name stored on every NPC character object
- A criminal record array on the Organization object (already present in the game state schema as `criminalRecord: [ Offence ]`)
- An Offence data structure: `{ turn, zoneId, poiId, actorId, victimId, description }`
- A monthly processing hook in the turn order that iterates offences and calculates fines (logic deferred — hook must exist and be callable)

### 6.4 Non-Player Characters (Planned — future update)

Human-like NPCs will not be present in the v1 release. The only living non-player characters in the initial release are animals, which occupy zones within POIs as huntable and hostile objects.

When NPCs are introduced in a future update, they will operate using the same underlying systems as player-controlled groups:

- **Goals and task lists** — NPCs will be assigned abstract goals and local actions using the same task system defined in Section 13.
- **Maintenance lifecycle** — NPCs will check their own food supply and injury state each turn using the same maintenance lifecycle logic as player groups.
- **Schedule** — NPCs will follow a 6-turn day cycle schedule identical in structure to player group schedules.
- **Factions and awareness** — NPCs will belong to factions and participate in the same zone awareness and disposition system already documented in Sections 4.2 and 4.4.
- **Criminal record integration** — NPC faction names will be matched against zone ownership for crime detection as described in Section 6.3.

The codebase must treat player-controlled groups and NPC groups as instances of the same underlying data structure from v1 onward, so that NPC behaviour can be activated by assigning goals and a controller without requiring structural changes.

---

## 7. Groups

A **group** is a named collection of one or more recruits that acts as a single unit on the Global Map and inside POIs.

- Groups are created and managed in the HQ Group Manager panel
- A group can contain a maximum of 9 recruits. A player can assign more than one group to the same POI simultaneously, allowing larger operations to be split across multiple groups.
- Groups move together; individual recruit actions are only available within the Zone Detail Panel
- A group moves at the speed of its slowest member — if any character in the group has a reduced movement speed (e.g. from a leg wound), the entire group's travel time is calculated using that character's speed
- Characters in a group do not share carrying capacity — each character holds items independently in their own inventory. The group's effective total capacity is the sum of each member's available space, but items must be assigned to specific characters and cannot exceed individual limits
- Groups can be split or merged at HQ or inside a POI zone where both groups are present

**Daily schedule:**

Each group has a **schedule** — a repeating 6-turn pattern that mirrors the in-game day cycle. Each of the six turn slots can be marked as a **rest turn** or left as an **active turn** by the player.

When a rest turn is reached in the cycle, characters in the group will prefer to rest rather than continue their current activity. If the group has no active task assigned for that turn they automatically perform the Rest action. If a task is in progress when a rest turn arrives the character pauses it and rests instead, resuming on the next active turn.

The default schedule has turn 6 set as the rest turn, matching the movement speed model in Section 5.3.3 — one rest turn per day cycle out of six. The player can adjust the schedule freely: more rest turns slow overall progress but improve wound recovery and fatigue management; fewer rest turns increase output but may cause characters to accumulate fatigue over time.

Schedules are set at the group level and apply to all members equally. Individual characters do not have separate schedules.

**Group roles:**

A group's role is a label assigned by the player to communicate intent and improve UI clarity. Roles are not mechanically enforced — there is no system restriction preventing a labelled Combat group from scouting, or a Supply group from fighting. However, roles are not purely cosmetic either: the recruits a player places in a group and the equipment they carry determine what the group is actually capable of. A group composed of recruits with high scouting capability and light equipment will perform better at reconnaissance regardless of its label. Role labels exist to help the player manage multiple groups at a glance.

- **Scout** — small, lightly equipped, prioritised for reconnaissance and awareness tasks
- **Combat** — larger, heavily equipped, prioritised for engaging hostile characters
- **Supply** — carries food and equipment to support deployed groups
- **Rescue** — dispatched to retrieve incapacitated recruits from inside POIs

**Group actions log:**

Every group maintains a rolling **actions log** — a record of actions performed by group members and events that affected them. The log retains entries for the last 20 turns and discards older entries automatically. Each log entry records the turn number, the character involved, and a description of the action or event.

The following are logged:

- Zone actions performed by any group member (Scout, Rest, Eat, Treat, Gather, etc.) — one entry per execution, not per attempt
- Damage received by any group member, including the wound applied and its severity
- Status effect changes — wounds progressing to a new stage, hunger stage transitions, incapacitation applied or cleared
- Loyalty changes — the amount and the cause (rest with food, upkeep payment, starvation penalty, etc.)
- Character state changes — a member becoming Incapacitated, recovering, or dying
- Items picked up, dropped, or transferred between group members
- Group movement — departure from and arrival at locations

**Access:**

Player-controlled groups expose the log in the Group panel, always accessible to the player. The log is displayed in reverse chronological order (most recent turn first) and can be scrolled.

NPC-controlled groups maintain the same log structure but their logs are not displayed in normal gameplay. They are accessible exclusively through Dev Mode (see Section 14), where they appear in the same format as player group logs. This allows inspection of NPC behaviour during testing and debugging without exposing it during normal play.

The log is included in the game state schema (see Section 11.2) and is serialised with the save file so it persists across sessions.

---

## 8. Points of Interest (POIs)

### 8.1 Properties

Every POI has:

- A **type** (dungeon, camp, ruins, forest zone, cave system, etc.)
- **Coordinates** — a fixed position on the global map expressed as an (x, y) pair. Used to calculate distance to other map points and to determine travel time for groups. Not displayed to the player as numbers.
- A **despawn timer** — number of turns until the POI disappears from the map
- A **zone graph** — the internal layout of zones and passages
- A **loot table** — an internal list of possible resources and artifacts the POI may contain. This is not visible to the player; discovering what a POI holds is part of the scouting challenge.

### 8.2 Spawn and Despawn

- POIs spawn dynamically during world generation and as the game progresses
- Small POIs persist for 2–10 in-game days
- Medium POIs persist for 10–60 in-game days
- Large POIs persist for 1–6 in-game months

### 8.3 Zone Graph Structure

Each POI contains between 3 and 20 zones depending on size. Zone graphs follow these patterns:

- **Linear** — zones form a single chain
- **Branching** — one or more forks in the path
- **Hub and spoke** — a central zone connects to several peripheral zones
- **Complex** — a non-trivial graph with loops and multiple paths

The entry zone is always revealed and scouted on arrival. All other zones begin as Unknown.

### 8.4 Zone Objects

#### 8.4.1 Object Characteristics

Every object that can appear in a POI zone has the following characteristics. Some are fixed by the object's type; others are assigned randomly when the POI spawns.

| Characteristic | Description |
|---|---|
| **Name** | A display label shown to the player. The name is distinct from the type — it conveys flavour and sets player expectations but has no effect on game logic. Two objects of the same type may have different names (e.g. "crumbling wall" and "stone barricade" are both of type Rock). |
| **Type** | A fixed classification that determines which actions can be applied to the object. The type list is defined in Section 8.4.2. |
| **Size** | A numeric value. Objects with a size small enough to fit within a character's available carrying capacity can be picked up and placed in their inventory using the Take action. |
| **Visibility** | The faction visibility value at which a new faction's awareness of this object is initialised when they first enter the zone. Derived from the object's `_discover_value` and `_visibility_value` generation parameters. See Section 4.3.1 for the full derivation. |
| **Upper bound visibility** | The numeric ceiling for faction visibility toward this object. Faction visibility cannot exceed this value. Rolled at spawn as a random value within ±20% of `faction_visibility_max`. See Section 4.3.1. |
| **Faction visibility map** | *Runtime state — not assigned at spawn.* Stores the current numeric faction visibility value for each faction that has entered the zone. Used to evaluate Undiscovered / Known / Located thresholds (see Section 4.3.2). Initialised for a faction when they first enter the zone, set to the object's `visibility` value. |
| **Gold value** | A range (min–max) representing how much the object is worth if carried back to a city and sold. The actual sale price is determined when the object is sold. Objects with a gold value of 0 have no market value. |
| **Tags** | A list of special modifiers that alter how the object interacts with the zone or with characters. See Section 8.4.3 for the tag list. |
| **Special actions** | One-off actions generated at spawn that apply only to this specific object instance. These are distinct from the standard actions available to all objects of the same type. |

#### 8.4.2 Object Types

The object type determines which standard actions are available on the object. Each type maps to a set of common and object-specific actions defined in Section 4.6.

| Type | Standard actions | Notes |
|---|---|---|
| **Rock** | Examine, Gather resources, Move | Yields stone or ore. Requires a tool for gathering. |
| **Tree** | Examine, Cut down, Gather resources | Yields timber and firewood. Cut down requires sharp tool ≥ 70%. |
| **Bush** | Examine, Gather resources, Search, Cut down | May yield berries, herbs, or concealed objects. |
| **Plant** | Examine, Gather resources | Yields food, herbs, or other organic material depending on species. |
| **Pond** | Examine, Use | Refills water flasks. May contain fish or hidden objects. |
| **River** | Examine, Use, Move (cross) | Refills water flasks. Crossing costs extra movement turns. |
| **Mushroom** | Examine, Take, Gather resources | Typically small enough to be taken directly into inventory. |
| **Fungal tree** | Examine, Cut down, Gather resources | Large fungal growth. Yields fungal material. Cut down requires sharp tool ≥ 50%. |
| **Wooden house** | Examine, Search | May contain containers and items. |
| **Wooden barn** | Examine, Search | Typically contains supplies, tools, or animals. |
| **Stone house** | Examine, Search | Sturdier structure; may contain locked areas. |

#### 8.4.3 Object Tags

Tags are modifiers attached to individual object instances at spawn. An object may carry multiple tags simultaneously.

| Tag | Effect |
|---|---|
| **Light source (X%)** | The object emits light. When the object is active or interacted with, the zone's effective light level is calculated as though increased by X%. Multiple light source tags in the same zone stack additively. |
| **Edible** | The object can be consumed as food. Carries a Food (value) tag indicating how many hunger charges are removed when one unit is consumed. May additionally carry a Delicious (X) tag granting X loyalty when consumed during Rest. Consumed via the Eat action or the Rest action. See Section 5.6.2. |
| **Toxic** | Contact or consumption applies a Poisoned wound instance to the character. |
| **Flammable** | The object can be set alight. When burning it gains a Light source tag and may spread fire to adjacent flammable objects. |
| **Concealment** | Characters hiding near or inside this object receive a bonus to their visibility reduction. |
| **Locked (condition)** | The object cannot be opened or accessed without satisfying the specified tool condition (e.g. key ≥ 100%). |
| **Delicious (X)** | When the object is consumed during the Rest action, X is added to the consuming character's loyalty in addition to the standard +1 rest loyalty bonus. Has no effect when consumed via the Eat action outside of rest. |
| **Fragile** | The object can be destroyed by a single Attack action or by incidental contact during combat. |
| **Heavy** | The object cannot be moved by a single character. Requires multiple characters or a specific tool condition to move. |

#### 8.4.4 Unknown Object Display

Objects that are Known but not yet Located are displayed with a placeholder representation rather than their true appearance. This allows a player to know something exists in a zone without knowing exactly what it is. The placeholder name and icon are determined by a broad category mapping:

| True type | Placeholder display name |
|---|---|
| Wooden house | Unidentified building |
| Wooden barn | Unidentified building |
| Stone house | Unidentified building |
| Rock | Stone formation |
| Tree / Fungal tree | Large growth |
| Bush / Plant | Vegetation |
| Pond / River | Water source |
| Mushroom | Small growth |

The placeholder removes type-specific detail but may still convey partial information — for example, a Known object categorised as "Unidentified building" tells the player a structure is present without revealing its contents or layout.

#### 8.4.5 Object Icons and Character Portraits

**Icon shape rules**

The shape of the icon displayed to the left of an entry in any object or character list is determined by whether the entry represents a living creature or a non-living object:

- **Non-living objects** — displayed with a **square** icon containing the object's image. This applies to all environmental objects, items, containers, structures, and any other non-living entity.
- **Living characters and animals** — displayed with a **rhombus** (diamond) icon containing the character's portrait. The rhombus shape is the universal visual indicator that an entry represents a living being capable of independent action.
- **Dead character bodies** — when a character dies their body becomes a non-living object. It is displayed with a **square** icon containing a standardised "dead body" image, not a portrait. The rhombus shape is never used for dead characters.

This distinction applies consistently across all UI contexts where object or character lists appear: the Zone Detail Panel object list, the roster, the group panel, and any other panel that displays a mix of living and non-living entries.

**Character portraits**

Every character in the game has a unique portrait image. Portraits are used wherever that character appears in the UI — in the Zone Detail Panel, the roster, the group panel, and the character detail view. All portrait images are displayed inside a rhombus shape.

Portraits represent the character's appearance and are assigned at initialisation. They do not change over time and are not affected by wounds, equipment, or status effects.

**Object icons**

Each object type has a distinct icon. Objects of the same type but different name or size may use variant icons to help the player distinguish them at a glance — for example, two objects both named "wild plant" may have different icons if one is visually larger or bears berries, enabling the player to prioritise task assignment without reading every entry in detail.

Icon variants are determined at spawn based on the object's size, name, and tags. The full icon set and portrait set are defined in the visual assets specification.

---

## 9. Economy

### 9.1 Gold

Gold is the single most important resource. It is earned by:

- Selling items staged in the Storeroom's For Sale section (see Section 9.4)
- Selling surplus equipment

Gold is spent on:

- Recruit upkeep (paid monthly, deducted on the first turn of the last day of each in-game month)
- Hiring new recruits
- Purchasing food, equipment, and weapons
- Relocating HQ

The guild dissolves (game over) if gold reaches zero and no income is expected within the next few turns (configurable threshold warning before hard game over).

### 9.2 Supplies

Food is purchased in the city market and stored at HQ. When a group departs on an expedition, the player allocates food from HQ stock to individual characters.

**During the Rest action**, a character consumes food from their inventory until their hunger charges are reduced to 20 or below in the Hunger stage, eating through Starvation and Famine stages if necessary. Each item consumed reduces charges by its Food (value). Loyalty bonuses from Delicious-tagged food also apply at this point (see Section 5.6.2).

**Between rest turns**, a character's hunger charges accumulate each turn. If hunger progresses to Starvation or Famine and the character performs the Eat action (5 AP), one food item is consumed and its Food value is subtracted from the current stage's charges, potentially reversing hunger progression. Characters with hunger charges above 80 automatically insert a Feed Self sub-goal into the group task list if the group's Allow self-management setting is enabled (see Section 5.6.2 and Section 13.5).

If a character has no food available and hunger is left untreated, it progresses through increasingly severe stages and ultimately causes death (see Section 5.6).

A supply group can be sent to a deployed group's current location to top up their food and swap out equipment.

### 9.3 Equipment and Weapons

Equipment (armour, tools) and weapons are purchased in the city Market. The Market displays items divided into three stock categories:

- **Permanent stock** — basic items always available. Inventory does not deplete and does not rotate.
- **Rotating stock** — higher quality items that appear in limited quantities and are replaced at the start of each new in-game week. Once a rotating stock item is purchased it is gone until the next rotation.
- **Rare drops** — occasionally a rare or unique item appears; limited to one or two units and does not restock once purchased.

The breadth and quality of rotating stock and rare drops is influenced by the city's **size**. Larger cities offer more rotating stock slots and a higher probability of rare drops appearing. Permanent stock is identical across all city sizes.

Each item in the Market displays its gold value range (min–max). The purchase mechanic — including the For Purchase staging box, the market tax, and the price roll — is documented in Section 9.5.

Unequipped items are stored at HQ in the Storeroom. Equipment is assigned to individual recruits in the Roster panel.

---

### 9.4 Selling Items

#### 9.4.1 Returning Items to the Storeroom

When a group returns to HQ, all non-equipped items carried by its members are automatically transferred to the Storeroom. Equipped items (those occupying a character's Garment, Head, Backpack, or Trinket slots) are not transferred — they remain on the character until manually unequipped. The Storeroom has no capacity limit and accepts all item types: equipment, weapons, food, and any objects picked up from POI zones.

#### 9.4.2 The For Sale Section

The Storeroom panel contains a dedicated **For Sale** section. The player can move any items from the main Storeroom inventory into the For Sale section at any time. Items staged here are not removed from the guild's possession until the player confirms the sale — they can be moved back to the main inventory freely before that point.

Each item in the For Sale section displays its gold value range (min–max) as defined on the item. The combined sale value of all staged items is shown as a running total, calculated by summing the lower bounds of all item ranges for the minimum and the upper bounds for the maximum:

> **total_min = Σ item.goldValueMin**
> **total_max = Σ item.goldValueMax**

For example, if the For Sale section contains a Stone [2–10] and a Bush [4–8], the displayed range is [6–18].

#### 9.4.3 Confirming a Sale

When the player clicks **Sell**, a single random integer is rolled in the range [total_min, total_max] inclusive. This value is added to the guild's gold balance immediately. All items in the For Sale section are removed from the guild's inventory. The transaction is recorded in the Ledger.

If the For Sale section is empty, the Sell button is disabled.

---

### 9.5 Purchasing Items

#### 9.5.1 The For Purchase Box

The Market panel contains a **For Purchase** box. The player drags items from any of the three stock categories into this box to stage them for purchase. Items can be removed from the box freely before confirming. The box displays each staged item with its gold value range (min–max).

The combined price range of all staged items is calculated by summing the lower bounds for the minimum and the upper bounds for the maximum, identical to the selling mechanic:

> **total_min = Σ item.goldValueMin**
> **total_max = Σ item.goldValueMax**

#### 9.5.2 Market Tax

All purchases are subject to a **market tax** displayed prominently in the Market panel. The default tax rate is **10%**. The tax is applied to the rolled price after the random value is determined (see Section 9.5.3). It exists to prevent players from profiting by purchasing items from the Market and immediately selling them back through the Storeroom, as the randomness in both operations would otherwise allow occasional arbitrage gains.

The taxed maximum is shown to the player alongside the base price range so they know the worst-case cost before committing:

> **taxed_max = ⌈total_max × (1 + tax_rate)⌉**

#### 9.5.3 Confirming a Purchase

When the player clicks **Buy**, the following steps are applied:

1. A single random integer is rolled in the range [total_min, total_max] inclusive.
2. The tax is applied: **final_price = ⌈rolled_value × (1 + tax_rate)⌉**
3. The final price is subtracted from the guild's gold balance.
4. All items in the For Purchase box are moved into the guild's Storeroom inventory.
5. The transaction is recorded in the Ledger.

All intermediate values are rounded up to the nearest integer at each step.

**Example:** the For Purchase box contains two items with ranges [6–20] and [8–30]. The combined range is [14–50]. The taxed maximum is ⌈50 × 1.10⌉ = 55. A value of 30 is rolled. The final price is ⌈30 × 1.10⌉ = 33 gold.

#### 9.5.4 Purchase Affordability Check

The player cannot initiate a purchase if the guild's current gold balance is less than the taxed maximum:

> **guild.gold ≥ taxed_max** must be true for the Buy button to be enabled.

This check uses the taxed maximum — the worst possible outcome — to guarantee the transaction can always complete regardless of the roll result. If the guild cannot afford the worst case, the Buy button is disabled. The player must remove items from the For Purchase box until the taxed maximum falls within the available balance.

If the For Purchase box is empty, the Buy button is disabled regardless of gold balance.

---

## 10. World Generation

The global map is procedurally generated at the start of each playthrough.

**Fixed elements:**
- Exactly 3 cities placed at stable positions with guaranteed spacing
- Road connections between cities
- One of the 3 cities is chosen at random to serve as the player's starting HQ city

**Dynamic elements:**
- Terrain (forest, plains, hills, water) generated via noise function
- POI spawn zones are determined by terrain type using internal affinity weights — certain POI types are more likely to appear in certain terrain. This logic is internal to the generation system and is not communicated directly to the player.
- POIs spawn over time in valid zones weighted by terrain affinity

**Map size:** set in the World Options menu at game start (see Section 12.2). Default is 128×128 tile units; valid range is 64–256. The map is always square. The WebGL canvas renders only the visible portion with pan and zoom.

---

## 11. Technical Architecture

### 11.1 Rendering

- WebGL 2D canvas fills the full viewport
- The Global Map and POI Layout are rendered directly on the WebGL canvas
- UI panels (City Layout, Zone Detail, HQ menus) are HTML/CSS overlaid on the canvas
- The canvas renders tile terrain, POI markers, group tokens, and zone graphs
- All sprite/tile graphics are procedurally drawn (colored shapes and outlines) — no external image assets required for v1

### 11.2 Game State

All game state is held in a single JavaScript object. The full schema reflecting all currently documented systems is below.

```
GameState {
  turn:          number,
  world:         World,
  organizations: [ Organization ],   // array supports future rival guilds
  playerOrgId:   string
}

// ── World ──────────────────────────────────────────────

World {
  tiles:  [ Tile ],        // terrain grid
  cities: [ City ],
  pois:   [ POI ]
}

Tile {
  x, y:        number,
  terrainType: string      // "forest" | "plains" | "hills" | "water"
}

City {
  id, name:    string,
  x, y:        number      // map coordinates
}

// ── POI ───────────────────────────────────────────────

POI {
  id, name:      string,
  type:          string,
  x, y:          number,   // map coordinates
  despawnTimer:  number,   // turns remaining
  zones:         [ Zone ],
  discovered:    bool      // whether the player can interact with this POI; all POIs are always visible on the map but only Discovered ones can be entered or travelled to
}

Zone {
  id, name:         string,
  size:             number,
  lightLevel:       number,   // 0–100
  weatherEffects:   [ string ],
  hazardConditions: [ HazardCondition ],
  ownerFaction:     string | null,
  objects:          [ ZoneObject ],
  state:            "Unknown" | "Scouted",
  factionAwareness: { [factionId]: { [objectId]: number } }
  // Stores the current faction visibility value per faction per object.
  // Awareness state (Undiscovered / Known / Located) is derived from this value
  // at runtime using the thresholds in Section 4.3.2. Not stored as a string enum.
}

HazardCondition {
  triggerAction:  string,     // action name that triggers the roll
  damageEvents:   [ { probability: number, damageMin: number, damageMax: number } ]
}

ZoneObject {
  id:                   string,
  name:                 string,
  type:                 string,     // see Section 8.4.2
  size:                 number,
  discoverValue:        number,     // 0–4; controls the scale of faction_visibility_max
  visibilityValue:      number,     // 0–4; controls starting position on the visibility spectrum
  visibility:           number,     // rolled at spawn; faction visibility initialised to this on zone entry
  upperBoundVisibility: number,     // rolled at spawn; hard ceiling for all faction visibility values
  goldValueMin:         number,
  goldValueMax:         number,
  tags:                 [ Tag ],
  specialActions:       [ Action ],
  isAlive:              bool        // runtime flag — false for static objects; true for living animals/characters. Changes to false when the entity dies; does not reset.
}

Tag {
  name:  string,              // e.g. "Food", "Delicious", "Light source", "Edible"
  value: number | null        // the X in Food(X), Delicious(X), Light source(X%)
}

// ── Organization ──────────────────────────────────────

Organization {
  id, name:      string,
  gold:          number,
  hqCityId:      string,
  criminalRecord: [ Offence ],   // prepared for future use; empty in v1
  recruits:      [ Recruit ],
  groups:        [ Group ],
  inventory:     {
    food:      [ Item ],
    equipment: [ Item ],
    weapons:   [ Item ],
    misc:      [ Item ],    // zone objects converted to items by being picked up
    forSale:   [ Item ]     // items staged in the For Sale section; cleared on sale
  }
}

Offence {
  turn, zoneId, poiId, actorId, victimId: string,
  description: string
}

// ── Recruit / Character ───────────────────────────────

Recruit {
  id, name:       string,
  portrait:       string,         // portrait asset reference
  race:           string,         // e.g. "human"; determines wound table and base characteristics
  level:          number,         // 1–10
  upkeepBaseRate: number,         // gold per level per month
  traits:         [ string ],     // max 2; trait names from Section 5.5
  characteristics: {
    health:          number,      // base health 90–130
    regeneration:    number,
    movementSpeed:   number,
    baseCarryCapacity: number,    // 5–10 before backpack
    strength:        number,
    dexterity:       number,
    perception:      number,
    loyalty:         number       // 0–30
  },
  equipment: {
    garment:  Item | null,
    head:     Item | null,
    backpack: Item | null,
    trinket:  Item | null,
    trinket2: Item | null         // available with Provident trait
  },
  inventory:    [ Item ],
  statusEffects: [ StatusEffect ],
  wounds:        [ Wound ],
  hunger:        HungerState,
  incapacitated: IncapacitatedState | null,
  state: "Available" | "Deployed" | "Wounded" | "Incapacitated" | "Dead"
  // Note: Wounded and Incapacitated are not mutually exclusive — a character may carry
  // active wounds (wounds[]) while also being Incapacitated (incapacitated != null).
  // The state field reflects the dominant condition for UI and scheduling purposes:
  // Incapacitated takes precedence over Wounded when both are true.
}

// ── Items ─────────────────────────────────────────────

Item {
  id, name: string,
  type:     string,
  size:     number,
  tags:     [ Tag ],
  goldValue: { min: number, max: number }
}

// ── Status Effects ────────────────────────────────────

StatusEffect {
  type:           string,
  charges:        number,
  maxCharges:     number,
  chargePerTurn:  number,   // positive = worsening; negative = healing
  effects:        [ Effect ]
}

Effect {
  stat:      string,        // e.g. "maxHealth", "strength", "movementSpeed"
  modifier:  number,        // flat or percentage depending on stat
  isPercent: bool
}

// ── Wounds ────────────────────────────────────────────

Wound {
  type:          string,    // e.g. "LightArmWound_Left"
  category:      "Progressive" | "Stackable" | "Special",
  stage:         string,    // current stage name
  charges:       number,
  maxCharges:    number,
  degenerationRange: { min: number, max: number },
  isBandaged:    bool,
  effects:       [ Effect ]
}

// ── Hunger ────────────────────────────────────────────

HungerState {
  stage:    "Hunger" | "Starvation" | "Famine" | "StarvedToDeath",
  charges:  number
  // StarvedToDeath is the terminal stage. When Famine reaches max charges the stage
  // transitions to StarvedToDeath and the character dies immediately. charges is set
  // to 1 (matching the status effect definition in Section 5.6.1) and does not change
  // further. This stage persists on the body after death so that examining characters
  // can identify starvation as the cause of death.
}

// ── Incapacitated ─────────────────────────────────────

IncapacitatedState {
  charges:       number,    // current; depletes by 20 per turn
  maxCharges:    200
}

// ── Groups ────────────────────────────────────────────

Group {
  id, name:    string,
  role:        "Scout" | "Combat" | "Supply" | "Rescue",
  memberIds:   [ string ],  // recruit ids
  locationPoiId: string | null,
  locationZoneId: string | null,
  currentCityId:  string | null,
  inTransitTo:    { type: "city" | "poi", targetId: string } | null,
  schedule:    [ bool ],    // 6-element array; true = rest turn
  taskList:    [ Task ],
  actionLog:   [ LogEntry ],  // rolling 20-turn history; [] for new groups
  config: {
    zoneTravelAllowed:            bool,
    giveUpTurnCount:              number,
    overrideMaintenanceLifecycle: bool,
    allowSelfManagement:          bool    // if false, characters cannot insert tasks autonomously
  }
}

LogEntry {
  turn:        number,
  characterId: string | null,  // null for group-level events (movement, arrival)
  type:        string,         // e.g. 'action', 'damage', 'status', 'loyalty', 'state', 'item', 'movement'
  description: string          // human-readable summary
}

Task {
  id:         string,
  action:     string,       // action name or goal type
  assigneeId: string | null,
  goalLifetimeRemaining: number | null,
  giveUpCounter:         number
}
```

### 11.3 Save and Load

Since GitHub Pages is static (no server), saving is handled client-side:

- **Save** — serializes `GameState` to JSON and triggers a browser file download
- **Load** — prompts a file upload dialog; parses the JSON and restores state
- **Autosave** — optionally writes state to `localStorage` as a backup every N turns

### 11.4 Single File Constraint

For v1 the entire application ships as a single `index.html` containing inlined CSS, JavaScript, and WebGL shaders. No build system, no npm, no external dependencies except Google Fonts (optional). This matches the pattern of the other projects in the repository.

---

## 12. UI Layout

Full interface documentation is maintained in a separate document: **Ironbound — Interface Design Document** (`ui.md`). That document specifies every screen, panel, interactive element, and navigation flow in detail. The high-level layout sketch in Section 12.4 provides a structural overview; implementers should refer to `ui.md` for the complete specification.

The Interface Design Document declares which version of this GDD it was written against. If the GDD version listed in `ui.md` does not match the current GDD version, the two documents may be out of sync and should be reconciled before implementation continues.

### 12.1 Launch Menu

The launch menu is the first screen shown when the game page is loaded in the browser. It is displayed before any world is initialised and replaces the main game view entirely until the player makes a selection.

**Options:**

- **Start new game** — transitions the player to the World Options menu (see Section 12.2).
- **Load game** — opens the browser's native file picker. The player selects a previously saved `.json` save file. Upon selection the file is parsed, the saved world state is restored, and the game opens at the Global Map layout from the point at which the game was saved.
- **Launch test scenario** — transitions the player to the Test Scenario menu (see Section 12.3).

**Behaviour notes:**
- No other interaction is available while the launch menu is visible.
- If a selected save file cannot be parsed or is invalid, an error message is displayed and the launch menu remains open.
- The launch menu has no back button — navigating away requires refreshing the page.

### 12.2 World Options Menu

Shown when the player selects Start new game. The player configures the parameters of the new world before generation begins. All fields have defaults so the player can proceed immediately without changing anything.

| Field | Input type | Default | Constraints | Description |
|---|---|---|---|---|
| Organization name | Text input | "The Guild" | Any non-empty string | The name of the player's organization. |
| Starting gold | Numeric input | 4200 | Any value | The gold balance the organization begins with. |
| Starting characters | Numeric input | 0 | 0 – 9 | Number of randomly generated recruits placed in the organization's roster at game start. These recruits are randomly generated with a random assortment of equipment. They are all placed in a single group located at HQ with no active goals or tasks. |
| Target world POIs | Numeric input | 20 | 10 – 50 | How many POIs the world attempts to sustain simultaneously. The spawn system will generate new POIs when the active count falls below this target. |
| Map size | Numeric input | 128 | 64 – 256 | The width and height of the generated world map in tile units. The map is always square. Larger maps spread cities and POIs further apart, increasing travel distances. |
| Target publicly known POIs | Numeric input | 8 | 6 – Target world POIs | How many POIs spawn as Discovered for all organizations by default, regardless of Gather Rumors activity. The remaining active POIs spawn as Undiscovered — their location is visible on the map but they cannot be interacted with until discovered via references or future Gather Rumors mechanics. Must not exceed Target world POIs. |

Once the player confirms, the world is generated using these parameters and the game opens at the Global Map layout.

### 12.3 Test Scenario Menu

Shown when the player selects Launch test scenario from the launch menu. The player is presented with a list of available pre-defined test scenarios. Each scenario is a fixed world state designed to test a specific system or mechanic.

Selecting a scenario from the list displays a brief description of what the scenario tests. The player then confirms to launch it. The game loads the pre-defined world state and opens at the Global Map layout.

**Behaviour notes:**
- Test scenarios are hardcoded into the game and do not require external files.
- The list of available scenarios expands as new systems are implemented.
- A Back button returns the player to the launch menu without loading anything.

### 12.4 In-game Layout Sketch

```
┌─────────────────────────────────────────────────────┐
│  TOOLBAR: Turn counter │ Gold │ Roster count │ [END TURN]  │
├─────────────────────────────────────────────────────┤
│                                                     │
│            WebGL Canvas (active layout)             │
│    Global Map  /  POI Zone Graph  /  City Menu      │
│                                                     │
├─────────────────────────────────────────────────────┤
│  BOTTOM PANEL: context-sensitive                    │
│  (group details / zone actions / market panel)      │
└─────────────────────────────────────────────────────┘
```

The toolbar is always visible. The canvas switches between layouts. The bottom panel responds to what the player has selected.

---

## 13. Group Automation and Task System

### 13.1 Overview

Each group maintains a **task list** — an ordered queue of tasks that defines what the group and its members will work on. Tasks are executed in a top-down priority order: tasks at the top of the list are checked for assignment before those below them. A player can manage this list manually, abstract goals can populate it automatically, and characters can add their own tasks to it in response to their needs.

### 13.2 Task Structure

A **task** is a pairing of an action and an optional assignee:

```
Task {
  action:   zone action or abstract goal (see Sections 4.5, 4.6, 13.3)
  assignee: character | null
}
```

- **action** — the specific action or goal the task represents.
- **assignee** — the character responsible for executing this task. May be null when a task is first added to the list.

Tasks with no assignee wait in the list until the group's task assignment step (see Section 13.6 turn order) pairs them with an available group member. Tasks are never discarded solely because they lack an assignee — they remain in the list until completed, cancelled, or expired.

### 13.3 Task Assignment

During the task assignment step of the turn order, each group's task list is scanned from top to bottom. For each task without an assignee, the system checks whether any group member is currently unoccupied (has no active task). If one is found, the best-suited available member is assigned — selected based on relevant characteristics and equipped tools. If no unoccupied member is available, the task remains unassigned and is checked again next turn.

A group member who already has an active task is not reassigned. They continue their current task until it completes or is cancelled.

**Character-initiated tasks:**

Characters may add tasks to the group task list themselves and self-assign, provided the group's **Allow self-management** setting is enabled (see Section 13.5). This happens automatically when a character determines that a personal maintenance action is needed — such as hunger management (see Section 5.6.2) or applying wound treatment. The self-assigned task is placed immediately below the task the character is currently performing, and the character marks themselves as the assignee immediately. When the character finishes their current task, they begin the self-assigned one.

If Allow self-management is disabled, characters cannot insert tasks into the group task list on their own. All maintenance is then driven by the maintenance lifecycle (Section 13.6) or by the player directly.

### 13.4 Task Types

Tasks fall into two categories:

**Local actions** — a direct instruction to perform a specific zone action (see Sections 4.5 and 4.6) at a specific location. Local actions map directly to a single action and do not require decomposition. Examples: scout a specific zone, gather resources from a specific object, treat an injured character.

**Abstract goals** — a high-level objective that the group breaks down automatically into a sequence of sub-tasks, each of which is added to the task list in order. Goals continue generating sub-tasks until the goal is complete, the goal lifetime expires, or the give-up threshold is reached.

**Available goals:**

| Goal | Description |
|---|---|
| **Gather valuables** | Scouts the POI to uncover objects, then prioritises and gathers items with the highest gold value. Continues until the goal lifetime expires or all reachable valuables have been collected. |
| **Gather specific resource** | Scouts until a source of the desired resource type is Located, gathers from it, then repeats until the lifetime expires or the group is full. |
| **Explore** | Systematically scouts zones within the POI to advance object awareness across as many zones as possible. |
| **Encamp** | Travels to the designated POI, builds a camp structure in the entry zone, then shifts to standard group maintenance lifecycle. |

**Goal decomposition example — Gather valuables:**

1. Travel to the designated POI
2. Scout the entry zone to surface Located objects
3. Identify the highest gold-value Located objects
4. Assign appropriate group members to gather or take those objects
5. Wait for current tasks to complete
6. Return to HQ

### 13.5 Group Task Configuration

The following settings can be configured per group:

| Setting | Default | Description |
|---|---|---|
| **Zone travel allowed** | Yes | Whether the group may move freely between internal zones of a POI to fulfil a task. If No, the group only acts within the zone it currently occupies. |
| **Give-up turn count** | 7 | If a sub-task generated by a goal fails to complete for this many consecutive turns, the parent goal is cancelled along with all its remaining sub-tasks, and the group proceeds to the next item in the task list. |
| **Goal lifetime** | Configurable | The maximum number of turns a goal may run, counted from the turn it begins executing. When the lifetime expires the goal is removed from the task list. |
| **Override maintenance lifecycle** | No | If Yes, the group ignores its maintenance lifecycle checks and no maintenance tasks are inserted automatically. |
| **Allow self-management** | Yes | If Yes, individual characters may insert sub-goals and tasks into the group task list on their own when personal maintenance conditions are met (hunger, wound treatment). If No, characters cannot modify the task list autonomously — all task management is handled by the maintenance lifecycle or the player. |

### 13.6 Group Maintenance Lifecycle

The group maintenance lifecycle is a set of automatic checks performed each turn before the task assignment step. When a maintenance condition is detected, a task is inserted at the top of the task list (above all existing tasks) and assigned to the relevant character immediately. The lifecycle can be suppressed per group using the override setting.

**Checks performed each turn:**

**Injured members:**
If any group member is wounded and has not yet received treatment, the lifecycle inserts a medical treatment task at the top of the list, assigned to the wounded character. If no medical supplies are available in the group's inventory, a supply-gathering sub-task sequence is generated first.

**Food supply:**
If any group member has no food items in their inventory, the lifecycle first checks whether redistributing food from members with surplus would resolve the shortfall. If yes, it inserts a food redistribution task. If no, it inserts a food-gathering sub-task sequence.

### 13.7 Turn Order

The following sequence defines the order in which all game systems resolve at the start of each turn:

1. **World consistency check** — expired POIs are removed. If the active POI count falls below the target, new POIs are generated and placed on the map.
2. **Monthly financial operations** *(first turn of the last day of a month)* — upkeep payments are deducted from the guild's gold balance. Criminal record fine processing is reserved here for a future update but is not active in v1.
3. **Weekly rotation** *(first turn of the first day of a new week)* — the recruit hiring pool in the Tavern is refreshed. Market item inventories in cities are rotated.
4. **Maintenance lifecycle checks** — each group's maintenance lifecycle is evaluated. Maintenance tasks are inserted into the task queue where applicable.
5. **Task assignment** — each group's task list is scanned top to bottom. Unassigned tasks are paired with available group members based on suitability. Members already executing a task continue without interruption.
6. **Character action determination** — each character evaluates their assigned task and determines which zone actions to take this turn based on available action points.
7. **Action resolution** — all character actions are applied. Actions resolve simultaneously.
8. **Status effect update** — wound charges, hunger charges, loyalty modifiers, and all other status effects are updated. Character states and characteristics are recalculated to reflect any changes.

---

## 14. Dev Mode

Dev Mode is a special overlay that exposes all hidden game state to the player for testing and debugging purposes. It can be toggled on or off at any time during play.

### 14.1 What Dev Mode Reveals

When Dev Mode is active the following normally hidden information becomes visible:

**Global Map:**
- All POIs currently active in the world are already visible on the map (see Section 3.1). Dev Mode additionally reveals full details on Undiscovered POIs — their type, despawn timer, and zone count — which are normally hidden from the player until the POI is Discovered. Undiscovered POIs are displayed with a distinct visual indicator in Dev Mode to distinguish them from Discovered ones.

**POI Layout:**
- All zones within the POI, including those in the Unknown state. Unknown zones are displayed with their actual contents rather than a question mark placeholder.

**Zone Detail Panel:**
- All objects in the zone regardless of their awareness state — Undiscovered, Known, and Located objects are all listed and displayed with their full characteristics: name, type, size, visibility value, gold value range, tags, and all other properties.
- The faction visibility map for each object, showing which factions have Located, Known, or not yet discovered each object.
- The current charges and max charges of all active wounds and hunger stages on every character in the zone.

### 14.2 Interaction Constraints

Dev Mode is view-only for information that the player's faction has not yet discovered. Seeing an object in Dev Mode does not grant the ability to interact with it. A player-controlled character can only act on objects that their faction has Located through normal awareness mechanics. Dev Mode does not bypass this restriction.

### 14.3 Visual Indicator

While Dev Mode is active a persistent indicator is displayed in the toolbar to make it clear the player is viewing hidden information. All Dev Mode-exclusive information is rendered with a distinct visual style (e.g. a different colour or opacity) so it is immediately distinguishable from normally visible game state.

---

## 15. Resources

### 15.1 Item List

Every item in the game has a defined set of tags, a gold value range, and a size. Tags determine what the item can be used for — as a tool, food source, equipment piece, or weapon. Gold value is rolled within the defined range at the point of sale. Size determines how many inventory capacity units the item occupies when unequipped.

Items are divided into three availability categories based on how they enter the game.

---

#### 15.1.1 Purchasable Items — Unlimited Supply

These items are always available in the city Market under permanent stock. They do not deplete and do not rotate.

| Item | Tags | Gold value | Size | Notes |
|---|---|---|---|---|
| **Knife** | One-Handed, Sharp tool (100%), Melee Weapon | 7–10 | 2 | — |
| **Rope** | Climbing tool (70%) | 5–5 | 3 | — |
| **Bandage** | Healing | 5–10 | 1 | — |
| **Bread** | Food (70), Delicious (1) | 8–8 | 2 | — |
| **Fruits** | Food (30), Delicious (1) | 2–12 | 1 | — |
| **Meat** | Food (80), Delicious (1) | 6–22 | 2 | — |
| **Backpack** | Equipment (backpack) | 20–30 | 4 | When equipped: carrying capacity +10, movement speed −1 |
| **Bow** | Hunting tool (100%), Ranged Weapon, Two-Handed | 25–40 | 4 | — |
| **Cloth** | Equipment (garment) | 10–12 | 4 | Armor value: 5 |
| **Torch** | Lighting tool (80%) | 4–4 | 2 | — |
| **Lamp** | Lighting tool (100%) | 20–20 | 1 | — |

---

#### 15.1.2 Purchasable Items — Limited Supply

These items appear in the city Market as rotating or rare stock. They are available in limited quantities and are replaced or removed at the weekly rotation. Once purchased they are gone until the next appearance.

| Item | Tags | Gold value | Size | Notes |
|---|---|---|---|---|
| **Exotic Fruits** | Food (40), Delicious (3) | 10–15 | 1 | — |
| **Dried Meat** | Food (100) | 15–15 | 1 | — |
| **Fish** | Food (60), Delicious (1) | 6–8 | 1 | — |
| **Sword** | Sharp tool (40%), Melee Weapon, Two-Handed | 40–45 | 5 | — |
| **Axe** | Sharp tool (20%), Melee Weapon, One-Handed | 15–17 | 4 | — |
| **Leather Armor** | Equipment (garment) | 20–30 | 6 | Armor value: 25 |
| **Chainmail** | Equipment (garment) | 80–80 | 8 | Armor value: 35. When equipped: dexterity −5 |
| **Plate Armor** | Equipment (garment) | 300–300 | 9 | Armor value: 50. When equipped: dexterity −8 |
| **Large Backpack** | Equipment (backpack) | 40–45 | 5 | When equipped: carrying capacity +15, movement speed −1, dexterity −4, perception −2 |
| **Metal Helmet** | Equipment (head) | 50–50 | 3 | Armor value: 30 |
| **Leather Cloak** | Equipment (head) | 10–20 | 3 | Armor value: 20 |
| **Fish Net** | Fishing tool (100%) | 20–20 | 3 | — |
| **Pickaxe** | Mining tool (100%), Two-Handed, Melee Weapon | 30–30 | 6 | — |

---

#### 15.1.3 Non-Purchasable Items

These items cannot be bought in any city Market. They are found exclusively through looting POI zones, gathering from zone objects, or hunting. They are the primary source of guild income from expeditions.

| Item | Tags | Gold value | Size | Notes |
|---|---|---|---|---|
| **Iron Ore** | Ore | 8–12 | 2 | — |
| **Copper Ore** | Ore | 5–10 | 2 | — |
| **Wood** | Resource | 1–2 | 2 | — |
| **Fiber** | Resource | 0–0 | 1 | — |
| **Stone** | Resource | 0–1 | 2 | — |
| **Unidentified Artifact** | Artifact | 150–300 | 1 | — |
| **Hunting Trophy (small)** | — | 10–15 | 1 | — |
| **Hunting Trophy (large)** | — | 50–55 | 3 | — |
| **Berries** | Food (30), Delicious (2) | 3–4 | 1 | — |
| **Exotic Meat** | Food (100), Delicious (1) | 50–50 | 2 | — |
| **Goblet** | — | 20–80 | 1 | — |
| **Jewelry** | — | 10–100 | 1 | — |
| **Exotic Jewelry** | — | 400–500 | 1 | — |
| **Glowing Crystal** | Lighting tool (50%) | 40–60 | 1 | — |
| **Fungal Wood** | Resource | 8–15 | 2 | — |
| **Glowing Mushroom** | Lighting tool (20%) | 20–22 | 1 | — |
| **Glowing Shard** | Lighting tool (60%) | 100–100 | 2 | — |
| **Ancient Artifact Alpha** | Artifact | 800–850 | 1 | — |
| **Ancient Artifact Beta** | Artifact | 900–1000 | 1 | — |
| **Ancient Bones** | — | 200–300 | 8 | — |
| **Basic Supplies** | Resource | 7–45 | 2 | — |

---

## 16. Scope — v1 Release

The following systems are in scope for the first release:

- [x] Global map with WebGL rendering, pan and zoom
- [x] Procedural world generation (terrain + cities)
- [x] Dynamic POI spawning and despawn timers
- [x] POI zone graph layout and rendering
- [x] Zone Detail Panel with action assignment
- [x] City Layout with Market and Recruitment panels
- [x] HQ staff task assignments (Gather Rumors, Trading Connections, Train Recruits, Treat Wounded)
- [x] Single player guild (one organization)
- [x] Recruit roster with traits and permadeath
- [x] Trait system (per-character permanent modifiers, mutual exclusivity)
- [x] Wound and lifecycle system (health, wound categories, progression, treatment)
- [x] Hunger system (staged starvation, feeding, loyalty bonuses)
- [x] Action execution cycle (action points, contribution factors, progress threshold, tool multipliers)
- [x] Group creation and management
- [x] Group automation and task system (local actions, abstract goals, maintenance lifecycle)
- [x] Turn-based time advance with event resolution
- [x] Gold economy and upkeep
- [x] Food supply and expedition management
- [x] Dead character body persistence and inspection
- [x] Character portraits and icon shape rules (rhombus for living, square for objects)
- [x] Zone object system (types, tags, visibility, gold value, faction awareness map)
- [x] Damage system (damage ratio, wound probability formula, incapacitation)
- [x] Race system (per-race wound tables, human race defined)
- [x] Launch menu (new game / load from file)
- [x] Save/load via JSON file
- [x] Dev Mode (full hidden state visibility for testing)

**Out of scope for v1 (planned v2+):**
- Rival organizations
- Diplomacy or faction standing systems
- Crafting
- Multiplayer

---

## 17. Developer Testing System

### 17.1 Overview

The game ships with a built-in developer testing system activated by appending `?dev` to the page URL. When active, a persistent **DEV panel** appears at the bottom of the screen containing a validation console, scenario launcher, and state injection tools. The system is entirely self-contained within the single `index.html` file and requires no external dependencies or build tools.

The DEV system is implemented as a self-executing module assigned to the global `DEV` object. It is inert when `?dev` is not present in the URL — no UI is rendered and no performance overhead is incurred.

### 17.2 Activation

Add `?dev` to the URL when opening the game in a browser:

```
file:///path/to/ironbound.html?dev
```

A `DEV` badge appears in the bottom-right corner of the screen. Clicking it opens the full dev panel. The panel can be collapsed and re-expanded without losing the log contents.

When dev mode is active, `DEV.validate()` is automatically called after every End Turn so the game state is continuously checked as time progresses.

### 17.3 Validation System

`DEV.validate()` runs all registered rules against the current `GameState` object and logs each result to the dev console as a pass or fail. Rules are derived directly from the GDD and are grouped by category:

| Category prefix | Coverage |
|---|---|
| `GS-xxx` | GameState top-level structure and cross-references |
| `WO-xxx` | World: city count, HQ assignment, map bounds, POI fields, tile grid dimensions |
| `OR-xxx` | Organization: required fields, gold type, inventory categories |
| `RC-xxx` | Recruits: required fields, level range, loyalty range, trait count, hunger stage and charge bounds |
| `GR-xxx` | Groups: required fields, member count, member ID validity, schedule format, role enum |
| `WD-xxx` | Wounds: category enum, charge bounds |

Each rule has a unique ID (e.g. `WO-002`), a human-readable description matching the GDD section it enforces, and a check function that receives the full `GameState` and returns either `true` (pass) or a failure description string. The pass/fail count is shown in the log header on each run.

**Rule structure:**

```javascript
{
  id:    'WO-002',
  desc:  'Exactly 3 cities (GDD §10)',
  check: gs => gs.world.cities.length === 3
             || 'Expected 3 cities, got ' + gs.world.cities.length
}
```

All rules are accessible at `DEV.RULES` and new rules can be pushed to this array by future implementations as new systems are added.

### 17.4 Scenarios

Scenarios are pre-defined operations that set up specific game states for testing. They are invoked via `DEV.runScenario(name)` or from the buttons in the dev panel toolbar.

| Scenario | Effect |
|---|---|
| `world_gen` | Creates a fresh game via the normal `startNewGame()` path and immediately runs validation. Used to confirm world generation always produces a valid state. |
| `poi_discovery` | Sets all POIs to `discovered: true` on the current game state and re-validates. Tests that full-discovery states are valid. |
| `corrupt_state` | Intentionally injects four known violations (negative gold, negative despawnTimer, zone count out of range, extra city) and validates. Used to confirm the validator catches all four failures. |

New scenarios are added by extending the `scenarios` object inside `DEV.runScenario()`.

### 17.5 State Injection Helpers

`DEV.inject` contains helper functions that mutate the current game state to create edge cases without requiring manual JSON editing.

| Helper | Effect |
|---|---|
| `DEV.inject.emptyOrg()` | Sets gold to 0, clears recruits and groups. Tests zero-roster states. |
| `DEV.inject.richOrg()` | Sets gold to 99,999 and injects 3 minimal but schema-valid test recruits. |
| `DEV.inject.badPOI()` | Sets the first POI's `despawnTimer` to −1 to trigger `WO-008`. |
| `DEV.inject.allDiscovered()` | Sets all POIs to discovered. |
| `DEV.inject.negativeGold()` | Sets gold to −1,000 to trigger `OR-003`. |

### 17.6 Logging API

Any future implementation code can write to the dev console using `DEV.log(message, level)`.

| Level | Appearance | Use |
|---|---|---|
| `'pass'` | Green ✓ | Rule passed |
| `'fail'` | Red ✗ | Rule failed or invariant violated |
| `'warn'` | Amber ⚠ | Non-fatal unexpected state |
| `'info'` | Dim · | General information |
| `'head'` | Separator | Section divider in the log |

`DEV.IS_DEV` is a boolean that can be checked anywhere in the codebase to conditionally execute diagnostic code without affecting production behaviour.

### 17.7 Extending the System

As new game systems are implemented, the DEV system should be extended in parallel:

- Add new validation rules to the `RULES` array covering GDD constraints for the new system.
- Add new scenarios to `runScenario()` that exercise the new system end-to-end.
- Add new inject helpers for edge cases specific to the system.

The convention is to keep rule IDs stable once defined. If a rule is superseded, mark it as deprecated in its description rather than deleting it, so existing failure logs remain interpretable.

---

*End of document — v4.0-draft*
