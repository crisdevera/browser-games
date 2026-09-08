# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A small collection of standalone, single-file HTML browser games/toys. There is no build system, package manager, server, or test suite — each `.html` file is fully self-contained (inline `<style>` and `<script>`, no bundler, no npm dependencies) and runs by opening it directly in a browser.

Current files:
- `tic-tac-toe.html` — two-player tic-tac-toe with a chalkboard visual theme (chalk-drawn X/O strokes, animated win-line strike-through). Score is persisted per-browser via `localStorage`.
- `sector-sweep.html` — a 2D top-down retro arcade shooter (Canvas 2D, `requestAnimationFrame` game loop). Arrow keys move, mouse aims/fires. Sprites are procedurally drawn pixel art (character-grid strings mapped through a palette, cached to offscreen canvases) rather than image assets, since there are no external art files in the repo.

## Running/testing

Just open the file in a browser (double-click, or `open tic-tac-toe.html` / `open sector-sweep.html` on macOS). No install or build step. If a browser blocks `file://` access to something (rare for these, since neither fetches external data), serve the directory instead: `python3 -m http.server` and visit `http://localhost:8000/<file>.html`.

There is no automated test suite — verification is manual, in-browser (play the game).

## Architecture notes

Each HTML file is a complete, independent artifact — there is no shared code between them and no reason to introduce cross-file imports or a build pipeline for two static pages. When editing one, everything relevant (markup, styles, game logic) lives in that one file.

Both files pull a Google Fonts stylesheet (`fonts.googleapis.com`) for display typography and otherwise have zero external runtime dependencies — no CDN-hosted JS libraries, no analytics, nothing fetched at runtime beyond that font CSS.

### `sector-sweep.html` internal structure

The single `<script>` block is organized into commented sections, in dependency order — keep new code in the matching section rather than scattering logic:
- **Constants/config** — arena size, speeds, `LEVELS` table, `ENEMY_DEFS`
- **Sprite pixel data** — `PALETTE` map + character-grid rows for the player, gun, and enemy sprites, built via a `row(char, count, ...)` helper (guarantees consistent row width — any new sprite row must sum to the same width as its siblings, or the sprite will render skewed)
- **Sprite renderer** — `buildSprite()` caches each unique frame to an offscreen canvas once; `drawSprite()` does the scaled/rotated `drawImage` each frame
- **Input handling** — keyboard state object, mouse-to-canvas coordinate conversion (`getCanvasPos`, accounts for DPR/CSS scaling so aim doesn't drift)
- **Entity factories** — plain-object constructors for player/enemy/bullet/particle (no classes)
- **Game state** — single `game` object holds all mutable session state; `setState()` centralizes which DOM overlay (menu/level-complete/game-over/victory/pause) is shown
- **Update logic** — movement, aim, shooting, enemy seek+separation steering, wave spawning (round-robins enemy spawn points across the four arena edges so enemies visibly approach from multiple angles), collision (circle-distance checks), particles
- **Render logic** — draws to the canvas each frame
- **UI wiring / canvas setup / main loop** — DOM button handlers, DPR-aware canvas sizing, the `requestAnimationFrame` loop

Menu, HUD, and end-of-round screens are DOM overlay `<div>`s toggled via the `hidden` attribute (not canvas-drawn text), positioned over the canvas.
