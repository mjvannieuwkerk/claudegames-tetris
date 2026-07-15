# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Two independent, functionally-equivalent implementations of Tetris that must be kept in sync:

- `python/tetris.py` — single-file Python/Pygame version.
- `web/` — vanilla HTML5 canvas/JS port (no build tooling, no dependencies).

Both share the same rules: 10x20 board, 7-bag randomizer, ghost piece, ten levels (0-9) with the same fall-speed curve, and the same scoring table. If you change game rules/behavior in one, mirror the change in the other unless told otherwise.

## Commands

Run the Python version:
```bash
python3 python/tetris.py
```
Install its only dependency: `pip install pygame`.

Run the web version (needs a static file server, not `file://`, since it fetches `tetris.js`/`style.css`):
```bash
python3 -m http.server 8123 --directory web
```
Then open `http://localhost:8123/index.html`.

There is no build step, linter, or test suite configured in this repo.

## Architecture

### `python/tetris.py`

Split into three parts:

- **Shape generation** (`_rotate_grid_cw`, `_build_rotations`): the `T`, `S`, `Z`, `J`, `L` tetrominoes are not hand-authored per rotation. Each is defined once as a 3x3 base grid and rotated 90° clockwise around the true matrix center to derive the other three states. `I` and `O` are hand-picked instead since they're symmetric enough that generated rotations wouldn't look right. If you need to tweak a shape, edit the base grid in `SHAPES`, not the generated rotations.
- **Game state** (`Piece`, `Tetris`): pure logic, no rendering or pygame calls. `Tetris` owns the board (`ROWS x COLS` grid of `None`/color), the current/next piece via a 7-bag randomizer (`_next_from_bag`), scoring, level/speed progression, and line clearing. Player actions (`move`, `rotate`, `soft_drop`, `hard_drop`) and the per-frame `update(dt_ms)` tick all funnel through `_collides`/`_lock_piece`. Rotation uses a small wall/floor kick table (`rotate`) that tries the natural position first, then nearby offsets, so rotating near a wall or the floor doesn't just fail.
- **Rendering** (`draw_cell`, `draw_board`, `draw_mini_piece`, `draw_side_panel`) and the **main loop** (`main`): straightforward pygame drawing and event handling, kept separate from `Tetris` so the game logic can be reasoned about independently of pygame.

### `web/tetris.js`

Mirrors the same three-part split as the Python version (shape generation → `Piece`/`Tetris` game state → canvas rendering/main loop), using the same function and field names translated to camelCase (e.g. `_lockPiece`, `ghostRow`). Notable differences from the Python version, both due to running in a browser rather than a native window:

- Controls are Arrow keys / Space / P / R only — there's no Esc-to-quit, since a browser tab can't reliably close itself.
- Key repeat for movement relies on native browser keydown repeat (guarded with `event.repeat` to stop rotate/hard-drop/pause from firing on repeat) instead of `pygame.key.set_repeat`.

Levels run 0-9 in both versions; level increases every 10 lines cleared and increases fall speed via `LEVEL_SPEEDS_MS`.
