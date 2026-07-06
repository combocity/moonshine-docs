---
layout: page
title: Session Lifecycle
---

The ROM lifecycle explains how content is created, uploaded, and published. The session lifecycle is narrower: it describes what happens during one playable attempt, from the moment a player presses play to the moment Avalon can validate the submitted result.

For the publication flow, see [ROM Lifecycle]({% link rom-lifecycle.md %}).

## What A Session Represents

A session is the auditable boundary around one playable attempt.

It is not just the Lua runtime running on the player's machine, and it is not just a save file. A session is the unit Avalon can later verify. It binds a starting context, a stream of player inputs, and the resulting state into one coherent record.

In other words, a session answers one question:

> Starting from this exact state, with this exact ROM version and this exact input stream, did the game really produce this submitted result?

This is what makes server-backed play trustworthy. If the replayed result matches the submitted result, the session can be accepted as evidence for player progression, save state updates, unlocked milestones, badges, crash reports, and ranking data when supported. If it does not match, Avalon can reject that session branch and rebuild player state from validated sessions.

For Authors, the practical rule is simple: anything that affects a submitted result must be deterministic and reproducible from the session context.

## Local Maker Sessions

In local maker mode, Moonshine starts sessions without Avalon. It loads the ROM folder directly, validates the manifest, loads Lua scripts and resources from disk, and creates a local session ticket.

Local sessions are meant for fast iteration. Moonshine can persist selected menu inputs, save state, milestones, and badges locally so Authors can test progression gates without a server round-trip.

There is no server audit in local maker mode. If a run unlocks a milestone locally, that helps your local test loop; it does not prove that the same result would be accepted by Avalon in server-backed play.

## Server-Backed Sessions

In server-backed play, Moonshine starts from Avalon catalog data. The catalog tells Moonshine which ROM version is available to the player, which variants can be played, and which progression state is currently known by the server.

Before a session can start, Moonshine makes sure the ROM file is available locally. If it is missing, Moonshine downloads it from Avalon. If it is already cached, Moonshine verifies its hash. A missing ROM file or hash mismatch prevents the session from starting.

This protects the local side of the run: Moonshine only starts the session from the ROM version Avalon expects. Avalon also keeps the server-side cartridge used later for audit, so validation does not depend on whatever happens to exist on the player's machine after the run.

Moonshine then loads the ROM file, resolves the selected variant, restores menu choices when possible, and asks the player to confirm any startup options. Once the selection is fixed, Moonshine sends Avalon a session creation request.

Avalon creates the session as authoritative state and returns a session ticket. From that point on, Moonshine and Avalon share the same starting point: the same player, ROM version, selected variant, menu selection, RNG seed, save state, milestones, and badges.

## Runtime Contract

Once Moonshine has a session ticket, it starts the Lua runtime from that fixed context.

The exact Lua lifecycle is documented in the Lua API reference: `init()` is optional, `update()` and `draw()` are required, `api.session` exposes the launch context, `api.save` exposes persistent ROM state, and `api.progress` exposes milestones and badges.

For the session lifecycle, the important part is that Moonshine records the player's inputs every frame. Those inputs become part of the session report and are later replayed by Avalon during audit.

Keep gameplay state changes in `update()`. Use `draw()` for rendering. Avoid changing save state, progression, score, or outcome from rendering code, because server audit replays the run headlessly.

## Ending A Run

There are two lifecycle calls:

```lua
api.session.end_game()
api.session.shutdown()
```

`api.session.end_game()` freezes the result that should be submitted. Save state, newly unlocked milestones and badges, recorded inputs, outcome, crash information, and ranking data when supported are captured around that transition. Calling it more than once does not create multiple reports.

`api.session.shutdown()` tells Moonshine the ROM is finished and can return control to the surrounding UI.

The safest pattern is:

1. Update all final state.
2. Call `api.session.end_game()`.
3. Optionally show a short result or acknowledgement screen.
4. Call `api.session.shutdown()`.

Do not rely on long post-result gameplay. Once the result has been frozen, later changes should not be treated as part of the submitted run.

## Session Report

When a run ends, Moonshine submits a session report containing all the evidences needed to audit the run.

If Lua crashed during a server-backed session, the submitted crash is still treated as a session result. Avalon only turns that crash into trusted Author-facing information after audit reproduces the same crashed result.

## Audit And Determinism

Audit is Avalon checking whether the submitted session can be reproduced.

Avalon does not trust the final save state, unlocked milestones, badges, crash information, or ranking data just because the player submitted them. It replays the session from the same starting context and applies the recorded inputs in a headless runtime. The replayed result must match the submitted report.

Avalon also knows when the session was created and when the report was received. That wall-clock window can be used to reject or flag sessions whose real elapsed time is inconsistent with their frame duration. This complements replay audit: the ROM result must be reproducible, and the session must still make sense as one real playable attempt.

If the replay matches, the session is validated. Validated sessions can update player progression and add submitted score entries to their associated ranking tables when score submission is supported by the runtime. If the replay does not match, Avalon rejects the submitted result and handles the session according to server policy.

This is why deterministic ROM creation matters. The Lua runtime is sandboxed: direct filesystem and operating system access, native modules, and blocked modules such as `os`, `io`, and `debug` are removed. Stay inside that sandbox instead of trying to work around it.

For gameplay results, rely on the session context: the seeded `math.random`, `api.input`, `api.session.selection`, save state, and progression APIs. Keep result-changing logic in `update()`, and use rendering APIs, including text measurements, for presentation only because audit runs headlessly.

## Related

- **[ROM Lifecycle]({% link rom-lifecycle.md %})** - How a ROM moves from local work to Draft and Preview.
- **[Lua API v1 Reference]({% link lua-api-v1.md %})** - Runtime lifecycle and API details.
- **[Menus & Configuration]({% link menus-configuration.md %})** - Configure startup choices used by `api.session.selection`.
- **[Progression System]({% link progression-milestones.md %})** - Unlock milestones and badges.
- **[Leaderboards]({% link leaderboards.md %})** - Ranking table status and limitations.
