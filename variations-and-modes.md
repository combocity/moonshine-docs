---
layout: page
title: Variations & Modes
---

Variations allow players to choose different ways to play your mod. Each variation can have its own configuration, menus, and progression gates.

## What are Variations?

A **variation** is a distinct playable mode or configuration of your game. Think of it as "a different way to play."

Players see variations in a selector screen and choose which one to play.

### Examples

**Puzzle Mod:**
- Variation 1: "Beginner Puzzles"
- Variation 2: "Intermediate Puzzles"
- Variation 3: "Expert Puzzles"

**Roguelike Mod:**
- Variation 1: "Story Mode"
- Variation 2: "Classic Mode"
- Variation 3: "Daily Challenge"

**Survival Mod:**
- Variation 1: "Peaceful"
- Variation 2: "Normal"
- Variation 3: "Hard"

## Single Variation

Every mod needs at least one variation. A simple mod with one variation:

```json
{
  "name": "My Game",
  "variations": [
    {
      "id": "default",
      "label": "Play Game"
    }
  ]
}
```

Players won't see a selector screen. They'll go straight into your mod.

## Multiple Variations

With multiple variations, players choose which one to play:

```json
{
  "name": "My Game",
  "variations": [
    {
      "id": "easy",
      "label": "Easy Mode",
      "description": "Relaxed difficulty"
    },
    {
      "id": "normal",
      "label": "Normal Mode",
      "description": "Balanced challenge"
    },
    {
      "id": "hard",
      "label": "Hard Mode",
      "description": "For experienced players"
    }
  ]
}
```

### Variation Properties

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `id` | string | Yes | Internal ID (≤32 chars, unique) |
| `label` | string | Yes | Display name (≤32 chars) |
| `description` | string | No | Longer description |
| `menuInputs` | array | No | Configuration options (see Menus) |
| `requiredMilestone` | string | No | Milestone to unlock this variation |
| `visibleFromMilestone` | string | No | Milestone to show as "coming soon" |

## Progression-Gated Variations

Use milestones to require completion before unlocking harder variations:

```json
{
  "milestones": ["beat_easy", "beat_normal"],
  "variations": [
    {
      "id": "easy",
      "label": "Easy Mode"
    },
    {
      "id": "normal",
      "label": "Normal Mode",
      "requiredMilestone": "beat_easy"
    },
    {
      "id": "hard",
      "label": "Hard Mode",
      "requiredMilestone": "beat_normal"
    }
  ]
}
```

**Result:**
- Easy Mode: Always available
- Normal Mode: Locked until player earns "beat_easy"
- Hard Mode: Locked until player earns "beat_normal"

### Visibility vs. Accessibility

- **`requiredMilestone`** - Variation is greyed out until milestone earned
- **`visibleFromMilestone`** - Variation hidden until milestone earned (doesn't appear in selector)

```json
{
  "id": "teaser",
  "label": "Secret Mode",
  "visibleFromMilestone": "unlocked_secret",
  "requiredMilestone": "completed_all"
}
```

**Result:**
- Not visible: Doesn't appear in selector
- After "unlocked_secret": Appears as "Coming Soon" (greyed)
- After "completed_all": Fully accessible

## Communicating with Lua

Your Lua script needs to know which variation was selected:

```lua
-- Access the selected variation
local variation_id = GetSelectedVariation()  -- Returns "easy", "normal", etc.

-- Adjust difficulty based on variation
if variation_id == "easy" then
  difficulty_multiplier = 0.5
elseif variation_id == "normal" then
  difficulty_multiplier = 1.0
elseif variation_id == "hard" then
  difficulty_multiplier = 2.0
end
```

## Adding Menus to Variations

Each variation can have configuration menus:

```json
{
  "id": "hardcore",
  "label": "Hardcore Mode",
  "description": "Extreme challenge",
  "menuInputs": [
    {
      "id": "starting_health",
      "label": "Starting Health",
      "type": "select",
      "options": [
        { "label": "Low (50)" },
        { "label": "Normal (100)" },
        { "label": "High (200)" }
      ]
    }
  ]
}
```

(See [Menus & Configuration]({{ site.baseurl }}{% link menus-configuration.md %}) for more details)

## Design Patterns

### Progression Tiers
```json
"variations": [
  { "id": "t1", "label": "Tier 1" },
  { "id": "t2", "label": "Tier 2", "requiredMilestone": "completed_t1" },
  { "id": "t3", "label": "Tier 3", "requiredMilestone": "completed_t2" }
]
```

### Parallel Modes
```json
"variations": [
  { "id": "story", "label": "Story Mode" },
  { "id": "sandbox", "label": "Sandbox Mode" },
  { "id": "challenge", "label": "Challenge Mode" }
]
```
(No progression gates. All variations are accessible.)

### Content Packs
```json
"variations": [
  { "id": "standard", "label": "Standard Pack" },
  { "id": "expansion1", "label": "Expansion 1", "requiredMilestone": "dlc_purchased" },
  { "id": "expansion2", "label": "Expansion 2", "requiredMilestone": "dlc_purchased" }
]
```

### Teaser Pattern
```json
"variations": [
  { "id": "current", "label": "Current Season" },
  {
    "id": "next",
    "label": "Next Season (Coming Soon)",
    "visibleFromMilestone": "season_end",
    "requiredMilestone": "season_start"
  }
]
```

## Best Practices

✅ **DO:**
- Keep variation labels short and clear
- Use progression gates for content tiers
- Show "coming soon" teases for unreleased content
- Provide descriptions for hard-to-understand variations
- Make the first variation accessible to all players

❌ **DON'T:**
- Create variations with all menu options locked
- Use unclear variation names
- Gate the only variation (always have at least one free option)
- Make progression paths confusing

## Complete Example

```json
{
  "name": "Battle Royale Mini",
  "author": "Game Designer",
  "version": "1.0.0",
  "modeType": "Solo",
  "variations": [
    {
      "id": "training",
      "label": "Training",
      "description": "Learn the basics"
    },
    {
      "id": "normal",
      "label": "Normal Battle",
      "description": "50 players, standard rules",
      "requiredMilestone": "completed_training"
    },
    {
      "id": "hardcore",
      "label": "Hardcore Battle",
      "description": "100 players, brutal rules",
      "requiredMilestone": "ranked_level_10",
      "menuInputs": [
        {
          "id": "starting_loadout",
          "label": "Starting Loadout",
          "type": "select",
          "options": [
            { "label": "Random" },
            { "label": "Weapons" },
            { "label": "Armor" }
          ]
        }
      ]
    },
    {
      "id": "daily",
      "label": "Daily Challenge",
      "visibleFromMilestone": "ranked_level_5",
      "requiredMilestone": "daily_unlocked"
    }
  ],
  "milestones": [
    "completed_training",
    "ranked_level_5",
    "ranked_level_10",
    "daily_unlocked"
  ]
}
```

---

**Next:**
- **→ [Menus & Configuration]({{ site.baseurl }}{% link menus-configuration.md %})** - Add options and configuration
- **→ [Progression System]({{ site.baseurl }}{% link progression-milestones.md %})** - Gate features with milestones

**Related:**
- **← [Getting Started]({{ site.baseurl }}{% link getting-started.md %})** - Create your first mod
- **→ [Complete Example]({{ site.baseurl }}{% link example-progression-mode.md %})** - See full working code

**Back to:** [Home]({{ site.baseurl }}{% link index.md %})
