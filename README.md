# Tiny Chess — a 4 KB chess engine

A chess engine in a single `main.py` (3,877 bytes), Python 3.13 standard
library only. Built for make.fun's tiny-chess-01: 8 games against the house
gauntlet through the env's HTTP protocol (`/game`, `/move`, `/resign`).

## How it works

- 0x88 board; FEN parsed each move; pseudo-legal move generation with a
  make/unmake legality filter.
- Negamax alpha-beta with iterative deepening to a fixed 100,000-node
  budget — deterministic: no clock, no randomness.
- Transposition table (Zobrist hashing), MVV-LVA + killer-move ordering,
  quiescence over captures, promotions and check replies.
- Evaluation: material (P100 N320 B330 R500 Q900) + full piece-square
  tables + bishop pair +30 + passed pawns + king safety.
- Repetition rebuilt from the env's move list; mate scored as mate-in-N.

## Practice results

Local harness, 3 practice trials vs the house gauntlet (depth-2 to
nodes-500k): Elo 1,279 / 997 / 1,545, mean ~1,274. Practice floor is 1,154.

## Determinism

The same position and history always give the same move. Replays are
byte-identical; perft verified (startpos 20/400/8,902; Kiwipete
48/2,039/97,862).

## Run

`python3 -P -s main.py` — stdlib only, reads no other file.

Original work, MIT licensed.
