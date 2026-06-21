---
layout: page
title: Best Practices & Edge Cases
---

Design guidelines to avoid common pitfalls and create smooth player experiences.

## Menu Design

### ✅ DO: Always Have Accessible Options

Every menu input should have at least ONE accessible option:

```json
{
  "id": "difficulty",
  "options": [
    { "label": "Easy" },  // Always accessible
    { "label": "Hard", "requiredMilestone": "beat_easy" }
  ]
}
```

### ❌ DON'T: Create Inaccessible Menus

This creates a broken menu where nothing can be selected:

```json
{
  "id": "difficulty",
  "options": [
    { "label": "Hard", "requiredMilestone": "beat_easy" },
    { "label": "Extreme", "requiredMilestone": "beat_hard" }
  ]
}
```

**Problem:** At game start, both options are locked. Player can't proceed.

**Fix:** Add at least one accessible option:
```json
{
  "id": "difficulty",
  "options": [
    { "label": "Easy" },  // No requirement
    { "label": "Hard", "requiredMilestone": "beat_easy" }
  ]
}
```

## Variation Design

### ✅ DO: Provide Entry Points

Always have at least one playable variation accessible at game start:

```json
{
  "variations": [
    { "id": "intro", "label": "Tutorial" },  // Always accessible
    { "id": "main", "label": "Main Game", "requiredMilestone": "beat_intro" }
  ]
}
```

### ❌ DON'T: Gate All Variations

This traps the player:

```json
{
  "variations": [
    { "id": "main", "label": "Main Game", "requiredMilestone": "has_played" },
    { "id": "hard", "label": "Hard", "requiredMilestone": "beat_main" }
  ]
}
```

**Problem:** No variation is playable on first launch.

**Fix:** Provide an unlocked entry point:
```json
{
  "variations": [
    { "id": "main", "label": "Main Game" },  // No requirement
    { "id": "hard", "label": "Hard", "requiredMilestone": "beat_main" }
  ]
}
```

## Progression Design

### ✅ DO: Create Natural Progression Paths

Design progression that feels like advancement:

```json
{
  "milestones": ["beat_t1", "beat_t2", "beat_t3"],
  "variations": [
    { "id": "tier_1", "label": "Tier 1" },
    { "id": "tier_2", "label": "Tier 2", "requiredMilestone": "beat_t1" },
    { "id": "tier_3", "label": "Tier 3", "requiredMilestone": "beat_t2" }
  ]
}
```

Each tier unlocks logically.

### ❌ DON'T: Create Confusing Milestones

Unclear progression confuses players:

```json
{
  "milestones": ["x", "y", "z"],  // Meaningless names
  "variations": [
    { "id": "a", "label": "A" },
    { "id": "b", "label": "B", "requiredMilestone": "x" },
    { "id": "c", "label": "C", "requiredMilestone": "y" }
  ]
}
```

**Fix:** Use descriptive milestone names:
```json
{
  "milestones": ["beat_campaign", "master_rank", "skilled_enough"],
  "variations": [...]
}
```

### ✅ DO: Use Descriptive Milestone Names

Clear naming helps you and players understand progression:

```json
{
  "milestones": [
    "tutorial_complete",
    "level_1_defeated",
    "boss_rank_earned"
  ]
}
```

### ❌ DON'T: Create Impossible Milestones

Don't reference milestones that can never be earned:

```lua
-- Never triggered in the game
function update()
  -- No code that calls RecordMilestone("impossible_goal")
end
```

**Fix:** Only gate content behind milestones players can actually earn.

## Edge Cases

### Case 1: All Menu Options Locked

**Scenario:** Players start with all difficulty options locked.

**Solution:** Always have one free option:
```json
{
  "options": [
    { "label": "Easy" },  // Always free
    { "label": "Hard", "requiredMilestone": "earned_right_to_hard" }
  ]
}
```

### Case 2: Pre-Configured Selection Becomes Inaccessible

**Scenario:** Manifest has pre-configured menu choice, but player loses that milestone.

**Solution:** System automatically falls back to first accessible option. No action needed from you. Players will simply get a sensible default choice.

### Case 3: Nested Menus with No Accessible Path

**Scenario:** Parent option accessible, but all sub-options locked.

**Solution:** Ensure nested options have accessible defaults:
```json
{
  "id": "mode",
  "options": [
    {
      "label": "Campaign",
      "inputs": [
        {
          "id": "difficulty",
          "options": [
            { "label": "Easy" },  // Always accessible
            { "label": "Hard", "requiredMilestone": "beat_easy" }
          ]
        }
      ]
    }
  ]
}
```

