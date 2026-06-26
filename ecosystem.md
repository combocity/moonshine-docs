---
layout: page
title: Moonshine Ecosystem
---

Moonshine is a place to play, create, and share community-made games. The ecosystem is built around players, authors, and Discord communities so new content can start privately, be tested with a trusted group, and later become available more broadly.

This page explains how Moonshine, Avalon, and Discord work together before you dive into ROM manifests, Lua scripts, cartridges, or publication workflows.

## The three parts

| Part | Purpose |
|------|---------|
| **Moonshine** | The game client. Players use it to play and browse available content. Authors use it to create, test, and upload their work. Moonshine is online by design. |
| **Avalon** | The trusted backend server. It owns player identity, author permissions, ROM identity, ROM versions, cartridge storage, session data, replays, and ranking tables. |
| **Discord** | The community and governance layer. A Discord server acts as a Moonshine community where players gather, administrators review trust decisions, and authors coordinate testing and publication. |

Moonshine is where the game happens. Avalon is where the authoritative decisions live. Discord is where identity and community boundaries start.

Using Discord as the entry point keeps registration simple: one Moonshine player is linked to one Discord user, and players do not need a separate email-based signup just to reserve a Moonshine identity. Discord servers also give communities separate spaces where they can approve their own authors and decide which content they want to make available to their members.

## Roles at a glance

| Role | What they do |
|------|--------------|
| **Player** | Registers through Discord, completes account setup in Moonshine, logs in, plays authorized content, and submits session results. |
| **Author** | Starts as a player, gets approved for a specific Discord server, creates Draft ROMs, and publishes Preview versions for community testing. |
| **Discord administrator** | Reviews author access requests, decides who is trusted to create content for the server, and helps organize community testing and publication decisions. |

Author access is scoped to a Discord server. Being approved as an author in one community does not automatically make someone an author everywhere.

## Player path

1. Join a Discord server that uses Moonshine.
   For now, Moonshine does not list the Discord servers where the Moonshine bot is available. You need to know which community to join through an invite or an external announcement. A future Moonshine feature should make those communities discoverable from inside the game.
2. Start the registration flow from Discord with `/public moonshine`, and choose a player tag.
3. Avalon reserves the tag and provides a registration PIN.
4. Go to the Moonshine registration screen to complete the registration flow.
5. Log in with your player tag and password.
6. Browse and play the content available to you.
7. If you later join another Discord server that uses the Moonshine bot, run `/player link-server` in that server to link it to the same Moonshine account and access that community's content.

The PIN connects the Discord registration step to the in-game account setup. Keep it private and complete registration before the reservation expires.

Moonshine is an online game, so your account can be used from any computer or arcade cabinet where Moonshine is installed. That also means anyone who knows your player tag and password can sign in as you. Captain Obvious says: choose a strong password and do not share it.

## Author path

1. Register as a Moonshine player.
2. Join the Discord server where you want to publish content. This is often the community where you first registered as a player, but it can be another Moonshine community linked to your account.
3. Request author access for that server with `/author request`.
4. Once author access is granted, the Moonshine maker tools let you create, test, debug, and upload a cartridge for your Draft ROM.
5. Publish a Preview with `/author publish-preview` when you are ready for community testing.
6. Use `/author rom-crashes` and `/author rom-crash` to inspect confirmed Lua crash groups for your Preview ROMs.
7. Iterate with feedback before moving toward a wider release.

An author is still a player first. The author role adds publishing permissions for a specific community.

## Discord administrator path

1. Add the Moonshine bot to your Discord server.
2. Review and grant author access requests from players in your server with `/admin author-requests` when they fit your community's rules or expectations.
3. Organize preview testing channels, tester groups, or feedback processes.
4. Decide when content is mature enough for broader community visibility.

Administrators provide the human governance layer. Avalon can enforce permissions, but the community decides who is trusted.

## Publication stages

Moonshine does not treat content as only private or public. A ROM can move through staged visibility levels as it matures.

| Stage | Meaning | Use it when |
|-------|---------|-------------|
| **Draft** | Private author work associated with a Discord server. | You are building, testing locally, or making frequent changes. |
| **Preview** | A test candidate promoted from Draft, with an explicit SemVer version. | You want people in the community to try a version before a stable release. |
| **Guild Live** | A community release intended for a Discord server. | The community accepts the version for wider server visibility. |
| **Public Live** | A global release intended for all Moonshine players. | The content is ready to be visible beyond one Discord community. |

Draft and Preview workflows are the early publishing path for authors. Guild Live and Public Live describe the broader publication model; their exact promotion rules may evolve as the ecosystem grows.

## Where to go next

- New to Moonshine ROM authoring? Start with the [Authoring Introduction]({{ site.baseurl }}{% link introduction.md %}).
- Creating your first ROM? Follow [Getting Started]({{ site.baseurl }}{% link getting-started.md %}).
- Preparing a valid ROM manifest? Read the [Manifest Reference]({{ site.baseurl }}{% link manifest.md %}).
- Designing unlocks, menus, or leaderboards? See [Variants & Modes]({{ site.baseurl }}{% link variants-and-modes.md %}), [Menus & Configuration]({{ site.baseurl }}{% link menus-configuration.md %}), [Progression System]({{ site.baseurl }}{% link progression-milestones.md %}), and [Leaderboards]({{ site.baseurl }}{% link leaderboards.md %}).
- Polishing a ROM before sharing it? Check [Best Practices & Edge Cases]({{ site.baseurl }}{% link best-practices.md %}).

## Current limitations and planned work

The ecosystem is still growing. These notes separate what exists today from the parts that are planned or only partially wired.

| Area | Current status |
|------|----------------|
| Discord community discovery | Moonshine does not currently list the Discord servers where the Moonshine bot is available. Players need an invite or an external announcement to find a community. In-game community discovery is planned. |
| Player registration | The current player registration path starts from Discord with `/public moonshine`, then finishes in Moonshine with the registration PIN. A non-Discord registration path is planned, but it is not available yet. |
| Author access | Author access requests and administrator review are available through `/author request` and `/admin author-requests`. Access remains scoped to the Discord server that approved it. |
| Crash review | Avalon groups confirmed Lua crashes for Preview ROMs, and authors can inspect them with `/author rom-crashes` and `/author rom-crash`. Automatic community notification around those crash groups is still expected to evolve. |
| Guild Live and Public Live | Avalon and Moonshine already model server-scoped and global catalog visibility. The full Discord/admin promotion workflow from Preview to Guild Live or Public Live is still being defined. |
