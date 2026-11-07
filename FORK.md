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

Two extra callbacks (neither is a window proc), both reported to the
web client as strings:

1. **`shim_monster_state`** — once per command cycle, the engine
   reports every monster the hero can see as a string of
   `mnum,x,y,band,flags` records:

   - **band** — a qualitative HP band (0 unhurt, 1 hurt, 2 wounded,
     3 near death), computed inside the engine in C. Exact hit points
     never cross into JavaScript; the game deliberately shows bands,
     not numbers.
   - **flags** — a status bitmask: 1 sleeping, 2 peaceful, 4 tame,
     8 stunned, 16 confused, 32 fleeing, 64 blinded.

   The web client uses this to render live condition bands and status
   badges in its portrait gallery.

2. **`shim_death_cause`** (Stage D) — once per actual death, the
   engine reports its own death attribution — the same record the
   tombstone and logfile are built from — as one pipe-separated
   string, `how|format|mnum|name`:

   - **how** — the `enum game_end_types` value; only true deaths
     (DIED through GENOCIDED) are reported, never quits, escapes, or
     ascensions. Averted deaths (life-saving, wizard refusal) do not
     fire it: the emitter sits at the top of `really_done`, which is
     reached only when the death actually happens.
   - **format** — the killer format code (KILLED_BY_AN / KILLED_BY /
     NO_KILLER_PREFIX).
   - **mnum** — the killing monster's species when the death came
     through `done_in_by`, else -1.
   - **name** — `killer.name` verbatim.

   The web client uses this to build the death screen: the killer's
   portrait plus the cause line for monster kills, the cause line
   alone otherwise.

## Files changed

- `win/shim/winshim.c` — both callbacks and their emitters.
- `include/extern.h` — the emitters' declarations.
- `src/allmain.c` — one `shim_monster_state` call per command cycle
  in `moveloop`, with the turn fully settled.
- `src/end.c` — the killing monster's species noted in `done_in_by`;
  the death-cause emitter called at the top of `really_done`.

Each changed file carries a "Portraits of the Doomed (v0.2)" notice
at the change site. Nothing else in the tree is modified; comments
aside, the patch is exactly these four files.

## Building

The engine builds with the upstream WASM pipeline (Emscripten);
see the `wasm-367` package tooling in the parent project. The
shipped game embeds the resulting `nethack.wasm`, built from this
branch.

## License

NetHack is distributed under the NetHack General Public License;
see the license files in this repository. This fork preserves all
upstream copyright and license notices.
