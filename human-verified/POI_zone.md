# POI Zone Layout

POI Zone layout is opened once the player clicked on a specific Zone from the POI Zone Graph. This layout is where most of the game actions and activities take place.

# Layout structure

Toolbar is available on the top of the screen just like on any other Layout. It has turn counter, gold, "END TURN" button, "HQ" button and "Group Management" button. In this layout it should also have "Back to POI Zones" button.

Small leftmost section of the screen must be allocated for the Group task queue. It should occupy no more than 7% of the screen width. See "Group task queue".

Middle section of the screen (approx. 33% of screen width) is occupied by the Group Overview panel. Here the state of each group is displayed with more details. See "Groups panel".

Rightmost section of the screen takes the remaining screen space (roughly 60%) and is allocated for the displaying of the zone objects. It is the largest and the most item-dense area of the layout. See "Zone objects panel".

# Group task queue

Group tasks are displayed here. When a user is hovering over a group in the Group panel - the tasks of a group over which the user is currently hovering are displayed. Otherwise the tasks of a group with the last selected character are displayed.

Group task queue is a vertical list of "task" items.

Each "task" is an action that might have a target (square shaped slot), might have an assignee (diamond-shaped slot) and has a progress bar below it displaying its current progress.

Whether task has a target or not is determined by the task type. All tasks always have an assignee slot but characters may be currently assigned or not assigned to the task.

Task types examples (not exhaustive, see game documentation for a full list of tasks):

- Harvest (has a zone object as a target)
- Loot (has a zone object as a target)
- Gather resources (has a zone object as a target)
- Scout (has no target)

A character from a group can be an assignee of a task.

# Groups panel

Group panel displays groups. Each group is a large structure with the following elements:

- Name: text name of a group
- List of characters with portraits: displayed in diamond-shaped form each. Empty character slots must be displayed as empty diamond-shaped slots.
- Inventory: expandable area with item icons and names displaying arbitrary number of items with square icons
- Group log button that opens a text log popup
- Group configuration button that opens a group configuration menu

Whenever a player hovers over a group its tasks are displayed in the leftmost "Group task queue" section. Whenever a player hovers over a character in a group - those tasks that are assigned to the character are highlighted in the Group task queue.

Whenever the player clicks on a character portrait - that character becomes selected and stays selected until a player clicks on an empty space. That characters tasks become highlighted in the Group task queue and stay highlighted until the character is deselected and an object from the Zone objects panel which is the target of the current task being performed by that character (if there is one) is highlighted in the Zone objects panel - the Zone objects panel must be scrolled to that highlighted object if necessary. If the character is not assigned to any tasks - nothing happens. Zone objects scrolling only happens once when a player clicks on a character portrait. After that the player can scroll Zone objects panel however he wants. If a player clicks on an already selected character - Zone objects are scrolled to the target of the task that character is assigned to once again.

Selected characters must become highlighted and they stay highlighted for as long as the character is the currently selected one.

# Zone objects panel

Zone objects panel must be divided into three main sections:

- Located objects (45% of zone panel height): a list of objects player can interact with
- Known objects (45% of zone panel height): a list of objects that the player knows exist but didn't yet located, objects from that list render with their names and icons replaced with type-based "known state appearance"
- Undiscovered (10% of zone panel height): the smallest section of all three, it only holds an explanatory text "There might be unknown objects in this zone. Proceed to exploration in order to uncover them."

All objects existing in the zone, including character groups are displayed in a list of Zone objects in the corresponding category (based on their visibility for the viewer).

"Located-Known" sections of the zone are resizeable so that the player may change the size of each section based on the current needs. These sections are automatically scrolled to a particular object if that object is a target of the task in the task queue and the tasks assignee is being selected.

Each object is displayed with the following properties:

- Icon (Square for non-living objects, Diamond-shaped for characters)
- Character icon if that character is currently performing actions with that object or is assigned a task that targets that object (characters icon is diamond-shaped), displayed to the right of the objects icon
- Name (text), displayed on top of the objects icon
- Visibility percentage (for the viewer), displayed to the left of the objects icon
- Size (in game units, static objects property)
- Material (resource type, displayed only for objects with 100% visibility), to the left of the objects size
- Content (objects inventory, displayed only for objects with 100% visibility), to the left of the objects material

# Object interactions

Known and located objects in the Zone objects panel can be interacted with. When a player clicks on an object it must display a dropdown menu with all possible actions that a currently selected character can perform on that object (characters are selected in the Groups panel). If a player clicks on an option of the actions dropdown menu - that action is added to the tasks list of the currently selected characters Group with the currently selected character as an assignee at the bottom of the task list queue.

# Test scenario
