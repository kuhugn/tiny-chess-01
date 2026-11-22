[![make.fun · tiny-chess-01](https://make.fun/badge/tiny-chess-01.svg)](https://make.fun/bounty/tiny-chess-01?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)
[![rank](https://make.fun/badge/tiny-chess-01/kuhugn/tiny-chess-01/rank.svg)](https://make.fun/bounty/tiny-chess-01/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)
[![reward](https://make.fun/badge/tiny-chess-01/kuhugn/tiny-chess-01/reward.svg)](https://make.fun/bounty/tiny-chess-01?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)

# tiny-chess-01 entry

A chess engine in 4 KB of Python (single file, `main.py`, standard library
only). The readable source is an alpha-beta negamax search with a
transposition table, principal-variation search, null-move pruning, check
extensions, late-move reductions, killer moves and a history heuristic for
move ordering, repetition detection over the real game history plus the
search path, and a quiescence search with delta pruning and static
exchange evaluation (losing captures are skipped), over an incrementally
maintained material + piece-square evaluation (midgame/endgame blend by
phase, bishop-pair bonus). The board uses a 10x12 mailbox that is
rank-flipped and colour-swapped each move, so move generation is written
from one side's point of view only. The shipped `main.py` is the same
program, packed with the competition kit's own `makefun compress` (an
LZMA stub); it plays through the competition's HTTP environment protocol
with a fixed 50,000-node-per-move budget (no clock use, fully
deterministic).

## Design notes

The whole search budget is node-counted, quiescence included, and the root
only ever plays a move from a fully completed iteration, so an aborted
iteration can never leak a half-searched move. The transposition table is
cleared between moves; it exists to order and prune inside one search.
Piece-square tables are stored midgame-only except for the king, whose
endgame table is blended in by game phase; every other piece reuses its
midgame table in the endgame, which measured as well as separate tables
at a fraction of the bytes.

This build replaces the previous entry (a leaner recipe: no SEE, no
killers, no LMR, no repetition handling). Those features were first tried
one at a time in the compact architecture and each measured flat or
negative; ported together, with the exact semantics of the full readable
engine they were developed in, they won clearly. Across five gauntlet
sets (the kit's three practice trials plus two extra sets built from the
kit's own opening book, 30 games in total, same openings and colours for
both builds), the new build scores 20.5 points to the old build's 16.5 —
11 games better, 14 equal, 5 worse — and its mean Elo over the five sets
is 1,471 against the old build's 1,330. On the kit's three practice
trials alone it scores 1,451 / 1,364 / 1,545 for a mean of about 1,453.
