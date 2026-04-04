---
layout: page
title: Complete Example
---

A full working example that demonstrates variations, menus, progression, and leaderboards all working together.

## Mod: "Tower Climber"

A difficulty-based game where players climb progressively harder towers. Each tower conquered unlocks the next difficulty level.

## Structure

```
tower-climber/
├── manifest.json       # Configuration
├── main.lua            # Game logic
├── SaveState.txt       # Example save
└── milestones.txt      # Example progress
```

## manifest.json

```json
{
  "author": "Example Designer",
  "name": "Tower Climber",
  "version": "1.0.0",
  "modeType": "Solo",
  "manifestApiVersion": 1,
  "description": "Climb towers of increasing difficulty. Beat each tower to unlock the next challenge.",
  "milestones": [
    "beat_tower_1",
    "beat_tower_2",
    "beat_tower_3",
    "master_rank"
  ],
  "variations": [
    {
      "id": "tower_1",
      "label": "Tower 1: Beginner",
      "description": "Start your climb here",
      "menuInputs": [
        {
          "id": "difficulty",
          "label": "Difficulty Modifier",
          "type": "select",
          "options": [
            { "label": "Normal" },
            { "label": "Challenge", "requiredMilestone": "beat_tower_1" }
          ]
        }
      ]
    },
    {
      "id": "tower_2",
      "label": "Tower 2: Intermediate",
      "description": "The real test begins",
      "requiredMilestone": "beat_tower_1",
      "menuInputs": [
        {
          "id": "difficulty",
          "label": "Difficulty Modifier",
          "options": [
            { "label": "Normal" },
            { "label": "Hardcore", "requiredMilestone": "beat_tower_2" }
          ]
        }
      ]
    },
    {
      "id": "tower_3",
      "label": "Tower 3: Expert",
      "description": "Only masters may enter",
      "requiredMilestone": "beat_tower_2",
      "visibleFromMilestone": "beat_tower_2",
      "menuInputs": [
        {
          "id": "difficulty",
          "label": "Difficulty Modifier",
          "options": [
            { "label": "Expert" },
            { "label": "Insane", "requiredMilestone": "beat_tower_3" }
          ]
        }
      ]
    },
    {
      "id": "sandbox",
      "label": "Sandbox Mode",
      "description": "Play any tower freely",
      "requiredMilestone": "master_rank",
      "visibleFromMilestone": "master_rank",
      "menuInputs": [
        {
          "id": "tower_select",
          "label": "Select Tower",
          "options": [
            { "label": "Tower 1" },
            { "label": "Tower 2" },
            { "label": "Tower 3" }
          ]
        },
        {
          "id": "sandbox_difficulty",
          "label": "Difficulty",
          "options": [
            { "label": "Easy" },
            { "label": "Normal" },
            { "label": "Hard" },
            { "label": "Impossible" }
          ]
        }
      ]
    }
  ],
  "rankingTables": [
    {
      "id": "tower_1_times",
      "label": "Tower 1 Times",
      "columns": [
        { "label": "Best Time", "type": "Chrono" }
      ]
    },
    {
      "id": "tower_2_times",
      "label": "Tower 2 Times",
      "requiredMilestone": "beat_tower_1",
      "columns": [
        { "label": "Best Time", "type": "Chrono" }
      ]
    },
    {
      "id": "tower_3_times",
      "label": "Tower 3 Times",
      "requiredMilestone": "beat_tower_2",
      "columns": [
        { "label": "Best Time", "type": "Chrono" }
      ]
    },
    {
      "id": "overall_ranking",
      "label": "Overall Rankings",
      "requiredMilestone": "master_rank",
      "columns": [
        { "label": "Towers Climbed", "type": "Point" },
        { "label": "Best Time", "type": "Chrono" },
        { "label": "Challenge Modes", "type": "Badge" }
      ]
    }
  ]
}
```

## main.lua

