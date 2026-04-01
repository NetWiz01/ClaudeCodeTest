# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running the Games

No build step. Open any HTML file directly in a browser:

```
start shooter.html
start tictactoe.html
```

Or double-click from Explorer. There are no dependencies, servers, or install steps.

## Git Workflow

**Commit and push after every meaningful unit of work** — a completed feature, a bug fix, a refactor. Do not batch multiple unrelated changes into one commit. This ensures work is never lost and any change can be reverted cleanly.

```bash
git add <specific-files>
git commit -m "concise present-tense summary

Optional body explaining why, not just what."
git push
```

- Stage specific files by name — never `git add -A` or `git add .`
- Commit messages: short subject line (≤72 chars), present tense, no period
- Always push immediately after committing — local-only commits defeat the purpose
- If `gh auth` causes a push failure, run `gh auth setup-git` then retry

Remote: `https://github.com/NetWiz01/ClaudeCodeTest` (branch: `main`)

## Project Structure

Two self-contained single-file HTML games. All HTML, CSS, and JS live inside each file — no external assets, no modules, no bundler.

### shooter.html — Pixel Assault (top-down shooter)

The entire game is one `<script>` block organized into sections (marked with `// ──` banners):

| Section | What it does |
|---|---|
| **CONSTANTS** | `CW/CH` (800×600), physics values (`PLAYER_SPEED`, `FIRE_RATE`, etc.), color palette object `C` |
| **INPUT** | `Keys` object (keydown/keyup map) and `Mouse` object (`x/y/down/clicked`). `Mouse.clicked` is a single-frame flag consumed at the end of each loop tick. |
| **SPRITE DRAW FUNCTIONS** | Procedural `ctx.fillRect`-based sprite renderers: `drawPlayerBody`, `drawGunRelative`, `drawRunnerSprite`, `drawTankSprite`, `drawZigzagSprite`, `drawMuzzleFlash`. All sprite functions draw centered at `(0,0)` — callers must `ctx.save/translate/restore`. |
| **PARTICLES** | Plain-object array `particles[]`. `spawnDeath/spawnMuzzle/spawnHit` add entries; capped at 250. Each particle has `{x,y,vx,vy,life,ml,sz,col}`. |
| **BULLET** | `class Bullet` — moves each frame, deactivates on out-of-bounds or age > 2s. |
| **PLAYER** | `class Player` — reads `Keys`/`Mouse` each frame, handles movement, aiming (`Math.atan2`), shooting (spawns `Bullet` + muzzle particles), invincibility frames (`invTimer`). |
| **ENEMY** | `class Enemy` (base) + `class ZigzagEnemy extends Enemy`. Enemy types: `runner` (fast, 1HP), `tank` (slow, 4HP), `zigzag` (sine-wave lateral movement, 2HP). Death animation: 0.35s scale+rotate+fade driven by `deathT`. |
| **WAVE/LEVEL SYSTEM** | `LEVELS[]` config array + `Wave` singleton object. `Wave.done()` = all enemies spawned, all killed, none alive. `Wave.next()` increments `lv` — called inside `initLevel()`, NOT directly on level-complete detection. |
| **COLLISION** | `overlap()` = squared-distance circle test. Bullets vs enemies, enemies vs player. Player gets knockback + 0.8s invincibility on hit. |
| **GAME STATE** | `state` string: `MENU → PLAYING → LEVEL_COMPLETE → GAME_OVER / VICTORY`. `initGame()` = full reset. `initLevel()` = partial reset (keeps score/HP, calls `Wave.next()`). |
| **SCREENS** | `drawMenu`, `drawWorld`, `drawHUD`, `drawLevelComplete`, `drawGameOver`, `drawVictory`. In-canvas buttons via `drawBtn()` which checks `Mouse.clicked` and bounds — no DOM event listeners on buttons. |
| **SCANLINES** | Pre-rendered to an offscreen canvas once at startup (`initScanlines`); `drawImage`'d every frame as the final layer. |
| **GAME LOOP** | `requestAnimationFrame` loop, `dt` capped at 50ms. `Mouse.clicked` reset at end of each tick. |

**Key invariant:** `Wave.lv` always holds the *current* level number while in `LEVEL_COMPLETE` state (it is not pre-incremented). `Wave.next()` / `initLevel()` advances it when the player proceeds.

**Mouse coordinates** use `getBoundingClientRect()` with scale correction so they stay accurate if the canvas is CSS-scaled.

### tictactoe.html — Tic Tac Toe

Single-file game with DOM-based UI (divs, not canvas). Minimax AI for the computer opponent. Score tracked in JS variables; game mode (2-player vs computer) toggled by a button.

## Adding a New Enemy Type to shooter.html

1. Add a `draw<Name>Sprite(ctx, walkFrame)` function in the SPRITE section
2. Add color entries to `C` at the top
3. Add a spawn probability branch in `mkEnemy(c)`
4. Update relevant `LEVELS[]` entries with the new type's probability field
5. Add a `spawnDeath` case for the new type's particle colors
