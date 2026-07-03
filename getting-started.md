---
layout: page
title: Getting Started
---

This guide walks you through creating a first Moonshine Lua ROM with the current
Lua API.

## Prerequisites

- Moonshine game installed.
- An Author account linked to a Discord server. See [Moonshine Roles Ecosystem]({{ site.baseurl }}{% link ecosystem.md %}) for the community flow.
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
stubs for autocompletion (initially in the /Roms folder, move them accordingly).

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

Create the LUA entry point `main.lua` next to the manifest:

```lua
local frame = 0

function init() -- called once before update()
  api.log.info("My First ROM started")
  api.save.play_count = (api.save.play_count or 0) + 1
end

function update() -- called every frame
  frame = frame + 1
  if frame >= 600 then
    api.session.end_game()
  end

  if frame >= 800 then
    api.session.shutdown()
  end
end

function draw() -- called every frame (will try)
  api.graphics.draw_text(24, 24, "Hello Moonshine")
end
```

Moonshine calls `init`, `update`, and `draw` when running this script. You will need to call `api.session.shutdown()` to explicitely end your session, so Moonshine can terminate your script. For the full flow, see [Session Lifecycle]({{ site.baseurl }}{% link session-lifecycle.md %}).

## Step 3: Test Your ROM

1. Launch Moonshine.
2. Open Maker Mode.
3. Select your local ROM.
4. Choose a variant and menu options.
5. Start the session.

Check the logs for Lua errors or `api.log` output.

## Troubleshooting

### ROM Won't Load

Moonshine validates your manifest before offering to run the ROM. If something is invalid or inconsistent, it reports the issues with a list of human-readable messages. Read them, fix the manifest, then try loading the ROM again.

### Script is crashing

Crashes happen; nothing wrong with moonshing here (i hope). Read the error message, check the stack trace, and use EmmyLua debugging tools to inspect what happened.

## Next Steps

- **[Lua API v1 Reference]({{ site.baseurl }}{% link lua-api-v1.md %})** - Runtime lifecycle and `api.*` modules.
- **[Session Lifecycle]({{ site.baseurl }}{% link session-lifecycle.md %})** - How a Lua session starts, submits, and ends.
- **[Manifest Reference]({{ site.baseurl }}{% link manifest.md %})** - Full manifest fields and validation rules.
- **[Menus & Configuration]({{ site.baseurl }}{% link menus-configuration.md %})** - Variant menu inputs.
- **[Progression System]({{ site.baseurl }}{% link progression-milestones.md %})** - Milestones and badges.
