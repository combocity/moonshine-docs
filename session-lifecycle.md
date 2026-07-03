---
layout: page
title: Runtime Session Lifecycle
---

A session is one playable run of one ROM version, one variant, and one set of menu selections.

The ROM lifecycle explains how content is created, uploaded, and published. The session lifecycle is narrower. It describes what Moonshine prepares when a player presses play, what Lua receives, how a run produces a result, and what Avalon does with that result afterward.

For the publication flow, see [ROM Lifecycle]({% link rom-lifecycle.md %}).

## What A Session Represents

When a player chooses a ROM entry from the Moonshine catalog, they are not just choosing a game name. They are choosing a specific ROM version and cartridge. When they then choose a variant and menu options, Moonshine has enough information to start one run.

That run has its own session id, player id, random seed, starting save state, starting milestones, starting badges, menu selection, recorded inputs, result, and audit status.

The Lua runtime exposes the selected variant as `api.session.variant_id`.

## Local Maker Sessions

In local maker mode, Moonshine starts sessions without Avalon. It loads the manifest folder directly, validates the manifest, loads Lua scripts and resources from disk, and creates a local session ticket with a generated session id, the local player id when one is available, a random seed, the current time, and the saved state stored for that manifest folder.

Local sessions are meant for fast iteration. Moonshine persists the selected menu inputs, save state, milestones, and badges locally. If Lua crashes, local maker mode can show a more direct crash message with phase, file, line, and reason.

There is no server audit in local maker mode. If a run unlocks a milestone locally, that helps your local test loop; it does not prove that the same result would be accepted by Avalon in server-backed play.

## Server-Backed Sessions

In server-backed play, Moonshine starts from Avalon catalog data. The catalog entry tells Moonshine which ROM id, version id, cartridge hash, cartridge file name, variants, visible milestones, and earned progress are available to the player.

If the cartridge is not already cached locally, Moonshine downloads it from Avalon and checks the cartridge hash. If the cartridge is missing or the hash does not match, the session does not start.

Moonshine then loads the cartridge, resolves the selected variant, restores the last menu selection for that cartridge when possible, and shows the menu prompt when the variant has configurable inputs. Once the player confirms the menu, Moonshine sends Avalon a session creation request. The request body contains the ROM version id, variant id, and input selection.

Avalon creates the session as authoritative state. Before opening the new session, it closes any still-open session for the same player, ROM, version, and variant as unreported. Then it finds the previous pending or validated session in that same chain, assigns the next node number, loads the player's current progression for that ROM version, chooses an RNG seed, and stores the starting state.

The response sent back to Moonshine contains the session id, player id, RNG seed, creation date, starting save state, starting milestones, and starting badges. Moonshine also refreshes its local catalog snapshot with those milestones and badges so menus and visibility checks are aligned with the server.

## Lua Startup

After Moonshine has a session ticket, it builds the Lua runtime.

The runtime seeds `math.random` with the session seed, then removes `math.randomseed` so the ROM cannot reseed itself. This matters for server audit: Avalon must be able to replay the same cartridge, same seed, same selection, same starting save state, and same inputs and get the same result.

Moonshine exposes the launch context through `api.session`:

```lua
api.session.player_id
api.session.variant_id
api.session.debug
api.session.selection
```

`api.session.debug` is true in local maker mode and false for normal server-backed sessions. `api.session.selection` is a read-only table keyed by menu input id; values are strings selected before the session started.

Persistent ROM state is exposed as:

```lua
api.save
```

It is always a table. On a fresh session, the save blob is empty and Moonshine provides an empty table. On later sessions, Moonshine decodes the previous save blob before `init()` runs.

The API tables themselves are read-only, but the content of `api.save` is mutable. This is intentional:

```lua
function init()
  api.save.play_count = (api.save.play_count or 0) + 1
end
```

Keep save data small and boring. The current encoded save state limit is 16 KB. Save data should be made of simple Lua values: tables, strings, finite numbers, booleans, and nil. Cycles, metatables, functions, userdata, threads, and non-finite numbers are not valid persistent state.

## The Lua Frame Loop

`init()` is optional. If present, Moonshine calls it once after the API is ready and before the first frame.

`update()` and `draw()` are required. Moonshine calls `update()` with the current input snapshot available through `api.input`, then calls `draw()` so the ROM can render the frame. The session duration is counted in frames after `update()` runs.

Use `update()` for game state, save state, progression, ending conditions, and anything that affects the result. Use `draw()` for rendering. Avoid making `draw()` change save state, progression, score, or run outcome. Server audit replays the run headlessly, and result timing is much easier to reason about when rendering has no gameplay side effects.

Moonshine records held inputs every frame. When the result is submitted, those inputs are compressed and sent with the report. Avalon later decompresses them and replays the cartridge headlessly to verify the submitted duration, save state, milestones, badges, scores, outcome, and crash information.

