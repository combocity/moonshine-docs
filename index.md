---
layout: page
title: Moonshine Lua ROM Authoring Documentation
---

**Create, integrate, and share your own games.**

## Why Moonshine?

Like many programmers, I created a Tetris clone, specifically a *Tetris The Grandmaster* variant. Twenty years ago, clones were one of the only ways for the emerging Western player community to experience this arcade-exclusive, Japan-only game at home. Beyond recreation, I saw an opportunity to experiment with online features. Unfortunately, it didn't stay online long due to technical limitations and licensing issues.

Over the years, people in the TGM community have built amazing projects, each with its own ideas and strengths. Still, I wanted to revisit my own game, this time on a foundation designed with online and community features in mind from the start.

Moonshine is an attempt to be a platform that lets anyone create a puzzle game simply, while providing built-in community features out of the box: player progression tracking, server-backed sessions, and foundations for rankings and replays. No need to rebuild the foundation every time: focus on your game design instead.

## Authoring Overview

Moonshine was originally designed for puzzle games, and its API is still intentionally small. However, because ROMs are scripted in Lua, authors have enough flexibility to experiment with many kinds of game ideas beyond the original puzzle-game focus.

## What Is a ROM?

In Moonshine, a **ROM** is the playable content you author: basically, a game.

When it is uploaded or distributed, it is packed into a **ROM file** (`*.t3rom`) that contains:

- a manifest
- one or more Lua scripts
- optional resources, such as images, sounds, and music

Internally, this packaged artifact may be called a cartridge, but authors and players can simply think of it as the ROM file.

## Variants, Menus, and Progression

You can create several **variants** per ROM, such as *Easy*, *Normal*, or *Hard* modes. A variant can be seen as a game mode that appears as an entry in the *Play Menu*.

Each **variant** can also have its own **Custom Menu**, letting the player tweak settings before starting. For example, if the player completes your *Hard* variant with 100 levels, you could then allow them to start from a specific level.

You can also define modes that only appear after specific conditions are met. For example, an *Extreme* mode could become visible only after the player beats *Hard* mode.

This is possible with **Milestones**. Milestones allow you to customize the visibility and accessibility of variants, menu options, badges, and ranking table definitions depending on player progression.

Milestones work like achievements internally, but they are not displayed directly to players. If you want a visible achievement-like reward, you can define **Badges** instead, although when and how they are shown in the UI is still a work in progress.

When a session starts, Moonshine provides the current player progression and the variant save state so your game can react to what the player has already done.

## Leaderboards

Leaderboards are still work in progress.

The manifest can describe `rankingTables`, but Moonshine does not yet expose a public Lua API for ranking consultation.

## Getting Started

New to Moonshine ROM authoring? Start here:

1. **[Moonshine Roles Ecosystem]({{ site.baseurl }}{% link roles-ecosystem.md %})** - Get the gist of Moonshine, Avalon, and Discord roles
2. **[Getting Started]({{ site.baseurl }}{% link getting-started.md %})** - Set up your development environment and create your first ROM

## Core Concepts

Master the key features of Moonshine ROM authoring:

- **[Variants & Modes]({{ site.baseurl }}{% link variants-and-modes.md %})** - Design multiple game variants, difficulty tiers, and parallel game modes
- **[Session Lifecycle]({{ site.baseurl }}{% link session-lifecycle.md %})** - Understand how a Lua session starts, submits results, and ends
- **[Menus & Configuration]({{ site.baseurl }}{% link menus-configuration.md %})** - Create interactive menus and let players customize their experience before playing
- **[Progression System]({{ site.baseurl }}{% link progression-milestones.md %})** - Build progression paths using milestones to unlock content and gate features
- **[Leaderboards]({{ site.baseurl }}{% link leaderboards.md %})** - Work-in-progress ranking table definitions

## Reference & Examples

- **[Lua API v1 Reference]({{ site.baseurl }}{% link lua-api-v1.md %})** - Runtime lifecycle and `api.*` modules
- **[Manifest Essentials]({{ site.baseurl }}{% link manifest.md %})** - Complete technical reference for `manifest.json` configuration
- **[Best Practices & Edge Cases]({{ site.baseurl }}{% link best-practices.md %})** - TODO rewrite from concrete behavior
- **[Complete Example]({{ site.baseurl }}{% link example-progression-mode.md %})** - TODO rewrite for the current Lua API

---

**Last Updated:** June 2026
