---
layout: page
title: Moonshine Roles Ecosystem
---

Moonshine authoring is not only a local Lua workflow. Published content moves through an online ecosystem made of the Moonshine client, the Avalon backend, and Discord communities.

This page explains who does what, where permissions are decided, and how players, authors, and Discord administrators take part in registration, testing, and publication workflows.

## How the ecosystem fits together

| Part | Purpose |
|------|---------|
| **Moonshine** | The game client. Players use it to play and browse available content. Authors use it to create, test, debug, and upload ROMs. |
| **Avalon** | The trusted backend and source of truth. It owns player identities, community roles, permissions, ROM identity and versioning, ROM file storage, session records, audits, replays, crash reports, and ranking tables. |
| **Discord** | The community and governance layer. A Discord server acts as a Moonshine community where players gather, administrators review trust decisions, and authors coordinate testing and publication. |

Moonshine is where games are played and authored. Avalon is where authoritative state and permissions live. Discord is where communities organize trust, access, and visibility.

Using Discord as the entry point keeps early community workflows simple. One Moonshine player is linked to one Discord user, and each Discord server can act as a separate Moonshine community with its own trusted authors, testers, and content visibility decisions.

## Roles at a glance

| Role | What they do |
|------|--------------|
| **Player** | Registers through Discord, completes account setup in Moonshine, logs in, plays authorized content, and submits session results. |
| **Author** | Starts as a player, gets approved for a specific Discord server, creates Draft ROMs, and publishes Preview versions for community testing. |
| **Discord administrator** | Reviews author access requests, decides who is trusted to create content for the server, and helps organize community testing and publication decisions. |

Author access is scoped to a Discord server. Being approved as an author in one community does not automatically make someone an author everywhere.

## Player path

1. Join a Discord server that uses Moonshine. For now, this usually happens through an invite or an external announcement.
2. Start the registration flow from Discord with `/public moonshine`, and choose a player tag.
3. Avalon reserves the tag and provides a registration PIN.
4. Go to the Moonshine registration screen to complete the registration flow.
5. Log in with your player tag and password.
6. Browse and play the content available to you.
7. If you later join another Discord server that uses the Moonshine bot, run `/player link-server` in that server to link it to the same Moonshine account and access that community's content.

The PIN connects the Discord registration step to the in-game account setup. Keep it private and complete registration before the reservation expires.

Moonshine is an online game, so your account can be used from any computer or arcade cabinet where Moonshine is installed. That also means anyone who knows your player tag and password can sign in as you. Choose a strong password and keep it private.

## Author path

1. Register as a Moonshine player.
2. Join the Discord server where you want to create and publish content. This is often the community where you first registered as a player, but it can be another Moonshine community linked to your account.
3. Request author access for that server with `/author request`.
4. Once author access is granted, the Moonshine maker tools let you create, test, debug, and upload a ROM file for your Draft ROM.
5. Publish a Preview with `/author publish-preview` when you are ready for community testing.
6. Use `/author rom-crashes` and `/author rom-crash` to inspect confirmed Lua crash groups for your Preview ROMs.
7. Iterate with feedback before moving toward a wider release.

An author is still a player first. The author role adds publishing permissions for a specific community.

## Discord administrator path

1. Add the Moonshine bot to your Discord server.
2. Review and grant author access requests from players in your server with `/admin author-requests` when they fit your community's rules or expectations.
3. Organize preview testing channels, tester groups, or feedback processes. This workflow is still evolving.
4. Decide when content is mature enough for broader community visibility.

Administrators provide the human governance layer. Avalon can enforce permissions, but the community decides who is trusted.

## Where to go next

- New to Moonshine ROM authoring? Return to the [main introduction]({{ site.baseurl }}{% link index.md %}).
- Creating your first ROM? Follow [Getting Started]({{ site.baseurl }}{% link getting-started.md %}).
- Want to understand how a ROM moves from local development to publication? read [ROM Lifecycle]({{ site.baseurl }}{% link rom-lifecycle.md %})
- Want to understand what happens when a ROM starts, runs, submits results, and ends? Read [Runtime Session Lifecycle]({{ site.baseurl }}{% link session-lifecycle.md %}).
- Preparing a valid ROM manifest? Read the [Manifest Reference]({{ site.baseurl }}{% link manifest.md %}).
- Designing unlocks, menus, or leaderboards? See [Variants & Modes]({{ site.baseurl }}{% link variants-and-modes.md %}), [Menus & Configuration]({{ site.baseurl }}{% link menus-configuration.md %}), [Progression System]({{ site.baseurl }}{% link progression-milestones.md %}), and [Leaderboards]({{ site.baseurl }}{% link leaderboards.md %}).
- Polishing a ROM before sharing it? Check [Best Practices & Edge Cases]({{ site.baseurl }}{% link best-practices.md %}).

## Current limitations and planned work

The ecosystem is still growing. These notes highlight the parts that are planned, incomplete, or expected to evolve.

| Area | Current status |
|------|----------------|
| Discord community discovery | Moonshine does not currently list the Discord servers where the Moonshine bot is available. Players need an invite or an external announcement to find a community. In-game community discovery is planned. |
| Player registration | The current registration path starts from Discord with `/public moonshine`, then finishes in Moonshine with a registration PIN. A non-Discord registration path is planned, but not available yet. |
| Crash review | Avalon groups confirmed Lua crashes for Preview ROMs, and authors can inspect them with `/author rom-crashes` and `/author rom-crash`. Notifications, triage workflows, and community-facing crash review may evolve. |
| Preview testing | Authors can publish Preview versions for community testing, but tester groups, feedback channels, and related Discord workflows are still being defined. |
| Guild Live and Public Live | Avalon and Moonshine already model server-scoped and global catalog visibility. The full promotion workflow from Preview to Guild Live or Public Live is still being defined. |
