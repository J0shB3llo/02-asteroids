# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A clone of the arcade classic **Asteroids**, built in pure HTML5 Canvas — no dependencies, no bundler, no build step. The entire game lives in one file: `game.js`.

## Running / testing

There is no build, lint, or test tooling. To run the game:

```bash
npx serve .
```

Then open `http://localhost:3000`. Alternatively, open `index.html` directly in a browser. To verify a change works, reload the page and play — there is no automated test suite, so manual verification in a browser is the only way to confirm correctness.

## Architecture

Everything runs in `game.js`, loaded directly by `index.html` (no modules, no imports). The structure is a classic real-time game loop over a small set of entity classes, all rendered on a single `800×600` canvas (`W`/`H` constants).

- **Game loop**: `requestAnimationFrame(loop)` at the bottom drives `update(dt)` then `draw()` each frame, with `dt` clamped to 0.05s to avoid large jumps after tab-switch/lag.
- **Global mutable state**: `ship`, `bullets`, `asteroids`, `particles` arrays/objects plus `score`, `lives`, `level`, `state` (`'playing' | 'dead' | 'gameover'`) are module-level `let` bindings reassigned by `initGame()` / `nextLevel()` / `killShip()`. There's no state container object — any new feature that needs game state will likely add another top-level `let`.
- **Entities** (`Bullet`, `Asteroid`, `Ship`, `Particle`) each follow the same shape: constructor sets initial physics state, `update(dt)` advances position/timers and sets `this.dead = true` when expired, `draw()` renders via canvas 2D calls. Dead entities are swept with `.filter(e => !e.dead)` after each update pass — follow this pattern for any new entity type.
- **Toroidal space**: position updates wrap via the `wrap(v, max)` util so ships/asteroids/bullets that exit one edge reappear on the opposite edge. Any new moving entity should wrap its position the same way.
- **Collision detection**: plain O(n²) pairwise distance checks using the `dist(a, b)` util (bullets vs asteroids, ship vs asteroids) — no spatial partitioning, intentionally simple given entity counts.
- **Asteroid splitting**: `size` is 1–3 (small/medium/large); `RADII`, `SPEEDS`, `POINTS` arrays are indexed by size. `Asteroid.split()` produces two smaller asteroids on destruction (size 1 asteroids just disappear). Each asteroid's silhouette is a randomly generated irregular polygon (`verts`) computed once in the constructor, not regenerated per frame.
- **Input**: raw keydown/keyup listeners populate `keys` (held state) and `justPressed` (edge-triggered, consumed via `pressed(code)` — call sites must consume it, since it's cleared on read). Arrow keys + Space have their default browser scrolling prevented.
- **Game state transitions**: `killShip()` moves to `'dead'` (with `deadTimer` countdown) or `'gameover'` depending on remaining lives; `update()` branches on `state` at the top and returns early for the `'dead'`/`'gameover'` cases before running normal gameplay logic.

## Conventions

- Spanish is used for in-game text (HUD labels, game-over message) and code comments (section banner comments like `// ── Ship ───...`); keep new user-facing strings and section comments consistent with this.
- No semicolon-free style — standard semicolons and 2-space indentation throughout.
- Tunable gameplay constants (speeds, cooldowns, radii, drag) are declared as local `const` near their point of use (e.g. `ROT`, `THRUST`, `DRAG` inside `Ship.update`) rather than centralized — follow this pattern rather than introducing a config object.



