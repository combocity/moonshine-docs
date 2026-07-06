---
layout: page
title: Session Lifecycle
---

The ROM lifecycle explains how content is created, uploaded, and published. The session lifecycle is narrower. It describes what Moonshine prepares when a player presses play, what Lua receives, how a run produces a result, and what Avalon does with that result afterward.

For the publication flow, see [ROM Lifecycle]({% link rom-lifecycle.md %}).

## What A Session Represents

A session is the authoritative record of one playable attempt. consistency boundary that lets Moonshine and Avalon connect everything that makes a result meaningful : 
- the player
- the selected ROM version, variant, and start up options
- the ROM save state and player's progression
- the recorded inputs
- the updated save state
- newly unlocked milestones and badges
- the audit status...

In other words, a session answers a simple question:

> Starting from this exact state, with this exact ROM version and this exact input stream, did the game really produce this submitted result?

This is what makes server-backed play auditable. When a result is submitted, Avalon can replay the session headlessly and check whether the reported outcome can be reproduced. If the replay matches, the session can be treated as valid evidence for updating player progression, save state, milestones, badges, crash reports, and ranking data when supported. If it does not match, Avalon can reject that session branch and act accordingly.

For Authors, this means a session should be thought of as the atomic unit of player progress. Anything that affects the submitted result should be deterministic and tied to the session: use the session seed, the menu selection, the starting save state, the progression provided at startup, and the recorded inputs. Avoid depending on wall-clock time, external state, rendering side effects, or anything the audit runtime cannot reproduce.

## Local Maker Sessions

In local maker mode, Moonshine starts sessions without Avalon. It loads everything from the ROM folder directly, validates the manifest, loads Lua scripts and resources from disk, and creates a local session ticket. Local sessions are meant for fast iteration. Moonshine persists the selected menu inputs, save state, milestones, and badges locally. If Lua crashes, local maker mode can show a more direct crash message with phase, file, line, and reason.

There is no server audit in local maker mode. If a run unlocks a milestone locally, that helps your local test loop; it does not prove that the same result would be accepted by Avalon in server-backed play.

## Server-Backed Sessions

In server-backed play, Moonshine starts from Avalon catalog data. The catalog tells Moonshine which ROM id, version id.. etc, are available to the player.

If the ROM (version) file  is not already cached locally, Moonshine downloads it from Avalon and checks the ROM file hash for consistency. otherwise the session cannot start. This as the effect to protect your 

Once the ROM is loaded, Moonshine will resolves from the player's progression the menu prompt if available. Once the player confirms it's choices, Moonshine sends Avalon a session creation request.

Avalon creates the session as authoritative state and send back to Moonshine all necessary data to stay in sync.

## Lua Startup and Ending a run

After Moonshine has a session ticket, it builds the Lua runtime and the gameplay start..

i'm not going into details about the Lua Frame loop, `init()`, `update()` and `draw()` functions. Go to Getting started/lua-api-v1 (?) for more informations blablabla..

To end a Run, there are two lifecycle calls:

```lua
api.session.end_game()
api.session.shutdown()
```

`api.session.end_game()` tells Moonshine that the run result should be captured and submitted. It is idempotent: calling it more than once does not create multiple reports.

Think of `end_game()` as the point where the result is frozen. Save state, milestones, badges, duration, inputs, outcome, crash information, and future ranking results are captured for submission around that transition. Changes made later should not be treated as part of the submitted run.

`api.session.shutdown()` tells Moonshine the ROM is finished and can return control to the surrounding UI. If a server-backed session is open, Moonshine still builds and submits a result when the session finishes.

In practice, do not rely on long post-result gameplay. Once `end_game()` has requested a report, Moonshine can leave the session as soon as result submission has been processed. Use the post-result window for short acknowledgement screens, not for additional gameplay state.

## Session Report

When a run ends, Moonshine submits a session report to Avalon.

The report contains the evidence needed to audit the run: the recorded input stream, the duration, the resulting save state, newly unlocked progression, the final outcome, and structured crash information when the run crashed. Score data may also be included when supported by the runtime.

Avalon accepts a report only for an open session owned by the player. A session can be reported once. After the report is accepted, the session moves to pending audit.

## Crashes

crash blabla comme dans ton exemple ou fusionné plus haut ????


## Audit And Determinism

on a peut être pas besoin d'entrer dans les details technique.. on a déjà expliqué qu'il y a un audit. les details d'implémentation du worker on s'en fout. par contre c'est interessant de rappeler que la ROM est chargé coté serveur : il y a une double securité avec la vérification du hash. a minima coté local et coté serveur on s'assure que la ROM est intègre. En plus le timestamp du debut de session et fin permet de s'assurer que le joueurs de fait pas durer une session de 10min en 3jours pour faire un TAS.

The audit worker is Avalon checking whether the submitted report can be reproduced.

For each pending session, Avalon loads the stored cartridge, rebuilds the same Lua configuration, provides the same player id, variant id, menu selection, starting save state, starting milestones, starting badges, and RNG seed, then replays the recorded inputs in a headless runtime. The replayed result must match the submitted result: duration, save state, progress, badges, outcome, crash info, and scores.

If it matches, the session is validated. If the replayed result does not match, Avalon rejects that session branch, invalidates later pending/open sessions in that branch, and rebuilds player progress from validated session deltas.

This is why deterministic ROM creation matters. Do not base gameplay results on wall-clock time, filesystem state, network calls, operating system APIs, random reseeding, rendering measurements, or anything else that the audit runtime cannot reproduce. Use the session seed through `math.random`, use `api.input`, use `api.session.selection`, and keep result-changing logic in `update()`.

## Related

- **[ROM Lifecycle]({% link rom-lifecycle.md %})** - How a ROM moves from local work to Draft and Preview.
- **[Lua API v1 Reference]({% link lua-api-v1.md %})** - Runtime API details.
- **[Menus & Configuration]({% link menus-configuration.md %})** - Feed `api.session.selection`.
- **[Progression System]({% link progression-milestones.md %})** - Unlock milestones and badges.
- **[Leaderboards]({% link leaderboards.md %})** - Ranking table status and limitations.
