# Chess

A chess game in a single HTML file. No build step, no dependencies, no network
requests — open `index.html` in a browser and play.

It ships with its own engine: alpha-beta search with a transposition table,
quiescence search and a tapered evaluation, running in a Web Worker so the board
stays responsive while the computer thinks.

## Playing

Open `index.html`, or host the file anywhere static (GitHub Pages works).

- **Move** by clicking a piece and then its destination, or by dragging it.
  Dragging works with a mouse, a finger or a pen.
- **Keyboard:** arrow keys move the cursor, <kbd>Enter</kbd> selects and moves,
  <kbd>Esc</kbd> cancels, <kbd>f</kbd> flips the board, <kbd>u</kbd> takes a move back.
- **Modes:** play the computer as either colour, or two players at one screen.
- **Difficulty:** ten levels. Level 1 plays like a beginner — shallow, and it
  drops a piece often enough to be beatable; level 10 searches three seconds a
  move and does not miss much.
- **Themes:** four boards (Walnut, Green, Slate, Ink) and four piece sets
  (Classic, Outline, Neo, Letters), mixed and matched freely and switchable
  mid-game. Your choice is remembered in the browser.

Undo, flip, resign, offer a draw, and import or export the game as FEN or PGN
from the buttons in the side panel. Every piece set is drawn locally — Unicode
glyphs or inline SVG, never a webfont or an image file.

The version number under the title tells you which build you are looking at.

### Levels

Three dials weaken the engine from the top down: how deep it looks, how far
below best it will settle (randomness), and how often it throws a move away
(blunder).

| Level | Depth | Time | Randomness | Blunder | ~Elo |
|---|---|---|---|---|---|
| 1 | 1 | 0.1s | 220cp | 22% | 800 |
| 2 | 1 | 0.15s | 170cp | 14% | 950 |
| 3 | 2 | 0.2s | 130cp | 8% | 1100 |
| 4 | 2 | 0.3s | 100cp | 4% | 1250 |
| 5 | 3 | 0.4s | 75cp | 1.5% | 1400 |
| 6 | 4 | 0.6s | 50cp | — | 1550 |
| 7 | 5 | 0.9s | 30cp | — | 1700 |
| 8 | 6 | 1.2s | 15cp | — | 1800 |
| 9 | 8 | 2s | — | — | 1950 |
| 10 | unlimited | 3s | — | — | 2100 |

**The Elo column is an estimate**, derived from the settings rather than
measured against rated opposition, and the levels have only been checked
against each other. Read them as a ladder, not as a rating. There is no opening
book, so the top levels are deterministic — the same position always gets the
same move.

## Rules

The full rules are implemented, including the ones games often skip: castling
(and the fact that you may not castle out of, through or into check), en
passant, underpromotion, and every way a game can be drawn — stalemate, the
fifty-move rule, threefold repetition and insufficient material.

Move generation is verified against the standard `perft` positions, including
4,865,609 nodes at depth 5 from the initial position and 4,085,603 at depth 4
from "kiwipete".

## Engine

| | |
|---|---|
| Board | flat 64-entry array; moves packed into a single integer |
| Search | negamax, alpha-beta, principal variation search, iterative deepening on a time budget |
| Pruning | null-move, late-move reductions, delta pruning in quiescence, mate-distance pruning |
| Ordering | transposition-table move, MVV-LVA captures, killers, history heuristic |
| Table | Zobrist hashing, two 32-bit halves — one keys the table, the other verifies the hit |
| Evaluation | material, piece-square tables tapered between middlegame and endgame, pawn structure, mobility, king safety, bishop pair, rooks on open files |

`make`/`unmake` mutate the position in place, so the search allocates nothing per
node.

## Accessibility

The board is a `role="grid"` with a labelled cell per square, so a screen reader
announces "e4, white pawn" and follows the keyboard cursor via
`aria-activedescendant`. Moves and results are announced in a live region. The
whole game is playable from the keyboard, and all animation is disabled under
`prefers-reduced-motion`.

## Code layout

Everything lives in `index.html`:

- **`ChessEngine()`** — the engine, as one self-contained factory. It closes over
  nothing outside itself, because the page builds its Web Worker by stringifying
  it into a Blob. Anything added inside it must keep that property or the worker
  will break.
- **The UI** below it — rendering, input, dialogs, and the search driver.

Chrome refuses to start a Blob worker from a `file://` page, so opening the file
directly falls back to a search that runs one deepening iteration per task and
lets the page repaint between them. Served over HTTP, the real worker is used.

## Licence

MIT — see [LICENSE](LICENSE).