## Progress During A Session

At startup, `api.progress` contains the milestones and badges that were already earned for this ROM version. During the run, the ROM can unlock more:

```lua
api.progress.unlock_milestone("beat_normal")
api.progress.unlock_badge("first_clear")
```

The progress API behaves like a set. Unlocking the same id twice is harmless. `has_milestone`, `has_badge`, `get_milestones`, and `get_badges` reflect the session's current view, including ids unlocked earlier in the same run.

Only newly earned ids are submitted as deltas. Avalon validates that milestones and badges are declared by the ROM version's manifest. It also enforces current limits: milestone ids and badge ids must be 32 characters or fewer, a session's final milestone set must contain no more than 120 ids, and a session's final badge set must contain no more than 32 ids.

Progress is applied when Avalon accepts the report, then audited afterward. If a later audit rejects a session branch, Avalon rebuilds player progress from validated sessions. This is another reason to keep session behavior deterministic.

## Ending A Run

There are two lifecycle calls:

```lua
api.session.end_game()
api.session.shutdown()
```

`api.session.end_game()` tells Moonshine that the run result should be captured and submitted. It is idempotent: calling it more than once does not create multiple reports.

Think of `end_game()` as the point where the result is frozen. Save state, milestones, badges, duration, inputs, outcome, crash information, and future ranking results are captured for submission around that transition. Changes made later should not be treated as part of the submitted run.

`api.session.shutdown()` tells Moonshine the ROM is finished and can return control to the surrounding UI. If a server-backed session is open, Moonshine still builds and submits a result when the session finishes.

The clearest pattern is to update all final state first, call `end_game()`, and then call `shutdown()` when you no longer need the ROM to remain alive:

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
    api.save.best_frame = frame
    api.progress.unlock_milestone("survived_600_frames")
    api.session.end_game()
    ending = true
  end
end
```

In practice, do not rely on long post-result gameplay. Once `end_game()` has requested a report, Moonshine can leave the session as soon as result submission has been processed or the ROM reports that it is finished. Use the post-result window for short acknowledgement screens, not for additional gameplay state.

## What Gets Submitted

Moonshine submits the session report to Avalon as MessagePack. The report includes:

- recorded inputs
- duration in frames
- save state
- newly added milestones and badges
- session outcome
- structured Lua crash information when the outcome is crashed
- ranking score fields when score submission is supported by the runtime

The server currently validates score payloads against ranking table definitions, but the public Lua API does not yet expose a complete score submission surface. Treat leaderboards and score submission as still evolving unless the dedicated leaderboard documentation says otherwise.

Avalon accepts reports only for an open session owned by the player. A session can be submitted once. After the report is accepted, the session moves to pending audit.

## Crashes

If Lua crashes during `init()`, `update()`, or `draw()`, Moonshine records structured crash information when it can: phase, file, line, reason, and raw error text. Local maker mode shows that detail directly. In server-backed play, Moonshine marks the outcome as crashed and submits the report after the player acknowledges the error, as long as a server session was already open.

Avalon requires crash information when the submitted outcome is crashed, and rejects crash information for non-crashed outcomes. Confirmed Game Maker-facing crash groups are created only after audit reproduces the crashed result.

Crash grouping uses the Preview version, phase, file, line, and normalized reason. This means two crashes in the same line can still be separate groups if their reason changes, and a crash fixed in a new Preview belongs to a different version history than the old Preview.

## Audit And Determinism

The audit worker is Avalon checking whether the submitted report can be reproduced.

For each pending session, Avalon loads the stored cartridge, rebuilds the same Lua configuration, provides the same player id, variant id, menu selection, starting save state, starting milestones, starting badges, and RNG seed, then replays the recorded inputs in a headless runtime. The replayed result must match the submitted result: duration, save state, progress, badges, outcome, crash info, and scores.

If it matches, the session is validated. If the replayed result does not match, Avalon rejects that session branch, invalidates later pending/open sessions in that branch, and rebuilds player progress from validated session deltas.

This is why deterministic authoring matters. Do not base gameplay results on wall-clock time, filesystem state, network calls, operating system APIs, random reseeding, rendering measurements, or anything else that the audit runtime cannot reproduce. Use the session seed through `math.random`, use `api.input`, use `api.session.selection`, and keep result-changing logic in `update()`.

## Related

- **[ROM Lifecycle]({% link rom-lifecycle.md %})** - How a ROM moves from local work to Draft and Preview.
- **[Lua API v1 Reference]({% link lua-api-v1.md %})** - Runtime API details.
- **[Menus & Configuration]({% link menus-configuration.md %})** - Feed `api.session.selection`.
- **[Progression System]({% link progression-milestones.md %})** - Unlock milestones and badges.
- **[Leaderboards]({% link leaderboards.md %})** - Ranking table status and limitations.
