---
layout: page
title: Progression System
---

Milestones and badges represent player progression for a ROM. They are declared
in the manifest and handled in Lua through `api.progress`.

## Milestones

A milestone is an internal progression flag:

```json
{
  "milestones": [
    "tutorial_completed",
    "beat_easy",
    "beat_normal"
  ]
}
```

Milestone ids must contain only letters, numbers, `_`, or `-`, and must be 32
characters or fewer.

Use milestones to:

- Unlock variants.
- Unlock or reveal menu inputs and options.
- Unlock or reveal ranking tables.
- Let Lua check whether a player has already completed something.

## Unlocking Milestones in Lua

```lua
if boss_defeated then
  api.progress.unlock_milestone("beat_easy")
end
```

Check existing progression:

```lua
if api.progress.has_milestone("beat_easy") then
  bonus_available = true
end

local earned = api.progress.get_milestones()
for _, milestone in ipairs(earned) do
  api.log.info("Earned milestone: " .. milestone)
end
```

`get_milestones()` returns a snapshot. Mutating that Lua table does not change
the player's progression. Use `unlock_milestone` to add progress.

## Session Sync

For a server-backed session, Avalon sends the player's earned milestones when
the catalog and session are prepared. When the session ends, Moonshine reports
newly unlocked milestones back to Avalon.

In local maker mode, Moonshine may persist test progress locally so Game Makers can
try gates without a server round-trip. Public ROM code should still use
`api.progress`; do not rely on hand-edited progress files as a Game Maker API.

## Gating Variants

```json
{
  "milestones": ["beat_easy"],
  "variants": [
    {
      "id": "easy",
      "label": "Easy"
    },
    {
      "id": "hard",
      "label": "Hard",
      "requiredMilestone": "beat_easy"
    }
  ]
}
```

`easy` is available immediately. `hard` becomes available once the player earns
`beat_easy`.

## Gating Menu Options

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

The first option of a menu input cannot be milestone-gated.

## Visibility vs Access

`requiredMilestone` and `visibleFromMilestone` work on variants, menu inputs,
menu options, badges, and ranking tables.

```json
{
  "id": "extreme",
  "label": "Extreme",
  "visibleFromMilestone": "beat_normal",
  "requiredMilestone": "beat_hard"
}
```

- Before `beat_normal`: hidden.
- After `beat_normal`: visible but unavailable.
- After `beat_hard`: available.

## Badges

Badges are visible rewards declared separately from milestones:

```json
{
  "badges": [
    {
      "id": "gm",
      "label": "GM",
      "description": "Reached GM rank",
      "iconIndex": 0,
      "requiredMilestone": "beat_hard"
    }
  ]
}
```

Lua can unlock and query badges:

```lua
if rank == "GM" then
  api.progress.unlock_badge("gm")
end

if api.progress.has_badge("gm") then
  api.log.info("GM badge already earned")
end
```

Badge UI presentation is still evolving, but the manifest and Lua progress API
already support badge ids.

## Complete Example

```json
{
  "name": "Progressive Adventure",
  "version": "1.0.0",
  "modeType": "Solo",
  "manifestApiVersion": 1,
  "milestones": [
    "beat_chapter_1",
    "beat_chapter_2",
    "master_rank"
  ],
  "variants": [
    {
      "id": "chapter_1",
      "label": "Chapter 1"
    },
    {
      "id": "chapter_2",
      "label": "Chapter 2",
      "requiredMilestone": "beat_chapter_1"
    },
    {
      "id": "master",
      "label": "Master",
      "visibleFromMilestone": "beat_chapter_2",
      "requiredMilestone": "master_rank"
    }
  ]
}
```

```lua
function update()
  if chapter_completed then
    api.progress.unlock_milestone("beat_chapter_" .. current_chapter)
  end

  if api.progress.has_milestone("beat_chapter_2") and score >= 10000 then
    api.progress.unlock_milestone("master_rank")
  end
end
```

## Next

- **[Menus & Configuration]({{ site.baseurl }}{% link menus-configuration.md %})** - Configure variants before launch.
- **[Leaderboards]({{ site.baseurl }}{% link leaderboards.md %})** - Ranking tables and score submission.
