<!--
# SPDX-FileCopyrightText: Copyright 2025 Eden Emulator Project
# SPDX-License-Identifier: GPL-3.0-or-later

# SPDX-FileCopyrightText: 2018 yuzu Emulator Project
# SPDX-License-Identifier: GPL-2.0-or-later
-->

<p align="center">
  <img src="dist/branding/lichii_icon_with_logo.png" alt="Lichii" width="600">
</p>

<p align="center">
  <a href="#elganflow">Elganflow</a> |
  <a href="#about-this-fork">Crowded-areas changes</a> |
  <a href="#lichii">Lichii</a> |
  <a href="#compatibility-and-issue-reports">Compatibility</a> |
  <a href="#building-and-contributing">Building</a> |
  <a href="#credits-and-license">License</a>
</p>

## Elganflow

**Elganflow** is a personal, unofficial fork of [Eglantine](https://github.com/Djihads80/Eglantine). The work is on the `elganflow` branch. Please report problems caused by its changes here, not upstream.

It merges [Lichii](https://github.com/zortos293/Lichii) (described at the end) for its game fixes, including The Legend of Zelda: Ocarina of Time, and builds on [KektJ's crowded-areas fork](https://github.com/KektJ/Eglantine) (described below). It adds:

- **Larger Vulkan descriptor payload:** `GUEST_FRAME_PAYLOAD_SIZE` goes from 512 KiB to 2 MiB. In Fire Emblem: Fortune's Weave's battles, the old size overflowed about 13 times a second, and each overflow made the emulator wait for the GPU. It costs about 300 MB more RAM.
- **Two experimental GPU split options, off by default:** `EDEN_NX2_WRITER_WAIT=auto` and `EDEN_NX2_ACQUIRE_PEEK=1` skip some waits between the GPU threads. They halve the GPU split drains but haven't raised the frame rate in the tests so far.
- **Name:** the window, menus and executable are called Elganflow (`elganflow.exe`). Settings, saves and shader caches stay in the same `eden` folder as Eglantine's.

Releases are Windows x64 builds made with MSVC. Use the Fire Emblem: Fortune's Weave settings from the section below: Texture array limit 64 and Run ahead of the GPU set to Relaxed.

## About this fork

These changes come from [KektJ's fork](https://github.com/KektJ/Eglantine), branch `perf/crowded-areas`, aimed at raising the frame rate in crowded areas of Fire Emblem: Fortune's Weave. The text below is KektJ's, and so are the builds and results it describes.

### Results

Measured in crowded areas on a Ryzen 7 5800X3D and RTX 4070 Ti at 2x resolution. The game targets 30 FPS.

| Area | Before | With both settings below |
|---|---|---|
| Crowded town | 17 FPS | at least 26 FPS |
| Heaviest crowd tested | about 9 FPS | 28-30 FPS |

These come from play testing in a limited part of the game on one PC, so other areas may behave differently. They were measured on the PGO build.

### Which build to download

| Build | Videos | Notes |
|---|---|---|
| **MSVC** | Play normally | The one to choose if you want the game's movies. It hasn't had the same fps comparison |
| **PGO** (clang, MinGW) | **Don't play.** The screen stays blank and the game skips them | Its FFmpeg package has no AV1 decoder, and this game's movies are AV1. The results above come from this build |

### What changed

- **Texture array limit setting:** shaders that pick a texture at runtime made the emulator bind up to 1024 textures per array on every draw. In these crowds, that was nearly half of the GPU back thread's busy time. A lower limit skips most of that work.
- **Run ahead of the GPU setting:** adds a **Relaxed** mode, which lets the game prepare its next frame while the GPU threads finish the current one, without the extra wait in the existing run-ahead mode.
- **Storage buffer sizing:** upstream sized every native compute storage buffer from its mapping, up to 8 MiB, which caused flickering lines on the world map and a slower title screen in this game. This fork uses upstream's narrower version from [#561](https://github.com/Djihads80/Eglantine/pull/561), limited to DLSS passes. Upstream later reverted that change in [#569](https://github.com/Djihads80/Eglantine/pull/569) without giving a reason, so it may affect other games.
- **Fire Emblem: Fortune's Weave 60 FPS setting:** keeps the game's own frame rate target at 60, which reached 43-46 FPS in the heaviest crowd tested and looked like normal speed. This hasn't been timed, and the game may run slower wherever it can't hold 60 FPS. It only works on the game version with build ID `DB93B0125F485060`.
- **Thread names on MinGW builds:** profilers and debuggers can now tell the emulator's threads apart.

### Using the settings

Both settings are on the advanced graphics tab and can be set per game. They are off by default, so the emulator behaves normally until you change them. For Fortune's Weave, set:

| Setting | Value | If something goes wrong |
|---|---|---|
| Texture array limit | 64 | Wrong textures: raise it to 128 or 256 |
| Run ahead of the GPU | Relaxed | Freezes, crashes or rubberbanding: use **Previous frame** or **Auto** |
| Fire Emblem: Fortune's Weave: 60 FPS (optional) | On | Slow motion: turn it off or lower the resolution. Don't use `EDEN_NX2_SWAP_INTERVAL` with it, or the game runs too fast |

Keep a backup save when using **Relaxed**. Running ahead can make a game overwrite data the GPU still needs, and upstream has seen freezes and crashes from this in other games. Neither setting has been tested with other games, so set them per game rather than globally.

The environment variables `EDEN_NX2_TEXTURE_ARRAY_CAP` and `EDEN_NX2_FRAMES_AHEAD` override these settings when set.

### Known issues

- Short hitches can still happen, with the worst single frames taking up to about 0.8 seconds.
- A torch's light in one cave disappears at a certain distance. This may be an older bug, but that hasn't been confirmed.
- Settings of 64 and Relaxed have only been tested in the parts of the game played so far.
- Videos don't play on the PGO build. Use the MSVC build if you want to see them.
- Real NVIDIA DLSS is optional and off by default. It needs NVIDIA's Streamline, which the Graphics tab can download. Without it, the game uses its own DLSS network at 1x and FSR 3.1 at higher resolutions.

### Credits

The 60 FPS setting relies on the frame rate target's location and behavior as documented by [seedds/nxnative](https://github.com/seedds/nxnative) and [weile10000/FEFW-60fps](https://github.com/weile10000/FEFW-60fps). The setting is this fork's own implementation and doesn't use their files or cheat code. Please don't report problems with it to them.

## Lichii

The rest of this page is Lichii's README.

Lichii is an experimental, open-source Nintendo Switch and Switch 2 emulator written in C++. It builds on Eglantine, Eden, yuzu and Sudachi, preserving their contributions while developing this fork's game compatibility and renderer fixes.

This is an independent fork, not an official release of those upstream projects. Lichii is not affiliated with Nintendo.

## Downloads and setup

Published builds belong on this fork's [releases page](https://github.com/zortos293/Eglantine/releases). The repository still uses its historical Eglantine name; the desktop application is now Lichii.

On Windows, extract the entire build archive and run **lichii.exe**. Only use games and system files you have legally obtained. Games, keys and firmware are not included.

### Existing saves and settings

The rebrand intentionally retains the existing Eden/Eglantine data directories and configuration keys. It does not move or delete saves, settings or shader caches. Back up your saves before testing experimental builds.

## Compatibility and issue reports

Compatibility and performance vary by game, hardware and driver. Rendering errors, stutters and crashes remain possible. Passing automated tests does not establish that a game renders correctly, and this project makes no universal frame-rate guarantee.

When [reporting an issue](https://github.com/zortos293/Eglantine/issues), include:

- The exact build, game version, CPU, GPU and driver version.
- Relevant settings and steps to reproduce the problem.
- A screenshot and the log from the affected session, after checking it for personal information.

Do not upload game files, keys, firmware or private account information.

## Building and contributing

See the [build guide](docs/Build.md), [user documentation](docs/user/README.md) and [coding guidelines](docs/Coding.md). Some inherited documentation and internal build target names still use upstream names. The GUI build target remains `yuzu`; its output is `lichii.exe` on Windows and `lichii` on other desktop platforms.

Keep changes focused and distinguish code-level checks from actual gameplay testing.

## Artwork
Art created by YOB (yobslab)
The supplied Lichii logo, icon and library artwork are in [dist/branding](dist/branding). Windows, Linux and macOS icon resources are generated from that artwork by [Generate-LichiiIcons.ps1](tools/Generate-LichiiIcons.ps1).

## Support Lichii

Lichii is free and stays free. If you want to help, you can donate in crypto. Donations help pay for the AI tools and build machines used to develop this open-source project faster.

**Solana (SOL)**
```text
JepEjfyWjGjhJvHSrqj52KtLjcbbbmUh1iGqerYBF4w
```

**Ethereum (ETH)**
```text
0x055ABa1b80Bc35C836Ce7E9F47c2b3d6C75b5534
```

**Bitcoin (BTC)**
```text
bc1qcvxdpf4qvdhqp8ee4s355cjmxrzsphsdwpl8w4
```

**BNB Smart Chain (BNB)**
```text
0x055ABa1b80Bc35C836Ce7E9F47c2b3d6C75b5534
```

**Polygon**
```text
0x055ABa1b80Bc35C836Ce7E9F47c2b3d6C75b5534
```

Send each coin on its matching network. Coins sent on the wrong network can't be recovered.

> [!NOTE]
> Donations are voluntary gifts and are non-refundable. A donation does not buy any product, service, perk, early access, exclusive build, priority support or say in the project. Every release stays free and public for everyone.

## Credits and license

Lichii inherits the work of the Eglantine, Eden, yuzu and Sudachi projects and their contributors, including Eglantine's original developer Camille LaVey. Original copyright notices and third-party licenses are retained.

See [LICENSE.txt](LICENSE.txt) and the source files for applicable license terms.
