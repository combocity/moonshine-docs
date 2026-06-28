---
layout: page
title: Leaderboards
---

Leaderboards are work in progress.

The manifest can already declare ranking table definitions, and Avalon/Moonshine
already model those definitions in the catalog. Moonshine does not yet expose a
public Lua API for submitting or querying scores from a ROM.

## Current Manifest Shape

Declare ranking tables with `rankingTables`:

```json
{
  "rankingTables": [
    {
      "id": "master",
      "label": "Master",
      "columns": [
        {
          "label": "Grade",
          "type": "Badge"
        },
        {
          "label": "Time",
          "type": "Chrono"
        },
        {
          "label": "Score",
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
| `Badge` | A badge/rank-like value. |
| `Chrono` | A time value. |
| `Point` | A numeric score value. |

Column labels are required and must be 32 characters or fewer.

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

## Not Available Yet

The following pieces are intentionally not documented as working API yet:

- Submitting scores from Lua.
- Querying a player's best score from Lua.
- Querying global rankings from Lua.
- Final ranking display behavior.

Until those are implemented, treat `rankingTables` as a manifest contract and UI
catalog placeholder, not as a complete leaderboard feature.

## Next

- **[Manifest Reference]({{ site.baseurl }}{% link manifest.md %})** - Full validation rules.
- **[Progression System]({{ site.baseurl }}{% link progression-milestones.md %})** - Milestone gates.
