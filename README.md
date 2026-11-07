# Hookshot: the whole picture goes white at some sun angles (RX 9060 XT)

Evidence for issue [#36 — Severe overexposure / bloom issue](https://github.com/NotRexed/Hookshot/issues/36).

Not a Hookshot repository: this only holds captures and a log from one player's machine, so the author can see the
frame instead of a description of it.

## Machine and build

| | |
|---|---|
| GPU | AMD Radeon RX 9060 XT 16 GB, Vulkan 1.4, driver `0x8001a0` |
| OS | Windows 11 build 26300 (UBR 9711) |
| Build | HookshotBuilder build of 2026-11-07 ~20:50 local (`hookshot.exe` timestamp 20:58), predates #166 |
| Game | `zh-Hant`, Insert menu: frame generation 3x, frame pacing on, resolution Auto, upscaling Catmull-Rom, sharpness 60 |
| Where | outside Hyrule Castle, on the rock face, with the sun on the scene |

## What the frames are

- `renderer_*.png` — 1280×720, written by Hookshot itself (`RX_VK_SHOTS`), i.e. the frame the renderer produced.
  The number in the name is the game's present counter.
- `player_*.png` — the game window as it was on the monitor (`PrintWindow`), so the glow around the edges is the
  window frame, not the game.

| file | present | mean luma (8×8 grid) | |
|---|---|---|---|
| `renderer_normal_present_4200.png` | 4200 | 64.7 | scene visible |
| `renderer_white_present_1500.png` | 1500 | **249.6** | white |
| `renderer_white_present_2100.png` | 2100 | **251.6** | white |
| `renderer_white_present_2700.png` | 2700 | **251.6** | white |
| `renderer_bright_present_6600.png` | 6600 | 221.6 | part way to white |
| `player_normal.png` | — | 160.6 | what the player sees, normal |
| `player_white.png` | — | **245.9** | what the player sees, white (37 consecutive captures) |

The white stretch holds while the camera holds the angle (it is not a one-frame flash), and the renderer is healthy
inside it — same presents from `hookshot_log.txt`:

```
44280 presents, 35.6 fps; a frame: 895 draws (0 skipped), 112 dispatches, 106 passes, 144 barriers,
6 submits, ... GPU-bound wait 0.0 ms; 4855 pipelines (63198 ms), 81 programs (10 failed), 1315 textures (3817 MB)
GPU time 20.6 ms a frame
```

## The one number worth looking at

`hookshot_log.txt` was taken with `RX_VK_EXPOSURE_LOG=2`, which logs the final pass's exposure
(`c3+0x38` of fragment stage `73320e720a317817`) at every present:

```
vk: exposure 1325 1
...
vk: exposure 2100 1
...
vk: exposure 6823 1
```

2,621 samples between present 1325 and 6823, covering the white stretch at 1500–2700: **the value is 1.0 everywhere**.
So the number the final pass is given does not move when the picture blows out.

## Reproduce

Beside `hookshot.exe`:

```
set RX_VK_EXPOSURE_LOG=2
set RX_VK_SHOTS=<dir>
hookshot.exe
```

Then walk out of Hyrule Castle and turn until the sun is behind the scene.

Windows Error Reporting and the other sessions of this machine are in the gist linked from the issue comment.
