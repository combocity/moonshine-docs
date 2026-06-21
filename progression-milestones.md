---
layout: page
title: Progression System
---

Milestones track player achievements and gate features based on progress. Players earn milestones by completing challenges, and new features unlock as they progress.

## What are Milestones?

A **milestone** is an achievement or checkpoint. Examples:
- "beat_easy_mode"
- "reached_level_10"
- "earned_master_rank"
- "defeated_boss_1"

Milestones are identifiers (strings) that represent progress. Use them to:
- Gate features (unlock harder variations after beating easy)
- Show hints (display "coming soon" for locked content)
- Create progression tiers (tier 1 → tier 2 → tier 3)

## Defining Milestones

List all possible milestones in your manifest:

```json
{
  "milestones": [
    "tutorial_completed",
    "beat_easy",
    "beat_normal",
    "beat_hard",
    "earned_rank_master"
  ]
}
```

These milestone IDs are referenced throughout your manifest to gate content.

## How Players Earn Milestones

Players earn milestones through gameplay. Your Lua script detects achievements and records them:

```lua
-- In your Lua script
if player_defeated_boss then
  RecordMilestone("beat_easy")
end
```

Earned milestones are saved in `milestones.txt` next to your entry point:

```
# milestones.txt
tutorial_completed
beat_easy
beat_normal
```

One milestone per line. Lines starting with `#` are comments.

## Gating Features with Milestones

Use milestones to control what's accessible:

### Gating Variations

```json
{
  "variations": [
    {
      "id": "easy",
      "label": "Easy Mode"
    },
    {
      "id": "hard",
      "label": "Hard Mode",
      "requiredMilestone": "beat_easy"
    }
  ]
}
```

- Easy Mode: Always accessible
- Hard Mode: Locked until player earns "beat_easy"

### Gating Menu Options

```json
{
  "id": "difficulty",
  "label": "Difficulty",
  "options": [
    { "label": "Easy" },
    {
      "label": "Hard",
      "requiredMilestone": "beat_easy"
    }
  ]
}
```

### Gating Specific Menu Inputs

```json
{
  "id": "advanced_settings",
  "label": "Advanced Settings",
  "requiredMilestone": "expert_unlocked",
  "options": [...]
}
```

The entire menu input is locked until "expert_unlocked" is earned.

## Visibility vs. Accessibility

Control how locked content appears to players:

### Required Milestone (Greyed Out)

```json
{
  "requiredMilestone": "beat_easy"
}
```

**Result:** Content appears but is greyed out. Shows "coming soon" or similar message.

### Visible From Milestone (Hidden Until Hint)

```json
{
  "visibleFromMilestone": "teaser_hint"
}
```

**Result:** Content completely hidden until "teaser_hint" is earned. Then appears greyed out.

### Both (Progressive Reveal)

```json
{
  "visibleFromMilestone": "teaser_hint",
  "requiredMilestone": "full_unlock"
}
```

**Timeline:**
1. Initially: Hidden completely
2. After "teaser_hint": Shows as greyed out ("Coming Soon")
3. After "full_unlock": Fully accessible

## Design Patterns

### Simple Progression (Easy → Hard)
```json
{
  "milestones": ["beat_easy", "beat_hard"],
  "variations": [
    { "id": "easy", "label": "Easy" },
    { "id": "hard", "label": "Hard", "requiredMilestone": "beat_easy" }
  ]
}
```

### Multi-Tier Progression
```json
{
  "milestones": ["beat_tier_1", "beat_tier_2", "beat_tier_3"],
  "variations": [
    { "id": "tier_1", "label": "Tier 1" },
    { "id": "tier_2", "label": "Tier 2", "requiredMilestone": "beat_tier_1" },
    { "id": "tier_3", "label": "Tier 3", "requiredMilestone": "beat_tier_2" }
  ]
}
```

### Parallel Unlocks
```json
{
  "milestones": ["story_complete", "sandbox_unlocked"],
  "variations": [
    { "id": "story", "label": "Story Mode" },
    { "id": "sandbox", "label": "Sandbox", "requiredMilestone": "story_complete" }
  ]
}
```

