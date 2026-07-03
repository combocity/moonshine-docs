---
layout: page
title: ROM Lifecycle
---

A ROM does not become public content in one jump. It starts as a local folder on your machine, becomes a Draft stored by Avalon, can be published as a Preview for a Discord community, and may later move toward wider visibility.

This page follows that creation and publication flow. If you want to understand what happens during one playable run, read [Runtime Session Lifecycle]({% link session-lifecycle.md %}) instead.

## The Big Picture

Moonshine is the client for Game Makers and players. Avalon is the server that owns identity, permissions, ROM records, versions, cartridge storage, session records, crash groups, audit status, and progression. Discord is where human trust decisions happen.

The normal path is:

1. Build and test the ROM locally from its manifest folder.
2. Upload it from Moonshine as a Draft attached to one Discord server.
3. Update that Draft as many times as needed while iterating.
4. Publish the current Draft as a Preview from Discord.
5. Let trusted players test the Preview.
6. Review confirmed Lua crash groups and feedback.
7. Upload a new Draft and publish a new Preview when you are ready.

Guild/server live and global/public live states are already modeled by Avalon and Moonshine, but the full promotion workflow beyond Preview is still being defined. Today, Preview is the meaningful community testing state for Game Makers.

## Local Work Comes First

Your ROM begins as a manifest folder: a `manifest.json`, Lua files, and optional resources such as images, sounds, music, and fonts. Local maker mode loads that folder directly. It validates the manifest, discovers Lua scripts, resolves resources from disk, enables the local debugger when available, and starts local sessions without asking Avalon to create a server session.

This local loop is where you should solve the obvious problems: missing files, invalid manifest data, broken menu definitions, missing `update()` or `draw()`, Lua crashes, bad progression ids, and save state mistakes. Moonshine can show Lua crash details directly in local maker mode, including the phase, file, line, and reason when those details can be parsed.

Local testing is not the same as server-backed play. It is intentionally faster and more permissive for iteration. The save state, milestones, badges, and menu choices are stored locally for the manifest folder. There is no server audit and no community visibility.

## Uploading the First Draft

When you upload a ROM for the first time, the manifest does not yet contain a server ROM identifier (`modId` in manifest JSON). Moonshine therefore treats the upload as Draft creation.

Moonshine asks which Discord server the Draft belongs to, because Game Maker access is scoped to a server. Avalon checks that your Moonshine player account is allowed to create content for that server. If you were approved as a Game Maker in one Discord community, that does not automatically grant access in another.

Moonshine then packs the manifest folder into the current cartridge format, sends the cartridge to Avalon, and includes basic manifest metadata: API version, ROM name, optional version, and mode type. Avalon validates the uploaded cartridge before it creates anything authoritative.

Several details matter here:

- Only Solo ROMs are currently accepted by the Game Maker upload path.
- The cartridge limit is currently 20 MB.
- Avalon rejects unsupported manifest API versions.
- The declared upload metadata must match the manifest inside the cartridge.
- The cartridge must contain a valid index, manifest, declared scripts, and declared resources.
- Draft cartridge contents must not already be obfuscated. Obfuscation is chosen later when publishing Preview.
- A Game Maker can currently have only two Draft ROMs at the same time.
- Avalon rejects another Draft with the same Game Maker, ROM name, and mode type.

If the upload succeeds, Avalon creates a ROM identity and a Draft version, stores the cartridge, and returns the new ROM id. Moonshine writes that id back into your manifest as the `modId` JSON field.

Do not treat that field as cosmetic. It is the current JSON field name for the server-side ROM id. If you remove it or copy it into another unrelated project, Moonshine may create a new Draft or try to update the wrong one.

## Updating a Draft

After the first upload, the manifest contains the server ROM identifier, so Moonshine no longer creates a new Draft. It updates the existing one.

The update flow is incremental. Moonshine packs the current folder, builds an index of the declared cartridge files, and asks Avalon what changed compared with the current server Draft. If nothing changed, Avalon can report that the Draft is already up to date. If files changed, Moonshine uploads a Draft patch containing only changed or added files plus the list of deleted files.

Avalon applies the patch to the current Draft cartridge, validates the rebuilt cartridge, and replaces the Draft version if the resulting file index changed. This means a Draft can have several internal Draft version ids over time even though the ROM id stays the same.

Avalon protects this flow with a base Draft version check. If the server Draft changed between the diff request and the patch upload, the patch is rejected with a Draft version mismatch. In practice, refresh and upload again.

Changing the manifest name or mode type after a Draft exists is not treated as a rename. Avalon compares the uploaded metadata with the existing ROM identity and rejects mismatches. The safe path is to choose the public name and mode type before the first upload.

