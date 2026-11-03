# Changelog

All notable changes to this repository. Dates are `YYYY-MM-DD`.

This file is not translated: it is a historical record that grows with each version, and
translating it would mean two histories that drift apart.

## [1.0.5] — 2026-11-03

### Fixed

- **The by-hand example passed the prefix of the first game, and the build did not link.**
  `tools/README.md` and the docstring of `tools/parche_uwp_build.py` both told you to run the step
  with `marioparty`. That prefix is what the entry point is renamed to, so it produced
  `marioparty_uwp_main`, while everything else here — `uwp\main.cpp` included — expects
  `marioparty3_uwp_main`. Running that step by hand therefore ended in the very
  `LNK2019: unresolved external symbol` that the same README explains three lines above. Both now
  say `marioparty3`.

  **Only the by-hand path was affected.** `compilar.ps1` already defaults to
  `-Prefijo marioparty3`, which is why every build made with it worked and the error never showed
  up here. The example tree in the docstring, `C:\mp\src`, now reads `C:\mp3\src` for the same
  reason: it was the first game's, copied over.

## [1.0.4] — 2026-11-03

**Caught up with the game's v0.2.13, which fixes the last two faults this port had open.** The
game went from v0.2.10 to v0.2.13 in a day: #55 in v0.2.11, #54 in v0.2.12, and a memory fix from
another user's report in v0.2.13.

### Changed

- **`tools/compilar.ps1`: the default `-Version` is now `v0.2.13`.**
- The recipe was **re-checked against v0.2.13, not assumed**: `tools/parche_uwp_build.py` was run
  over a clean v0.2.13 checkout and its three changes applied, leaving `CMakeLists.txt` and
  `src/main/main.cpp` **identical** to the tree this port is built from. No tool needed changing.
  The three library commits this family pins — `rt64` 5473732, `N64ModernRuntime` cdf5abb and
  `RecompFrontend` b2c5c18 — do not move between the two versions either, so worleydl's UWP
  patches and the four conflict resolutions land exactly as before.
- The upstream fault table: **#54 is fixed in v0.2.12** and **#55 in v0.2.11**. All six faults
  this port found while playing are now fixed upstream, and none of them was the port's. #55 was
  checked here by measuring the same scene with interpolation on, which is the condition for the
  fault: the zigzag on Peach's dress drops from 0.54 of the median to **0.009**, and the mean
  per-pixel difference between consecutive frames from 6.40 to **0.90**. #54 was checked by
  playing a turn change.
- `README.md` / `README.es.md`: built from v0.2.13, package `0.1.0.6`. The controller
  play-through on record is still the one from package `0.1.0.3`; the 0.1.0.6 package has been
  booted and measured, and the table says so.

### Added

