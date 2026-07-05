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

The ROM id is the stable identity. The Draft name can change on update, but the mode type must still match the existing ROM.

## Publishing a Preview

Uploading a Draft does not make it visible to players. A Draft is Author-side content stored by Avalon.

When you are ready for community testing, publish the current Draft as a Preview from Discord:

```text
/game-maker publish-preview
```

The command runs inside a Discord server, and Avalon only lists Drafts that belong to that server and to your Moonshine player account. If there is one Draft, Discord shows the publish prompt directly. If there are several, Discord asks you to select one.

Preview publication requires a valid SemVer. If the Draft manifest already contains a valid version, Discord can propose it. You can also provide another SemVer from the Discord prompt. If a Preview already exists, the new SemVer must be strictly greater than the current Preview version.

Publishing creates a new Preview from the current Draft. Obfuscation is enabled by default in the Discord prompt.

A new Preview replaces the previous one. Treat Preview sessions and rankings as temporary test data.

## What Players See

Players do not see Drafts. They see catalog entries that Avalon says they can access.

ROM visibility depends on scope. Global live ROMs can appear to any registered player, server live ROMs to players active in that Discord server, and Previews to server players allowed to test them.

Today, Preview access is effectively server-wide; it may become more selective later.

When a player starts a catalog entry, Moonshine downloads the ROM file if needed, verifies and caches it locally, and starts a server-backed session.

SemVer still matters: it helps Authors, testers, Moonshine, and Avalon talk about which ROM version is being played.

## Crash Review Is Based on Confirmed Crashes

Preview testing is useful because server-backed sessions produce server-side evidence. When a Lua session crashes, Moonshine can submit the crash outcome and structured crash information to Avalon. Avalon does not immediately turn every submitted crash into an Author-facing crash group. It audits the session by replaying the submitted inputs against the stored ROM.

If the replay reproduces the submitted crashed result, Avalon marks the session validated and groups the crash by ROM version, phase, file, line, and normalized reason. Authors can inspect confirmed crash groups from Discord:

```text
/game-maker rom-crashes
/game-maker rom-crash
```

The list command shows recent confirmed groups for your Preview ROMs on that server. The detail command shows one group, its count, phase, location, reason signature, and recent linked sessions.

This distinction is important. A crash group is not merely "a player reported a crash"; it is a crash Avalon could confirm through the audit pipeline. If a session cannot be audited yet because of a technical failure, or if the replay does not match the submitted result, it will not become the same kind of confirmed Author signal.

## Draft, Preview, And Live States

A ROM currently moves through the Author workflow as Draft, then Preview. Draft is the editable upload; Preview is the version exposed for community testing.

Server live and global live are modeled in Avalon and Moonshine, but they are not part of the current publication workflow yet.

## Related

- **[Moonshine Roles Ecosystem]({% link ecosystem.md %})** - Moonshine, Avalon, Discord, roles, permissions, and community governance.
- **[Runtime Session Lifecycle]({% link session-lifecycle.md %})** - What happens during one playable run.
- **[Getting Started]({% link getting-started.md %})** - Create and test your first ROM.
- **[Manifest Reference]({% link manifest.md %})** - Prepare a valid ROM manifest.
