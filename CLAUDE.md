# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

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
- Speed: `max(100, 1000 - (level-1)*90)` ms, set in `clearLines`; level = `max(startLevel, floor(lines/10)+1)` (`startLevel` from the pause-menu `#start-level` select, read in `init()`).
- Changing `COLS`/`ROWS`/`BLOCK` requires updating the `<canvas id="board">` width/height in `index.html` manually.
- Pause (`P`/`Esc`) opens `#pause-menu` (separate from game-over `#overlay`): Reanudar / Reiniciar / Ver controles / Nivel inicial. While `paused`, keydown goes to `pauseMenuKey` only (game inputs ignored). `togglePause` restarts `loop` with a fresh `lastTime` to avoid a big `dt` jump; `endGame` cancels the rAF.

- Records: block near end of `game.js`; `localStorage` key `ftetris.records` (`{top[5], bestCombo, maxLines}`), all access in try/catch. `#start-screen` overlay gates the game (`started` flag, `startGame()`; initial `init()` loop is cancelled by `showStartScreen`); `endGame` → `showGameOverRecords()`; combo tracked via `trackCombo(cleared)` in `clearLines`. Keydown ignores input targets. `loop` does not re-queue after game over.

## Theme

Dark by default; `#theme-toggle` (panel) switches to light. Colors are CSS variables in `style.css` (`:root` dark, `:root[data-theme="light"]`). `applyTheme(theme, persist)` in `game.js` sets `data-theme`, caches `--grid`/`--highlight` into `gridColor`/`highlightColor` (canvas can't use CSS vars), saves to `localStorage` (`tetris-theme`, try/catch) and redraws (needed while paused/game over). It runs before `init()`; the toggle `blur()`s so Space doesn't activate it.
- Skins: `SKINS` (`retro|neon|pastel|pixel`) each with `colors[8]`, `bg`, `grid` (null = CSS/theme) and `drawBlock(ctx,px,py,color,size)`. `COLORS` is a `let` derived from the active skin by `applySkin` (persists in `localStorage` `ftetris.skin`, redraws board+next). `drawBlock(context,x,y,idx,size,alpha)` delegates to the skin; `paintBg` paints `skin.bg`.
