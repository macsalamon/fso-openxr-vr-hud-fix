# Optional: FotG autotarget circle follows the HUD + softer targeting-computer tint

**For Fate of the Galaxy (FotG_R) 1.1.2 and 1.1.3** (1.1.3 did not change this file, so the change is identical). Included with the
permission of the FotG team (for the circle change); it is their file with changes. Not an official FotG release.

It contains two changes to FotG's script `autoaim-sct.lua`. It only makes sense together with the engine fix from this repo.

## 1. The autotarget circle follows the HUD

The autotarget circle is drawn by FotG's script from the `On Frame` hook, which runs after the HUD panel, so it is drawn directly
on each eye's image and tilts with head roll. This change draws the circle from an `On HUD Draw` hook instead, so it is part of
the ship-fixed HUD panel and no longer tilts. The aiming logic, lock sounds and the targeting-computer gauge stay as they were.

## 2. Targeting-computer tint (v0.2.1)

When the targeting computer is switched on (to lock on with torpedoes etc.), FotG tints the **whole screen** orange. On a flat
screen that is fine, in VR it covers the whole view and is annoying. The red box around the centre reticle already shows that the
computer is on. Now the tint is a soft glow around the middle of the HUD only, drawn into the HUD panel. You can change this
in the settings at the top of the file (open it with any text editor, game closed, then restart the game):

```lua
occluder_color = {165, 103, 43, 20},
occluder_mode = "center",
occluder_center_radius = 0.21,
occluder_center_steps = 6,
```

| Setting | Meaning |
|---|---|
| `occluder_mode` | `"center"` = soft glow around the middle of the HUD (default here). `"off"` = no tint at all. `"full"` = whole screen, as in the original. |
| `occluder_color` | Red, green, blue and **opacity** (0 - 255) of the tint. The last number is the strength: `20` is subtle, `50` is the original FotG value, `0` = invisible. |
| `occluder_center_radius` | `"center"` mode: size of the glow as a fraction of the screen height. `0.21` roughly covers the area used for target lock. |
| `occluder_center_steps` | `"center"` mode: number of stacked circles that make the fade. More = smoother edge, but each circle needs an opacity of at least ~5, so the script limits the count to `opacity / 5` (opacity 20 -> at most 4 circles, 50 -> 10). Without that limit, many very faint circles lose their green and blue in the 8-bit HUD texture and the glow turns red. |

To get the original FotG behaviour back: set `occluder_mode = "full"` and `occluder_color = {165, 103, 23, 50}`, or restore FotG's
original file (a FotG update / "Verify/Repair" does that as well).

## Install

1. Back up the original: `Library\FotG_R\FotG_R-<your version>\data\tables\lua\autoaim-sct.lua`.
2. Unpack `VRHUDfix_optional_FotG_circle_script_FotG1.1.3.zip` (or `..._FotG1.1.2.zip`, from the Releases page) into your
   Knossos folder and confirm "Replace", or copy `Library/FotG_R/FotG_R-1.1.3/data/tables/lua/autoaim-sct.lua` (resp. 1.1.2)
   from this folder over the original.
3. A FotG update or "Verify/Repair" will overwrite the file; repeat step 2 afterwards (and re-apply your settings).

## What changed

See [`autoaim-sct.diff`](autoaim-sct.diff). Circle: `OnFrame` gets a `hud_pass` argument, the circle is only drawn from
`On HUD Draw`, and one `On HUD Draw` hook is added. Tint: `DrawSecondary` draws the glow (or the full-screen rectangle, or
nothing) according to `occluder_mode`, and the four settings above are new.

Note: the circle's arc is deliberately partial when the target is outside the aim-assist cone (that is the mod's design,
not a bug).
