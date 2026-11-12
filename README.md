# tiny-chess-01 — karhutlah.ai

A small chess engine in a single `main.py` (4,087 bytes). It unpacks an
lzma-compressed payload and execs it; the payload and unpacker together are
the whole entry, standard library only, no other file read.

## Engine

- 0x88 board, full legal move generation (castling, en passant, promotion).
- Negamax with alpha-beta, principal-variation search, transposition table
  (exact / lower / upper flags), killer moves, MVV-LVA capture ordering.
- Quiescence search over captures and promotions (all evasions in check).
- Null-move pruning (R=2, guarded: not in check, no double null, side has
  non-pawn material). Check extensions via the quiescence search.
- Iterative deepening to a fixed node budget of 100,000 nodes per move.
- Evaluation: incremental material plus piece-square tables, blended
  endgame king centralisation. No bishop-pair bonus (cut for size).
- Fully deterministic: fixed node budget only, no clock, no randomness.

## Size

`main.py` is 4,087 bytes as committed (limit 4,096). The source is
hand-minified, then compressed with lzma (FORMAT_ALONE, preset 9) and
base85-encoded; the unpacker is two lines. Repo total is under 8,192 bytes.

## Practice results (local kit, no docker)

- Compressed final (this file, 4,069 bytes), full trials (16 games each):
  trial-0 Elo 1087 (3W/2D/11L), trial-1 Elo 1131 (4W/1D/11L),
  trial-2 Elo 1131 (4W/1D/11L). **Mean Elo 1116** across the three.
- Uncompressed v6 (same search, fuller eval), full trial-0: Elo 1326,
  7W/0D/9L — won all 6 vs depth-2/3/4, plus game 13 vs nodes-500k.
  Equals the sunfish baseline trial-0 (1326). The size cuts (simplified
  eval, one killer per ply) cost ~239 Elo on trial-0; see build notes.
- Determinism: the kit scorer reports "working 1" (replay reproduced
  the moves) on all three trials of the compressed final.
- Perft verified: start d1–d4 20/400/8902/197281, Kiwipete and positions
  3–6 correct; mate-in-1 (Qxf7) found.
