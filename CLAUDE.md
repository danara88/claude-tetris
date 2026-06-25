# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running the game

No build step. Open directly or serve locally:

```bash
open index.html                  # macOS direct open
python3 -m http.server 8000      # then visit http://localhost:8000
```

## Architecture

Single-file game logic in `game.js` (~300 lines), no dependencies, no bundler.

**Board model**: `board` is a `ROWS × COLS` (20×10) matrix; `0` = empty, `1–7` = piece color index.

**Piece object**: `{ type, shape, x, y }` where `shape` is a 2D matrix of color indices.

**Key functions and their roles**:
- `collide(shape, ox, oy)` — boundary + overlap check; used before every move/rotation
- `tryRotate()` — applies `rotateCW` then tries wall kicks `[0, -1, 1, -2, 2]` offsets
- `lockPiece()` → `merge()` + `clearLines()` + `spawn()` — the piece-settling chain
- `ghostY()` — projects piece straight down to find landing row; used for ghost rendering
- `loop(ts)` — `requestAnimationFrame` loop; accumulates `dropAccum` against `dropInterval`
- `init()` — full reset, call to restart

**Speed formula**: `dropInterval = Math.max(100, 1000 − (level − 1) × 90)` ms; level increments every 10 lines.

**Canvas layout**: `#board` canvas is `300×600` px (10×20 blocks at 30 px each); `#next-canvas` is `120×120` px for piece preview.

## Tunable constants (top of `game.js`)

| Constant | Default | Note |
|---|---|---|
| `COLS` / `ROWS` | 10 / 20 | Changing these requires matching canvas `width`/`height` in `index.html` |
| `BLOCK` | 30 | Pixel size per cell |
| `COLORS` | 7 colors | Index-matched to `PIECES` |
| `LINE_SCORES` | `[0,100,300,500,800]` | Multiplied by current level |
