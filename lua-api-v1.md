---
layout: page
title: Lua API v1 Reference
---

Moonshine exposes the runtime API through the global `api` table. The selected
API version comes from `manifestApiVersion` in `manifest.json`. Version `1` is
the only public Lua API version today.

This page describes the API as Authors should use it. For a first runnable ROM,
start with [Getting Started]({{ site.baseurl }}{% link getting-started.md %}).

The SDK stubs shipped with Moonshine mirror this API for EmmyLua completion:

```text
sdk/api/v1/
├── api.lua
├── audio.lua
├── globals.lua
├── graphics.lua
├── input.lua
├── log.lua
├── progress.lua
├── safety.lua
├── session.lua
└── types.lua
```

## Runtime Lifecycle

Moonshine loads the Lua entry module, prepares the `api` table, restores the
session context, and then calls the lifecycle functions.

| Function | Required | When it runs |
|----------|----------|--------------|
| `init()` | No | Once before the first frame, after `api` is ready. |
| `update()` | Yes | Once per frame. Put gameplay, input handling, save changes, progression changes, and result decisions here. |
| `draw()` | Yes | Once per rendered frame after `update()` in interactive play. Use it for presentation. |

```lua
local frame = 0

function init()
  api.log.info("ROM started")
end

function update()
  frame = frame + 1

  if api.input.Start.DownCount == 1 then
    api.session.end_game()
  end
end

function draw()
  api.graphics.draw_text(24, 24, "Frame: " .. frame)
end
```

Keep result-changing logic in `update()`. Server audit replays sessions in a
headless runtime, so rendering code should not decide save state, progression,
score, or outcome.

## Lua Environment

The runtime is sandboxed. Direct filesystem and operating system access are not
part of the public API.

Moonshine removes or blocks:

- `os`, `io`, and `debug`.
- `dofile`, `loadfile`, and `load`.
- `package.searchpath`.
- `rawset`.
- Native module loading in normal runtime.

`require()` can load ROM modules. In local maker mode, modules are resolved under
the ROM package root. In packed or server-backed play, modules are loaded from
the package module map. Module names use dot notation, such as
`require("lib.helper")`.

Do not try to work around the sandbox. If a result depends on hidden filesystem
state, operating system APIs, native modules, or rendering measurements that
headless audit cannot reproduce, Avalon may not be able to validate the
session.

`math.random` is seeded from the session ticket. `math.randomseed` is removed
after the initial seed, so ROM code cannot reseed randomness during play.

Most API tables are read-only. `api.save` is the mutable table intended for
persistent ROM state.

## API Root

| Field | Description |
|-------|-------------|
| `api.version` | Current Lua API version. For this page, `1`. |
| `api.session` | Session context and lifecycle helpers. |
| `api.save` | Mutable persistent save table. |
| `api.progress` | Milestone and badge helpers. |
| `api.input` | Per-frame input snapshot. |
| `api.graphics` | Rendering and visual resource helpers. |
| `api.audio` | Sound and music helpers. |
| `api.log` | Host logging helpers. |
| `api.safety` | Reserved safety hooks. Currently no-op. |

## `api.session`

`api.session` contains launch context and lifecycle helpers. The table and its
`selection` child are read-only.

| Field or function | Type | Description |
|-------------------|------|-------------|
| `api.session.player_id` | `string \| nil` | Current player id when supplied by the host. |
| `api.session.variant_id` | `string` | Selected variant id from the manifest. |
| `api.session.debug` | `boolean` | `true` when launched with local maker debugging enabled. |
| `api.session.selection` | `table<string, string>` | Selected menu values keyed by menu input id. |
| `api.session.end_game()` | function | Requests submission of the session result once. |
| `api.session.shutdown()` | function | Ends the ROM immediately and returns control to Moonshine. |

```lua
function init()
  local variant = api.session.variant_id
  local difficulty = api.session.selection.difficulty or "Normal"

  api.log.info("Variant: " .. variant)
  api.log.info("Difficulty: " .. difficulty)
end
```

Use bracket access for menu ids that are not valid Lua field names:

