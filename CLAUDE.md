# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

A classic Tetris implementation in vanilla JavaScript, HTML5 Canvas, and CSS. No dependencies, no build step, no package.json, no test suite — just three files (`index.html`, `style.css`, `game.js`).

## Running the game

There is nothing to install or build.

```bash
open index.html              # macOS, open directly
python3 -m http.server 8000  # or serve locally, then visit http://localhost:8000
npx serve .
php -S localhost:8000
```

There is no lint, build, or test command in this project.

## Architecture

Everything lives in `game.js` (~300 lines), a single flat script with module-level `let`/`const` state — no classes, no modules, no bundler. `index.html` just provides two `<canvas>` elements (`board` for the playfield, `next-canvas` for the preview) plus HUD/overlay DOM nodes that `game.js` reads and writes directly by id.

Key pieces of state and logic in `game.js`:

- **Board model**: `board` is a `ROWS × COLS` matrix; each cell is `0` (empty) or a piece-color index `1–7`.
- **Pieces**: `PIECES` defines each tetromino as a square matrix of color indices. `rotateCW` rotates by transposing + reversing rows.
- **Collision** (`collide`): checks board bounds and existing fixed blocks.
- **Wall kicks** (`tryRotate`): on rotation collision, retries at x offsets `[0, -1, 1, -2, 2]` before giving up.
- **Game loop** (`loop`): driven by `requestAnimationFrame`; accumulates elapsed time in `dropAccum` and advances the piece one row once `dropInterval` is exceeded.
- **Locking** (`lockPiece`): merges the current piece into `board`, clears completed lines, spawns the next piece.
- **Line clearing** (`clearLines`): scans bottom-to-top, splicing out full rows and unshifting empty ones at the top.
- **Scoring**: `LINE_SCORES = [0, 100, 300, 500, 800]` multiplied by `level`; hard drop adds 2 points per row dropped, soft drop adds 1 point per row.
- **Leveling**: level increases every 10 cleared lines; `dropInterval = max(100, 1000 - (level-1)*90)` ms.
- **Ghost piece** (`ghostY`): projects the current piece straight down to its landing row, drawn at `globalAlpha = 0.2`.

Control flow: `init()` builds the board, seeds `next`, calls `spawn()`, and kicks off the `requestAnimationFrame` loop. Keydown handling (`ArrowLeft/Right/Down/Up`, `KeyX`, `Space`, `KeyP`) mutates `current` directly and is ignored while `paused` or `gameOver`. If a freshly spawned piece immediately collides, `endGame()` fires and shows the Game Over overlay; `restartBtn` calls `init()` again.

## Tunable constants (in `game.js`)

`COLS`, `ROWS`, `BLOCK` (cell pixel size), `COLORS`, `LINE_SCORES`, `dropInterval`. If `COLS`/`ROWS`/`BLOCK` change, update the `width`/`height` attributes of `<canvas id="board">` in `index.html` to match (`COLS × BLOCK` and `ROWS × BLOCK`).
