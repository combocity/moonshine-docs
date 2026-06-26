---
layout: page
title: Leaderboards
---

Leaderboards track and display player performance. Players can see global rankings and compare their scores against others.

## What are Leaderboards?

A **leaderboard** is a ranked list of player scores. Each leaderboard tracks a specific metric or achievement:

- **Time-based:** Who completed a challenge fastest?
- **Score-based:** Who earned the most points?
- **Multi-column:** Who has the best combination of time + score + badges?

## Creating a Leaderboard

Define leaderboards in your manifest:

```json
{
  "rankingTables": [
    {
      "id": "fastest_times",
      "label": "Fastest Times",
      "columns": [
        {
          "label": "Time",
          "type": "Chrono"
        }
      ]
    }
  ]
}
```

### Leaderboard Properties

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `id` | string | Yes | Internal ID (≤32 chars, unique) |
| `label` | string | Yes | Display name (≤32 chars) |
| `columns` | array | Yes | Score columns to track |
| `description` | string | No | Longer description |
| `requiredMilestone` | string | No | Milestone to see this leaderboard |
| `visibleFromMilestone` | string | No | Milestone to show as "coming soon" |

### Column Types

Each leaderboard can have multiple columns. Three types are supported:

#### Chrono (Time)
Tracks fastest times (lower is better):
```json
{
  "label": "Time",
  "type": "Chrono"
}
```
Displayed as MM:SS or HH:MM:SS

#### Point (Score)
Tracks highest scores (higher is better):
```json
{
  "label": "Points",
  "type": "Point"
}
```
Displayed as numeric values

#### Badge (Achievement)
Tracks earned achievements:
```json
{
  "label": "Badges",
  "type": "Badge"
}
```
Displays badge icons or counts

## Single-Column Leaderboards

Track one metric:

```json
{
  "id": "high_scores",
  "label": "High Scores",
  "description": "All-time highest scores",
  "columns": [
    {
      "label": "Score",
      "type": "Point"
    }
  ]
}
```

**Result:** Simple ranking showing player name and score.

## Multi-Column Leaderboards

Track multiple metrics:

```json
{
  "id": "ranked_stats",
  "label": "Ranked Statistics",
  "columns": [
    {
      "label": "Time",
      "type": "Chrono"
    },
    {
      "label": "Score",
      "type": "Point"
    },
    {
      "label": "Badges",
      "type": "Badge"
    }
  ]
}
```

**Result:** Players ranked by time (primary), then score, then badges.

**Display Example:**
```
Rank | Player | Time   | Score | Badges
-----|--------|--------|-------|-------
1    | Player1| 5:23   | 8500  | 12
2    | Player2| 5:45   | 7800  | 10
3    | Player3| 6:01   | 9200  | 8
```

## Gating Leaderboards

Use milestones to control leaderboard visibility:

### Locked Leaderboard
```json
{
  "id": "expert_scores",
  "label": "Expert Scores",
  "requiredMilestone": "expert_status",
  "columns": [...]
}
```
Greyed out until "expert_status" is earned.

### Hidden Leaderboard (Teaser)
```json
{
  "id": "season_2",
  "label": "Season 2 Rankings",
  "visibleFromMilestone": "season_1_complete",
  "requiredMilestone": "season_2_active",
  "columns": [...]
}
```
Hidden until "season_1_complete", then shown as greyed until "season_2_active".

## Submitting Scores in Lua

Your script submits scores to leaderboards:

```lua
-- Submit a single score
SubmitLeaderboardScore("high_scores", {
  score = 8500
})

-- Submit multiple columns
SubmitLeaderboardScore("ranked_stats", {
  time = 320,      -- 5:20 in seconds
  score = 8500,
  badges = 12
})
```

### Score Format

| Type | Format | Example |
|------|--------|---------|
| Chrono | Seconds (integer) | 320 (= 5:20) |
| Point | Integer | 8500 |
| Badge | Integer (count) | 12 |

## Design Patterns

### Performance Tracking
```json
{
  "id": "completion_times",
  "label": "Completion Times",
  "columns": [
    { "label": "Time", "type": "Chrono" }
  ]
}
```

