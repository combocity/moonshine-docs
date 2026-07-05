---
layout: page
title: ROM Lifecycle
---

A ROM does not become public content in one jump. It starts as a local folder on your machine, becomes a Draft stored by Avalon, can be published as a Preview for a Discord community, and may later move toward wider visibility.

This page follows that creation and publication flow. If you want to understand what happens during one playable run, read [Runtime Session Lifecycle]({% link session-lifecycle.md %}) instead.

## Lifecycle At A Glance

The normal path is:

1. Build and test the ROM locally from its manifest folder.
2. Upload it from Moonshine as a Draft attached to one Discord server.
3. Update that Draft as many times as needed while iterating.
4. Publish the current Draft as a Preview from Discord.
5. Let trusted players test the Preview.
6. Review confirmed Lua crash groups and feedback.
7. Upload a new Draft and publish a new Preview when you are ready.

Today, this page focuses on the current local -> Draft -> Preview authoring path. Server live and global live are already modeled by Avalon and Moonshine, but the promotion workflow beyond Preview is still being defined.

## Local Work Comes First

Before Avalon knows about a ROM, it is just a local folder loaded directly by Moonshine. This is the fast iteration space: you can run the ROM, adjust its manifest and scripts, and test behavior without creating a server session.

Local sessions are not server-backed: there is no Avalon session, audit, and community visibility. Those only matter once the ROM is uploaded as a Draft or published as a Preview.

## Uploading the First Draft

When you upload a ROM for the first time, the manifest does not yet contain a server ROM identifier. Moonshine therefore treats the upload as Draft creation.

If you have several Discord server linked to your author account, and since a ROM can only be linked to a single server, Moonshine will ask you which one you want.

For now, only Solo ROMs are accepted (single player), ROM files are limited to 20 MB, and Avalon rejects another Draft with the same Author, ROM name, and mode type.

If the upload and validation succeeds, Avalon creates the ROM as Draft, stores the ROM file, and returns the new ROM id. Moonshine writes that id back into your manifest as the `romId` JSON field.

## Updating a Draft

After the first upload, the manifest contains the server ROM identifier, so Moonshine no longer creates a new Draft. It updates the existing one.

The update flow is incremental. Moonshine rebuilds the ROM content index and uploads only the changes to Avalon, if there are any.

Avalon applies the patch to the current Draft cartridge, validates the rebuilt cartridge, and replaces the Draft version if the resulting file index changed. This means a Draft can have several internal Draft version ids over time even though the ROM id stays the same.

The ROM id is the stable identity. The Draft name can change on update, but the mode type must still match the existing ROM.

## Publishing a Preview

Uploading a Draft does not make it visible to players. A Draft is Author-side content stored by Avalon.

When you are ready for community testing, publish the current Draft as a Preview from Discord:

```text
/game-maker publish-preview
```

The command runs inside a Discord server, and Avalon only lists Drafts that belong to that server and to your Moonshine player account. If there is one Draft, Discord shows the publish prompt directly. If there are several, Discord asks you to select one.

Preview publication requires a valid SemVer. If the Draft manifest already contains a valid version, Discord can propose it. You can also provide another SemVer from the Discord prompt. If a Preview already exists, the new SemVer must be strictly greater than the current Preview version.

When the Preview is published, Avalon reads the current Draft cartridge, writes the selected Preview version and server-side author into the manifest stored inside the Preview cartridge, signs the cartridge, optionally obfuscates content, and creates a new Preview version. Obfuscation is enabled by default in the Discord prompt.

Publishing a new Preview replaces the previous active Preview for that ROM. Treat Preview data as test data tied to a specific Preview version. When a Preview is replaced, old Preview sessions and ranking data are not something an Author should rely on as lasting public history.

## What Players See

Players do not see Drafts. They see catalog entries that Avalon says they can access.

A global live ROM can appear to any registered player. A server live ROM can appear to players who are active in that Discord server. A Preview can appear to players who have access to the server that owns the ROM. Moonshine downloads the cartridge for a catalog entry only when needed, verifies the expected cartridge hash, caches it locally, and then starts a server-backed session.

The catalog can contain more than one entry for the same ROM when distinct visible versions exist. Moonshine labels catalog entries with a scope prefix in the version text: global, server, or preview. If two visible entries would have the same SemVer, Avalon avoids adding a duplicate catalog entry for that same version.

This is why SemVer matters. It is not only display text. It helps Authors, testers, Moonshine, and Avalon talk about which cartridge is being played.

## Crash Review Is Based on Confirmed Crashes

Preview testing is useful because server-backed sessions produce server-side evidence. When a Lua session crashes, Moonshine can submit the crash outcome and structured crash information to Avalon. Avalon does not immediately turn every submitted crash into an Author-facing crash group. It audits the session by replaying the submitted inputs against the stored cartridge.

If the replay reproduces the submitted crashed result, Avalon marks the session validated and groups the crash by ROM version, phase, file, line, and normalized reason. Authors can inspect confirmed crash groups from Discord:

```text
/game-maker rom-crashes
/game-maker rom-crash
```

The list command shows recent confirmed groups for your Preview ROMs on that server. The detail command shows one group, its count, phase, location, reason signature, and recent linked sessions.

This distinction is important. A crash group is not merely "a player reported a crash"; it is a crash Avalon could confirm through the audit pipeline. If a session cannot be audited yet because of a technical failure, or if the replay does not match the submitted result, it will not become the same kind of confirmed Author signal.

## Draft, Preview, And Live States

A ROM identity can have several kinds of versions over time:

- **Draft** is the Author's current editable upload.
- **Preview** is the current testable community version.
- **Server live** is modeled for server-scoped release.
- **Global live** is modeled for broader public release.

The current Author workflow is strongest around Draft and Preview. Server live and global live are present in Avalon and Moonshine's catalog model, cartridge access checks, and display scopes, but the final human workflow for promoting content into those states is still expected to evolve.

## Practical Author Guidance

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