```lua
local start_level = api.session.selection["start-level"]
```

### Ending A Session

Call `api.session.end_game()` after all final result state has been updated.
The current `update()` finishes, then Moonshine can collect the session result.
Calling `end_game()` more than once has no additional effect.

The ROM keeps control until it calls `api.session.shutdown()`. This allows a
short result screen or acknowledgement after the result boundary.

```lua
local result_sent = false
local shutdown_countdown = 0

local function finish_run()
  api.save.best_score = math.max(api.save.best_score or 0, score)
  api.progress.unlock_milestone("first_clear")
  api.session.end_game()
  result_sent = true
  shutdown_countdown = 120
end

function update()
  if won and not result_sent then
    finish_run()
    return
  end

  if result_sent then
    shutdown_countdown = shutdown_countdown - 1
    if shutdown_countdown <= 0 then
      api.session.shutdown()
    end
  end
end
```

For the broader server-backed flow, see
[Session Lifecycle]({{ site.baseurl }}{% link session-lifecycle.md %}).

## `api.save`

`api.save` is a persistent table loaded before `init()` and encoded when the
session result is collected. On the first execution of a ROM for a player, it
starts as an empty table.

The `api` root is read-only, so a ROM cannot replace `api.save`. The table
contents are mutable:

```lua
function init()
  api.save.play_count = (api.save.play_count or 0) + 1
end

local function remember_best(score)
  api.save.best_score = math.max(api.save.best_score or 0, score)
end
```

Assigning `nil` removes a field from the next saved state:

```lua
api.save.temporary_flag = nil
```

Nested tables can be updated in place, but initialize them first:

```lua
if type(api.save.settings) ~= "table" then
  api.save.settings = {}
end

api.save.settings.speed = "fast"
```

Lists are regular Lua tables. Keep them compact with `table.insert` and
`table.remove`:

```lua
if type(api.save.items) ~= "table" then
  api.save.items = {}
end

table.insert(api.save.items, "new_item")
table.remove(api.save.items, 1)
```

Save data should stay plain and small. Supported values are:

- Booleans, finite numbers, and strings.
- Tables with string or finite-number keys.
- Nested tables without metatables or cycles.

Do not store functions, threads, userdata, tables as keys, boolean keys,
metatables, or self-referencing tables. If Moonshine cannot encode `api.save`,
it keeps the previous save state blob instead of saving the invalid value.

The encoded save state is currently limited to 16 KiB. Treat save data as player
progress and preferences, not as a full replay log or large content store.

## `api.progress`

Progression is represented as milestones and badges. Declare the public ids in
the manifest, then unlock them from Lua when the player earns them.

| Function | Description |
|----------|-------------|
| `api.progress.has_milestone(id)` | Returns `true` when the milestone is already unlocked in this session context. |
| `api.progress.unlock_milestone(id)` | Adds a milestone to the session result. Blank ids are ignored. |
| `api.progress.get_milestones()` | Returns a sorted snapshot array of unlocked milestone ids. |
| `api.progress.has_badge(id)` | Returns `true` when the badge is already awarded in this session context. |
| `api.progress.unlock_badge(id)` | Adds a badge to the session result. Blank ids are ignored. |
| `api.progress.get_badges()` | Returns a sorted snapshot array of awarded badge ids. |

```lua
if level_completed then
  api.progress.unlock_milestone("beat_level_1")
end

if api.progress.has_milestone("beat_level_1") then
  bonus_mode_available = true
end
```

The runtime treats ids case-insensitively. `get_milestones()` and
`get_badges()` return copies; mutating those arrays does not change progression.

```lua
for _, milestone in ipairs(api.progress.get_milestones()) do
  api.log.info("Milestone: " .. milestone)
end
```

Only milestones and badges added during the session are included as newly added
progress in the session result.

## `api.input`

`api.input` is a read-only snapshot of the current frame input state.

Available keys:

```text
A, B, C, D, Up, Down, Left, Right, Start
```

Each key is an `InputState` table:

| Field | Description |
|-------|-------------|
| `Key` | Input key name. |
| `DownCount` | `0` when up, `1` on the first pressed frame, greater than `1` while held. |

