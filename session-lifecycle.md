---
layout: page
title: Runtime Session Lifecycle
---

A session is one playable run of one ROM variant.

A ROM can define many variants, menus, save data, progression rules, badges, ranking tables, and resources. A session is the specific run Moonshine starts when a player chooses one of those variants.

This page focuses on the Lua runtime flow: what Moonshine prepares before Lua starts, how `init()`, `update()`, and `draw()` are called, and how a ROM submits and shuts down a completed run.

For the broader authoring and publication flow, see the [Moonshine ROM LifeCycle]({{ site.baseurl }}{% link rom-lifecycle.md %}).

## Before Lua Starts

When the player starts a ROM variant, Moonshine prepares:

- the selected variant
- the menu selections
- the player's current progression
- the saved state for that ROM

In local maker mode, Moonshine creates the session locally so you can test quickly.

In server-backed play, Moonshine asks Avalon to create the session and later submits the result back to Avalon.

Moonshine then prepares the Lua runtime and exposes the session context through `api.session`:

| Field | Description |
|-------|-------------|
| `api.session.player_id` | Current player id when supplied by the host. |
| `api.session.variant_id` | Variant selected from the manifest. |
| `api.session.debug` | `true` in local maker mode. |
| `api.session.selection` | Menu selections keyed by menu input id. |

Moonshine also exposes `api.save`, a persistent save table for your ROM. It is loaded before `init()` runs, so your script can read existing state immediately.

## Runtime Flow

Once the Lua runtime is ready, Moonshine drives the script in this order:

1. `api.save` is loaded.
2. `init()` runs once.
3. `update()` runs once per frame with the current input state available through `api.input`.
4. `draw()` runs so the ROM can render the current frame.

During `update()`, your ROM usually reads input, updates game state, writes to `api.save`, unlocks progression through `api.progress`, and decides when the run is complete.

## Ending a Session

For a normal completed run, use both lifecycle calls:

```lua
api.session.end_game()
api.session.shutdown()
```

`api.session.end_game()` submits the run result once. The submitted result includes the current save state, newly unlocked milestones and badges, session duration, and recorded inputs.

The ROM keeps running after `end_game()`. This lets you show a result screen, play a fanfare, or wait for the player to acknowledge the end of the run.

When the ROM is ready to stop, call `api.session.shutdown()`. This ends the ROM session and lets Moonshine continue its own flow.

Any save or progression changes made after `api.session.end_game()` are not included in the submitted result. Treat `end_game()` as the moment where the run's result is frozen.

## Practical Pattern

A simple pattern is to separate the gameplay state from the ending state:

```lua
local frame = 0
local ending = false
local ending_frame = 0

function update()
  if ending then
    ending_frame = ending_frame + 1

    if ending_frame >= 120 then
      api.session.shutdown()
    end

    return
  end

  frame = frame + 1

  if frame >= 600 then
    api.session.end_game()
    ending = true
  end
end
```

This freezes the result at frame 600, keeps the ROM alive for two more seconds at 60 FPS, then shuts the session down.

## Related

- **[Moonshine Roles Ecosystem]({{ site.baseurl }}{% link ecosystem.md %})** - Understand Moonshine, Avalon, Discord, roles, permissions, and publication workflows.
- **[Getting Started]({{ site.baseurl }}{% link getting-started.md %})** - Create a first ROM.
- **[Lua API v1 Reference]({{ site.baseurl }}{% link lua-api-v1.md %})** - Runtime API details.
- **[Menus & Configuration]({{ site.baseurl }}{% link menus-configuration.md %})** - Feed `api.session.selection`.
- **[Progression System]({{ site.baseurl }}{% link progression-milestones.md %})** - Unlock milestones and badges.
