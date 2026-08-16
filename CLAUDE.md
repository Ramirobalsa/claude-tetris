# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

A classic Tetris implementation in vanilla JavaScript with HTML5 Canvas. No dependencies, no build step, no framework — three files (`index.html`, `style.css`, `game.js`) that run directly in the browser.

## Running the game

There is no build/lint/test tooling (no `package.json`). To run:

```bash
# Just open the file
start index.html       # Windows

# Or serve locally (recommended for consistent behavior)
python3 -m http.server 8000
npx serve .
```

Then verify changes by opening the page in a browser and playing — there are no automated tests.

## Architecture

Everything lives in `game.js` (~300 lines), structured as a single global game-state object implicitly held in module-level `let` variables (`board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, `dropInterval`, etc.) rather than a class or state container.

**Board model**: a `ROWS × COLS` (20×10) matrix where each cell is `0` (empty) or an integer 1–7 identifying which piece color occupies it (see `COLORS`/`PIECES` arrays, index 0 unused as a sentinel).

**Pieces**: each of the 7 tetrominoes is a small square matrix in `PIECES`. Rotation is done generically via matrix transpose+reverse (`rotateCW`), not per-piece rotation tables. `tryRotate()` layers wall-kick behavior on top by attempting horizontal offsets `[0, -1, 1, -2, 2]` after rotating, keeping the first offset that doesn't collide.

**Game loop**: driven by `requestAnimationFrame` in `loop(ts)`. Elapsed time accumulates in `dropAccum`; once it exceeds `dropInterval` the current piece drops one row (or locks if it can't). `draw()` is called every frame regardless (grid → locked board → ghost piece → current piece, in that order — layering matters for correctness).

**Piece lifecycle**: `spawn()` promotes `next` to `current` and generates a new `next`; if the new `current` immediately collides, `endGame()` fires. `lockPiece()` calls `merge()` (bakes the piece into `board`) → `clearLines()` → `spawn()`.

**Scoring/leveling**: line-clear points come from `LINE_SCORES = [0, 100, 300, 500, 800]` multiplied by `level`; hard drop adds 2 pts/row, soft drop 1 pt/row. `level` increases every 10 lines, and `dropInterval` is recalculated as `max(100, 1000 - (level-1)*90)` each time.

**Rendering**: two canvases — `#board` (main play field) and `#next-canvas` (next-piece preview) — both use the same `drawBlock()` helper, just with different cell sizes (`BLOCK` vs the local `NB` in `drawNext()`). The ghost piece reuses `ghostY()` (projects `current` straight down until collision) and is drawn with `globalAlpha = 0.2`.

**Input**: a single `keydown` listener switches on `e.code`, gated by `paused`/`gameOver` flags (pause toggle itself bypasses the gate). There's no key-repeat throttling beyond the browser's native key-repeat behavior.

## Tunable constants (top of `game.js`)

`COLS`, `ROWS`, `BLOCK` control board dimensions/cell size — if changed, the `#board` canvas `width`/`height` in `index.html` must be updated to match (`COLS×BLOCK` and `ROWS×BLOCK`). `COLORS` and `LINE_SCORES` are the other easily-tunable arrays.
