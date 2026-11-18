# Tiny Chess — a 4 KB chess engine

[![make.fun · tiny-chess-01](https://make.fun/badge/tiny-chess-01.svg)](https://make.fun/bounty/tiny-chess-01?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)
[![rank](https://make.fun/badge/tiny-chess-01/riandi21/tiny-chess-4k/rank.svg)](https://make.fun/bounty/tiny-chess-01/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)
[![reward](https://make.fun/badge/tiny-chess-01/riandi21/tiny-chess-4k/reward.svg)](https://make.fun/bounty/tiny-chess-01?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)

A chess engine in a single `main.py` (3,931 bytes), Python 3.13 standard
library only. Built for make.fun's tiny-chess-01: 8 games against the house
gauntlet through the env's HTTP protocol (`/game`, `/move`, `/resign`).

## How it works

- 0x88 board; FEN parsed each move; pseudo-legal move generation with a
  make/unmake legality filter.
- Negamax alpha-beta with iterative deepening to a fixed 100,000-node
  budget — deterministic: no clock, no randomness. Principal variation
  search: the first move searches the full window, later moves probe a
  null window first and re-search on fail-high.
- Transposition table (Zobrist hashing), MVV-LVA + killer-move ordering,
  quiescence over captures, promotions and check replies. Check
  extension: a position with the side to move in check searches one ply
  deeper. Late-move reductions apply to quiet non-pawn moves only —
  quiet pawn pushes are exempt, so their positional payoff (freed
  pieces, passed pawns) is searched at full depth.
- Evaluation: material (P100 N320 B330 R500 Q900) + full piece-square
  tables + bishop pair +30 + passed pawns + king safety.
- One-entry opening book: Alekhine Defence as White, the position after
  1.e4 Nf6 2.e5 Nd5 3.d4 d6 4.Nf3 Bg4 plays 5.c4. The house's depth-2/3/4
  engines answer ...Nb4 there, a blunder Stockfish rates +301..+304 for
  White; the line was found by playing Stockfish principal variations
  against the real house engine over all 40 gauntlet openings x 2 colours
  x 8 opponents (640 lines, the only winning line).
- Repetition rebuilt from the env's move list; mate scored as mate-in-N.

## Practice results

Local harness, v19 (PVS + Alekhine book + check extension + LMR pawn
exemption) full trial-0, 6 games vs the house gauntlet (depth-2 to
nodes-500k): 4 wins / 1 draw / 1 loss, local Elo 1,545, 357 plies to
spare — the strongest version so far (v11: 4/0/2, Elo 1,451 local, 445
plies to spare; v6: same 4/0/2, Elo 1,279 on the server grade). Wins by
checkmate vs depth-2 (115 plies), depth-3 (86), depth-4 (105) and
nodes-250k (137). Draw vs nodes-100k (Berlin, 200 plies, adjudicated).
The one loss is as Black vs nodes-500k (Philidor, 117 plies). Practice
floor is 1,154. Think time ~3.8 s/move mean, max ~6.7 s (limit 10 s).

## Determinism

The same position and history always give the same move. Replays are
byte-identical; perft verified (startpos 20/400/8,902; Kiwipete
48/2,039/97,862).

## Run

`python3 -P -s main.py` — stdlib only, reads no other file.

Original work, MIT licensed.