```lua
local function pressed(key)
  return api.input[key].DownCount == 1
end

local function held(key)
  return api.input[key].DownCount > 0
end

function update()
  if pressed("A") then
    jump()
  end

  if held("Left") then
    move_left()
  end
end
```

All public keys are present every frame. Missing host input state is exposed as
`DownCount = 0`.

## `api.graphics`

Graphics functions are for presentation. In headless replay, drawing functions
are no-op, while resource handle lookups and `measure_text()` remain
deterministic. Do not use graphics calls or measured text sizes to decide
gameplay results.

Color tables use `r`, `g`, `b`, and `a` channels from `0` to `255`. `nil` color
or missing channels default to `255`.

```lua
local red = { r = 255, g = 64, b = 64, a = 255 }
```

### Built-In Fonts

`api.graphics.FontName` contains built-in font constants:

```lua
api.graphics.FontName.PixelVa
api.graphics.FontName.B2BS
api.graphics.FontName.ScoreDozer
```

Use the constants by name rather than depending on their numeric values.

### Text

| Function | Description |
|----------|-------------|
| `api.graphics.draw_text(x, y, text)` | Draws text with the default font. |
| `api.graphics.draw_text_ex(x, y, text, fontNameOrHandle, scale, color)` | Draws text with a built-in font constant or custom font handle. |
| `api.graphics.draw_text_loc(x, y, fontHandle, key)` | Draws localized text from a custom font resource. |
| `api.graphics.measure_text(text, fontName)` | Returns `{ width, height }` for visual layout. |
| `api.graphics.get_font_handle(fontId)` | Returns a custom font handle from manifest `resources.fonts`, or `-1` when missing. |

```lua
api.graphics.draw_text(24, 24, "Ready")

api.graphics.draw_text_ex(
  24,
  48,
  "Danger",
  api.graphics.FontName.B2BS,
  1.5,
  { r = 255, g = 64, b = 64, a = 255 }
)
```

Custom fonts come from manifest `resources.fonts`:

```lua
local font = -1

function init()
  font = api.graphics.get_font_handle("main")
end

function draw()
  api.graphics.draw_text_ex(24, 80, "Custom font", font, 1.0, nil)
end
```

`get_font_handle(fontId)` returns a stable handle for a resolved font. It
returns `-1` when the id is blank, the manifest resource is missing, or the
current renderer cannot provide font handles. In headless replay, a manifest
font can still receive a deterministic positive handle.

`draw_text_loc(x, y, fontHandle, key)` resolves `key` through the localization
map of the font resource associated with `fontHandle`. If the key is missing,
the key itself is drawn. If the handle is invalid in interactive play, Moonshine
falls back to drawing the key with the default text renderer. In headless replay,
the draw call is a no-op.

```lua
function draw()
  api.graphics.draw_text_loc(24, 112, font, "title")
end
```

`measure_text(text, fontName)` returns a table:

```lua
local size = api.graphics.measure_text("Ready", api.graphics.FontName.B2BS)
api.graphics.draw_text(24, 24 + size.height, "Go")
```

Headless replay uses a deterministic approximation for measured text size, so
measurements are useful for layout only.

### Sprites

| Function | Description |
|----------|-------------|
| `api.graphics.get_sprite_handle(imageId, spriteId)` | Returns a sprite handle from manifest `resources.images`, or `-1` when missing. |
| `api.graphics.draw_sprite(spriteHandle, x, y)` | Draws a sprite at top-left coordinates using its source size. |
| `api.graphics.draw_sprite_ex(spriteHandle, x, y, width, height, rotation, originX, originY, color)` | Draws a sprite with explicit size, rotation, origin, and tint. |
| `api.graphics.set_background(index)` | Selects the active background by index. |

```lua
local player_sprite = -1

function init()
  player_sprite = api.graphics.get_sprite_handle("characters", "player")
end

function draw()
  api.graphics.draw_sprite(player_sprite, 100, 120)
end
```

