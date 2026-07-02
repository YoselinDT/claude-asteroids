# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

A clone of the classic arcade game **Asteroids**, built with pure HTML5 Canvas and vanilla JavaScript (ES6+). No dependencies, no bundler, no build step, no package.json.

## Running the game

Open `index.html` directly in a browser, or serve it locally:

```bash
npx serve .
```

Then visit `http://localhost:3000`.

There is no lint, test, or build tooling in this repo — changes are verified by opening the game in a browser and playing it.

## Architecture

Everything lives in a single file, `game.js` (~420 lines), loaded directly by `index.html` via a `<script>` tag (no modules). `index.html` just sets up an 800×600 `<canvas>` and a black page background; all rendering happens in `game.js` via the 2D canvas context (`ctx`).

The file is organized top-to-bottom into sections (marked by `// ── Section ──` comments), each building on the last:

1. **Input** — `keys` tracks currently-held keys; `justPressed` tracks single-frame key-down edges (used for shooting/restart so holding the key doesn't repeat-fire). `pressed(code)` consumes a `justPressed` flag.
2. **Utils** — `wrap(v, max)` implements toroidal space (objects exiting one edge reappear on the opposite edge); `dist`, `rand`, `randInt` are shared helpers.
3. **Entity classes** — `Bullet`, `Asteroid`, `Ship`, `Particle`. Each has `update(dt)` and `draw()`, and sets `this.dead = true` when it should be removed. Asteroids are randomly-generated irregular polygons (`verts`) sized 1–3 (small/medium/large, see `RADII`/`SPEEDS`/`POINTS`); `Asteroid.split()` produces two smaller asteroids when a size-2/3 asteroid is destroyed.
4. **Game state** — module-level `let` variables (`ship`, `bullets`, `asteroids`, `particles`, `score`, `lives`, `level`, `state`, `deadTimer`) hold the entire game state; there is no state container object. `state` is one of `'playing' | 'dead' | 'gameover'`. `initGame()` resets everything for a new game; `nextLevel()` advances the level and respawns asteroids; `killShip()` handles a ship hit and transitions `state`.
5. **`update(dt)`** — the single per-frame update function, branching on `state`. Handles input-driven shooting, moving all entities, bullet↔asteroid collisions (splits asteroids, awards `POINTS`, spawns explosion particles), ship↔asteroid collisions (respects `ship.invincible`), and level-complete detection (`asteroids.length === 0` → `nextLevel()`).
6. **`draw()`** — clears the canvas and draws particles → asteroids → bullets → ship → HUD (score/level/lives), plus a `GAME OVER` overlay when applicable.
7. **Main loop** — `requestAnimationFrame`-driven `loop(ts)` computes `dt` in seconds (clamped to 0.05s to avoid large jumps after tab-switch), then calls `update(dt)` and `draw()`.

Collisions are simple circle-vs-circle checks using each entity's `radius` and `dist()`. All movement/physics constants (thrust, drag, rotation speed, bullet speed, asteroid speeds/radii/points) are defined as local constants near their point of use rather than centralized — check the relevant class/function when tuning gameplay feel.

Text in the UI (HUD labels, game-over message) is in Spanish; keep new user-facing strings consistent with that.
