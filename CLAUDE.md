# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

A single-file Python/Pygame implementation of Tetris. The `web/` directory exists but is currently empty (no web implementation yet).

## Commands

Run the game:
```bash
python3 python/tetris.py
```

Install the only dependency:
```bash
pip install pygame
```

There is no build step, linter, or test suite configured in this repo.

## Architecture

Everything lives in `python/tetris.py`, split into three parts:

- **Shape generation** (`_rotate_grid_cw`, `_build_rotations`): the `T`, `S`, `Z`, `J`, `L` tetrominoes are not hand-authored per rotation. Each is defined once as a 3x3 base grid and rotated 90° clockwise around the true matrix center to derive the other three states. `I` and `O` are hand-picked instead since they're symmetric enough that generated rotations wouldn't look right. If you need to tweak a shape, edit the base grid in `SHAPES`, not the generated rotations.
- **Game state** (`Piece`, `Tetris`): pure logic, no rendering or pygame calls. `Tetris` owns the board (`ROWS x COLS` grid of `None`/color), the current/next piece via a 7-bag randomizer (`_next_from_bag`), scoring, level/speed progression, and line clearing. Player actions (`move`, `rotate`, `soft_drop`, `hard_drop`) and the per-frame `update(dt_ms)` tick all funnel through `_collides`/`_lock_piece`. Rotation uses a small wall/floor kick table (`rotate`) that tries the natural position first, then nearby offsets, so rotating near a wall or the floor doesn't just fail.
- **Rendering** (`draw_cell`, `draw_board`, `draw_mini_piece`, `draw_side_panel`) and the **main loop** (`main`): straightforward pygame drawing and event handling, kept separate from `Tetris` so the game logic can be reasoned about independently of pygame.

Levels run 0-9; level increases every 10 lines cleared and increases fall speed via `LEVEL_SPEEDS_MS`.