### Score Chase
```json
{
  "id": "highest_scores",
  "label": "Highest Scores",
  "columns": [
    { "label": "Score", "type": "Point" }
  ]
}
```

### Multi-Metric Ranking
```json
{
  "id": "overall_ranking",
  "label": "Overall Rankings",
  "columns": [
    { "label": "Wins", "type": "Point" },
    { "label": "Average Time", "type": "Chrono" },
    { "label": "Achievements", "type": "Badge" }
  ]
}
```

### Seasonal Leaderboards
```json
{
  "id": "season_1",
  "label": "Season 1",
  "requiredMilestone": "season_1_available",
  "columns": [
    { "label": "Score", "type": "Point" }
  ]
},
{
  "id": "season_2",
  "label": "Season 2",
  "visibleFromMilestone": "season_1_done",
  "requiredMilestone": "season_2_started",
  "columns": [
    { "label": "Score", "type": "Point" }
  ]
}
```

### Difficulty-Specific Rankings
```json
{
  "id": "easy_times",
  "label": "Easy Mode Times",
  "columns": [
    { "label": "Time", "type": "Chrono" }
  ]
},
{
  "id": "hard_times",
  "label": "Hard Mode Times",
  "requiredMilestone": "beat_easy",
  "columns": [
    { "label": "Time", "type": "Chrono" }
  ]
}
```

## Complete Example

**manifest.json:**
```json
{
  "name": "Speed Runner",
  "rankingTables": [
    {
      "id": "all_times",
      "label": "All-Time Rankings",
      "columns": [
        {
          "label": "Best Time",
          "type": "Chrono"
        }
      ]
    },
    {
      "id": "difficulty_scores",
      "label": "Difficulty Challenges",
      "columns": [
        {
          "label": "Difficulty Level",
          "type": "Point"
        },
        {
          "label": "Time",
          "type": "Chrono"
        }
      ]
    },
    {
      "id": "daily_challenge",
      "label": "Today's Challenge",
      "requiredMilestone": "daily_unlocked",
      "columns": [
        {
          "label": "Score",
          "type": "Point"
        }
      ]
    }
  ]
}
```

**main.lua:**
```lua
function on_level_complete(completion_time, difficulty_level, score)
  -- Submit to all-times leaderboard
  SubmitLeaderboardScore("all_times", {
    time = completion_time
  })

  -- Submit to difficulty leaderboard
  SubmitLeaderboardScore("difficulty_scores", {
    difficulty_level = difficulty_level,
    time = completion_time
  })

  -- Submit to daily challenge if available
  if HasMilestone("daily_unlocked") then
    SubmitLeaderboardScore("daily_challenge", {
      score = score
    })
  end
end
```

## Best Practices

✅ **DO:**
- Create meaningful leaderboards that match your gameplay
- Use appropriate column types (Chrono for times, Point for scores)
- Submit scores after significant events (level complete, session end)
- Gate leaderboards behind progression for progression-based games
- Keep leaderboard IDs descriptive
- Provide clear column labels

❌ **DON'T:**
- Create leaderboards with meaningless data
- Mix incompatible column types unnecessarily
- Submit scores constantly (only for significant events)
- Gate the only leaderboard
- Use vague column names
- Create more leaderboards than necessary

## Querying Leaderboard Data

In your script, access submitted scores:

```lua
-- Get player's best score
local best_score = GetLeaderboardScore("high_scores")

-- Get global rankings (top 10)
local top_rankings = GetTopLeaderboardScores("high_scores", 10)

-- Get player's rank
local rank = GetLeaderboardRank("high_scores")
```

---

**Next:**
- **→ [Best Practices]({{ site.baseurl }}{% link best-practices.md %})** - Design guidelines
- **→ [Complete Example]({{ site.baseurl }}{% link example-progression-mode.md %})** - Full working code

**Related:**
- **← [Progression System]({{ site.baseurl }}{% link progression-milestones.md %})** - Gate features with milestones
- **← [Getting Started]({{ site.baseurl }}{% link getting-started.md %})** - Create your first ROM

**Back to:** [Home]({{ site.baseurl }}{% link index.md %})

