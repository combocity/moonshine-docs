---
layout: page
title: Manifest Reference
---

Complete reference for `manifest.json`, the file that describes a Moonshine Lua
ROM.

For a first ROM, start with [Getting Started]({{ site.baseurl }}{% link getting-started.md %}).

## Overview

- JSON properties are case-insensitive.
- `manifestApiVersion` currently supports only `1`.
- `modeType` currently supports only `Solo`.
- `variants` is required and must contain at least one variant.
- `milestones` is required and may be empty.
- Ids must contain only letters, numbers, `_`, or `-`, and must be 32
  characters or fewer.
- Ids must not collide across variants, menu inputs, badges, and ranking tables.

## Minimal Manifest

```json
{
  "author": "Your Name",
  "name": "My ROM",
  "version": "1.0.0",
  "modeType": "Solo",
  "manifestApiVersion": 1,
  "milestones": [],
  "variants": [
    {
      "id": "classic",
      "label": "Classic"
    }
  ]
}
```

## Metadata

| Property | Required | Description |
|----------|----------|-------------|
| `author` | No | Informational author display name. |
| `name` | Yes | ROM display name. |
| `version` | Yes | SemVer version string. |
| `modeType` | Yes | Must be `Solo`. |
| `manifestApiVersion` | No | Defaults to `1`; only `1` is supported. |
| `description` | No | Longer ROM description. |
| `scripts` | No | Relative Lua script paths. If provided, it must include `main.lua`. |

`main.lua` must be present in the ROM. Script paths cannot be absolute, cannot
use `..`, and must end with `.lua`.

## Milestones

```json
{
  "milestones": ["beat_easy", "beat_normal"]
}
```

- Required, but can be an empty array.
- Maximum 120 milestones.
- Each id must be 32 characters or fewer.
- Each id may contain only letters, numbers, `_`, or `-`.
- Manifest gates must reference milestones declared here.

Runtime progression is handled through `api.progress`, not through public
author-managed files.

## Access Gates

Variants, menu inputs, menu options, badges, and ranking tables can define:

| Property | Meaning |
|----------|---------|
| `requiredMilestone` | Element is visible but unavailable until this milestone is earned. |
| `visibleFromMilestone` | Element is hidden until this milestone is earned. |

Both must reference known milestone ids.

## Variants

```json
{
  "id": "hard",
  "label": "Hard",
  "description": "A harder ruleset",
  "refreshRate": 60,
  "inputBuffer": 3,
  "requiredMilestone": "beat_easy",
  "menuInputs": []
}
```

| Property | Required | Description |
|----------|----------|-------------|
| `id` | Yes | Unique variant id, max 32 chars. |
| `label` | Yes | Display label, max 32 chars. |
| `description` | No | Longer description. |
| `earnedDescription` | No | Text for earned/available states when supported. |
| `refreshRate` | No | Valid range is 30 to 120. |
| `inputBuffer` | No | Valid range is 0 to 5. |
| `menuInputs` | No | Variant-specific menu inputs. |

## Menu Inputs

```json
{
  "id": "difficulty",
  "label": "Difficulty",
  "type": "Select",
  "options": [
    { "label": "Easy" },
    { "label": "Hard", "requiredMilestone": "beat_easy" }
  ]
}
```

| Property | Required | Description |
|----------|----------|-------------|
| `id` | Yes | Unique menu input id, max 32 chars. |
| `label` | Yes | Display label, max 32 chars. |
| `type` | No | `Select` by default. `Radio` is also valid. |
| `description` | No | Longer description. |
| `earnedDescription` | No | Text for earned/available states when supported. |
| `options` | Yes | At least two options. |

The first option cannot define `requiredMilestone` or `visibleFromMilestone`.

Options support:

| Property | Required | Description |
|----------|----------|-------------|
| `label` | Yes | Display label and Lua selection value, max 32 chars. |
| `description` | No | Longer description. |
| `earnedDescription` | No | Text for earned/available states when supported. |
| `inputs` | No | Nested menu inputs. |

Lua reads selected values from `api.session.selection`.

## Badges

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

| Property | Required | Description |
|----------|----------|-------------|
| `id` | Yes | Unique badge id, max 32 chars. |
| `label` | Yes | Display label, max 32 chars. |
| `description` | No | Locked/unearned description. |
| `earnedDescription` | No | Earned description. |
| `iconIndex` | Yes | Integer from 0 to 31, unique across badges. |
| `overrideBadge` | No | Badge id replaced by this badge. |

At most 32 badges can be declared.

## Ranking Tables

Ranking tables are currently a manifest definition. Score submission from Lua is
not available yet.

```json
{
  "rankingTables": [
    {
      "id": "master",
      "label": "Master",
      "columns": [
        { "label": "Time", "type": "Chrono" },
        { "label": "Score", "type": "Point" }
      ]
    }
  ]
}
```

| Property | Required | Description |
|----------|----------|-------------|
| `id` | Yes | Unique ranking table id, max 32 chars. |
| `label` | Yes | Display label, max 32 chars. |
| `columns` | Yes | At least one column. |

Column `type` must be `Badge`, `Chrono`, or `Point`. Column `label` is required
and must be 32 characters or fewer.

## Resources

```json
{
  "resources": {
    "images": [
      {
        "id": "blocks",
        "fileName": "blocks.png",
        "sprites": [
          { "id": "red", "x": 0, "y": 0, "width": 8, "height": 8 }
        ]
      }
    ],
    "sfx": [
      { "id": "clear", "fileName": "clear.ogg" }
    ],
    "musics": [
      { "id": "theme", "fileName": "theme.ogg" }
    ],
    "fonts": [
      { "id": "main", "fileName": "main.ttf", "ttfFontSize": 16 }
    ]
  }
}
```

Resource file names are relative to the matching asset folder in the ROM.

## Cartridge Option

`obfuscatePackageContent` controls whether manifest, scripts, and resources are
stored obfuscated inside the packed `.t3rom` cartridge. It defaults to `true`.

```json
{
  "obfuscatePackageContent": true
}
```

## Related

- **[Variants & Modes]({{ site.baseurl }}{% link variants-and-modes.md %})** - Variant design.
- **[Menus & Configuration]({{ site.baseurl }}{% link menus-configuration.md %})** - Menu definitions.
- **[Progression System]({{ site.baseurl }}{% link progression-milestones.md %})** - Milestones and badges.
- **[Leaderboards]({{ site.baseurl }}{% link leaderboards.md %})** - WIP ranking table definitions.