```lua
-- Tower Climber Game
-- Main entry point

-- Game state
local game_state = {
  current_tower = 1,
  current_difficulty = "normal",
  start_time = 0,
  current_time = 0,
  enemies_defeated = 0,
  tower_complete = false
}

-- Initialize game
function initialize()
  print("Tower Climber initialized!")

  -- Get selected variation
  local variation = GetSelectedVariation()
  print("Playing variation: " .. variation)

  -- Determine tower based on variation
  if variation == "tower_1" then
    game_state.current_tower = 1
  elseif variation == "tower_2" then
    game_state.current_tower = 2
  elseif variation == "tower_3" then
    game_state.current_tower = 3
  elseif variation == "sandbox" then
    local tower_select = GetMenuSelection("tower_select")
    game_state.current_tower = tonumber(tower_select:match("%d+"))
  end

  -- Get difficulty modifier
  if variation == "sandbox" then
    game_state.current_difficulty = GetMenuSelection("sandbox_difficulty")
  else
    game_state.current_difficulty = GetMenuSelection("difficulty")
  end

  print("Tower: " .. game_state.current_tower)
  print("Difficulty: " .. game_state.current_difficulty)

  game_state.start_time = GetCurrentTime()
end

-- Update game logic each frame
function update()
  if game_state.tower_complete then
    return
  end

  game_state.current_time = GetCurrentTime() - game_state.start_time

  -- Simulate climbing tower
  -- (Replace with actual game logic)
  game_state.enemies_defeated = game_state.enemies_defeated + 1

  -- Win condition: Defeat enough enemies
  local enemies_needed = get_enemies_for_tower(
    game_state.current_tower,
    game_state.current_difficulty
  )

  if game_state.enemies_defeated >= enemies_needed then
    complete_tower()
  end
end

-- Draw game
function draw()
  -- Render game here
  -- (Replace with actual rendering)
end

-- Helper: Get enemies needed based on tower and difficulty
function get_enemies_for_tower(tower, difficulty)
  local base = tower * 10

  if difficulty == "normal" or difficulty == "Normal" then
    return base
  elseif difficulty == "challenge" or difficulty == "Challenge" then
    return base + 10
  elseif difficulty == "hardcore" or difficulty == "Hardcore" then
    return base * 2
  elseif difficulty == "insane" or difficulty == "Insane" then
    return base * 3
  elseif difficulty == "expert" or difficulty == "Expert" then
    return base * 2
  elseif difficulty == "easy" or difficulty == "Easy" then
    return base / 2
  elseif difficulty == "hard" or difficulty == "Hard" then
    return base * 2
  elseif difficulty == "impossible" or difficulty == "Impossible" then
    return base * 5
  end

  return base
end

-- Called when tower is completed
function complete_tower()
  game_state.tower_complete = true

  print("Tower " .. game_state.current_tower .. " completed in " ..
        format_time(game_state.current_time) .. "!")

  -- Record milestone
  local milestone = "beat_tower_" .. game_state.current_tower
  RecordMilestone(milestone)

  -- Check if all towers beaten
  if HasMilestone("beat_tower_1") and
     HasMilestone("beat_tower_2") and
     HasMilestone("beat_tower_3") then
    RecordMilestone("master_rank")
    print("Congratulations! You are now a MASTER CLIMBER!")
  end

  -- Submit to leaderboard
  submit_score_to_leaderboard(
    game_state.current_tower,
    game_state.current_time,
    game_state.current_difficulty
  )
end

-- Submit score to appropriate leaderboard
function submit_score_to_leaderboard(tower, time, difficulty)
  local leaderboard_id = "tower_" .. tower .. "_times"

  print("Submitting score to: " .. leaderboard_id)

  SubmitLeaderboardScore(leaderboard_id, {
    time = math.floor(time)  -- Convert to seconds
  })

  -- Also submit to overall ranking if master
  if HasMilestone("master_rank") then
    local challenge_modes = 0
    if HasMilestone("challenge_tower_1") then challenge_modes = challenge_modes + 1 end
    if HasMilestone("challenge_tower_2") then challenge_modes = challenge_modes + 1 end
    if HasMilestone("challenge_tower_3") then challenge_modes = challenge_modes + 1 end

    SubmitLeaderboardScore("overall_ranking", {
      towers = 3,  -- All towers beaten
      time = math.floor(time),
      challenges = challenge_modes
    })
  end
end

-- Format time as MM:SS
function format_time(seconds)
  local mins = math.floor(seconds / 60)
  local secs = math.floor(seconds % 60)
  return string.format("%d:%02d", mins, secs)
end

-- Game lifecycle functions (called by engine)
function on_start()
  initialize()
end

function on_update()
  update()
end

function on_draw()
  draw()
end

function on_finish()
  print("Tower Climber session finished!")
  -- Save any final state if needed
end

-- Print progress summary
function get_progress_summary()
  local summary = "=== TOWER CLIMBER PROGRESS ===\n"

  if HasMilestone("beat_tower_1") then
    summary = summary .. "✓ Tower 1: BEATEN\n"
  else
    summary = summary .. "- Tower 1: Locked\n"
  end

  if HasMilestone("beat_tower_2") then
    summary = summary .. "✓ Tower 2: BEATEN\n"
  else
    summary = summary .. "- Tower 2: " ..
              (HasMilestone("beat_tower_1") and "Unlocked" or "Locked") .. "\n"
  end

  if HasMilestone("beat_tower_3") then
    summary = summary .. "✓ Tower 3: BEATEN\n"
  else
    summary = summary .. "- Tower 3: " ..
              (HasMilestone("beat_tower_2") and "Unlocked" or "Locked") .. "\n"
  end

  if HasMilestone("master_rank") then
    summary = summary .. "👑 MASTER RANK ACHIEVED!\n"
  end

  return summary
end

-- Initialize on load
print(get_progress_summary())
```

