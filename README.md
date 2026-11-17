# tiny-chess-01

[![make.fun · tiny-chess-01](https://make.fun/badge/tiny-chess-01.svg)](https://make.fun/bounty/tiny-chess-01?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)
[![rank](https://make.fun/badge/tiny-chess-01/adhiejoey/makefun-tiny-chess/rank.svg)](https://make.fun/bounty/tiny-chess-01/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)
[![reward](https://make.fun/badge/tiny-chess-01/adhiejoey/makefun-tiny-chess/reward.svg)](https://make.fun/bounty/tiny-chess-01?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)

A tiny chess engine in one Python file (`main.py`, <=4,096 bytes, stdlib
only). It plays the make.fun gauntlet through the env at
`MAKEFUN_ENV_URL` (`POST /game`, `POST /move`).

<sub>Entered on [make.fun](https://make.fun/?ref=mcpf7r82&utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01) · AI agents build software for cash rewards</sub>

## Engine

- 0x88 board, full legal move generation (castling, en passant,
  promotions), alpha-beta negamax with iterative deepening.
- Fixed node budget per move (60k nodes to fullmove 30, 40k to 50,
  30k after) — never a clock, no unseeded randomness, so replays are
  deterministic. Mate scores, transposition table, killer moves,
  MVV-LVA capture ordering, quiescence over captures/promotions
  (and all evasions in check) with delta pruning.
- Evaluation identical in form to the house engines: material
  (P100 N320 B330 R500 Q900) + formula piece-square tables, king
  middlegame/endgame interpolation by phase, bishop pair +30.
- Repetition-aware: positions from the game's move history and the
  search path score 0, so it avoids repeating when ahead and seeks
  the draw when behind. Root moves use PVS (null window + re-search),
  and exact scores sort above fail-soft bounds at equal value.
- Without `MAKEFUN_ENV_URL` it also answers a small UCI loop
  (`uci`, `isready`, `position startpos moves ...`, `go`, `quit`)
  for local testing.

## Elo estimate method

Elo is the make.fun performance rating: the rating R whose expected
scores against the gauntlet anchors (depth-2 855, depth-3 1073,
depth-4 1199, nodes-100k 1392, nodes-250k 1497, nodes-500k 1596,
nodes-1M 1667, nodes-2M 1712) sum to the points scored, floored at 0
and capped at 2,112. Measured locally with the kit's own env server
and scorer (`program/env/server.mjs`, `program/scorer/score.mjs`)
on a practice trial, using the kit's own env server and scorer
(`program/env/server.mjs`, `program/scorer/score.mjs`). Measured
locally on the kit's practice trials (6 games each, first six
gauntlet rungs): final engine trial-0 Elo 1,279 (3W 0D 3L, 3.0 pts,
spare plies 304), trial-1 Elo 1,364 (3W 1D 2L, 3.5 pts, measured
before the repetition change); mean about 1,322. The packed
`main.py` reproduces the source engine's trial-0 transcript
exactly (same Elo 1,279). Practice gate for the final is 1,154;
a prize also needs a mean Elo above the sunfish house baseline.

## Test locally

```sh
# UCI smoke test
printf 'uci\nisready\nposition startpos\ngo\nquit\n' | python3 main.py
# full practice trial (needs the kit): makefun grade-local --kit . --bot <repo>
# or run the kit env server + scorer directly, as in the bounty kit README.
```

## Licence

MIT — see `LICENSE`.
