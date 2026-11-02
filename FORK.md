# Fork notes — Portraits of the Doomed engine patch

This fork of [apowers313/NetHack](https://github.com/apowers313/NetHack)
carries the engine modifications for **Portraits of the Doomed:
A NetHack Chronicle**, a web chronicle built on the NetHack 3.6.7
WASM port. It exists so the modified engine's corresponding source
is public, as the NetHack General Public License expects.

## Branch and base

- Branch: `wasm-3.6.7`
- Base commit: `d1e3a5089aee31090906ac0ee4d0a4b58c0c28a4`
  (apowers313/NetHack, `wasm-3.6.7`, unmodified)
- The patch is the commit(s) on top of that base on this branch.

## What the patch does

One addition: the `shim_monster_state` callback. Once per command
cycle, the engine reports every monster the hero can see as a string
of `mnum,x,y,band,flags` records:

- **band** — a qualitative HP band (0 unhurt, 1 hurt, 2 wounded,
  3 near death), computed inside the engine in C. Exact hit points
  never cross into JavaScript; the game deliberately shows bands,
  not numbers.
- **flags** — a status bitmask: 1 sleeping, 2 peaceful, 4 tame,
  8 stunned, 16 confused, 32 fleeing, 64 blinded.

The web client uses this to render live condition bands and status
badges in its portrait gallery.

## Files changed

- `win/shim/winshim.c` — the callback and the emitter that walks
  the level's monster list.
- `include/extern.h` — the emitter's declaration.
- `src/allmain.c` — one call per command cycle in `moveloop`, with
  the turn fully settled.

Each changed file carries a "Portraits of the Doomed (v0.2)" notice
at the change site. Nothing else in the tree is modified; comments
aside, the patch is exactly these three files.

## Building

The engine builds with the upstream WASM pipeline (Emscripten);
see the `wasm-367` package tooling in the parent project. The
shipped game embeds the resulting `nethack.wasm`, built from this
branch.

## License

NetHack is distributed under the NetHack General Public License;
see the license files in this repository. This fork preserves all
upstream copyright and license notices.
