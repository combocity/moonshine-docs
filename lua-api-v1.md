---
layout: page
title: Lua API v1 Reference
---

Moonshine exposes its runtime API through a global `api` table. The API version
is selected by `manifestApiVersion` in `manifest.json`. API v1 may still change
while the project evolves before release.

The SDK stubs shipped with Moonshine mirror this API and should be used for
EmmyLua autocompletion:

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

## Lifecycle

Moonshine requires `update` and `draw` to start a ROM. `init` is optional.

| Function | Required | Called when |
|----------|----------|-------------|
| `init()` | No | Once after the API tables are ready, when defined. Initialize state and load handles here. |
| `update()` | Yes | Every frame. Update gameplay state and react to input here. |
| `draw()` | Yes | Every frame after `update()`. Render the current frame here. |

```lua
local ticks = 0

function init()
  api.log.info("Session started")
end

function update()
  ticks = ticks + 1

  if api.input.Start.DownCount == 1 then
    api.session.end_game()
  end
end

function draw()
  api.graphics.draw_text(24, 24, "Ticks: " .. ticks)
end
```

## API Root

| Field | Description |
|-------|-------------|
| `api.version` | Current Lua API version. For this page, `1`. |
| `api.session` | Session context and lifecycle helpers. |
| `api.save` | Mutable persistent save table. |
| `api.progress` | Milestone and badge helpers. |
| `api.input` | Per-frame input snapshot. |
| `api.graphics` | Rendering helpers. |
| `api.audio` | Sound and music helpers. |
| `api.log` | Logging helpers. |
| `api.safety` | Reserved safety hooks. Currently no-op. |

Most API tables are read-only. `api.save` is the mutable table intended for ROM
save data.

## `api.session`

`api.session` contains the immutable launch context and session lifecycle
helpers.

| Field or function | Type | Description |
|-------------------|------|-------------|
| `api.session.player_id` | `string \| nil` | Current player id when supplied by the host. |
| `api.session.variant_id` | `string` | Selected variant id from the manifest. |
| `api.session.debug` | `boolean` | `true` when launched in local maker mode. |
| `api.session.selection` | `table<string, string>` | Selected menu values keyed by menu input id. |
| `api.session.end_game()` | function | Submits the session result once. The ROM keeps control until it calls `shutdown()`; later changes are not included in the submitted result. |
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

## `api.save`

`api.save` is a persistent table loaded before `init()` and included in
the session result when the host finalizes the session.

`api.save` is always a table. On a fresh save it is empty, so initialize your
own fields when they are `nil`.

```lua
function init()
  api.save.play_count = (api.save.play_count or 0) + 1
end
```

The API root is read-only, so the ROM cannot replace `api.save`. The content of
`api.save` is mutable:

```lua
function finish_game(score)
  local previous_best = api.save.best_score or 0
  api.save.best_score = math.max(previous_best, score)
  api.session.end_game()
end
```

Save state should contain plain data only: strings, numbers, booleans, and
simple tables. Do not store runtime objects, functions, or tables that reference
themselves. The encoded save state is currently limited to 16 KiB; if Moonshine
cannot encode the save table, it keeps the previous save state instead.
Assigning `nil` to a save field removes that field from the next saved state.
Nested tables can be updated in place:

```lua
api.save.settings.speed = "fast"
```

Lists are tables too. Use `table.insert` and `table.remove` to keep list indexes
compact:

```lua
if type(api.save.items) ~= "table" then
  api.save.items = {}
end

table.insert(api.save.items, "new item")
table.remove(api.save.items, 1)
```

Only insert plain save data into lists. If a list contains a runtime object,
function, self-reference, or too much data, Moonshine will keep the previous save
state instead of saving the invalid value.

## `api.progress`

Progression is backed by milestones and badges declared in the manifest.