- ⚠⚠ **`docs/HOW-IT-WORKS.md` trap 4 now carries its corollary: what the stamp keeps out never
  arrives.** v0.2.13's only change is bumping `lib/N64Recomp-browser`, and the fix it brings for
  [#62](https://github.com/n64hub/mario-party-3-recomp/issues/62) lives inside that submodule's
  `patches/N64ModernRuntime.patch` — the web patch this recipe stamps **without** applying. The
  version jump looks empty and is not. It makes no difference to the appx (the UWP patch already
  sets `allocation_size` to 512 MB), but it does to a desktop `.exe` built from the same tree.
  `tools/compilar.ps1` says the same beside its `Sellar` function.

### Worth knowing before you build it

Going from v0.2.10 to v0.2.13 is **not** a re-link. `us.toml` gains **seven hooks** and
`patches/widescreen.c` (+28), `patches/syms.ld` (+1), `src/game/camera_list.cpp` (+20) and
`CMakeLists.txt` (+2) all change, so the `recompilar` step has to run again before `pc`, `libs`
and `appx`.

## [1.0.3] — 2026-11-03

### Changed

- **This repository no longer explains itself by pointing at other repositories of this
  catalogue.** Comments and documentation introduced a decision, a trap or a default as "the same
  as in such-and-such other port of ours" — which asked anyone who came here for this game to go
  and read about a different one first. Each of those comparisons carried a fact that reads better
  stated outright, so that is what is written now.

  What is still cited is everything that is **not** ours: the upstream project, the third-party
  forks the recipe is built on, and the files taken from them under their own license.

**This release also carries 1.0.2**, which was written up in this file on 2026-11-02 but never
tagged: catching up with the game's v0.2.10, with the six anchors re-checked against it.

## [1.0.2] — 2026-11-02

**Caught up with the game's v0.2.10, and the recipe checked against it rather than assumed.**
The game moved from v0.2.7 to v0.2.10 in a day, touching four files this recipe cares about:
`CMakeLists.txt` (+7), `src/main/main.cpp` (+1/-1, the version string), `us.toml` (+33) and
`patches/widescreen.c` (+6/-3).

### Changed

- **`tools/compilar.ps1`: the default `-Version` is now `v0.2.10`.**
- The six anchors `tools/parche_uwp_build.py` matches on were re-checked against the upstream's
  clean files: all six still appear exactly once, on `v0.2.10` and on the current `main` alike.
  **No tool needed changing** — recorded here because it was verified, not assumed.
- `README.md` / `README.es.md`: the port is built from v0.2.10, package `0.1.0.5`. The
  controller play-through on record is still the one from package `0.1.0.3` (v0.2.7); 0.1.0.5 has
  been booted and measured but not played through, and the table now says so.
- The upstream fault table went from three entries to six. #48 and #49, listed as open, were
  fixed in v0.2.9 and v0.2.8; #52 was fixed in v0.2.10; #54 and #55 are new and still open.

### Noted, not fixed

- The game's `main` carries three commits that are in no release (PR #56, `model_ids.cpp`),
  which are the fix for #55. The recipe still builds from the `v0.2.10` tag.

### Housekeeping

- `tools/compilar.ps1` shows as 255 lines changed for a one-line edit. Its blob had been stored
  with CRLF; this commit stores it with LF, which is what `.gitattributes` has asked for since
  1.0.0 (`*.ps1 text eol=crlf` means LF in the repository and CRLF in the working tree). A clone
  still gets CRLF and the BOM, byte for byte as before — only the stored form changed.

## [1.0.1] — 2026-11-02

**A gap in the recipe, found by following it on a game it was not written for.** Taking
`n64hub/rush-2049-recomp` to the same console with these tools, everything applied — the same four
conflicts, resolved by the same script — and then the appx refused to link:

```text
main.obj : error LNK2019: unresolved external symbol recompui_uwp_data_path
```

### Added

- **`tools/parche_data_path.py`.** It defines `recompui_uwp_data_path` in RecompFrontend and makes
  `get_app_folder_path` return it before looking at `portable.txt`. That change is **in none of
  worleydl's patches** — it is this recipe's own, and until now it had travelled by copying
  already-patched files rather than as a tool. Anyone starting from a clean checkout would have hit
  the same wall at the very last step.
- `tools/compilar.ps1` calls it at the end of the `parches` step, so the default path just works.

### Changed

- `docs/HOW-IT-WORKS.md` trap 9 and `docs/INSTALL.md`'s symptom table now name the error and the
  tool that fixes it, in both languages. `tools/README.md` lists four tools instead of three.

## [1.0.0] — 2026-11-02

First release. The recipe for taking **Mario Party: Recompiled** to an Xbox Series X in
Developer Mode, as it was actually done on 2026-10-31.

### Added

- `tools/compilar.ps1` — the whole recipe in six steps (`rom`, `parches`, `recompilar`, `pc`,
  `libs`, `appx`), each repeatable on its own, with the reason for each decision beside it.
- `tools/parche_uwp_build.py` — the three changes the game itself needs, by exact anchors,
  idempotent.
- `tools/resolver_conflictos.py` — the four conflicts worleydl's UWP patches hit on this tree,
  and why all four resolve by keeping both sides.
- `tools/sellar.cmake` — re-stamps `N64Recomp-browser`'s web patches before every configure.
- `docs/HOW-IT-WORKS.md` — the two builds, the library commits this family pins, and the ten
  traps, each with the error it actually produces.
- `docs/INSTALL.md` — requirements, the commands, getting it onto the console, and a symptom
  table.
- `NOTICE.md` — what is not distributed, and the 65 lines of other people's code this repository
  quotes, measured and attributed one by one.
- Spanish versions of the README, the notice and the docs.

### Measured on the console

Xbox Series X in Developer Mode, package 0.1.0.4, on 2026-10-31: boots, draws, takes controller
input, validates the ROM from `LocalState` on its own, and runs the intro complete at 1920×1080
and **60 FPS**. Not yet played through with a controller in hand.

### Known not to be covered

- The `uwp/` wrapper project is not here: it derives from worleydl's forks.
- worleydl's `.patch` files are not here either; the recipe applies them from his repositories.
- A Series S has never been tried.
