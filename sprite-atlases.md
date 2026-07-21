---
layout: page
title: Sprite Atlases
---

An image resource can expose several named sprites from one texture. Moonshine
supports two ways to define those sprite rectangles:

| Mode | Use it when |
|------|-------------|
| `manual` | You want to write and maintain every rectangle in `manifest.json`. |
| `mask` | You want Moonshine to detect framed sprites and keep their rectangles synchronized. |

The mode belongs to each image resource, so one ROM can mix manual and mask
atlases. The `mode` field is required and must be either `manual` or `mask`.

## Manual Atlases

In `manual` mode, define an `id` and a `rect` for every sprite:

```json
{
  "resources": {
    "images": [
      {
        "id": "characters",
        "fileName": "characters.png",
        "mode": "manual",
        "sprites": [
          {
            "id": "snake_head_up",
            "rect": { "x": 9, "y": 4, "w": 16, "h": 16 }
          }
        ]
      }
    ]
  }
}
```

The image origin is its top-left pixel. `x` increases to the right, `y`
increases downward, and `w` and `h` are the sprite width and height.

![Manual sprite coordinates]({{ site.baseurl }}/assets/images/sprite-atlases/sprite_manual.png)

Moonshine uses these rectangles as written. If you move a sprite inside the
texture, update its `rect` yourself.

## Mask Atlases

In `mask` mode, draw a one-pixel frame around each sprite using the image's
`maskColor`. The usual starting point is an empty sprite list:

```json
{
  "resources": {
    "images": [
      {
        "id": "characters",
        "fileName": "characters.png",
        "mode": "mask",
        "maskColor": "#ff00ff",
        "sprites": []
      }
    ]
  }
}
```

![Sprites surrounded by mask frames]({{ site.baseurl }}/assets/images/sprite-atlases/sprite_mask_1.png)

The mask detector follows these rules:

- `maskColor` is required and uses the `#RRGGBB` format.
- Color matching is exact on RGB values. Alpha is not used for matching.
- Frames are one pixel thick, closed, and have no anti-aliasing.
- The generated `rect` is the area inside the frame; the technical border is
  excluded.
- Sprites may have different sizes and positions. No grid is required.
- When several closed rectangles overlap as possible candidates, Moonshine
  keeps the minimal frames rather than larger composite rectangles.
- Incomplete frames are ignored.

Treat the mask color as reserved technical data. Using it inside the artwork can
create shapes that are difficult to detect unambiguously.

## Stable Sprite Markers

During synchronization, Moonshine gives every detected sprite a unique
`keyColor`. It writes that color into the frame's top edge, immediately before
the top-right corner at `(right - 1, top)`:

```text
MMMMMMKM
M      M
M SPR  M
M      M
MMMMMMMM
```

`M` is the `maskColor` and `K` is the generated `keyColor`. Both are technical
pixels outside the sprite rectangle. When frames share a border, a marker on a
top border belongs to the frame below it.

![Mask markers and ignored incomplete frames]({{ site.baseurl }}/assets/images/sprite-atlases/sprite_mask_2.png)

After synchronization, the manifest contains the detected rectangles and their
markers:

```json
{
  "id": "characters",
  "fileName": "characters.png",
  "mode": "mask",
  "maskColor": "#ff00ff",
  "sprites": [
    {
      "id": "id_1",
      "keyColor": "#000001",
      "rect": { "x": 9, "y": 4, "w": 16, "h": 16 }
    },
    {
      "id": "id_2",
      "keyColor": "#000002",
      "rect": { "x": 26, "y": 4, "w": 16, "h": 16 }
    }
  ]
}
```

Generated sprite ids such as `id_1` can be renamed. Keep them unique inside the
image, and keep each generated `keyColor` attached to its frame.

## Synchronizing In Game Maker

Load the ROM manifest in the local Game Maker flow. When the manifest contains
at least one mask image, the editor enables **sync sprite atlases**.

The action processes every image whose `mode` is `mask`:

1. It analyzes the texture and builds a synchronization plan.
2. It matches existing sprites by `keyColor`, then by an unchanged rectangle
   when a marker is missing.
3. It assigns ids and markers to new unambiguous frames.
4. It updates moved rectangles and restores missing markers when safe.
5. It writes the texture and manifest together for each conflict-free image.

The summary reports updated, unchanged, conflicting, and failed images. Detailed
per-image information is written to the application log. An image with conflicts
is left unchanged while other conflict-free mask images can still be updated.

## Moving And Adding Sprites

To move a sprite, move its artwork, complete mask frame, and `keyColor` pixel
together. On the next synchronization, Moonshine recognizes the marker and
updates only the rectangle in the manifest.

![Mask frames moved with their stable markers]({{ site.baseurl }}/assets/images/sprite-atlases/sprite_mask_3.png)

You can also draw a new closed frame without a marker. If the existing sprites
can all be matched, Moonshine creates a new id and `keyColor`, writes the marker
into the frame, and adds the sprite to the manifest.

If a marker disappears but its frame remains at the same rectangle, Moonshine
can restore it. If both the marker and the positional match are lost, the old
sprite and the new frame cannot be associated safely; synchronization reports a
conflict instead of guessing.

## Generated Hashes

Mask synchronization maintains two optional technical fields on the image:

| Field | Meaning |
|-------|---------|
| `hash` | SHA-256 of the saved texture bytes. |
| `definitionHash` | SHA-256 of the canonical image definition: `id`, `fileName`, `mode`, `maskColor`, and ordered sprite `id`, `keyColor`, and `rect` values. |

When both hashes still match, Moonshine can determine that no synchronization
work is required. These fields are generated by Moonshine and normally should
not be edited by hand.

## Conflicts And Limitations

Mask synchronization deliberately avoids guessing. Common conflicts include:

- the same `keyColor` appearing in more than one frame;
- an existing sprite whose marker and previous rectangle can no longer be
  found;
- an unmarked frame that cannot be associated while an existing sprite is
  missing.

Avoid changing generated markers, mixing markers between frames, moving a
marker separately from its frame, or partially erasing a frame. Incomplete
frames are ignored rather than imported.

## Related

- **[ROM Resources]({% link resources.md %})** - Resource folders, runtime handles, audio, and fonts.
- **[Manifest Reference]({% link manifest.md %})** - Complete manifest field reference.
- **[Lua API v1 Reference]({% link lua-api-v1.md %})** - Loading and drawing sprites at runtime.