| Function | Description |
|----------|-------------|
| `api.progress.has_milestone(id)` | Returns `true` when the milestone is already unlocked. |
| `api.progress.unlock_milestone(id)` | Unlocks a milestone for the end-session result. |
| `api.progress.get_milestones()` | Returns a sorted snapshot array of unlocked milestone ids. |
| `api.progress.has_badge(id)` | Returns `true` when the badge is already awarded. |
| `api.progress.unlock_badge(id)` | Awards a badge for the end-session result. |
| `api.progress.get_badges()` | Returns a sorted snapshot array of awarded badge ids. |

```lua
if level_completed then
  api.progress.unlock_milestone("beat_level_1")
end

if api.progress.has_milestone("beat_level_1") then
  bonus_mode_available = true
end
```

Snapshot arrays returned by `get_milestones()` and `get_badges()` are copies.
Changing those arrays does not change progression.

```lua
for _, milestone in ipairs(api.progress.get_milestones()) do
  api.log.info("Milestone: " .. milestone)
end
```

## `api.input`

`api.input` is a per-frame snapshot of the current input state.

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

## `api.graphics`

Rendering helpers draw text and manifest-declared sprites.

### Text

| Function | Description |
|----------|-------------|
| `api.graphics.draw_text(x, y, text)` | Draws text with the default font. |
| `api.graphics.draw_text_ex(x, y, text, fontNameOrHandle, scale, color)` | Draws text with a font, scale, and optional color. |
| `api.graphics.draw_text_loc(x, y, fontHandle, key)` | Draws localized text from a manifest font localization key. |
| `api.graphics.measure_text(text, fontName)` | Returns `{ width, height }` for layout. |

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

Built-in font names:

```lua
api.graphics.FontName.PixelVa
api.graphics.FontName.B2BS
api.graphics.FontName.ScoreDozer
```

`measure_text` is for visual layout only. Headless audit replay uses a
deterministic approximation, so do not use measured text size to decide gameplay
outcomes, progress, or scores.

### Sprites

| Function | Description |
|----------|-------------|
| `api.graphics.get_sprite_handle(imageId, spriteId)` | Returns a sprite handle from manifest `resources.images`. Returns `-1` when missing. |
| `api.graphics.draw_sprite(spriteHandle, x, y)` | Draws a sprite at top-left coordinates. |
| `api.graphics.draw_sprite_ex(spriteHandle, x, y, width, height, rotation, originX, originY, color)` | Draws a sprite with size, rotation, origin, and optional tint. |
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

Color tables use `r`, `g`, `b`, and `a` channels from `0` to `255`. Missing
channels default to `255`.

## `api.audio`

Audio helpers use resources declared in the manifest.

| Function | Description |
|----------|-------------|
| `api.audio.play_sfx(sfxId)` | Plays a sound effect declared in `resources.sfx`. |
| `api.audio.get_music_handle(musicId)` | Returns a music handle declared in `resources.musics`. Returns `-1` when missing. |
| `api.audio.play_music(musicHandle, startLoopMs, endLoopMs)` | Plays music with loop bounds in milliseconds. |
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

## `api.safety`

`api.safety` is reserved for runtime safety hooks.

| Field or function | Description |
|-------------------|-------------|
| `api.safety.enabled` | Reserved flag. Currently `false`. |
| `api.safety.checkpoint()` | Reserved checkpoint hook. Currently no-op. |

Do not build gameplay behavior around this module yet.

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
- Network or HTTP access from Lua.
- File system access from Lua.
- Multi-player Lua session APIs.

Leaderboards can currently be declared as manifest `rankingTables`, but Moonshine
does not yet expose a public Lua score API.

## Related

- **[Getting Started]({{ site.baseurl }}{% link getting-started.md %})** - First ROM setup.
- **[Session Lifecycle]({{ site.baseurl }}{% link session-lifecycle.md %})** - Session start, result submission, and shutdown flow.
- **[Manifest Reference]({{ site.baseurl }}{% link manifest.md %})** - Manifest fields and validation.
- **[Menus & Configuration]({{ site.baseurl }}{% link menus-configuration.md %})** - `api.session.selection`.
- **[Progression System]({{ site.baseurl }}{% link progression-milestones.md %})** - Milestones and badges.
