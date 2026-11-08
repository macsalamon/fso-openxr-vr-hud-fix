# FSO OpenXR VR HUD fix (unofficial)

A free, community fix for the HUD in **FreeSpace Open 26.0.1 in OpenXR VR**, made while playing
[Fate of the Galaxy](https://www.fateofthegalaxy.net/) on a Valve Index.

> **Status: early test release.** Confirmed working by one person on one headset (Valve Index) with
> Fate of the Galaxy 1.1.2. Reports from other VR users are very welcome - see "Feedback". If it holds up,
> the plan is to submit it to the FreeSpace Open project as a pull request.
> Not affiliated with the FreeSpace Open or Fate of the Galaxy teams.

## What it fixes

1. **HUD "swimming"** - reticle, target brackets and other projected HUD elements moved against head movement
   while the cockpit and the backdrop stayed fixed. The engine projected the HUD with a symmetric field of view,
   but VR eyes have asymmetric ones.
2. **HUD rotating with head roll, drifting with distance from the centre, and showing in a different place for each eye.**
   The HUD is now rendered once per frame, as seen from the ship, and shown as a flat panel fixed to the ship
   (like a monitor in the cockpit).
3. **Blurry HUD text** - the HUD is rendered at 2x resolution. Cockpit monitors and gauges are unchanged.

## Download and install (no compiling)

1. Download `VRHUDfix_FSO26.0.1_AVX_package.zip` from the [Releases](../../releases) page.
2. Close Knossos and the game. **Back up** your Knossos `Library` folder (at least `Library\bin\FSO-26.0.1\windows\x64_avx`).
3. Unpack the zip into your Knossos folder so that
   `Library\bin\FSO-26.0.1\windows\x64_avx\fs2_open_26_0_1_x64_AVX.exe` is overwritten (confirm "Replace").
   It replaces only the engine executable; Knossos keeps launching "FSO 26.0.1" as usual.
4. Start the game from Knossos.

**Uninstall:** copy your backup back over the folder. (Knossos "Verify/Repair" of the engine also restores the stock
exe - unpack the zip again afterwards.)

Requirements: Windows, Knossos with the `x64_avx` engine folder (the AVX build runs on any CPU with AVX, i.e.
essentially everything since 2011), OpenXR VR.

## Known limitations

- Only tested on a Valve Index with Fate of the Galaxy 1.1.2.
- Built **without text-to-speech and voice recognition** (not needed for VR play; the official build has them).
- In Fate of the Galaxy, the autotarget circle is drawn by one of the mod's own scripts and still tilts with head roll
  unless you also install the [optional script change](fotg/README.md) (FotG 1.1.2 only; included with the FotG team's permission).
- The HUD texture is 2x the render size, which uses some extra GPU memory. If performance suffers, lower
  `VRHudResolutionScale` (below).

## Optional: FotG autotarget circle

[`fotg/`](fotg/README.md) contains a one-file change to FotG's `autoaim-sct.lua` that makes the autotarget circle follow the HUD
instead of tilting with head roll. Download `VRHUDfix_optional_FotG_circle_script.zip` from the Releases page and unpack it
over your Knossos folder (back up the original first).

## Optional settings

In `fs2_open.ini`, section `[Default]`, with the game closed
(portable Knossos: `HardLightProductions\FreeSpaceOpen\fs2_open.ini`):

| Key | Default | Meaning |
|---|---|---|
| `VRHudResolutionScale` | `2` | HUD texture resolution factor, 1.0 - 4.0 (1 = engine default) |
| `VRHudScale` | - | HUD size factor, 0.2 - 1.5 (only affects mods whose HUD uses a global scale) |
| `VRHudDistance` | `1000` | Stereo depth of the HUD panel in metres (the panel covers a fixed angle, so this changes depth, not size) |

These keys have not been independently verified to be read when added by hand - please report if they don't work.

## Source code

[`patch/vr_hud_projection_fix.patch`](patch/vr_hud_projection_fix.patch) contains **all** changes against the official
release tag [`release_26_0_1`](https://github.com/scp-fs2open/fs2open.github.com/releases/tag/release_26_0_1)
(commit `4b06a04908238ed726c6c0d82e43de3f8604a411`) of
[scp-fs2open/fs2open.github.com](https://github.com/scp-fs2open/fs2open.github.com). It applies cleanly:

```
git clone https://github.com/scp-fs2open/fs2open.github.com.git
cd fs2open.github.com
git checkout release_26_0_1
git apply /path/to/vr_hud_projection_fix.patch
```

Build settings used for the release exe: Windows, MSVC 2022 (Build Tools), Release,
`-DFORCED_SIMD_INSTRUCTIONS=AVX -DFSO_USE_SPEECH=OFF -DFSO_USE_VOICEREC=OFF`.
The patch touches 18 files (about 580 added lines): the 3D projection and view setup, the HUD, and the OpenGL
render-target/state code.

## Feedback

Please open an issue with your headset, GPU, mod, and whether the HUD is steady, sharp and correctly placed
(including the cockpit monitors). Screenshots help.

## License

The changes in this repository are released under the [Unlicense](LICENSE) (public domain).
The original FreeSpace 2 source remains under Volition's notice ([NOTICE-Volition.md](NOTICE-Volition.md)):
you may not sell or otherwise commercially exploit it or things created from it. This fix is free and must stay free.
Provided "as is", without warranty.