## Publishing a Preview

Uploading a Draft does not make it visible to players. A Draft is Game Maker-side content stored by Avalon.

When you are ready for community testing, publish the current Draft as a Preview from Discord:

```text
/game-maker publish-preview
```

The command runs inside a Discord server, and Avalon only lists Drafts that belong to that server and to your Moonshine player account. If there is one Draft, Discord shows the publish prompt directly. If there are several, Discord asks you to select one.

Preview publication requires a valid SemVer. If the Draft manifest already contains a valid version, Discord can propose it. You can also provide another SemVer from the Discord prompt. If a Preview already exists, the new SemVer must be strictly greater than the current Preview version.

When the Preview is published, Avalon reads the current Draft cartridge, writes the selected Preview version into the manifest stored inside the Preview cartridge, signs the cartridge, optionally obfuscates content, and creates a new Preview version. Obfuscation is enabled by default in the Discord prompt.

Publishing a new Preview replaces the previous active Preview for that ROM. Treat Preview data as test data tied to a specific Preview version. When a Preview is replaced, old Preview sessions and ranking data are not something a Game Maker should rely on as lasting public history.

## What Players See

Players do not see Drafts. They see catalog entries that Avalon says they can access.

A global live ROM can appear to any registered player. A server live ROM can appear to players who are active in that Discord server. A Preview can appear to players who have access to the server that owns the ROM. Moonshine downloads the cartridge for a catalog entry only when needed, verifies the expected cartridge hash, caches it locally, and then starts a server-backed session.

The catalog can contain more than one entry for the same ROM when distinct visible versions exist. Moonshine labels catalog entries with a scope prefix in the version text: global, server, or preview. If two visible entries would have the same SemVer, Avalon avoids adding a duplicate catalog entry for that same version.

This is why SemVer matters. It is not only display text. It helps Game Makers, testers, Moonshine, and Avalon talk about which cartridge is being played.

## Crash Review Is Based on Confirmed Crashes

Preview testing is useful because server-backed sessions produce server-side evidence. When a Lua session crashes, Moonshine can submit the crash outcome and structured crash information to Avalon. Avalon does not immediately turn every submitted crash into a Game Maker-facing crash group. It audits the session by replaying the submitted inputs against the stored cartridge.

If the replay reproduces the submitted crashed result, Avalon marks the session validated and groups the crash by ROM version, phase, file, line, and normalized reason. Game Makers can inspect confirmed crash groups from Discord:

```text
/game-maker rom-crashes
/game-maker rom-crash
```

The list command shows recent confirmed groups for your Preview ROMs on that server. The detail command shows one group, its count, phase, location, reason signature, and recent linked sessions.

This distinction is important. A crash group is not merely "a player reported a crash"; it is a crash Avalon could confirm through the audit pipeline. If a session cannot be audited yet because of a technical failure, or if the replay does not match the submitted result, it will not become the same kind of confirmed Game Maker signal.

## Draft, Preview, And Live States

A ROM identity can have several kinds of versions over time:

- **Draft** is the Game Maker's current editable upload.
- **Preview** is the current testable community version.
- **Server live** is modeled for server-scoped release.
- **Global live** is modeled for broader public release.

The current Game Maker workflow is strongest around Draft and Preview. Server live and global live are present in Avalon and Moonshine's catalog model, cartridge access checks, and display scopes, but the final human workflow for promoting content into those states is still expected to evolve.

## Practical Game Maker Guidance

Keep local iteration fast. Use local maker mode to test manifest validity, resource loading, menu behavior, save state, progression, and Lua crashes before involving a server.

Upload Drafts when you want Avalon to remember the ROM identity and cartridge. Publish Preview only when you want other people in the Discord community to test that exact cartridge.

Do not rely on Draft version ids. They can change on each upload. The stable identity is the server ROM identifier in your manifest, and the testable public identity is the Preview SemVer you publish.

Use Preview SemVer deliberately. A new Preview replaces the previous active Preview, and the next Preview must be strictly greater than the current one.

Watch confirmed crash groups after testers play. They are often more useful than screenshots because they point to the phase and location that Avalon could reproduce.

## Related

- **[Moonshine Roles Ecosystem]({% link ecosystem.md %})** - Moonshine, Avalon, Discord, roles, permissions, and community governance.
- **[Runtime Session Lifecycle]({% link session-lifecycle.md %})** - What happens during one playable run.
- **[Getting Started]({% link getting-started.md %})** - Create and test your first ROM.
- **[Manifest Reference]({% link manifest.md %})** - Prepare a valid ROM manifest.
