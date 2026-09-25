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
| `name` | Yes | ROM display name. |
| `version` | Yes | SemVer version string. |
| `modeType` | Yes | Must be `Solo`. |
| `manifestApiVersion` | No | Defaults to `1`; only `1` is supported. |
| `description` | No | Longer ROM description. |
| `scripts` | No | Relative Lua script paths. If non-empty, it must include every variant's effective entry point with exact casing. |

Script paths are relative to the manifest folder. They cannot be absolute, use
backslashes, contain empty, `.` or `..` path segments, or contain a colon, and
they must end with `.lua`.

## Variant Entry Points

Each variant can select its own Lua entry point with `entryPoint`:

```json
{
  "variants": [
    {
      "id": "classic",
      "label": "Classic"
    },
    {
      "id": "sprint",
      "label": "Sprint",
      "entryPoint": "variants/sprint.lua"
    }
  ]
}
```

If `entryPoint` is omitted, that variant uses `main.lua`. Consequently,
`main.lua` is required only when at least one variant omits `entryPoint`.
The fallback does not apply to an empty or whitespace-only value, which is
invalid. The selected file follows the normal Lua lifecycle: `init()` is
optional, while `update()` and `draw()` are required.

Entry point paths follow stricter rules than other script paths:

- The extension must be exactly lowercase `.lua`.
- Each slash-separated segment before the extension must be non-empty and
  cannot contain `.`. For example, `variants/sprint.lua` is valid, while
  `variants/sprint.v2.lua` is not.
- Moonshine converts the path to a Lua module name by removing `.lua` and
  replacing `/` with `.`. For example, `variants/sprint.lua` becomes
  `variants.sprint`.
- When `scripts` is non-empty, it must contain the effective entry point of
  every variant with exactly the same path and casing.
- Multiple variants may share the exact same entry point. Paths that differ
  only by casing are rejected.
- Two different script paths cannot resolve to the same Lua module name. For
  example, `foo/bar.lua` and `foo.bar.lua` both resolve to `foo.bar` and cannot
  coexist.

During packaging, Moonshine discovers the literal `require()` graph starting
from every variant entry point and packages the union of those scripts. Shared
entry points and dependencies are included only once. Calls whose module name
is computed at runtime are not discovered. Package generation rewrites
`scripts` with the discovered union, so source manifests normally omit that
property instead of maintaining it manually.

### Reserved entry-point module names

An entry point at the manifest root cannot use one of these exact,
case-sensitive module names reserved by the Lua runtime or Moonshine:

- `_G`
- `coroutine`
- `debug`
- `emmy_core`
- `io`
- `math`
- `os`
- `package`
- `string`
- `table`
- `utf8`

This restriction applies to the complete entry point module name. A nested
entry point such as `modes/math.lua`, which resolves to `modes.math`, remains
valid.

## Default Save State

A ROM may place an optional `default-save-state.json` companion file next to
its manifest. This is a fixed file name, not a manifest property or configurable
path. Moonshine converts its root JSON object into the initial `api.save` table
when the player has no persisted save.

The file belongs to the ROM folder rather than to a variant. All variants and
all manifests in that folder therefore share the same default. See
[Default Save State]({{ site.baseurl }}{% link default-save-state.md %}) for the
format, precedence, safety limits, and packaging behavior.

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
maker-maintained files.

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
  "entryPoint": "variants/hard.lua",
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
| `entryPoint` | No | Relative canonical Lua entry point. Defaults to `main.lua`. |
| `refreshRate` | No | Valid range is 30 to 120. |
| `inputBuffer` | No | Number of additional logical updates applied as input delay. Defaults to `0`. |
| `menuInputs` | No | Variant-specific menu inputs. |

An `inputBuffer` of `0` delivers input immediately. A positive value `N`
delays press, hold, and release states by exactly `N` logical updates.

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

Ranking tables define the result columns a ROM can submit through
`api.ranking.submit_score()`.

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

Column `type` must be `Badge`, `Chrono`, or `Point`. Column `label` is required,
must be 32 characters or fewer, and must be unique inside the table without
regard to case. A `Chrono` value is a non-negative integer duration in
milliseconds.

See [Leaderboards]({{ site.baseurl }}{% link leaderboards.md %}) for the Lua
submission API, validation rules, and ranking behavior.

## Resources

```json
{
  "resources": {
    "images": [
      {
        "id": "blocks",
        "fileName": "blocks.png",
        "mode": "manual",
        "sprites": [
          {
            "id": "red",
            "rect": { "x": 0, "y": 0, "w": 8, "h": 8 }
          }
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
      {
        "id": "main",
        "fileName": "main.ttf",
        "ttfFontSize": 16,
        "localization": {
          "title": "Ready"
        }
      }
    ]
  }
}
```

Each image supports:

| Property | Required | Description |
|----------|----------|-------------|
| `id` | Yes | Unique image id used by the Lua graphics API. |
| `fileName` | Yes | File name under `assets/images/`, without a path. |
| `mode` | Yes | `manual` or `mask`. |
| `maskColor` | In `mask` mode | Exact frame color in `#RRGGBB` format. |
| `hash` | No | Texture SHA-256 generated by mask synchronization. |
| `definitionHash` | No | Canonical image-definition SHA-256 generated by mask synchronization. |
| `sprites` | No | Sprite definitions inside the image. |

Each sprite supports:

| Property | Required | Description |
|----------|----------|-------------|
| `id` | Yes | Sprite id, unique inside its image. |
| `keyColor` | No | Technical `#RRGGBB` marker used by mask synchronization. |
| `rect` | Yes | `{ "x", "y", "w", "h" }` source rectangle; position must be non-negative and size positive. |

Resource file names are relative to the matching asset folder in the ROM.
For practical resource usage, see
[ROM Resources]({{ site.baseurl }}{% link resources.md %}). For sprite atlas
authoring and mask synchronization, see
[Sprite Atlases]({% link sprite-atlases.md %}).

## Preview Obfuscation

Preview obfuscation is chosen during Preview publication. It is not a public
manifest property.

## Related

- **[Variants & Modes]({{ site.baseurl }}{% link variants-and-modes.md %})** - Variant design.
- **[Menus & Configuration]({{ site.baseurl }}{% link menus-configuration.md %})** - Menu definitions.
- **[Progression System]({{ site.baseurl }}{% link progression-milestones.md %})** - Milestones and badges.
- **[Leaderboards]({{ site.baseurl }}{% link leaderboards.md %})** - Ranking tables and score submission.
- **[Default Save State]({{ site.baseurl }}{% link default-save-state.md %})** - Optional initial `api.save` data.
- **[ROM Resources]({{ site.baseurl }}{% link resources.md %})** - Images, audio, fonts, and badge assets.
- **[Sprite Atlases]({% link sprite-atlases.md %})** - Manual sprite rectangles and mask synchronization.
