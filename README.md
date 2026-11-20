# tiny-chess-01 entry

A chess engine in 4 KB of Python (single file, `main.py`, standard library
only). The readable source is an alpha-beta negamax search with a
transposition table, principal-variation search, null-move pruning, check
extensions, a history heuristic for quiet-move ordering, and a
capture-only quiescence search with delta pruning, over an
incrementally maintained material + piece-square evaluation (midgame/endgame
blend by phase, bishop-pair bonus). The board uses a 10x12 mailbox that is
rank-flipped and colour-swapped each move, so move generation is written
from one side's point of view only. The shipped `main.py` is the same
program, minified and packed with lzma + base85 under a small unpacker;
it plays through the competition's HTTP environment protocol with a fixed
50,000-node-per-move budget (no clock use, fully deterministic).

## Design notes

The whole search budget is node-counted, quiescence included, and the root
only ever plays a move from a fully completed iteration, so an aborted
iteration can never leak a half-searched move. The transposition table is
cleared between moves; it exists to order and prune inside one search.
Piece-square tables are stored midgame-only except for the king, whose
endgame table is blended in by game phase; every other piece reuses its
midgame table in the endgame, which measured as well as separate tables
at a fraction of the bytes.

Several textbook features were implemented, measured on the kit's own
practice gauntlet, and removed again when the numbers said no: repetition
detection inside the search, late-move reductions, killer moves, a more
aggressive null-move reduction, and a wider quiescence delta margin all
scored below the plain recipe above. Reverse futility pruning matched
the shipped build's headline score with wider swings, and combining it
with the history heuristic cancelled both gains out, so it stayed out
too. What survived is exactly what the measurements kept.

Measured on the kit's three practice trials (six games each, 50,000
nodes per move), the shipped build scores 1,364 / 1,451 / 1,279 Elo for
a mean of about 1,365, with ten and a half points from eighteen games:
the wins come against the fixed-depth anchors, the points dropped are
mostly draws the node-based anchors hold by repetition. Against the
same trials without the history heuristic the build gains half a point
net, changing only six of the eighteen game results.
