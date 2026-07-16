---
layout: page
title: ROM Resources
---

ROM resources are the files a Lua ROM uses for presentation: images, sprites,
sound effects, music, custom fonts, localized text attached to those fonts, and
badge icons.

The manifest declares which resources exist. The files themselves live under the
ROM `assets` folder. Lua then uses resource ids to request handles and draw or
play the resource at runtime.

For the exact JSON field reference, see
[Manifest Reference]({{ site.baseurl }}{% link manifest.md %}). For the runtime
function signatures, see
[Lua API v1 Reference]({{ site.baseurl }}{% link lua-api-v1.md %}).

## Folder Layout

Moonshine expects resources under fixed folders next to `manifest.json`:

```text
my-rom/
├── manifest.json
├── main.lua
└── assets/
    ├── images/
    ├── sfx/
    ├── musics/
    └── fonts/
```

The `fileName` declared in the manifest is only the file name. Do not include
`assets/images/`, `assets/sfx/`, `assets/musics/`, `assets/fonts/`, `../`, or an
absolute path.

Moonshine resolves each resource into its matching folder:

| Manifest block | Folder |
|----------------|--------|
| `resources.images` | `assets/images/` |
| `resources.sfx` | `assets/sfx/` |
| `resources.musics` | `assets/musics/` |
| `resources.fonts` | `assets/fonts/` |

Resource ids follow the usual Moonshine id rules: use letters, numbers, `_`, or
`-`, and keep ids 32 characters or fewer.

## Minimal Resource Block

```json
{
  "resources": {
    "images": [
      {
        "id": "tiles",
        "fileName": "tiles.png",
        "sprites": [
          { "id": "block", "x": 0, "y": 0, "width": 16, "height": 16 }
        ]
      }
    ],
    "sfx": [
      { "id": "clear", "fileName": "clear.ogg" }
    ],
    "musics": [
      { "id": "theme", "fileName": "theme.ogg" }
    ],
    "fonts": [
      {
        "id": "main",
        "fileName": "main.ttf",
        "ttfFontSize": 16,
        "localization": {
          "title": "Ready"
        }
      }
    ]
  }
}
```

The `resources` block is optional. A ROM can start with no custom resources and
add them later.

## Images And Sprites

Images are declared in `resources.images` and loaded from `assets/images/`.

Each image has:

| Field | Meaning |
|-------|---------|
| `id` | Image id used by Lua when requesting a sprite handle. |
| `fileName` | Image file name under `assets/images/`. PNG is the normal format used by Moonshine examples. |
| `sprites` | Optional list of named rectangles inside the image. |

Each sprite has:

| Field | Meaning |
|-------|---------|
| `id` | Sprite id, unique inside that image. |
| `x` / `y` | Top-left pixel in the image. |
| `width` / `height` | Sprite rectangle size in pixels. |

Sprite ids only need to be unique within one image. Two different images can
both define a sprite called `idle`.

```json
{
  "resources": {
    "images": [
      {
        "id": "characters",
        "fileName": "characters.png",
        "sprites": [
          { "id": "player_idle", "x": 0, "y": 0, "width": 24, "height": 24 },
          { "id": "player_run", "x": 24, "y": 0, "width": 24, "height": 24 }
        ]
      }
    ]
  }
}
```

Lua uses both ids:

```lua
local player_idle = -1
local character_sprites = {}

function init()
  player_idle = api.graphics.get_sprite_handle("characters", "player_idle")
  character_sprites = api.graphics.get_sprite_handles("characters")
end

function draw()
  api.graphics.draw_sprite(player_idle, 100, 120)
  api.graphics.draw_sprite(character_sprites.player_run, 130, 120)
end
```

`get_sprite_handle(imageId, spriteId)` returns `-1` when the image, sprite, or
texture cannot be resolved. Drawing with an invalid handle is safe and does
nothing.

Use `get_sprite_handles(imageId)` when you want every sprite declared in one
image resource. It returns a table keyed by sprite id and uses the same cached
handles as `get_sprite_handle()`. Missing images or unavailable textures return
an empty table.

Use `draw_sprite_ex()` when you need an explicit destination size, rotation,
origin, or tint:

```lua
api.graphics.draw_sprite_ex(
  player_idle,
  100,
  120,
  48,
  48,
  0,
  24,
  24,
  { r = 255, g = 255, b = 255, a = 255 }
)
```

## Sound Effects

Sound effects are short sounds declared in `resources.sfx` and loaded from
`assets/sfx/`. They use `.ogg` files.

```json
{
  "resources": {
    "sfx": [
      { "id": "confirm", "fileName": "confirm.ogg" },
      { "id": "clear", "fileName": "clear.ogg" }
    ]
  }
}
```

Lua plays a sound effect directly by id:

```lua
if api.input.A.DownCount == 1 then
  api.audio.play_sfx("confirm")
end
```

Sound effects are fire-and-forget. Use them for actions like confirms, hits,
clears, or warnings.

## Music

Music resources are declared in `resources.musics` and loaded from
`assets/musics/`. They use `.ogg` files.

```json
{
  "resources": {
    "musics": [
      { "id": "theme", "fileName": "theme.ogg" }
    ]
  }
}
```

