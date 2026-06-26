---
layout: page
title: Getting Started
---

This guide walks you through creating your first Moonshine Lua ROM.

## Prerequisites

- Moonshine game installed
- A player account linked to a Discord server. See [Moonshine Ecosystem]({{ site.baseurl }}{% link ecosystem.md %}) if you are not sure why Discord is part of the flow.
- VS Code installed with the EmmyLua extension.
- Basic familiarity with JSON and Lua (or willingness to learn!)
- An idea for a game you want to create

## Directory Structure

Create your ROM under the Moonshine local ROM workspace root named `Roms`:

```
GAME_ROOT/Roms/
└── my-first-rom/
    ├── .emmyrc.json            # Recommended: EmmyLua workspace settings
    ├── manifest.json           # Required: ROM configuration
    ├── main.lua                # Required: main Lua script
    ├── sdk/                    # Recommended: Moonshine Lua SDK for autocomplete
    │   └── api/
    │       └── v1/
    ├── SaveState.txt           # Optional: player data
    ├── milestones.txt          # Optional: player progression
    └── assets/                 # Optional: images, resources
        └── images/
```

**Important:** Your ROM must live under `GAME_ROOT/Roms/my-first-rom`. `main.lua` must sit next to the manifest. Open `my-first-rom` folder itself in VS Code so EmmyLua picks up `.emmyrc.json` and the local `sdk/api` stubs for autocompletion and debugging.

## Step 1: Create the Manifest

Create a file called `manifest.json` in your ROM directory:

```json
{
  "author": "Your Name",
  "name": "My First Mode",
  "version": "1.0.0",
  "modeType": "Solo",
  "manifestApiVersion": 1,
  "description": "A simple solo game mode",
  "variants": [
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
| `version` | Yes | Your ROM's SemVer version (e.g., `1.0.0`) |
| `modeType` | Yes | Use `Solo` for now |
| `manifestApiVersion` | No | Use `1` (currently only version supported, defaults to `1`) |
| `description` | No | Longer description of your mode |
| `variants` | Yes | At least one variant (see below) |
| `milestones` | Yes | Can be empty for simple ROMs |

### Variants

Each variant is a playable mode. At minimum you need one:

```json
"variants": [
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

Create `main.lua` in your ROM directory:

```lua
-- My First Mode
print("Welcome to my first Moonshine ROM!")

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

## Step 3: Test Your ROM

### Using Maker Mode

1. Launch Moonshine
2. Navigate to **Maker Mode**
3. Select your ROM from the available ROMs list
4. Choose a variant
5. Press **Start**

Your ROM should launch and print welcome message to the console.

### Viewing Logs

Check the game logs to see console output:
- Console.log file in the game directory
- In-game debug console (if available)

## Step 4: Add a Menu (Optional)

To add player configuration options, add `menuInputs` to your variant:

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
  "variants": [
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

The "Hard Mode" variant will only be accessible after the player earns the "completed_easy" milestone.

## Directory Examples

### Simple ROM (No Progression)
```
my-simple-rom/
├── manifest.json
├── main.lua
```

### Mode with Menus
```
my-menu-rom/
├── manifest.json
├── main.lua
└── assets/
    └── images/
        └── icon.png
```

### Mode with Progression
```
my-progression-rom/
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

### ROM Won't Load
- Check manifest JSON syntax (use a JSON validator)
- Verify `main.lua` exists next to the manifest
- Ensure the ROM is in the `GAME_ROOT/Roms/` directory

### Manifest Validation Error
- Review error message carefully
- Check for unknown milestones
- Verify all required fields are present

### Script Not Running
- Check console output for Lua errors
- Ensure `main.lua` has valid Lua syntax

## Next Steps

- **→ [Moonshine Ecosystem]({{ site.baseurl }}{% link ecosystem.md %})** - Understand Discord communities, author access, and publication stages
- **→ [Manifest Essentials]({{ site.baseurl }}{% link manifest.md %})** - Deep dive into manifest options
- **→ [Variants & Modes]({{ site.baseurl }}{% link variants-and-modes.md %})** - Create multiple game variants
- **→ [Menus & Configuration]({{ site.baseurl }}{% link menus-configuration.md %})** - Build interactive menus

---

**Back to:** [Authoring Introduction]({{ site.baseurl }}{% link introduction.md %}) | [Home]({{ site.baseurl }}{% link index.md %})
