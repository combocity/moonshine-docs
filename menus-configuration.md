---
layout: page
title: Menus & Configuration
---

Menus let players customize their game experience before playing. Players make selections that your Lua script receives and uses to configure the game.

## Basic Menu

Add a simple menu to a variation:

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

**Result:** Player sees a menu with three difficulty options to choose from.

## Menu Input Properties

Each menu input is a configuration option:

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `id` | string | Yes | Internal ID (≤32 chars, unique) |
| `label` | string | Yes | Display name (≤32 chars) |
| `type` | string | No | Input type (currently only "select") |
| `options` | array | Yes | Available choices |
| `requiredMilestone` | string | No | Milestone to access this input |
| `visibleFromMilestone` | string | No | Milestone to show as "coming soon" |

## Options

Each option represents a choice the player can make:

```json
"options": [
  {
    "label": "Easy",
    "description": "Perfect for beginners"
  },
  {
    "label": "Normal",
    "description": "Balanced challenge"
  },
  {
    "label": "Hard",
    "description": "For experienced players"
  }
]
```

### Option Properties

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `label` | string | Yes | Display name & selection value (≤32 chars) |
| `description` | string | No | Longer description |
| `requiredMilestone` | string | No | Milestone to access this option |
| `visibleFromMilestone` | string | No | Milestone to show as "coming soon" |
| `inputs` | array | No | Nested sub-options (see Hierarchical Menus) |

## Accessing Menu Selections

In your Lua script, access player choices:

```lua
-- Get a single menu selection
local difficulty = GetMenuSelection("difficulty")
-- Returns: "Easy", "Normal", or "Hard"

if difficulty == "Hard" then
  enemy_health = 200
elseif difficulty == "Normal" then
  enemy_health = 100
else  -- Easy
  enemy_health = 50
end
```

## Multiple Menu Options

Add multiple configuration options:

```json
{
  "id": "custom",
  "label": "Custom Mode",
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
      "id": "map_size",
      "label": "Map Size",
      "options": [
        { "label": "Small (10x10)" },
        { "label": "Medium (20x20)" },
        { "label": "Large (40x40)" }
      ]
    },
    {
      "id": "game_speed",
      "label": "Game Speed",
      "options": [
        { "label": "Slow" },
        { "label": "Normal" },
        { "label": "Fast" }
      ]
    }
  ]
}
```

**In Lua:**
```lua
local difficulty = GetMenuSelection("difficulty")
local map_size = GetMenuSelection("map_size")
local speed = GetMenuSelection("game_speed")
```

## Hierarchical Menus

Create nested options where selecting one reveals sub-options:

```json
{
  "id": "advanced",
  "label": "Advanced Mode",
  "menuInputs": [
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
                { "label": "Tiny" },
                { "label": "Large" }
              ]
            }
          ]
        },
        {
          "label": "Survival",
          "inputs": [
            {
              "id": "survival_difficulty",
              "label": "Survival Difficulty",
              "options": [
                { "label": "Easy" },
                { "label": "Hardcore" }
              ]
            }
          ]
        }
      ]
    }
  ]
}
```

**Structure:**
```
Game Type [select]
├── Campaign
├── Sandbox
│   └── Sandbox Size [select]
│       ├── Tiny
│       └── Large
└── Survival
    └── Survival Difficulty [select]
        ├── Easy
        └── Hardcore
```

**In Lua:**
```lua
local game_type = GetMenuSelection("game_type")
local sandbox_size = GetMenuSelection("sandbox_size")  -- nil if Sandbox not selected
local survival_diff = GetMenuSelection("survival_difficulty")  -- nil if Survival not selected
```

## Progression-Gated Options

