# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Vanilla JavaScript Tetris on HTML5 Canvas. No dependencies, no `package.json`, no build step, no tests, no linter. UI text and the README are in Spanish; keep user-facing strings in Spanish.

## Running

```bash
python3 -m http.server 8000   # then open http://localhost:8000
# or simply: xdg-open index.html
```

## Architecture

Three files: `index.html` (DOM: `#board` canvas, side panel with HUD and `#next-canvas`, `#overlay` for pause/game over), `style.css`, and `game.js` (all logic, plain script with `'use strict'`, no modules).

Key points in `game.js`:

- **State is module-level globals** (`board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, `dropInterval`, `animId`, …), all reset by `init()`, which is also the restart-button handler.
- **Board** is a `ROWS × COLS` matrix of `0` or a piece type index `1–7`. The same index is used in `PIECES` shapes and as the lookup into `COLORS`, so the cell value encodes the color. Index `0` of both arrays is `null`.
- **Pieces** are square matrices; `rotateCW` transposes and reverses. `tryRotate` applies simple horizontal wall kicks `[0, -1, 1, -2, 2]` (not SRS).
- **`collide(shape, ox, oy)`** is the single collision check used by movement, rotation, ghost projection (`ghostY`), and spawn. Cells with `y < 0` are allowed (above the board).
- **Lock flow**: `lockPiece()` → `merge()` → `clearLines()` (updates score/lines/level/`dropInterval`) → `spawn()`; if the newly spawned piece already collides, `endGame()`.
- **Game loop**: `requestAnimationFrame`-based `loop(ts)` accumulates `dt` into `dropAccum` and drops one row per `dropInterval` ms. Pause/game over work by `cancelAnimationFrame(animId)`.
- Speed: `dropInterval = max(100, 1000 − (level − 1) × 90)`; level = `floor(lines / 10) + 1`. Scoring: `LINE_SCORES[cleared] * level`, +1 per soft-drop row, +2 per hard-drop row.

## Gotcha

Canvas sizes are hard-coded in `index.html`. If you change `COLS`, `ROWS`, or `BLOCK` in `game.js`, update `#board`'s `width`/`height` to `COLS × BLOCK` by `ROWS × BLOCK`. `drawNext` assumes a 4×4 preview grid of 30px cells (120×120 `#next-canvas`).
