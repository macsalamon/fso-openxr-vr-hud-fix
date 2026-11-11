# Optional: FotG autotarget circle follows the HUD

**For Fate of the Galaxy (FotG_R) 1.1.2 and 1.1.3** (1.1.3 did not change this file, so the change is identical). Included with the permission of the FotG team; it is their file with a small change.
Not an official FotG release.

The autotarget circle is drawn by FotG's script `autoaim-sct.lua` from the `On Frame` hook, which runs after the HUD panel,
so it is drawn directly on each eye's image and tilts with head roll. This change draws the circle from an `On HUD Draw`
hook instead, so it is part of the ship-fixed HUD panel and no longer tilts. The aiming logic, lock sounds and the
targeting-computer tint stay as they were.

It only makes sense together with the engine fix from this repo; on its own the circle is still drawn, but nothing else
changes.

## Install

1. Back up the original: `Library\FotG_R\FotG_R-<your version>\data\tables\lua\autoaim-sct.lua`.
2. Unpack `VRHUDfix_optional_FotG_circle_script_FotG1.1.3.zip` (or `..._FotG1.1.2.zip`, from the Releases page) into your
   Knossos folder and confirm "Replace", or copy `Library/FotG_R/FotG_R-1.1.3/data/tables/lua/autoaim-sct.lua` (resp. 1.1.2)
   from this folder over the original.
3. A FotG update or "Verify/Repair" will overwrite the file; repeat step 2 afterwards.

## What changed

See [`autoaim-sct.diff`](autoaim-sct.diff): `OnFrame` gets a `hud_pass` argument, the targeting-computer tint is only drawn from
`On Frame`, the circle only from `On HUD Draw`, and one `On HUD Draw` hook is added.

Note: the circle's arc is deliberately partial when the target is outside the aim-assist cone (that is the mod's design,
not a bug).
