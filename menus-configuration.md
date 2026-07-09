---
layout: page
title: Menus & Configuration
---

Menus let players configure a variant before the Lua session starts. The
selected values are provided to Lua in `api.session.selection`.

## Basic Menu

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

The option `label` is the value sent to Lua.

## Menu Input Properties

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `id` | string | Yes | Internal id, max 32 chars, unique across menu inputs. |
| `label` | string | Yes | Display label, max 32 chars. |
| `type` | string | No | `Select` by default. `Radio` is also accepted. |
| `description` | string | No | Longer text shown by the UI when supported. |
| `earnedDescription` | string | No | Text for already-earned/available states when supported. |
| `options` | array | Yes | At least two options. |
| `requiredMilestone` | string | No | Milestone required to access this input. |
| `visibleFromMilestone` | string | No | Milestone required before this input is visible. |

## Option Properties

```json
{
  "label": "Hard",
  "description": "For experienced players",
  "requiredMilestone": "completed_normal"
}
```

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `label` | string | Yes | Display label and Lua selection value, max 32 chars. |
| `description` | string | No | Longer text shown by the UI when supported. |
| `earnedDescription` | string | No | Text for already-earned/available states when supported. |
| `requiredMilestone` | string | No | Milestone required to select this option. |
| `visibleFromMilestone` | string | No | Milestone required before this option is visible. |
| `inputs` | array | No | Nested menu inputs revealed by this option. |

## Reading Selections in Lua

Use `api.session.selection`:

```lua
function init()
  local difficulty = api.session.selection.difficulty or "Easy"

  if difficulty == "Hard" then
    enemy_health = 200
  elseif difficulty == "Normal" then
    enemy_health = 100
  else
    enemy_health = 50
  end
end
```

You can also index with brackets, which is useful for ids that are not valid Lua
field names:

```lua
local start_level = api.session.selection["start-level"]
```

## Multiple Inputs

```json
{
  "id": "custom",
  "label": "Custom",
  "menuInputs": [
    {
      "id": "difficulty",
      "label": "Difficulty",
      "options": [
        { "label": "Easy" },
        { "label": "Hard" }
      ]
    },
    {
      "id": "speed",
      "label": "Speed",
      "options": [
        { "label": "Slow" },
        { "label": "Normal" },
        { "label": "Fast" }
      ]
    }
  ]
}
```

```lua
function init()
  local difficulty = api.session.selection.difficulty
  local speed = api.session.selection.speed
end
```

## Nested Inputs

Nested inputs are attached to a specific option. They are only relevant when
that option is selected.

```json
{
  "id": "game_type",
  "label": "Game Type",
  "options": [
    { "label": "Campaign" },
    {
      "label": "Sandbox",
      "inputs": [
        {
          "id": "sandbox_size",
          "label": "Sandbox Size",
          "options": [
            { "label": "Small" },
            { "label": "Large" }
          ]
        }
      ]
    }
  ]
}
```

```lua
function init()
  local game_type = api.session.selection.game_type
  local sandbox_size = api.session.selection.sandbox_size
end
```

If `Campaign` is selected, `sandbox_size` is not expected to be present.

## Progression-Gated Options

Use milestones to control access:

```json
{
  "id": "difficulty",
  "label": "Difficulty",
  "options": [
    { "label": "Easy" },
    {
      "label": "Normal",
      "requiredMilestone": "completed_easy"
    },
    {
      "label": "Hard",
      "requiredMilestone": "completed_normal"
    }
  ]
}
```

- `requiredMilestone` keeps the element visible but unavailable until earned.
- `visibleFromMilestone` hides the element until the milestone is earned.
- Both properties must reference a milestone declared in the manifest.

## Fallback Option Rule

Every menu input must keep a valid fallback choice available. This is why each
menu input must define at least two options, and why the first option must not
define either `requiredMilestone` or `visibleFromMilestone`.

Use the first option as the safe default for a player who has not unlocked
anything yet. Gate later options when progression should unlock stronger,
harder, or more advanced choices:

```json
{
  "id": "difficulty",
  "label": "Difficulty",
  "options": [
    { "label": "Easy" },
    {
      "label": "Normal",
      "requiredMilestone": "completed_easy"
    },
    {
      "label": "Hard",
      "visibleFromMilestone": "completed_normal"
    }
  ]
}
```

If a stored or preconfigured selection points to an unavailable option,
Moonshine falls back to the first accessible option. If no accessible option
remains, the menu input is not rendered.

For `Select` inputs, unavailable options are removed from the choice list. For
`Radio` inputs, unavailable options can remain visible but disabled.

## Saving Player Choices

Selections are provided at session start. If your ROM wants to remember the last
choice, write it into `api.save`:

```lua
function init()
  api.save.last_difficulty = api.session.selection.difficulty
end
```

## Next

- **[Progression System]({{ site.baseurl }}{% link progression-milestones.md %})** - Gate features with milestones.
- **[Manifest Reference]({{ site.baseurl }}{% link manifest.md %})** - Full validation rules.
