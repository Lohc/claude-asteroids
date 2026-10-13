# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Asteroids clone on a plain HTML5 canvas. No dependencies, no bundler, no build step, no tests, no linter. All game logic lives in `game.js`; `index.html` only hosts an 800×600 `<canvas id="canvas">` and loads the script. Comments and on-screen text are in Spanish — keep that convention.

## Running

Open `index.html` directly in a browser, or serve the folder:

```bash
npx serve .   # then visit http://localhost:3000
```

Verify changes by playing in the browser; there is no automated test harness.

## Architecture (`game.js`)

- **Main loop**: `loop(ts)` via `requestAnimationFrame` computes `dt` in seconds (clamped to 0.05) and calls `update(dt)` then `draw()`. All motion is time-based (px/s), so any new movement must be scaled by `dt`.
- **Global state**: module-level `let` variables (`ship`, `bullets`, `asteroids`, `particles`, `score`, `lives`, `level`, `state`, `deadTimer`). `initGame()` resets everything; `nextLevel()` resets ship/bullets/particles and spawns `3 + level` large asteroids away from the center (`SAFE_DIST`).
- **State machine**: `state` is `'playing' | 'dead' | 'gameover'`. `update()` branches on it: `'dead'` runs a respawn countdown (`deadTimer`) while asteroids keep moving; `'gameover'` waits for Space to call `initGame()`.
- **Entities**: `Bullet`, `Asteroid`, `Ship`, `Particle` classes each expose `update(dt)` and `draw()` and draw directly on the global `ctx`. Removal uses a `dead` flag; arrays are filtered after updates/collisions rather than spliced.
- **Toroidal space**: positions wrap with `wrap(v, max)` against `W`/`H` (particles are the exception — they don't wrap).
- **Asteroid sizes**: `size` 3/2/1 = large/medium/small, indexing the parallel arrays `RADII`, `SPEEDS`, `POINTS` (index 0 unused). `split()` returns two asteroids of `size - 1`, or none at size 1.
- **Collisions**: circle-distance checks with `dist()`. Bullet vs asteroid in `update()`; ship vs asteroid uses `ship.radius + a.radius * 0.82` and is skipped while `ship.invincible > 0` (3 s after each reset, shown as blinking).
- **Input**: `keys[e.code]` holds held keys (rotation/thrust); `pressed(code)` is a consume-once edge trigger built on `justPressed` (used for Space to shoot/restart). Shooting is also rate-limited by `ship.shootCooldown`.