Lock options until milestones are earned:

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
    },
    {
      "label": "Extreme",
      "requiredMilestone": "completed_hard"
    }
  ]
}
```

**Result:**
- Easy: Always accessible
- Normal: Greyed out until "completed_easy"
- Hard: Greyed out until "completed_normal"
- Extreme: Greyed out until "completed_hard"

### Visibility vs. Accessibility

- **`requiredMilestone`** - Option is greyed out until milestone earned
- **`visibleFromMilestone`** - Option hidden until milestone earned (doesn't appear)

```json
{
  "label": "Secret Difficulty",
  "visibleFromMilestone": "unlocked_secret",
  "requiredMilestone": "completed_all"
}
```

## Design Patterns

### Progressive Difficulty
```json
"options": [
  { "label": "Tutorial" },
  { "label": "Easy", "requiredMilestone": "beat_tutorial" },
  { "label": "Normal", "requiredMilestone": "beat_easy" },
  { "label": "Hard", "requiredMilestone": "beat_normal" },
  { "label": "Extreme", "requiredMilestone": "beat_hard" }
]
```

### Feature Unlocking
```json
{
  "id": "modifiers",
  "label": "Game Modifiers",
  "options": [
    { "label": "None" },
    { "label": "Double Speed", "requiredMilestone": "expert_status" },
    { "label": "Low Health", "requiredMilestone": "expert_status" },
    { "label": "Darkness", "requiredMilestone": "master_status" }
  ]
}
```

### Two-Level Configuration
```json
"menuInputs": [
  {
    "id": "game_mode",
    "label": "Game Mode",
    "options": [
      {
        "label": "Campaign",
        "inputs": [
          {
            "id": "campaign_difficulty",
            "label": "Difficulty",
            "options": [...]
          }
        ]
      },
      {
        "label": "Arena",
        "inputs": [
          {
            "id": "arena_size",
            "label": "Arena Size",
            "options": [...]
          },
          {
            "id": "arena_hazards",
            "label": "Hazards",
            "options": [...]
          }
        ]
      }
    ]
  }
]
```

## Complete Example

```json
{
  "id": "multiplayer",
  "label": "Multiplayer",
  "menuInputs": [
    {
      "id": "mode",
      "label": "Game Mode",
      "options": [
        {
          "label": "Free For All",
          "inputs": [
            {
              "id": "ffa_map",
              "label": "Map",
              "options": [
                { "label": "Arena" },
                { "label": "Forest" },
                { "label": "Urban" }
              ]
            }
          ]
        },
        {
          "label": "Team Deathmatch",
          "inputs": [
            {
              "id": "tdm_team_size",
              "label": "Team Size",
              "options": [
                { "label": "1v1" },
                { "label": "2v2" },
                { "label": "4v4" }
              ]
            }
          ]
        }
      ]
    },
    {
      "id": "time_limit",
      "label": "Time Limit",
      "options": [
        { "label": "Unlimited" },
        { "label": "5 Minutes" },
        { "label": "10 Minutes" }
      ]
    }
  ]
}
```

**In Lua:**
```lua
local mode = GetMenuSelection("mode")  -- "Free For All" or "Team Deathmatch"
local ffa_map = GetMenuSelection("ffa_map")  -- nil if not FFA
local tdm_size = GetMenuSelection("tdm_team_size")  -- nil if not TDM
local time_limit = GetMenuSelection("time_limit")  -- Always set
```

## Best Practices

✅ **DO:**
- Keep option labels short (≤32 chars)
- Provide descriptions for non-obvious options
- Use progression gates to unlock advanced options
- Set sensible defaults (first option is usually default)
- Organize related options logically

❌ **DON'T:**
- Create menus where all options are locked
- Use vague option names
- Nest menus too deeply (3+ levels get confusing)
- Make all options require different milestones
- Create unused/non-functional options

## Saving Player Choices

Your Lua script receives player selections and can save them:

```lua
-- In your mod's save/load logic
local selections = {
  difficulty = GetMenuSelection("difficulty"),
  map_size = GetMenuSelection("map_size")
}

SaveToFile("player_choices.json", selections)
```

---

**Next:**
- **→ [Progression System]({{ site.baseurl }}{% link progression-milestones.md %})** - Gate features with milestones
- **→ [Leaderboards]({{ site.baseurl }}{% link leaderboards.md %})** - Track player performance

**Related:**
- **← [Variations & Modes]({{ site.baseurl }}{% link variations-and-modes.md %})** - Create different game variants
- **→ [Complete Example]({{ site.baseurl }}{% link example-progression-mode.md %})** - See full working code

**Back to:** [Home]({{ site.baseurl }}{% link index.md %})

