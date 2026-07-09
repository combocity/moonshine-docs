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

Moonshine only adds a menu input to `api.session.selection` when that input is
visible and accessible. If the input is hidden by `visibleFromMilestone` or
locked by `requiredMilestone`, no selection key is sent for that input.

If the input is available but a stored or preconfigured selection points to an
unavailable option, Moonshine falls back to the first accessible option. If no
accessible option remains, the menu input is not rendered and no selection key
is sent.

For `Select` inputs, unavailable options are removed from the choice list. For
`Radio` inputs, unavailable options can remain visible but disabled.

## Saving Player Choices

Moonshine remembers menu choices for each variant. When a player starts the
same variant again, Moonshine restores the previous valid selection and sends it
through `api.session.selection`.

If the manifest changed, or if progression gates make a remembered option
unavailable, Moonshine resolves the selection again and falls back to the first
accessible option for that input. Hidden or locked inputs are omitted from
`api.session.selection`.

Do not copy menu choices into `api.save` just to make them persistent; Moonshine
already handles that. Use `api.save` only when the ROM needs its own gameplay
history derived from those choices.

If a choice leads to player progression, such as clearing a difficulty, record
that progression with milestones instead. Milestones are the data Moonshine uses
for unlocks, visibility, and progression gates.

## Next

- **[Progression System]({{ site.baseurl }}{% link progression-milestones.md %})** - Gate features with milestones.
- **[Manifest Reference]({{ site.baseurl }}{% link manifest.md %})** - Full validation rules.
