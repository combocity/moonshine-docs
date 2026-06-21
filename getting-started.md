---
layout: page
title: Getting Started
---

This guide walks you through creating your first Moonshine Lua mod.

## Prerequisites

- Moonshine game installed
- Text editor (Visual Studio Code, Sublime Text, Notepad++, etc.)
- Basic familiarity with JSON and Lua (or willingness to learn!)
- An idea for a game you want to create

## Directory Structure

Create your mod under the Moonshine `Mods` directory:

```
GAME_ROOT/Mods/
└── my-first-mod/
    ├── manifest.json           # Required: mod configuration
    ├── main.lua                # Required: main Lua script
    ├── SaveState.txt           # Optional: player data
    ├── milestones.txt          # Optional: player progression
    └── assets/                 # Optional: images, resources
        └── images/
```

**Important:** Your mod must live under `GAME_ROOT/Mods/`. `main.lua` must sit next to the manifest.

## Step 1: Create the Manifest

Create a file called `manifest.json` in your mod directory:

```json
{
  "author": "Your Name",
  "name": "My First Mode",
  "version": "1.0.0",
  "modeType": "Solo",
  "manifestApiVersion": 1,
  "description": "A simple solo game mode",
  "variations": [
    {
      "id": "classic",
      "label": "Classic Mode"
    }
  ],
  "milestones": []
}
```

### Manifest Fields Explained

| Field | Required | Description |
|-------|----------|-------------|
| `author` | No | Your name (informational) |
| `name` | Yes | Display name of your mode |
| `version` | Yes | Your mod's SemVer version (e.g., `1.0.0`) |
| `modeType` | Yes | Use `Solo` for now |
| `manifestApiVersion` | No | Use `1` (currently only version supported, defaults to `1`) |
| `description` | No | Longer description of your mode |
| `variations` | Yes | At least one variation (see below) |
| `milestones` | Yes | Can be empty for simple mods |

### Variations

Each variation is a playable mode. At minimum you need one:

```json
"variations": [
  {
    "id": "classic",
    "label": "Classic Mode",
    "description": "The original game mode"
  }
]
```

- **`id`** - Internal identifier (used in code, ≤32 chars)
- **`label`** - Display name shown to players (≤32 chars)
- **`description`** - Optional longer text

## Step 2: Create Your Lua Script

Create `main.lua` in your mod directory:

```lua
-- My First Mode
print("Welcome to my first Moonshine mod!")

-- Your game logic goes here
function update()
  -- Called each frame
  -- Add game mechanics here
end

function draw()
  -- Called each frame to render
  -- Add graphics here
end
```

The script must live in `main.lua` next to the manifest and define functions that Moonshine calls during gameplay.

## Step 3: Test Your Mod

### Using Modder Mode

1. Launch Moonshine
2. Navigate to **Modder Mode**
3. Select your mod from the available mods list
4. Choose a variation
5. Press **Start**

Your mod should launch and print welcome message to the console.

### Viewing Logs

Check the game logs to see console output:
- Console.log file in the game directory
- In-game debug console (if available)

## Step 4: Add a Menu (Optional)

To add player configuration options, add `menuInputs` to your variation:

```json
{
  "id": "classic",
  "label": "Classic Mode",
  "menuInputs": [
    {
      "id": "difficulty",
      "label": "Difficulty",
      "type": "select",
      "options": [
        { "label": "Easy" },
        { "label": "Normal" },
        { "label": "Hard" }
      ]
    }
  ]
}
```

Access the selected value in your Lua script using the selection context.

## Step 5: Add Progression (Optional)

To track player progress with milestones:

```json
{
  "milestones": ["completed_easy", "completed_hard"],
  "variations": [
    {
      "id": "easy",
      "label": "Easy Mode"
    },
    {
      "id": "hard",
      "label": "Hard Mode",
      "requiredMilestone": "completed_easy"
    }
  ]
}
```

The "Hard Mode" variation will only be accessible after the player earns the "completed_easy" milestone.

## Directory Examples

### Simple Mod (No Progression)
```
my-simple-mod/
├── manifest.json
├── main.lua
```

### Mode with Menus
```
my-menu-mod/
├── manifest.json
├── main.lua
└── assets/
    └── images/
        └── icon.png
```

### Mode with Progression
```
my-progression-mod/
├── manifest.json
├── main.lua
├── SaveState.txt
├── milestones.txt
└── assets/
```

## Common Tasks

### Access Player Selections from Menus

In your Lua script, access menu selections through the provided API:

```lua
-- Get player's menu choices
local difficulty = GetMenuSelection("difficulty")  -- "Easy", "Normal", or "Hard"
```

### Track Player Milestones

Create a `milestones.txt` file to track progress:

```
# milestones.txt
completed_easy
completed_normal
```

Each line is a milestone ID the player has earned.

### Save Game Data

Create a `SaveState.txt` file for persistent data:

```
score=1000
level=5
time=3600
```

This is automatically loaded on session restart.

## Troubleshooting

### Mod Won't Load
- Check manifest JSON syntax (use a JSON validator)
- Verify `main.lua` exists next to the manifest
- Ensure the mod is in the `GAME_ROOT/Mods/` directory

### Manifest Validation Error
- Review error message carefully
- Check for unknown milestones
- Verify all required fields are present

### Script Not Running
- Check console output for Lua errors
- Ensure `main.lua` has valid Lua syntax

## Next Steps

- **→ [Manifest Essentials]({{ site.baseurl }}{% link manifest.md %})** - Deep dive into manifest options
- **→ [Variations & Modes]({{ site.baseurl }}{% link variations-and-modes.md %})** - Create multiple game variations
- **→ [Menus & Configuration]({{ site.baseurl }}{% link menus-configuration.md %})** - Build interactive menus

---

**Back to:** [Introduction]({{ site.baseurl }}{% link introduction.md %}) | [Home]({{ site.baseurl }}{% link index.md %})