Music uses handles:

```lua
local theme = -1

function init()
  theme = api.audio.get_music_handle("theme")
  api.audio.play_music(theme, 0, -1)
end

function update()
  if should_stop_music then
    api.audio.stop_music()
  end
end
```

`get_music_handle(musicId)` returns `-1` when the music cannot be resolved.
Passing `-1` to `play_music()` is safe and does nothing.

For `play_music`, `startLoopMs = 0` starts at the beginning of the track and
`endLoopMs = -1` means the full track loop.

## Fonts And Localized Text

Custom fonts are declared in `resources.fonts` and loaded from `assets/fonts/`.
They use `.ttf` files.

```json
{
  "resources": {
    "fonts": [
      {
        "id": "main",
        "fileName": "main.ttf",
        "ttfFontSize": 16,
        "baseSet": "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789 .,:;!?-_",
        "outline": true,
        "localization": {
          "title": "Ready",
          "game_over": "Game Over"
        }
      }
    ]
  }
}
```

| Field | Meaning |
|-------|---------|
| `id` | Font id used by Lua. |
| `fileName` | `.ttf` file name under `assets/fonts/`. |
| `ttfFontSize` | Font generation size. Must be greater than 0. |
| `baseSet` | Optional characters to include in the generated bitmap font. |
| `outline` | Optional outline flag. |
| `localization` | Text keys attached to the font. Required for custom font resources. |

Moonshine builds the supported character set from `baseSet` plus all characters
used by the font localization values. If text can appear at runtime but is not
present in localization values, add those characters to `baseSet`.

Lua can draw custom fonts directly:

```lua
local main_font = -1

function init()
  main_font = api.graphics.get_font_handle("main")
end

function draw()
  api.graphics.draw_text_ex(24, 24, "Ready", main_font, 1.0, nil)
end
```

Lua can also draw a localization key attached to the font:

```lua
function draw()
  api.graphics.draw_text_loc(24, 48, main_font, "title")
end
```

`draw_text_loc()` resolves the key through the font localization map. If the key
is missing, Moonshine draws the key itself. The current API stores localized text
inside each font resource; a broader ROM localization workflow may evolve later.

Built-in fonts are separate from custom font resources:

```lua
api.graphics.draw_text_ex(
  24,
  72,
  "Built-in font",
  api.graphics.FontName.B2BS,
  1.0,
  nil
)
```

Use built-in `api.graphics.FontName` constants when you do not need a custom
font file.

## Badge Atlas

Badges are declared outside the `resources` block, but badge icons still require
an image file.

If the manifest declares badges, Moonshine expects:

```text
assets/images/badges.png
```

The current atlas layout is 32 slots, 8 columns by 4 rows, with 32x32 pixel
icons. That means `badges.png` should be 256x128 pixels. Each badge uses its
`iconIndex` from the manifest to choose one slot.

```json
{
  "badges": [
    {
      "id": "gm",
      "label": "GM",
      "description": "Reached GM rank",
      "iconIndex": 0
    }
  ]
}
```

The badge atlas is handled by Moonshine as badge UI data. You normally unlock
badges through `api.progress.unlock_badge(id)`, not by drawing the atlas
yourself.

## Runtime Behavior

Moonshine builds a resource index when it loads or packs the ROM. Each file gets
a content hash used for caching and verification.

During interactive play:

- Sprite handles are resolved from image and sprite ids.
- Music handles are resolved from music ids.
- Font handles are resolved from font ids.
- Missing handles return `-1` instead of throwing.

During headless audit replay:

- Drawing functions do nothing.
- Audio playback does nothing.
- Resource handle lookup remains deterministic when the resource exists.
- `measure_text()` returns a deterministic approximation, not renderer-perfect
  layout.

Do not base gameplay results, save state, milestones, badges, or score decisions
on rendering or audio behavior.

## Common Problems

| Symptom | What to check |
|---------|---------------|
| Resource file missing at load | The file is in the matching `assets/<type>/` folder, and `fileName` contains only the file name. |
| `get_sprite_handle()` returns `-1` | Image id, sprite id, sprite rectangle, and image file are declared correctly. |
| `get_sprite_handles()` returns an empty table | Image id, sprite rectangles, and image file are declared correctly. |
| `get_music_handle()` returns `-1` | Music id exists in `resources.musics`, the file is `.ogg`, and the renderer supports music. |
| `get_font_handle()` returns `-1` | Font id exists in `resources.fonts`, the file is `.ttf`, `ttfFontSize` is positive, and localization is not empty. |
| Localized text draws the key | The key is missing or has an empty value in that font's `localization` map. |
| Custom text misses characters | Add those characters to the font `baseSet` or include them in localization values. |
| Badge icons fail to load | `assets/images/badges.png` exists and is 256x128 when badges are declared. |

## Related

- **[Manifest Reference]({{ site.baseurl }}{% link manifest.md %})** - Exact JSON fields and validation rules.
- **[Lua API v1 Reference]({{ site.baseurl }}{% link lua-api-v1.md %})** - Runtime functions for graphics and audio.
- **[Progression System]({{ site.baseurl }}{% link progression-milestones.md %})** - Milestones and badges.