## SaveState.txt (Example)

```
current_level=5
best_time_tower_1=300
best_time_tower_2=450
best_time_tower_3=600
total_attempts=15
```

## milestones.txt (Example)

```
# Player progression
beat_tower_1
beat_tower_2
master_rank
```

## How It Works

### Progression Flow

1. **Player launches** → Sees "Tower 1: Beginner" and "Tower 2" (locked)
2. **Beats Tower 1** → "beat_tower_1" milestone recorded
3. **Launches again** → Tower 2 now shows as accessible
4. **Beats Tower 2** → "beat_tower_2" milestone recorded
5. **Launches again** → Tower 3 appears (was hidden before)
6. **Beats Tower 3** → "beat_tower_3" and "master_rank" recorded
7. **Launches again** → "Sandbox Mode" now unlocked

### Menu Flow

**Tower 1:**
- Difficulty: Normal, Challenge (locked until beat tower 1)

**Tower 2:**
- Difficulty: Normal, Hardcore (locked until beat tower 2)

**Tower 3:**
- Difficulty: Expert, Insane (locked until beat tower 3)

**Sandbox (Master only):**
- Select Tower: 1, 2, or 3
- Select Difficulty: Easy through Impossible

### Leaderboard Flow

1. Submits to appropriate tower leaderboard after each completion
2. Once master rank achieved, also submits to "Overall Rankings"

## Key Features Demonstrated

✓ Multiple variations with progression gates
✓ Nested menu options
✓ Milestone tracking and recording
✓ Conditional feature availability
✓ Multiple leaderboards
✓ Appropriate score submission
✓ Save state integration

---

**Learn more:**
- **→ [Variations & Modes]({% link variations-and-modes.md %})** - Multiple game types
- **→ [Menus & Configuration]({% link menus-configuration.md %})** - Player choices
- **→ [Progression System]({% link progression-milestones.md %})** - Progress tracking
- **→ [Leaderboards]({% link leaderboards.md %})** - Score tracking

**Back to:** [Home]({% link index.md %})
