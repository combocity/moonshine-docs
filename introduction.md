---
layout: page
title: Authoring Introduction
---

This documentation is designed for content creators and game designers who want to build and publish their own games on the Moonshine platform using Lua scripting. Moonshine was originally built for puzzle games with a basic API, but its current features can already support other kinds of game ideas. The API is expected to evolve with feedback from makers.

This guide covers the entire process, from designing your game mechanics to integrating progression systems, building interactive menus, and publishing your creation to the Moonshine community.

Before publishing, it helps to understand how Moonshine, Avalon, and Discord work together. See the [Moonshine Ecosystem]({{ site.baseurl }}{% link ecosystem.md %}) page for the player, author, and Discord administrator roles.

In Moonshine, a **ROM** is the playable content you author: basically, a game. When it is uploaded or distributed, it is packed into a **ROM file** (`*.t3rom`) that contains:
- a manifest
- one or more Lua scripts
- optional resources (images, sounds, and music).

Internally, this packaged artifact may be called a cartridge, but makers and players can simply think of it as the ROM file.

You can create several **variants**, such as *Easy*, *Normal*, or *Hard* modes. A variant can be seen as a game mode that appears as an entry in the *Play Menu*. Each **variant** can also have its own **Custom Menu**, letting the player tweak settings before starting. For example, if the player completes your *Hard* variant with 100 levels, you could then allow them to start from a specific level.

Additionally, you could propose an *Extreme* mode that shows up only after beating the *Hard* mode. That's possible with **Milestones**. Milestones allow you to customize the visibility and accessibility of variants, menu options, badges, and ranking table definitions depending on player progression.

Milestones work like achievements internally, but they are not displayed directly to players. If you want a visible achievement-like reward, you can define **Badges** instead, although when and how they are shown in the UI is still a work in progress. When a session starts, Moonshine provides the current player progression and the variant save state so your game can react to what the player has already done.

Leaderboards are still work in progress. The manifest can describe `rankingTables`, but Moonshine does not yet expose a public Lua API for score submission.

**Ready to start?** → [Getting Started]({{ site.baseurl }}{% link getting-started.md %})
