# POI Zone Layout

POI Zone layout is opened once the player clicked on a specific Zone from the POI Zone Graph. This layout is where most of the game actions and activities take place.

# Layout structure

Toolbar is available on the top of the screen just like on any other Layout. It has turn counter, gold, "END TURN" button, "HQ" button and "Group Management" button. In this layout it should also have "Back to POI Zones" button.

Small leftmost section of the screen must be allocated for the Group task queue. It should occupy no more than 10% of the screen width. See "Group task queue".

Middle section of the screen (approx. 30% of screen width) is occupied by the Group Overview panel. Here the state of each group is displayed with more details. See "Groups panel".

Rightmost section of the screen takes the remaining screen space (roughly 60%) and is allocated for the displaying of the zone objects. It is the largest and the most item-dense area of the layout. See "Zone objects panel".

"Group tasks" - "Group panel" - "Zone objects panel" sections are resizeable so that the player may change the width of each section based on the current needs by dragging the border of between sections.

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

# UI interactions

- When a player hovers over an object in the Zone objects panel - if that object is assigned as a task target of a players Group - that task is highlighter and the character assigned to that task becomes highlighted.
- When a player hovers over a Task in the Group task queue - that tasks target object and assignee character become highlighted.
- When a player clicks on an object it must display a dropdown menu with all possible actions that a currently selected character can perform on that object (characters are selected in the Groups panel). If a player clicks on an option of the actions dropdown menu - that action is added to the tasks list of the currently selected characters Group with the currently selected character as an assignee at the bottom of the task list queue.
- When a player hover over a character in the Group panel - that character, Tasks it is assigned to and Objects which are targets of the tasks that character has as a Task must be highlighted.
- All highlighted objects, tasks and characters must be deselected or highlighting must be cleared from them once a player clicks on an empty space.
- When player clicks on a Group in the Groups panel - that group becomes selected and its tasklist is displayed on the left side in the Task List queue

# Test scenario

For testing purposes a Zone with the following contents must be displayed:

- Group named "Test 1" with characters Bob and Alice in it
- It has two random tasks in its task queue, Bob is assigned to one of them, Alice is not assigned to any of them
- 36 objects including random assortment of "Rocks" and "Trees" with random visibility values must be generated and displayed in their corresponding Zone Objects panel sections (known and located ones, since the player can not see undiscovered objects but only generic text there)
- Group named "Enemy Test 2" with character Wolf in it, this group has no tasks and does not belong to the players faction

All Trees must be displayed with dark-green placeholder squares instead of their icons.

All Rocks must be displayed with dark-grey placeholder squares instead of their icons.

All characters must be displayed with blue placeholder diamond-shaped placeholders instead of their icons.

Every character has material "Meat". Every tree has material "Wood". Every rock has material "Stone". Trees might have a random value between 1 and 3 of an item "Wood" in their content or have none. Every stone might have a random value between 1 and 3 of an item "Iron" in their content or have none. Characters have an item "Sword" in their contents. Wolf has nothing in its content.

