# What I changed in the engine, in plain words

Base: OFS nightly `nightly_20261123_c9bbe61f3` (includes BMagnu's VR HUD panel, PR #8058).
Full change as a patch file: `nightly_billboard_hud_engine_changes.patch` (same folder).
Size: 5 files, about 545 lines added, 16 removed. About 85% of that is one new drawing function.

The targeting-computer rectangle is NOT part of this. It is only a FotG Lua script change and needs no engine change.

---

## How the nightly's VR HUD works (the starting point)

The game draws the whole HUD once per frame into a flat picture (a "layer"), as if seen by a camera at the ship that does not follow your head.
That picture is then shown to each eye as a flat panel that hangs in front of the ship at 1000 m.
This is what makes the HUD stay fixed to the ship, like a cockpit, and it is a good design.

Two things I wanted to improve on top of it:

## Change 1: the target frame is drawn separately, standing on the target ("billboard")

**The problem.** The target frame (the corner brackets around the target, with its name and distance) is drawn into the flat HUD picture, at the spot where the target appears in that picture.
Because the picture is flat and covers a limited field of view, the frame is stretched towards the edges, and it can be cut off or missing when the target is outside the picture.
The reason is that a flat picture cannot show something that is far to the side or behind the panel's view.

**What I did.** While the HUD is drawn, the frame is no longer drawn into the picture.
Instead the engine writes the target down in a short list (`hud_queue_vr_bracket` in `hudbrackets.cpp`):
- the 8 corners of the target's bounding box
- its position, colour, name, class and distance

Then, for each eye, right after the HUD panel is shown, a new function (`overlay_draw` in `hudvr.cpp`) draws the frame:
1. It works out the direction from the ship to the target.
2. It places the frame on a flat surface that stands at right angles to that direction, as if on a ball around the ship. The frame's "up" is the ship's up, so it is always upright and never stretched or tilted.
3. It projects the 8 box corners onto that surface to get the frame's size, with 5 px padding and a minimum size so it never shrinks to nothing.
4. It draws four corner brackets, each 1 px thick, with arms reaching halfway along each side, as quads.
5. The text (name, class, distance) is drawn into a second layer of the same size (`gr_screen_label_layer_*` in `2d.cpp`/`2d.h`), one slot per target, and shown as a picture under the frame. Using a layer means the text is the same font and same look as the normal HUD text.

Because the frame is drawn in 3D at the target's direction, it is not cut off at the edge of the HUD panel.

**Other details**
- The same target is only drawn once, even if two HUD gauges ask for it (`vr_overlay_add`).
- Works for ships, debris, weapons, asteroids and jump nodes.
- The player's current target always gets a frame, even outside the HUD panel's view.
- Up to 4 frames per image can have a text label (`VR_LABEL_SLOTS`). If more than 4 targets are bracketed at once, the extra frames are still drawn, but without text. (This number is arbitrary and can be raised.)
- It can be turned off with `VRHudBillboardBrackets=0` in `fs2_open.ini` under `[Default]`. Then the original behaviour is used again.
- It only does anything while the VR HUD panel is being drawn, so flat-screen play is not affected.
- OpenGL and Vulkan both handle the label picture's up/down difference (one line in `overlay_draw`). You have tested both.

## Change 2: the HUD is drawn at higher resolution (2x)

**The problem.** The nightly draws the HUD picture at about one HUD pixel per eye pixel.
The picture is then rotated and resampled onto the eye, which smears thin text. It looked "blurry as hell" in your test.

**What I did.** One line in `hud_vr_init` (`hudvr.cpp`): the HUD picture is made 2x bigger in each direction (4x the pixels).
A setting `VRHudResolutionScale` in `fs2_open.ini` under `[Default]` changes it (1.0 to 4.0, default 2.0).
The text and frame label use this bigger picture too, so they get sharper as well.

This is a different thing from `-native_res_hud`: that flag only helps when the headset's resolution is bigger than the game's render resolution, and in your setup it did nothing visible.

Cost: a bigger picture means a bit more graphics memory and a little more work per frame. You did not see any fps problem on your PC.

## Where the changes are

| File | What |
|---|---|
| `code/hud/hudvr.cpp` | The new drawing function (frame + labels), the queue, the two ini settings, the resolution multiplier. About 320 lines. |
| `code/hud/hudbrackets.cpp` | Collects target data into the queue instead of drawing the frame. About 155 lines. |
| `code/hud/hudvr.h` | Declarations. 26 lines. |
| `code/graphics/2d.cpp`, `2d.h` | The second layer for texts, made by reusing the layer code that already exists. About 60 lines. |

## Honest limits, for anyone reviewing it

- Tested by me and you only in play with a Valve Index on OpenGL and Vulkan, in FotG. Not tested with other headsets, other mods, or other HUD layouts.
- Only the HUD frame of the target is changed. Other gauges (the target box in the corner, speed, and so on) are unchanged.
- Only 4 text labels at a time, as mentioned above.
- The numbers (padding 5, minimum size 18, arm length 50%, label gap 4) are what looked right to me. They are constants at the top of `overlay_draw` and can be changed.
- It builds on the nightly's screen-layer code, so a bigger change to that code upstream could need a small rework.
- I have not talked to anyone about this, and nothing has been published or sent.
