---
layout: page
title: ROM Lifecycle
---

The ROM lifecycle describes how content moves from local development to community testing and, eventually, broader availability.

This page focuses on the ROM authoring and publication flow. For what happens during one playable run, see [Runtime Session Lifecycle]({{ site.baseurl }}{% link session-lifecycle.md %}).

## Overview

A typical flow looks like this:

1. Create and edit the ROM locally.
2. Test it in Moonshine maker tools.
3. Upload it as a Draft ROM.
4. Publish a Preview for community testing.
5. Inspect crashes and collect feedback.
6. Iterate on the ROM.
7. Move toward broader visibility when the ROM is ready.

Some parts of this flow are available today, while others are still evolving as Moonshine, Avalon, and Discord workflows mature.

## Local Development

A ROM starts locally. The author prepares a manifest, writes Lua scripts, adds optional resources, and tests the ROM through Moonshine maker tools.

During local development, the goal is to iterate quickly. The author can test gameplay, menus, progression rules, save behavior, and runtime errors without going through the full community publication flow.

In this mode, Moonshine can create local sessions for testing. These are useful for development, but they are not the same as server-backed sessions created through Avalon.

## Draft ROM

Once the ROM is ready to leave local development, the author can upload it as a **Draft ROM**.

A Draft ROM is still author-facing. It represents content that exists in Avalon but is not yet broadly visible to players.

Drafts are useful for keeping ROM identity, versioning, and uploaded files attached to the authoring workflow before the ROM is shared with a community.

## Preview Publication

When the author is ready for community testing, they can publish a **Preview** version.

A Preview is meant for testing with a trusted group before wider release. This fits the Discord-based community model: authors can share work with a server, gather feedback, inspect issues, and decide whether the ROM is mature enough to move forward.

Preview publication is also where server-backed behavior becomes more important. Avalon can track sessions, store submitted results, group confirmed Lua crashes, and provide data that helps authors debug real play sessions.

## Crash Review and Iteration

After a Preview is played, authors may discover Lua crashes or unexpected behavior.

Avalon groups confirmed Lua crashes for Preview ROMs, and authors can inspect them using Discord commands such as:

```text
/author rom-crashes
/author rom-crash
```

Crash review is part of the iteration loop. An author can fix the ROM, upload a new version, publish another Preview, and continue testing with the community.

## Guild Live and Public Live

Moonshine and Avalon are designed to support different visibility levels.

A ROM may eventually become visible within a specific Discord community, or more broadly across Moonshine. These broader publication states are usually described as **Guild Live** and **Public Live**.

| State | Meaning |
|-------|---------|
| **Preview** | A testable version shared with a trusted community group. |
| **Guild Live** | A ROM made available within a specific Discord community. |
| **Public Live** | A ROM made available more broadly across Moonshine. |

The full promotion workflow from Preview to Guild Live or Public Live is still being defined.

## ROM Lifecycle vs Session Lifecycle

The ROM lifecycle is about content publication.

The session lifecycle is about playing.

A ROM can go through many lifecycle states over time: local development, Draft, Preview, later versions, and wider publication. Each time a player starts one variant of that ROM, Moonshine creates a separate session.

For example:

```text
One ROM
→ has several versions over time
→ may be published as Preview or Live
→ can be played many times
→ each play creates one session
```

A session starts when a player launches a ROM variant and ends when the ROM submits or exits that run. Session results, save data, progression unlocks, recorded inputs, and replay-related data belong to the runtime flow.

For details, see [Runtime Session Lifecycle]({{ site.baseurl }}{% link session-lifecycle.md %}).

## Current Limitations and Planned Work

The ROM publication workflow is still evolving.

| Area | Current status |
|------|----------------|
| Draft ROMs | Authors can create and upload Draft ROMs through Moonshine maker tools. |
| Preview publication | Authors can publish Preview versions for community testing. |
| Crash review | Confirmed Lua crash groups can be inspected for Preview ROMs. |
| Guild Live | Server-scoped catalog visibility is modeled, but the full promotion workflow is still being defined. |
| Public Live | Global catalog visibility is modeled, but the full public release workflow is still being defined. |
| Testing workflows | Tester groups, feedback channels, and Discord-side review processes are still expected to evolve. |

## Related

- **[Moonshine Roles Ecosystem]({{ site.baseurl }}{% link ecosystem.md %})** - Understand Moonshine, Avalon, Discord, roles, permissions, and community governance.
- **[Runtime Session Lifecycle]({{ site.baseurl }}{% link session-lifecycle.md %})** - Understand what happens during one playable run.
- **[Getting Started]({{ site.baseurl }}{% link getting-started.md %})** - Create and test your first ROM.
- **[Manifest Reference]({{ site.baseurl }}{% link manifest.md %})** - Prepare a valid ROM manifest.
