# AGENTS.md

This file provides guidance to Codex (claude.ai/code) when working with code in this repository.

## Project

Vanilla JS Tetris (HTML5 Canvas). No dependencies, no `package.json`, no build, no tests, no linter. README is in Spanish; UI strings (`PAUSA`, `GAME OVER`, `Puntuación`) are Spanish too.

## Run

Open `index.html` directly, or serve statically: `python3 -m http.server 8000`.

## Architecture

Three files: `index.html` (DOM, canvases `#board` 300×600 and `#next-canvas`, `#overlay`), `style.css`, `game.js` (all logic, ~300 lines, single global scope, `'use strict'`).

- Game state lives in module-level `let` globals (`board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, ...) reset by `init()`. `restart-btn` re-calls `init()`.
- Board is `ROWS×COLS` matrix; cell = `0` or color/piece index 1–7 (indexes into `COLORS` and `PIECES`, which both have `null` at index 0). Piece shape cells hold their own type index.
- Flow: `loop` (rAF, accumulates `dropAccum` vs `dropInterval`) → `lockPiece` → `merge` → `clearLines` → `spawn` (calls `endGame` if new piece collides).
- `loop` gravity step and `softDrop` are separate code paths; score for soft drop (+1/row) and hard drop (+2/row) is added in the input handlers, not the loop.
- Rotation: `rotateCW` + `tryRotate` kicks `[0,-1,1,-2,2]` horizontally only.
- Speed: `max(100, 1000 - (level-1)*90)` ms, set in `clearLines`; level = `floor(lines/10)+1`.
- Changing `COLS`/`ROWS`/`BLOCK` requires updating the `<canvas id="board">` width/height in `index.html` manually.
- `togglePause` restarts `loop` with a fresh `lastTime` to avoid a big `dt` jump; `endGame` cancels the rAF.