### Teaser Content
```json
{
  "milestones": ["season_1_done", "season_2_available"],
  "variations": [
    { "id": "season_1", "label": "Season 1" },
    {
      "id": "season_2",
      "label": "Season 2 (Coming Soon)",
      "visibleFromMilestone": "season_1_done",
      "requiredMilestone": "season_2_available"
    }
  ]
}
```

### Ranked Unlock
```json
{
  "milestones": ["ranked_qualified", "ranked_elite"],
  "variations": [
    { "id": "casual", "label": "Casual" },
    { "id": "ranked", "label": "Ranked", "requiredMilestone": "ranked_qualified" },
    { "id": "elite", "label": "Elite", "requiredMilestone": "ranked_elite" }
  ]
}
```

## Recording Milestones in Lua

Your script detects when players complete challenges and records milestones:

```lua
-- Check if player beat the level
if player_health > 0 and level_boss_defeated then
  RecordMilestone("beat_easy")
end

-- Track progression
if current_level >= 10 then
  RecordMilestone("level_10_reached")
end

if total_score >= 5000 then
  RecordMilestone("expert_scorer")
end
```

Milestones are automatically saved to `milestones.txt`.

## Complete Example

**manifest.json:**
```json
{
  "name": "Progressive Adventure",
  "author": "Designer",
  "version": "1.0.0",
  "modeType": "Solo",
  "milestones": [
    "tutorial_complete",
    "beat_chapter_1",
    "beat_chapter_2",
    "beat_chapter_3",
    "master_rank"
  ],
  "variations": [
    {
      "id": "story",
      "label": "Story Mode"
    },
    {
      "id": "chapter_2",
      "label": "Chapter 2",
      "requiredMilestone": "beat_chapter_1"
    },
    {
      "id": "chapter_3",
      "label": "Chapter 3",
      "requiredMilestone": "beat_chapter_2"
    },
    {
      "id": "extreme",
      "label": "Extreme Challenge",
      "requiredMilestone": "beat_chapter_3",
      "visibleFromMilestone": "master_rank"
    }
  ]
}
```

**main.lua:**
```lua
function update()
  -- Game logic here

  if chapter_completed then
    if current_chapter == 1 then
      RecordMilestone("beat_chapter_1")
    elseif current_chapter == 2 then
      RecordMilestone("beat_chapter_2")
    elseif current_chapter == 3 then
      RecordMilestone("beat_chapter_3")
    end
  end

  if player_score >= 10000 then
    RecordMilestone("master_rank")
  end
end
```

## Best Practices

✅ **DO:**
- Create milestone IDs that clearly describe what was achieved
- Use snake_case for milestone names (e.g., "beat_level_1")
- Gate content progressively (don't lock everything)
- Always have at least one fully accessible option
- Use "visibleFromMilestone" for teasers
- Design progression that feels natural

❌ **DON'T:**
- Create milestones that are never earned
- Gate the only playable variation
- Make all options locked at game start
- Use confusing milestone names
- Create impossible progression paths
- Gate tutorial content behind milestones

## Checking Milestones in Lua

Query earned milestones in your script:

```lua
-- Check if player has earned a milestone
if HasMilestone("beat_easy") then
  -- Player completed easy mode
  show_hard_mode = true
end

-- Get all earned milestones
local earned = GetAllMilestones()
for i, milestone in ipairs(earned) do
  print("Player earned: " .. milestone)
end
```

---

**Next:**
- **→ [Leaderboards]({{ site.baseurl }}{% link leaderboards.md %})** - Track and display scores
- **→ [Best Practices]({{ site.baseurl }}{% link best-practices.md %})** - Design guidelines

**Related:**
- **← [Variations & Modes]({{ site.baseurl }}{% link variations-and-modes.md %})** - Create game variants
- **← [Menus & Configuration]({{ site.baseurl }}{% link menus-configuration.md %})** - Add configuration options
- **→ [Complete Example]({{ site.baseurl }}{% link example-progression-mode.md %})** - See full working code

**Back to:** [Home]({{ site.baseurl }}{% link index.md %})
