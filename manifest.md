---
layout: page
title: Manifest Reference
---

Complete technical reference for the manifest.json file. This document describes the JSON manifest format used by Moonshine games in detail.

For modders just getting started, see [Getting Started]({% link getting-started.md %}) or [Variations & Modes]({% link variations-and-modes.md %}).

## Overview

This document describes the complete manifest format and validation rules.

## Table of contents
- [Overview](#overview)
- [Required metadata](#required-metadata)
- [Progression and milestones](#progression-and-milestones)
- [Badges](#badges)
- [Variations and menus](#variations-and-menus)
- [Leaderboards (rankingTables)](#leaderboards-rankingtables)
- [Resources (resources.images)](#resources-resourcesimages)
- [Package obfuscation](#package-obfuscation)
- [Integration reminders for modders](#integration-reminders-for-modders)

## Overview
- The manifest is required and must be valid before a session can start.
- JSON properties are case-insensitive.
- `manifestApiVersion` is an integer; only value `1` is supported.
  Missing values default to `1`.
- There is no separate manifest-schema version field today. `manifestApiVersion`
  selects the Lua API/runtime compatibility level.
- Identifiers (`id`) must be unique within their collection and must not
  collide across badges, variations, leaderboards, and menu entries.
- Milestones define the progression tree and back the access/visibility rules
  of other blocks.

## Required metadata
- `author` (string, required): informational display.
- `manifestApiVersion` (int, optional): only value `1` is supported. Defaults to `1`.
- `name` (string, required): mode name.
- `scriptVersion` (string, required): free-form.
- `main.lua` (file, required): must exist next to the manifest.
- `description` (string, optional).

## Progression and milestones
- `milestones` (string array, required but may be empty).
- Moonshine can also read `milestones.txt` next to the manifest to know which
  milestones are unlocked for the player. It is separate from `SaveState.txt`.
  - One milestone id per line; only the first token before whitespace is used.
  - Lines starting with `#` are treated as comments.
- Any `AccessControlledElement` (badges, variations, menu entries/options,
  leaderboards) may define:
  - `requiredMilestone`: milestone that unlocks access.
  - `visibleFromMilestone`: milestone that makes the element visible before
    unlock.
- References to unknown milestones are invalid.

## Badges
- Optional `badges` array.
- Fields:
  - `id` (string, required, <= 32 chars).
  - `label` (string, required, <= 32).
  - `description` / `earnedDescription` (optional).
  - `iconIndex` (required): integer between `0` and `31`, unique across badges.
  - `overrideBadge` (optional): badge being replaced.
  - `requiredMilestone` / `visibleFromMilestone` (optional) must target known
    milestones.
- `id` values must be unique and must not collide with variation, leaderboard,
  or menu ids.

## Variations and menus
- `variations` array is required (at least one variation).
- Variation:
  - `id` (required, <= 32).
  - `label` (required, <= 32).
  - `description` / `earnedDescription` (optional).
  - `refreshRate` (optional): when present, must be between `30` and `120`
    (currently validated but not used at runtime).
  - `inputBuffer` (optional): when present, must be between `0` and `5`
    (currently validated but not used at runtime).
  - `requiredMilestone` / `visibleFromMilestone` (optional).
  - `menuInputs` (optional).
- `menuInputs` (top-level or nested in options):
  - `id` (required, <= 32) and `label` (required, <= 32).
  - `type` (optional): currently only `select` is supported. When missing or
    `select` (case-insensitive), `options` is required and must contain at
    least 2 entries.
  - `options` (optional otherwise).
  - `requiredMilestone` / `visibleFromMilestone` (optional).
- `options` (required for `select`):
  - `label` (required, <= 32): used as the selection value passed to Lua and
    the override file.
  - `description` / `earnedDescription` (optional).
  - `inputs` (optional): allows nesting new `menuInputs`.
  - `requiredMilestone` / `visibleFromMilestone` (optional).
- Uniqueness:
  - Variation ids are unique.
  - `menuInputs` ids must be unique across all variations.
  - Id collisions across `Badge`, `Variation`, `RankingTable`, and `MenuInput`
    are not allowed.

## Leaderboards (rankingTables)
- Optional `rankingTables` array.
- Fields:
  - `id` (required, <= 32, unique).
  - `label` (required, <= 32).
  - `columns` (required, non-empty).
  - `requiredMilestone` / `visibleFromMilestone` (optional, must target known
    milestones).
- Columns:
  - `label` (required, <= 32).
  - `type` (required): `Badge`, `Chrono`, or `Point`.

## Resources (`resources`)
- Optional `resources` block.
- `images`: list of images keyed by `id` (uniqueness enforced).
  - `fileName`: required file name under `assets/images`.
  - `sprites` (optional): each sprite has `id` (unique per image), `x`, `y`,
    `width`, `height`.
    - `id` values may repeat across different images but not within the same
      image.
- `sfx`: list of sound effects with `id` and `fileName` (`.ogg`).
- `musics`: list of musics with `id` and `fileName` (`.ogg`).
- `fonts`: list of fonts with `id`, `fileName` (`.ttf`), `ttfFontSize`,
  optional `baseSet`, `outline`, and `localization`.

## Package obfuscation
- `obfuscatePackageContent` (bool, default `true`):
  - If `true` (default), manifest, scripts, and images are obfuscated in the
    package to avoid trivial inspection.
  - If `false`, the package stores files in clear text and is expected to be
    read as such.

## Integration reminders for modders
- The manifest must live under `GAME_ROOT/Mods`; absolute paths are forbidden.
- `main.lua` must live next to the manifest.
- `SaveState.txt` next to the manifest is loaded automatically if present.
- `milestones.txt` is read separately to drive access control; do not put
  modder state in that file.

---

## Quick Navigation

**Beginner? Start here:**
- **[Getting Started]({% link getting-started.md %})** - Create your first mod
- **[Variations & Modes]({% link variations-and-modes.md %})** - Multiple game types

**Learn by doing:**
- **[Complete Example]({% link example-progression-mode.md %})** - Full working mod

**Specific topics:**
- **[Menus & Configuration]({% link menus-configuration.md %})** - Player options
- **[Progression System]({% link progression-milestones.md %})** - Milestone gates
- **[Leaderboards]({% link leaderboards.md %})** - Score tracking
- **[Best Practices]({% link best-practices.md %})** - Design guidelines

**Back to:** [Home]({% link index.md %})
