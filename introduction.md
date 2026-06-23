---
layout: page
title: Authoring Introduction
---

This documentation is designed for content creators and game designers who want to build and publish their own games on the Moonshine platform using Lua scripting. Moonshine was originally built for puzzle games with a basic API, but its current features can already support other kinds of game ideas. The API is expected to evolve with feedback from modders.

This guide covers the entire process, from designing your game mechanics to integrating progression systems, building interactive menus, and publishing your creation to the Moonshine community.

Before publishing, it helps to understand how Moonshine, Avalon, and Discord work together. See the [Moonshine Ecosystem]({{ site.baseurl }}{% link ecosystem.md %}) page for the player, author, and Discord administrator roles.

In Moonshine, a game is atomically a **bundle** (packaged into a single `*.t3pkg` file) that contains :
- a manifest
- one or more Lua scripts
- optional resources (images, sounds, and music).

 You can create several **variations**, such as *Easy*, *Normal*, or *Hard* modes. A variation can be seen as a game mode that appears as an entry in the *Play Menu*. Each **variation** can also have its own **Custom Menu**, letting the player tweak settings before starting. For example, if the player completes your *Hard* variation with 100 levels, you could then allow them to start from a specific level.

Additionally, you could propose an *Extreme* mode that shows up only after beating the *Hard* mode. That's possible too with the use of **Milestones**. Milestones allow you to customize the visibility and accessibility of variations, menu options, and leaderboards depending on player progression.

Milestones work like achievements internally, but they are not displayed directly to players. If you want a visible achievement-like reward, you can define **Badges** instead, although when and how they are shown in the UI is still a work in progress. When your script starts, Moonshine provides the current **Milestones**, **Badges**, and **saveState**, so your game can react to the player's progression.

When your game variation ends, it can submit the player's score to a **Leaderboard** whose format is totally up to you. Moonshine's server will verify the score and update the leaderboards accordingly.

**Ready to start?** → [Getting Started]({{ site.baseurl }}{% link getting-started.md %})
