---
layout: page
title: Default Save State
---

A ROM can provide an optional `default-save-state.json` file to initialize
`api.save` before a player has a persisted save state.

This is a read-only template shipped with the ROM. It is not the player's
mutable save file, and Moonshine never rewrites it.

## File Location And Name

Place the file next to the ROM manifest:

```text
my-rom/
├── manifest.json
├── default-save-state.json
└── main.lua
```

The exact, case-sensitive name is `default-save-state.json`. Names such as
`defaultSaveState.json` and `Default-Save-State.json` are not aliases and are
rejected to keep ROMs portable across filesystems.

The file applies to the whole ROM folder, so every variant uses the same
default. If a folder contains multiple manifests, they also share this file.
Put manifests in separate folders when they need different defaults.

The file is optional. If it is absent and the player has no save, `api.save`
starts as an empty table.

## File Format

The document root is the state object itself. Do not wrap it in a
`defaultSaveState` property:

```json
{
  "play_count": 0,
  "settings": {
    "speed": "normal",
    "show_ghost": true
  },
  "unlocked_skins": ["classic"]
}
```

The root must be a JSON object. `{}` is a valid empty default. A present but
empty file, malformed document, `null`, array, or scalar root is invalid and
prevents the ROM from loading or being packaged; Moonshine does not silently
fall back to an empty table. Standard UTF-8 and UTF-8 with a byte-order mark are
accepted. JSON comments and trailing commas are not.

Moonshine converts JSON values as follows:

| JSON value | Lua value |
|------------|-----------|
| Object | Table with case-sensitive string keys. |
| Array | Table with consecutive integer keys starting at `1`. |
| String | String. |
| Boolean | Boolean. |
| Integer | Integer, when representable as a signed 64-bit value. |
| Decimal or exponent number | Finite floating-point number. |

Nested JSON `null` values are not supported because Lua `nil` removes a table
entry. Duplicate object property names and invalid Unicode strings are also
rejected. Property names that differ by case remain distinct. Empty objects and
empty arrays both become empty Lua tables, and object properties are encoded in
a canonical order; do not rely on JSON property order for Lua iteration.

## Initialization And Persistence

Moonshine chooses the initial `api.save` table in this order:

1. A non-blank persisted or session-ticket save state, including an explicitly
   saved empty Lua table.
2. `default-save-state.json`, when present.
3. An empty Lua table.

The selected state is available before top-level entry-point code executes and
before `init()` runs. Moonshine never merges the default with a persisted save.
Changing the file in a later ROM release does not migrate or overwrite existing
player saves.

If the ROM leaves `api.save` in a form that cannot be encoded at the end of a
session, Moonshine keeps the initial state selected for that session instead of
persisting the invalid value.

## Safety Limits

Moonshine validates the JSON-to-Lua conversion with these limits:

| Limit | Maximum |
|-------|---------|
| Source JSON file, including a UTF-8 byte-order mark | 128 KiB |
| Nesting depth | 32 |
| UTF-8 bytes in one string value or object key | 65,535 |
| Entries in one object or array | 10,000 |
| Total values and containers | 50,000 nodes |
| Encoded save state | 16 KiB |

The 128 KiB source limit bounds the file before JSON parsing. The separate
16 KiB limit applies after conversion to Moonshine's encoded save-state format.
Keep defaults focused on compact player progress and preferences rather than
game content or replay data.

## Packaging

Moonshine includes the original `default-save-state.json` bytes in the
`.t3rom` package. Its hash is recorded in the package index, and the signed
index therefore covers the file. An unindexed file injected into a package is
not used as a default save state.
