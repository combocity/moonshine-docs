---
layout: page
title: Leaderboards
---

A ROM can declare ranking tables in its manifest and submit results from Lua.
Moonshine includes those results in the session report, and Avalon verifies them
during replay audit before persisting accepted ranking entries.

Querying rankings from Lua and the final player-facing ranking display are not
part of the Lua API v1 contract.

## Manifest Shape

Declare ranking tables with `rankingTables`:

```json
{
  "rankingTables": [
    {
      "id": "easy",
      "label": "Easy",
      "columns": [
        {
          "label": "Score",
          "type": "Point"
        },
        {
          "label": "Time",
          "type": "Chrono"
        },
        {
          "label": "Apples",
          "type": "Point"
        }
      ]
    }
  ]
}
```

## Ranking Table Properties

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `id` | string | Yes | Internal id, max 32 chars, unique across variants, menu inputs, badges, and ranking tables. |
| `label` | string | Yes | Display label, max 32 chars. |
| `columns` | array | Yes | At least one ranking column. |
| `requiredMilestone` | string | No | Milestone required to access this ranking table. |
| `visibleFromMilestone` | string | No | Milestone required before this ranking table is visible. |

## Column Types

| Type | Intended use |
|------|--------------|
| `Badge` | An integer badge/rank-like value, not a badge id string. |
| `Chrono` | A non-negative duration represented as an integer number of milliseconds. |
| `Point` | A numeric score value. |

Column labels are required, must be 32 characters or fewer, and must be unique
inside the table without regard to case.

## Submitting A Result

Call `api.ranking.submit_score()` once for the table before ending the session:

```lua
local submitted, submission_error = api.ranking.submit_score(
  "easy",
  score,
  {
    Score = score,
    Time = elapsed_ms,
    Apples = apples_eaten
  })

if not submitted then
  error(submission_error)
end

api.session.end_game()
```

```lua
api.ranking.submit_score(ranking_table_id, score, values)
-- returns: boolean, string|nil
```

| Argument | Description |
|----------|-------------|
| `ranking_table_id` | Id of a ranking table declared in `rankingTables`. |
| `score` | Integer technical ordering value, stored separately from the visible columns. For a simple score table, this can be the same value as the visible `Score` column. |
| `values` | Table containing one integer value for every declared column label. |

The function returns `true, nil` when the result is accepted, or `false` and an
error message when validation fails. Success means the result was added to the
current session report; it does not mean a public ranking entry has already been
audited or displayed.

A valid submission must follow these rules:

- The ranking table id must exist in the manifest.
- The technical score and every column value must be Lua integers.
- `values` must contain every declared column exactly once, with no extra columns.
- Column labels and the table id are matched without regard to case.
- `Chrono` values are non-negative durations in milliseconds.
- Only one successful result can be submitted to each ranking table during a session.
- Results must be submitted before `api.session.end_game()`.

Compute scores and durations from deterministic gameplay state in `update()`.
Do not derive submitted values from rendering, text measurement, or the operating
system clock, because Avalon replays the session headlessly during audit. In the
example, `elapsed_ms` is a value tracked by the game; `submit_score()` does not
measure time automatically.

Local maker sessions validate the same call, but they do not persist a local
leaderboard entry.

## Progression Gates

Ranking table definitions can use the same progression gates as variants and
menus:

```json
{
  "id": "expert",
  "label": "Expert",
  "requiredMilestone": "beat_normal",
  "columns": [
    {
      "label": "Time",
      "type": "Chrono"
    }
  ]
}
```

These gates control access and visibility. They do not automatically select a
ranking table for the current variant; the ROM chooses the table id it submits.

## API v1 Limitations

The public API does not provide:

- Querying a player's best score from Lua.
- Querying global rankings from Lua.
- Ranking ordering direction, tie-breaking, and final display behavior.

## Next

- **[Manifest Reference]({{ site.baseurl }}{% link manifest.md %})** - Full validation rules.
- **[Lua API v1 Reference]({{ site.baseurl }}{% link lua-api-v1.md %})** - `api.ranking.submit_score()` reference.
- **[Session Lifecycle]({{ site.baseurl }}{% link session-lifecycle.md %})** - Submission and audit flow.
- **[Progression System]({{ site.baseurl }}{% link progression-milestones.md %})** - Milestone gates.
