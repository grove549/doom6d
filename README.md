# Doom550D-6D

Canon EOS 6D port of [Doom550D](https://github.com/lauris-nl/doom550d), a
playable DOOM port for Magic Lantern.

This repository is a GitHub fork of the upstream project. The 6D port keeps
the upstream Doom engine, WAD loader, renderer, save system and software
synthesizer, and adds:

- Canon EOS 6D raw-key mapping
- 6D diagonal direction handling and the combined `UNPRESS_UDLR` event
- run toggle on the AF-point key, because the 6D has no zoom-out key
- quick-control wheel strafing and weapon selection
- module-level `Volume` menu for Canon ASIF audio output
- 6D build instructions and validation notes

The detailed Chinese port notes, key codes, test checklist and limitations
are in [README-6D.md](README-6D.md).

## Install

Copy the built module to the camera card:

```text
ML/MODULES/doom.mo
```

Put a Doom IWAD where the upstream module expects it. WAD files are not
included in this repository; use only data you legally own (for example
Freedoom).

The module must match the camera build. For a 6D.116 camera, use a
`doom.mo` built from this branch and the matching 6D.116 Magic Lantern
`autoexec.bin`/symbols. Do not install a 550D build on a 6D.

## Build

Place the `module/` directory in a Magic Lantern source tree:

```sh
cp -r doom550d-6d/module magiclantern/modules/doom550d
cd magiclantern
make -C modules/doom550d DOOM_6D=1 -j4
```

Output:

```text
modules/doom550d/build/doom.mo
```

`DOOM_6D=1` adds `-DDOOM_6D` and enables the 6D key map. Standalone module
builds do not necessarily define `CONFIG_6D`, so pass the flag explicitly.
A tree that already defines `CONFIG_6D` also enables it.

## 6D controls

| 6D control | Doom action |
| --- | --- |
| Arrow keys / multicontroller | Move forward/back, turn left/right; diagonals supported |
| `SET` | Fire; confirm in menus |
| `PLAY` | Open doors / use switches |
| Hold magnify (Zoom In) | Run |
| Tap AF-point key | Toggle run (the 6D has no zoom-out key) |
| Quick-control wheel left/right | Strafe left/right |
| Quick-control wheel up/down | Previous/next weapon |
| `INFO` | Automap; back in menus |
| `MENU` | Open/close the Doom menu (ignored for the first 5 seconds) |
| `TRASH` ×3 | Hidden cheat menu |
| `Q` | Exit Doom |
| Half/full shutter | Consumed by the module; not mapped to Doom actions |

If the wheel directions feel reversed in a particular firmware/build, swap
the `DOOM_BGMT_WHEEL_LEFT/RIGHT` and `DOOM_BGMT_WHEEL_UP/DOWN` handlers. If
the magnify key does not emit events in a shooting mode, use the AF-point
key to toggle run.

## Volume

Open `Games → Doom → Volume` and choose `1` to `ASIF_MAX_VOL` (10 on the
6D). The value is applied when Doom starts; exit with `Q` and restart to
change it.

Doom's own `F4` sound menu still controls the in-game SFX/music mix. The
module `Volume` controls the Canon ASIF output level.

## Compatibility

- Camera: Canon EOS 6D
- Magic Lantern: 6D.116 build
- Upstream engine: Doom550D (GPL-2.0)
- Display, palette, audio and FIO paths follow the upstream implementation;
  the 6D key map and volume path are the port-specific changes.

Known limitations are documented in [README-6D.md](README-6D.md), including
the relationship between the 6D magnify key and Canon digital zoom, the
AF-point key sharing, and the combined direction release event.

## Credits

- Upstream Doom550D: Bas Lichtjaar and contributors
  (https://github.com/lauris-nl/doom550d)
- Doom engine: id Software, GPL-2.0
- Magic Lantern project and contributors

## License

GPL-2.0, matching upstream. Doom engine and WAD data remain subject to their
own copyrights and licenses. No WAD or commercial game data is included.
