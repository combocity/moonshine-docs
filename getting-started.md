---
layout: page
title: Getting Started
---

This guide walks you through creating a first Moonshine Lua ROM with the current
Lua API.

## Prerequisites

- Moonshine game installed.
- A player account linked to a Discord server. See [Moonshine Ecosystem]({{ site.baseurl }}{% link ecosystem.md %}) for the community flow.
- VS Code with the EmmyLua extension.
- Basic JSON and Lua familiarity.

## Directory Structure

Create your ROM under the Moonshine local ROM workspace root named `Roms`:

```text
GAME_ROOT/Roms/
└── my-first-rom/
    ├── .emmyrc.json
    ├── manifest.json
    ├── main.lua
    ├── sdk/
    │   └── api/
    │       └── v1/
    └── assets/
        ├── images/
        ├── sfx/
        ├── musics/
        └── fonts/
```

`manifest.json` and `main.lua` are the important files. Open the ROM folder
itself in VS Code so EmmyLua can use `.emmyrc.json` and the local `sdk/api`
stubs for autocompletion.

## Step 1: Create the Manifest

Create `manifest.json`:

```json
{
  "author": "Your Name",
  "name": "My First ROM",
  "version": "1.0.0",
  "modeType": "Solo",
  "manifestApiVersion": 1,
  "description": "A simple solo ROM",
  "milestones": [],
  "variants": [
    {
      "id": "classic",
      "label": "Classic"
    }
  ]
}
```

Important fields:

| Field | Required | Description |
|-------|----------|-------------|
| `name` | Yes | Display name of your ROM. |
| `version` | Yes | SemVer version, for example `1.0.0`. |
| `modeType` | Yes | Use `Solo`. |
| `manifestApiVersion` | No | Use `1`; this is the only supported API version today. |
| `milestones` | Yes | List of possible progression milestones. Can be empty. |
| `variants` | Yes | At least one playable variant. |

## Step 2: Create `main.lua`

Create `main.lua` next to the manifest:

```lua
local frame = 0

function init()
  api.log.info("My First ROM started")
  api.save.play_count = (api.save.play_count or 0) + 1
end

function update()
  frame = frame + 1

  if frame >= 600 then
    api.session.end_game()
  end
end

function draw()
  api.graphics.draw_text(24, 24, "Hello Moonshine")
end
```

Moonshine calls `init`, `update`, and `draw` during the session. Your script can
use the global `api` table to read the session context, react to inputs, draw,
play sounds, save progress, and end the game. For the complete runtime surface,
see the [Lua API v1 Reference]({{ site.baseurl }}{% link lua-api-v1.md %}).

## Step 3: Add a Menu

Add `menuInputs` to a variant:

```json
{
  "id": "classic",
  "label": "Classic",
  "menuInputs": [
    {
      "id": "difficulty",
      "label": "Difficulty",
      "type": "Select",
      "options": [
        { "label": "Easy" },
        { "label": "Normal" },
        { "label": "Hard" }
      ]
    }
  ]
}
```

Read the selected value from Lua:

```lua
function init()
  local difficulty = api.session.selection.difficulty or "Easy"
  api.log.info("Difficulty: " .. difficulty)
end
```

## Step 4: Add Progression

Declare possible milestones in the manifest:

```json
{
  "milestones": ["completed_easy"],
  "variants": [
    {
      "id": "easy",
      "label": "Easy"
    },
    {
      "id": "hard",
      "label": "Hard",
      "requiredMilestone": "completed_easy"
    }
  ]
}
```

Unlock milestones from Lua when the player earns them:

```lua
if boss_defeated then
  api.progress.unlock_milestone("completed_easy")
end
```

For server-backed sessions, Avalon sends the player's earned milestones when
the catalog/session is created and receives newly unlocked milestones when the
session ends. In local maker mode, Moonshine may persist local test progress for
development, but authors should use the `api.progress` functions rather than
editing progress files by hand.

## Step 5: Test Your ROM

1. Launch Moonshine.
2. Open Maker Mode.
3. Select your local ROM.
4. Choose a variant and menu options.
5. Start the session.

Check the logs for Lua errors or `api.log` output.

## Troubleshooting

### ROM Won't Load

- Check that `manifest.json` is valid JSON.
- Verify `main.lua` exists next to the manifest.
- Ensure the ROM is under `GAME_ROOT/Roms/`.
- Check manifest validation errors for invalid ids or milestone references.

### Script Not Running

- Ensure `main.lua` defines valid Lua.
- Use `init`, `update`, and `draw` as entry points.
- Check the Moonshine logs for Lua runtime errors.

## Next Steps

- **[Lua API v1 Reference]({{ site.baseurl }}{% link lua-api-v1.md %})** - Runtime lifecycle and `api.*` modules.
- **[Manifest Reference]({{ site.baseurl }}{% link manifest.md %})** - Full manifest fields and validation rules.
- **[Menus & Configuration]({{ site.baseurl }}{% link menus-configuration.md %})** - Variant menu inputs.
- **[Progression System]({{ site.baseurl }}{% link progression-milestones.md %})** - Milestones and badges.