`get_sprite_handle(imageId, spriteId)` looks up an image resource and one of its
declared sprites. A resolved handle is stable and cached for the session. The
function returns `-1` when either id is blank, the resource is missing, or the
current renderer cannot provide texture handles.

Invalid sprite handles are safe: `draw_sprite()` and `draw_sprite_ex()` simply
do nothing.

```lua
api.graphics.draw_sprite_ex(
  player_sprite,
  100,
  120,
  32,
  32,
  0,
  16,
  16,
  { r = 255, g = 255, b = 255, a = 255 }
)
```

`rotation` is expressed in radians. `originX` and `originY` are passed to the
renderer as the rotation origin.

## `api.audio`

Audio functions use manifest resources when handles are needed. In headless
replay, audio playback is no-op.

| Function | Description |
|----------|-------------|
| `api.audio.play_sfx(sfxId)` | Plays a sound effect by id. |
| `api.audio.get_music_handle(musicId)` | Returns a music handle from manifest `resources.musics`, or `-1` when missing. |
| `api.audio.play_music(musicHandle, startLoopMs, endLoopMs)` | Plays music by handle with loop bounds in milliseconds. |
| `api.audio.stop_music()` | Stops the current music. |

```lua
local theme = -1

function init()
  theme = api.audio.get_music_handle("theme")
  api.audio.play_music(theme, 0, -1)
end

function update()
  if api.input.A.DownCount == 1 then
    api.audio.play_sfx("confirm")
  end
end
```

`get_music_handle(musicId)` returns `-1` when the id is blank, the manifest
resource is missing, or the current renderer cannot provide music handles.
Passing `-1` or any handle below `1` to `play_music()` is safe and does nothing.

For `play_music`, `startLoopMs = 0` means the start of the track and
`endLoopMs = -1` means the full track loop.

## `api.log`

Logging helpers forward messages to the host log sink.

| Function | Description |
|----------|-------------|
| `api.log.debug(message)` | Debug-level log. |
| `api.log.info(message)` | Info-level log. |
| `api.log.warn(message)` | Warning log. |
| `api.log.error(message)` | Error log. |

```lua
api.log.info("Loaded variant " .. api.session.variant_id)
```

Use logs for diagnostics, not gameplay state.

## `api.safety`

`api.safety` is reserved for future runtime safety hooks.

| Field or function | Description |
|-------------------|-------------|
| `api.safety.enabled` | Reserved flag. Currently `false`. |
| `api.safety.checkpoint()` | Reserved checkpoint hook. Currently no-op. |

Do not build gameplay behavior around this module.

## Types

| Type | Shape |
|------|-------|
| `SpriteHandle` | Integer returned by `api.graphics.get_sprite_handle`; `-1` means missing. |
| `FontHandle` | Integer returned by `api.graphics.get_font_handle`; `-1` means missing. |
| `MusicHandle` | Integer returned by `api.audio.get_music_handle`; `-1` means missing. |
| `Color` | `{ r?: integer, g?: integer, b?: integer, a?: integer }`, channel values `0` to `255`. |
| `TextSize` | `{ width: integer, height: integer }`. |
| `InputState` | `{ Key: string, DownCount: integer }`. |

## Not Available Yet

These are not part of Lua API v1:

- Leaderboard score submission.
- Leaderboard score querying.
- Multi-player Lua session APIs.

Ranking tables can currently be declared in the manifest, but Moonshine does not
yet expose a public Lua score API.

## Related

- **[Getting Started]({{ site.baseurl }}{% link getting-started.md %})** - First ROM setup.
- **[Session Lifecycle]({{ site.baseurl }}{% link session-lifecycle.md %})** - Session start, result submission, and shutdown flow.
- **[Manifest Reference]({{ site.baseurl }}{% link manifest.md %})** - Manifest fields and validation.
- **[ROM Resources]({{ site.baseurl }}{% link resources.md %})** - Images, audio, fonts, and badge assets.
- **[Menus & Configuration]({{ site.baseurl }}{% link menus-configuration.md %})** - `api.session.selection`.
- **[Progression System]({{ site.baseurl }}{% link progression-milestones.md %})** - Milestones and badges.
