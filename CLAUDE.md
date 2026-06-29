# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Vanilla JS Tetris. HTML5 Canvas + CSS. No dependencies, no build, no `package.json`, no tests. UI text is Spanish.

## Run

Open `index.html` directly, or serve statically: `python3 -m http.server 8000` / `npx serve .`. No build/lint/test commands exist.

## Architecture

Three files: `index.html` (DOM + two `<canvas>`), `style.css` (dark theme), `game.js` (all logic, ~300 lines, single IIFE-less global scope under `'use strict'`).

Key invariants in `game.js`:
- **Board model**: `ROWS × COLS` matrix; each cell is `0` (empty) or color index `1–7` matching `COLORS` / `PIECES` (both 1-indexed, index `0` is `null`).
- **Pieces** (`PIECES`): square matrices. Rotation = transpose + reverse rows (`rotateCW`); `tryRotate` applies wall kicks (`[0,-1,1,-2,2]` column offsets).
- **Collision** (`collide(shape, ox, oy)`): single source of truth for bounds + overlap; reused by movement, rotation, ghost, and lock detection.
- **Game loop** (`loop`): `requestAnimationFrame` + `dropAccum`/`dropInterval` accumulator. Pause cancels `animId`; resume resets `lastTime`.
- **Scoring**: `LINE_SCORES[cleared] * level`; soft drop +1/row, hard drop +2/cell.
- **Level/speed**: level = `floor(lines/10)+1`; `dropInterval = max(100, 1000 - (level-1)*90)`.
- State lives in module-level `let` vars (`board, current, next, score, ...`); `init()` resets all and is the restart entry point.

### Coupling to watch

Canvas pixel size is hard-coded in `index.html` (`board` = `300×600`). If you change `COLS`, `ROWS`, or `BLOCK` in `game.js`, update `width`/`height` to `COLS*BLOCK × ROWS*BLOCK` or rendering breaks. `game.js` queries DOM by fixed IDs (`board`, `next-canvas`, `score`, `lines`, `level`, `overlay`, `overlay-title`, `overlay-score`, `restart-btn`) — keep IDs in sync between the two files.
