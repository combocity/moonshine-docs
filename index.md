---
layout: page
title: Moonshine Lua ROM Creation Documentation
---

**Create, integrate, and share your own games.**

## Why Moonshine?

Like many programmers, I once created a Tetris clone, specifically a *Tetris The Grandmaster* variant. Twenty years ago, clones were one of the only ways for the emerging Western player community to experience this arcade-exclusive, Japan-only game at home. Beyond recreation, I also wanted to experiment with online features. Unfortunately, it did not stay online for long due to technical limitations and licensing issues.

Over the years, people in the TGM community have built amazing projects, each with its own ideas and strengths. Still, I wanted to revisit the online and community-driven ambitions behind my own game, this time on a foundation designed for them from the start.

Moonshine is that foundation: an online-first platform where people can play, create, compete, and share community-made games without every Game Maker having to rebuild the same surrounding systems from scratch.

New content can start privately, be tested with a trusted Discord group, and later become available more broadly. Competitive play and replay visualization are also part of the experience I want Moonshine to grow toward, so players can compare performances, study runs, share memorable sessions, and follow how a community evolves around a game.

The goal is simple: let Game Makers focus on game design while Moonshine handles the surrounding structure.

## Game Maker Overview

Moonshine was originally designed for puzzle games, and its API is still intentionally small. However, because ROMs are scripted in Lua, Game Makers have enough flexibility to experiment with many kinds of game ideas beyond the original puzzle-game focus.

### What Is a ROM?

In Moonshine, a **ROM** is the playable content you create: basically, a game.

When it is uploaded or distributed, it is packed into a **ROM file** (`*.t3rom`) that contains:

- a manifest
- one or more Lua scripts
- optional resources, such as images, sounds, and music

Internally, this ROM file is called a cartridge, but Game Makers and players can simply think of it as the ROM file.

### Core Game Maker Features

A ROM can define several **variants**, such as *Easy*, *Normal*, or *Hard*. A variant  can be seen as a game mode a game mode that appears as an entry in the *Play Menu*.

Each variant can also provide its own **Custom Menu**, letting the player tweak settings before starting. For example, after completing a *Hard* variant with 100 levels, the player could unlock an option to start from a specific level.

You can also define content that only becomes visible or available after specific conditions are met. For example, an *Extreme* variant could remain hidden until the player beats *Hard* mode.

This is handled through **Milestones**. Milestones allow you to control the visibility and accessibility of variants, menu options, badges, and ranking tables depending on player progression.

Milestones work like internal achievements, but they are not displayed directly to players. If you want a visible achievement-like reward, you can define **Badges** instead, although when and how they are shown in the UI is still a work in progress.

When a player starts a ROM variant, Moonshine provides the current player progression and the ROM save state, so your game can react to what the player has already done.

The manifest can also define one or more custom **ranking tables** for your ROM. Each ranking table can represent a different way to compare players, such as score, time, level, or any other result your game exposes. Access to these ranking tables can also be controlled through Milestones, allowing you to reveal rankings only after the player reaches specific progression goals.

ROM creation is an iterative process. Some features are still evolving, and edge cases are expected as the platform grows. The goal of this documentation is to make those behaviors explicit and help Game Makers build with confidence.

## Getting Started

New to Moonshine ROM creation? Start here:

1. **[Moonshine Ecosystem]({{ site.baseurl }}{% link ecosystem.md %})** - Get the gist of Moonshine, Avalon, and Discord roles
2. **[Getting Started]({{ site.baseurl }}{% link getting-started.md %})** - Set up your development environment and create your first ROM

## Core Concepts

Master the key features of Moonshine ROM creation:

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
