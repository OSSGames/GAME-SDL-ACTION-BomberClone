# BomberClone for ArcaOS / OS/2

BomberClone is an open-source clone of the classic Bomberman game. This is the
ArcaOS / OS/2 port based on upstream version 0.11.9.1.

## Version

**0.11.9.1 – ArcaOS Release 1**

## License

GNU GPL v2 — see `COPYING` for the full text.

## Requirements

- ArcaOS 5.x (or OS/2 Warp 4.5 with the appropriate runtime libraries)
- SDL2
- SDL2\_image (with PNG support)
- SDL2\_mixer

## How to Build

Use the provided OS/2 makefile with GCC / EMX:

```
make -f makefile.os2
```

The build produces `bomberclone.exe`.  All source files are compiled with
`-O2 -Wall` against the SDL2 headers found under `/@unixroot/usr/include/SDL2`.

## How to Run

```
bomberclone.exe
```

Game data must be located in the `data/` subdirectory next to the executable.

### Keyboard shortcuts

| Key         | Action                        |
|-------------|-------------------------------|
| Arrow keys  | Move                          |
| Space       | Drop bomb                     |
| F8          | Toggle fullscreen             |
| Alt+Enter   | Toggle fullscreen             |
| Ctrl+X      | Quit                          |

## Known Issues

### Music does not play

The ArcaOS SDL2\_mixer package currently ships with only the `CMD` and `WAVE`
decoders.  BomberClone's music files (`data/music/`) are in XM and MOD tracker
format, which require the XMP, MIKMOD, or MODPLUG decoder — none of which are
included.  As a result, background music is silent.

**Workaround:** convert the XM/MOD files to WAV using
[OpenMPT](https://openmpt.org/) (free, cross-platform):

1. Open each file in OpenMPT.
2. File → Export → Wave (22 050 Hz, 16-bit, stereo).
3. Replace the originals in `data/music/` with the exported `.wav` files.

The game will then play them through the WAVE decoder.  Sound effects are
unaffected; they are already in WAV format.

### Network

- Starting a LAN network game requires the **Notify Gamemaster** option to be
  disabled.
- The game uses UDP port 11000 by default. Make sure this port is open if
  playing over a network.

## ArcaOS Port — Change Log

### Release 1 (2026-08-18) — upstream 0.11.9.1

#### SDL1 → SDL2 Migration

- Replaced `SDL_SetVideoMode` with `SDL_CreateWindow` + `SDL_GetWindowSurface`.
- Updated `SDL_Init` flags to SDL2 equivalents.
- Replaced deprecated SDL1 surface, event, and timer APIs throughout.
- Updated `SDL_mixer` calls to the SDL2\_mixer API.
- Updated `SDL_image` calls to the SDL2\_image API.
- Keyboard input migrated from `SDL_GetKeyState` to `SDL_GetKeyboardState`
  with `SDL_Scancode` indices (replaces `SDLK_*` index arithmetic).
- Added `makefile.os2` for building with GCC / EMX on ArcaOS.

#### Fullscreen Scaling Fix

- Introduced a fixed-size **virtual back buffer** (`gfx.screen`) allocated with
  `SDL_CreateRGBSurface` at the game's native resolution.  All game rendering
  targets this surface unchanged.
- Added `gfx_present()`: blits the virtual buffer to the real window surface
  with aspect-ratio-preserving `SDL_BlitScaled` when the window is larger than
  the native resolution (fullscreen), or a plain `SDL_BlitSurface` in windowed
  mode, then calls `SDL_UpdateWindowSurface`.  This is the same pattern used by
  the Abe's Amazing Adventure SDL2 port.
- `gfx_blitupdaterectdraw()` now calls `gfx_present()` instead of
  `SDL_UpdateWindowSurfaceRects`.
- All scattered `SDL_UpdateWindowSurface(gfx.window)` calls in `game.c`,
  `mapmenu.c`, `configuration.c`, `multiwait.c`, and `network.c` replaced with
  `gfx_present()`.
- Fullscreen toggle sites (`sysfunc.c` Alt+Enter, `game.c` F8 / BCK\_fullscreen,
  `multiwait.c` BCK\_fullscreen): removed the stale
  `gfx.screen = SDL_GetWindowSurface()` re-assignment that overwrote the virtual
  buffer pointer; added an immediate `gfx_present()` call so the screen repaints
  at the moment of the toggle rather than on the next game-loop frame.

#### Compiler Warning Fixes (GCC 9.2 `-Wall`)

- **`extern inline` in headers** (`sysfunc.h`, `gfx.h`, `menu.h`, `single.h`,
  `player.h`, `bomb.h`): removed `inline` from forward declarations to silence
  "declared but never defined" warnings.
- **`-Wunused-but-set-variable`**: removed dead local variables and their
  assignments in `bomb.c` (`dist`), `gfx.c` (`ssfkt`), `mapmenu.c` (`x`),
  `netsrvlist.c` (`srvlst_entry`), `player.c` (`xs`, `coll_speed`, `oldm`,
  `oldd`), `single.c` (`aiplayer`), `tileset.c` (`sfkt`).
- **`-Wrestrict` / self-aliasing `sprintf`**: fixed cases where source and
  destination buffers aliased in `debug.c`, `netsrvlist.c`, and
  `ogcache-client.c` using the pointer-offset pattern or a temporary buffer.
- **`-Wformat-overflow`**: converted `sprintf` to `snprintf` with correct buffer
  size in `configuration.c`, `font.c`, `gfx.c`, `map.c`, `mapmenu.c`,
  `sound.c`, `tileset.c`, and others.
- **`-Wstringop-truncation`**: replaced `strncpy` with `snprintf` in
  `broadcast.c`, `configuration.c`, `menu*.c` (6 files), `ogcache-client.c`,
  `packets.c`, `sysfunc.c`, `tileset.c`, `udp.c`.
- **`-Wmisleading-indentation`**: added explicit braces to ambiguous `if` / `for`
  bodies in `chat.c`, `field.c`, `gfxpixelimage.c`, `map.c`, `mapmenu.c`,
  `netmenu.c`, `packets.c`, `player.c`, `single.c`.
- **`memcpy` on overlapping memory** (`chat.c`): replaced with `memmove`.
- **Removed unused mask variables** in `gfxpixelimage.c` (`rmask`, `gmask`,
  `bmask`, `amask`) along with the dead `#if SDL_BYTEORDER` assignment block.

## Authors

- Original game: Steffen Pohle — steffen@bomberclone.de
- ArcaOS / OS/2 port: Andrey Vasilkin

## Links

- Upstream project: https://www.bomberclone.de/
- Source: http://sourceforge.net/projects/bomberclone
- ArcaOS Port: https://github.com/OS2World/GAME-SDL-ACTION-BomberClone
