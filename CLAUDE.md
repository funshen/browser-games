# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

A collection of self-contained browser games. Each game is a **single HTML file** with no build tools, no external dependencies, and no server required — open directly in a browser.

## Running Games

```bash
open shooter.html
open tictactoe.html
```

No build step, no `npm install`, no local server needed.

## Git Workflow

Always commit after meaningful changes and push to GitHub immediately:

```bash
git add <file>
git commit -m "descriptive message"
git push
```

Remote: `https://github.com/funshen/browser-games` (branch: `main`)

## Architecture: shooter.html

Single `<script>` block structured in 12 labeled sections (marked with `// ===` banners):

1. **CONSTANTS & CONFIG** — `W/H`, `STATE` enum, `ENEMY_TYPES`, `POWERUP_TYPES`, `SHOP_ITEMS`, hardcoded `LEVELS[]`
2. **UTILITIES** — `clamp`, `randFloat`, `randInt`
3. **PIXEL ART PRIMITIVES** — `drawPixelMap(ctx, grid, palette, scale)` renders 2D palette-index arrays; `SPRITES` object holds grids + palettes for PLAYER, GRUNT, TANK, SPEEDER
4. **ENTITY CLASSES** — `Player`, `Bullet`, `Enemy`, `Coin`, `Powerup`
5. **PARTICLE SYSTEM** — flat `particles[]` array; spawners: `spawnMuzzleFlash`, `spawnHitSparks`, `spawnExplosion`, `spawnCoinSparkle`
6. **WAVE / LEVEL SYSTEM** — `LEVELS[1-2]` hardcoded; `generateLevel(n)` for n≥3; `waveState` object tracks spawning; `updateWaveSpawner(dt)` drives it
7. **SHOP SYSTEM** — HTML overlay `#shopModal` (position:absolute over canvas); `renderShopItems()` rebuilds the grid after each purchase
8. **GAME STATE MACHINE** — `G` global (coins/level/score/upgrades); `restartGame()`, `startNextLevel()`, `startLevelupSplash()`
9. **INPUT** — `input.keys` + `input.mouse`; keydown handles ESC (pause toggle) and Space (shockwave)
10. **MAIN LOOP** — `requestAnimationFrame` loop, dt capped at 50ms; `updatePlaying(dt)` / `renderPlaying()`
11. **SCREEN RENDERS** — `renderTitle`, `renderLevelup`, `renderPauseOverlay`, `renderGameOver`, `drawHUD`, `drawCrosshair`
12. **BOOT** — `init()` called at end

### State Machine
```
TITLE → PLAYING ⇄ PAUSED
PLAYING → LEVELUP → PLAYING
PLAYING → (all waves done) → SHOP → LEVELUP → PLAYING
PLAYING → (hp ≤ 0) → GAMEOVER → TITLE
```

### Key Patterns

- **Backwards splice**: all mid-loop array removals use `for (let i = arr.length-1; i >= 0; i--)` then `arr.splice(i, 1)`
- **Contact damage**: `player.takeDamage(enemy.damage * dt)` — rate per second, not per frame
- **dt cap**: `Math.min((ts - lastTime) / 1000, 0.05)` prevents teleportation on tab resume
- **HUD drawn after `ctx.restore()`** so it never shakes with the game world
- **Shop z-index**: `#gameWrapper` is `position:relative`; `#shopModal` is `position:absolute` on top of canvas
- **Player upgrades**: `G.upgrades` object is the source of truth; `Player.applyUpgrades()` reads it on construction — a new `Player` instance is created at the start of each level

### Shockwave Ability
- `player.shockwaveCooldown` tracks recharge (8s); `SHOCKWAVE_RADIUS = 150`
- `shockwaveAnim` object at module level drives the expanding ring visual
- Instant-kills GRUNT/SPEEDER within radius; deals 80 damage to TANK

### Adding a New Enemy Type
1. Add config object to `ENEMY_TYPES`
2. Add pixel map + palette to `SPRITES`
3. Add movement behavior branch in `Enemy.update()`
4. Reference the type string in level/wave definitions
