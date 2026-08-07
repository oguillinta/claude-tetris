# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

A classic Tetris implementation in vanilla JavaScript with HTML5 Canvas. No build process, no package manager, no external dependencies — three files (`index.html`, `style.css`, `game.js`) that run directly in the browser.

## Running the game

There is no build/lint/test tooling in this repo. To run it, just serve or open the static files:

```bash
start index.html        # Windows: open directly
python3 -m http.server 8000   # or any static file server, then visit localhost:8000
```

There are no automated tests. Verify changes by opening `index.html` in a browser and playing the game (use the `run` skill if available to launch and check it).

## Architecture

All game logic lives in `game.js` (~300 lines), organized around these pieces:

- **Board model**: a `ROWS × COLS` (20×10) matrix where each cell is `0` (empty) or a color index `1–7` identifying a locked piece.
- **Pieces**: defined as square matrices (tetrominoes). Rotation is computed via transpose + row-reverse in `rotateCW` (game.js:68).
- **Collision detection**: `collide(shape, ox, oy)` (game.js:55) checks board bounds and overlap with locked cells.
- **Wall kicks**: `tryRotate()` (game.js:77) attempts the rotation, then retries with ±1/±2 column offsets before giving up.
- **Game loop**: `loop(ts)` (game.js:243), driven by `requestAnimationFrame`, accumulates elapsed time and drops the piece one row once `dropInterval` is exceeded.
- **Line clearing**: `clearLines()` (game.js:96) scans bottom-to-top; full rows are removed and empty rows unshifted at the top.
- **Scoring**: classic table `[0, 100, 300, 500, 800]` multiplied by current level; hard drop adds 2 pts/cell, soft drop adds 1 pt/row.
- **Level/speed**: level increases every 10 cleared lines; drop speed is `max(100, 1000 - (level - 1) * 90)` ms.
- **Ghost piece**: `ghostY()` (game.js:115) projects where the current piece would land; drawn with `globalAlpha = 0.2`.

Control flow:

```
init()
  createBoard() → empty matrix
  next = randomPiece()
  spawn()        → moves `next` into `current`, generates a new `next`
  requestAnimationFrame(loop)

loop(timestamp)
  accumulate dt
  if dt >= dropInterval → drop the piece or lockPiece()
  draw()  (grid + board + ghost + current piece)
  requestAnimationFrame(loop)

keydown → move / rotate / soft-drop / hard-drop / pause
```

If a freshly spawned piece immediately collides (in `spawn()`, game.js:144), `endGame()` (game.js:221) fires and the Game Over overlay is shown.

## Tunable constants (game.js)

| Constant | Meaning | Default |
|---|---|---|
| `COLS` / `ROWS` | Board dimensions | `10` / `20` |
| `BLOCK` | Pixel size of each cell | `30` |
| `COLORS` | Piece color palette | 7 colors |
| `LINE_SCORES` | Points for 1–4 lines cleared | `[0,100,300,500,800]` |
| `dropInterval` | Initial fall speed (ms) | `1000` |

If you change `COLS`, `ROWS`, or `BLOCK`, also update the `width`/`height` attributes of `<canvas id="board">` in `index.html` to match (`COLS × BLOCK` and `ROWS × BLOCK`).