### Case 4: Player Progression Resets

**Scenario:** Player earned milestone, then lost it (rare scenario).

**Solution:** Lost access reverts to next best option automatically.

## Leaderboard Design

### ✅ DO: Choose Relevant Metrics

Track metrics that matter for your game:

```json
{
  "rankingTables": [
    {
      "id": "completion_times",
      "label": "Fastest Times",
      "columns": [
        { "label": "Time", "type": "Chrono" }
      ]
    }
  ]
}
```

### ❌ DON'T: Create Meaningless Leaderboards

Don't track metrics that don't reflect skill/effort:

```json
{
  "id": "random_numbers",
  "label": "Random",
  "columns": [
    { "label": "Random", "type": "Point" }
  ]
}
```

## Performance Considerations

### Text Measurement

`api.graphics.measure_text()` is safe for visual layout, but do not use its result to decide gameplay state.

During normal play Moonshine measures text through the renderer. During server audit/replay, the runtime uses a deterministic approximation because the server does not load SDL graphics. If a mod uses text measurements to update `api.state.save`, submit scores, unlock milestones, award badges, or decide the session outcome, the audited result may diverge from the local play result.

### Menu Updates

- **Menus built once** at initialization
- **Switching variations** hides/shows pre-built menus (fast)
- **Selecting options** expands/collapses children as needed

No performance concerns for normal-sized menus.

### Leaderboard Submissions

Submit scores sparingly:
```lua
-- ✅ Good: Submit at significant events
if level_completed then
  SubmitLeaderboardScore("completion_times", {time = elapsed})
end

-- ❌ Bad: Don't spam submissions
function update()
  SubmitLeaderboardScore(...)  -- Every frame!
end
```

## Visibility vs. Accessibility Cheatsheet

| Feature | Visible | Greyed? | Meaning |
|---------|---------|---------|---------|
| No gates | ✓ | ✗ | Always accessible |
| `requiredMilestone` | ✓ | ✓ | Locked (show hint) |
| `visibleFromMilestone` | ✗ | - | Hidden (teaser) |
| Both gates | [Timeline](#both-gates-progressive-reveal) | [Timeline](#both-gates-progressive-reveal) | Progressive reveal |

## Common Patterns

### Single Difficulty
```json
{
  "variations": [
    { "id": "game", "label": "Play Game" }
  ]
}
```

### Difficulty Tiers
```json
{
  "variations": [
    { "id": "easy", "label": "Easy" },
    { "id": "normal", "label": "Normal", "requiredMilestone": "beat_easy" },
    { "id": "hard", "label": "Hard", "requiredMilestone": "beat_normal" }
  ]
}
```

### Multiple Parallel Modes
```json
{
  "variations": [
    { "id": "campaign", "label": "Campaign" },
    { "id": "sandbox", "label": "Sandbox" },
    { "id": "arcade", "label": "Arcade Mode" }
  ]
}
```

### Progression + Parallel
```json
{
  "variations": [
    { "id": "tutorial", "label": "Tutorial" },
    { "id": "main", "label": "Main Game", "requiredMilestone": "beat_tutorial" },
    { "id": "sandbox", "label": "Sandbox", "requiredMilestone": "beat_main" },
    { "id": "endless", "label": "Endless", "requiredMilestone": "beat_main" }
  ]
}
```

## Testing Checklist

Before shipping your mod:

- [ ] Can a new player start the game?
- [ ] Does at least one variation always work?
- [ ] Can every menu selection be made?
- [ ] Are all milestones actually achievable?
- [ ] Do progression gates feel natural?
- [ ] Can leaderboards be accessed?
- [ ] Do descriptions make sense?
- [ ] Are all IDs unique and valid?

## Debugging

### Mod Won't Start
- Check manifest JSON syntax
- Verify at least one variation is accessible
- Ensure entry point file exists

### Menu Broken
- Check all menu options have at least one accessible choice
- Verify milestone IDs are defined
- Look for typos in milestone references

### Scores Not Submitting
- Check leaderboard ID exists in manifest
- Verify column names match submission code
- Ensure scores are in correct format (seconds for Chrono, etc.)

---

**Next:**
- **→ [Complete Example]({{ site.baseurl }}{% link example-progression-mode.md %})** - See full working code

**Related:**
- **← [Progression System]({{ site.baseurl }}{% link progression-milestones.md %})** - Gate features with milestones
- **← [Leaderboards]({{ site.baseurl }}{% link leaderboards.md %})** - Track player performance

**Back to:** [Home]({{ site.baseurl }}{% link index.md %})
