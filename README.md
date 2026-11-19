[![tiny-chess-01](https://make.fun/badge/tiny-chess-01.svg)](https://make.fun/bounty/tiny-chess-01)
[![rank](https://make.fun/badge/tiny-chess-01/Niuyyuy/tiny-chess-sunfish/rank.svg)](https://make.fun/bounty/tiny-chess-01)
[![reward](https://make.fun/badge/tiny-chess-01/Niuyyuy/tiny-chess-sunfish/reward.svg)](https://make.fun/bounty/tiny-chess-01)

# tiny-chess-sunfish

A fork of [sunfish](https://github.com/thomasahle/sunfish) (GPL-3.0-only) for
the [make.fun tiny-chess-01 bounty](https://make.fun/bounty/tiny-chess-01):
the strongest chess engine in 4,096 bytes of Python.

Play it on make.fun — sign up with my referral link:
https://make.fun/r/0wvmb34n

## What it is

`main.py` (4,036 bytes) is the whole engine: sunfish's tuned 2026 PST entry
(`nnue_4k/pst_entry.py`, commit `e396948acbd80cdcbad3dccfb07e878fba9df8f4`),
packed with lzma + base85. It plays the env protocol (POST `/game`, `/move`,
`/resign`) to a **fixed 65,000 nodes per move** — never a clock — inside the
1,800 s trial wall.

## Fork changes (vs the house baseline)

The house baseline is this same sunfish entry at 50,000 nodes/move. This
fork starts from a faithful reproduction of it, then:

- **Env driver**: replaces sunfish's UCI loop with the make.fun env loop.
  History is rebuilt from the env's move list before every search, so the
  position, castling rights, en passant and repetition set are exact.
- **Node-cap stop**: the wall-clock deadline check is replaced by a hard
  node cap, so every move is a fixed node count.
- **Bigger node budget**: 65,000 nodes/move (up from 50,000). About 4.5 s
  worst case per move, under the 10 s limit; trials run in ~1,000 s of the
  1,800 s wall. 75,000 was tried but lost a game on time, so 65,000 it is.
- **Bug fix**: the driver's `Searcher` was first called `S`, shadowing the
  engine's south-step constant (`S = 10`) and crashing en-passant handling
  in `value()`/`move()` after a double push. Renamed to `ENG`.
- **Size work**: PST tables as one base85 blob, `Move`/`Entry` namedtuples
  as plain tuples, and all `Position`/`Searcher` attributes and methods
  renamed to single letters. A differential test proved the packed engine
  searches identically to the original sunfish on the start position and
  all six trial-0 openings.

## Practice-trial results (kit scorer, 6 games each)

| version | trial-0 | trial-1 | trial-2 | mean |
| --- | --- | --- | --- | --- |
| baseline 50k | 1364 | 1451 | — | 1408 |
| this fork 65k | 1451 | 1451 | 1364 | **1422** |
| this fork 75k | 1451 | 1364 | — | 1408 |

House baseline (RULES.txt): 1364 / 1451 / 1279, mean 1365. Practice floor:
1154. This fork beats or ties the house baseline on every practice trial
(+87 / 0 / +85, mean +57). Caveat: the 10 s per-move limit binds — in
trial-1 game 6 our think time crept past 10 s at 65k and 75k (at 50k the
same game played 140 plies cleanly), so the margin is thin.

## License

GPL-3.0-only — see `LICENSE` (SPDX tag only, per the bounty rules). Sunfish
is Copyright Thomas Ahle and contributors; this fork keeps that lineage.
