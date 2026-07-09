# ShadowDash — Development Notes

## Origin

ShadowDash was built as a single-page HTML/JS/CSS game from a blank folder — no starter template, no game engine. The brief was deliberately minimal ("basic HTML/JS/CSS game, single page"), so the design choices below were made to keep the whole thing self-contained, fast to load, and easy to read as one file, while still giving the game a distinct identity rather than being a generic endless runner.

The name "ShadowDash" suggested the core hook before any code was written: a **dash mechanic** that's not just a speed boost but a *required* way past certain obstacles. That idea became the Shadow Gate — an obstacle that can't be jumped or ducked, only dashed through — which gives the dash meter (and orb collection, which charges it) a real strategic role instead of being a cosmetic extra.

## Why a single file, no framework, no build step

- **Zero setup cost.** Anyone can open `index.html` directly or drop it into any static host with no `npm install`, no bundler, no transpilation.
- **Matches the scope.** A 2D endless runner doesn't need component frameworks, routing, or state management libraries — a single `requestAnimationFrame` loop over a plain object graph is simpler to reason about and debug than the equivalent React/Canvas wrapper would be.
- **Portability.** It deploys identically to Netlify, GitHub Pages, a USB stick, or `file://` — there's no server-side requirement at all.

The trade-off is that the file is long (~830 lines). It's organized in clearly commented sections (Canvas setup → Audio → Constants → State → Input → Spawning → Collision → Update → Draw → Main loop) specifically so it stays navigable despite living in one place. See [EXTENDING.md](EXTENDING.md) for where each concern lives.

## Architecture

### Rendering
A single `<canvas>` element rendered at a **fixed logical resolution of 900×420**, scaled to fit the container via CSS and corrected for device pixel ratio in `resizeCanvas()`. This keeps all game-object coordinates resolution-independent — physics and collision math never has to think about screen size, only the CSS layer does.

### Game loop
One `requestAnimationFrame` loop (`loop(ts)`) drives everything:
1. Compute `dt` (delta time in seconds), clamped to 0.05s to avoid physics blowing up after a tab was backgrounded.
2. If `state === 'playing'`, call `update(dt)` — this advances physics, spawns obstacles/orbs, moves everything, and resolves collisions.
3. Always call `draw()` — rendering runs even when paused/menu'd so the background (stars, mountains) keeps animating behind overlays.

State is a plain string machine: `'menu' | 'playing' | 'paused' | 'gameover'`, switched via `setState()`, which also toggles the visibility of the three HTML overlay `<div>`s layered on top of the canvas.

### Physics
Simple constant-gravity kinematics — no physics engine:
- `GRAVITY = 2200`, `JUMP_VEL = -780` (px/s²  and px/s, in logical canvas units)
- The player has one collidable state axis (`grounded`/airborne) plus a cosmetic `ducking` flag that shrinks the collision box (`playerBox()`) rather than moving the sprite.

### Collision
Plain axis-aligned bounding box (AABB) overlap tests (`rectsOverlap`) between the player's hitbox and each obstacle/orb rectangle. No spatial partitioning — object counts are low enough (a handful of obstacles/orbs on screen at once) that brute-force `O(n)` checks per frame are irrelevant to performance.

### Obstacles
Three types, chosen randomly on each spawn tick (`spawnObstacle()`) with fixed probability bands:
- `spike` (38% of spawns) — ground-level, jump required
- `wraith` (30%) — head-height, duck required
- `gate` (32%) — full-height Shadow Gate, dash required; this is the type that enforces the dash mechanic

Spawn cadence (`spawnInterval`) and world scroll `speed` both scale continuously with `distance` traveled, which is what produces the difficulty ramp — there's no discrete "level" system, just two formulas tightening over time.

### Dash system
The dash meter (`player.dashMeter`, 0–100) is the resource gating dashes:
- Dash costs 34%, requires the meter at or above that threshold
- Orbs grant +20% on pickup; passive regen is +3%/second when not dashing
- While `dashing` is true: movement speed multiplies by 2.4×, a fading trail of ghost sprites is recorded (`player.trail`), and `invuln` is set so the player passes through spikes/wraiths for free and can clear a Shadow Gate

### Audio
No audio files at all. `beep(freq, dur, type, vol)` creates a short-lived `OscillatorNode` → `GainNode` (with an exponential decay envelope) → `AudioContext.destination` for every sound effect. Each named effect (`sfx.jump`, `sfx.dash`, `sfx.hit`, etc.) is just a specific frequency/duration/waveform combination. The `AudioContext` is created lazily on first input (`tryStart()`) because browsers block audio until a user gesture.

### Persistence
Only one piece of state persists: the high score, in `localStorage` under the key `shadowdash_best`. No backend, no accounts, no server-side leaderboard.

### Visual layers (draw order, back to front)
sky gradient → twinkling stars → parallax mountain silhouettes → ground line → orbs → obstacles → particle bursts → dash trail ghosts → player sprite → HTML overlays (HUD, menus) sit on top via CSS, outside the canvas entirely.

## Tooling used to build and ship this

- Authored directly as a single `index.html` — no scaffolding tool.
- Verified locally with a plain Python static file server (`python -m http.server`) and a real browser; screenshots were taken to visually confirm rendering and gameplay before shipping (canvas games can't be fully verified by type-checking or unit tests alone — you have to look at it).
- Version-controlled with git, hosted on GitHub at [MuhammadTausif/shadowdash](https://github.com/MuhammadTausif/shadowdash).
- Deployed to Netlify via `netlify-cli` (`netlify deploy --prod`), currently live at https://shadowdash-651.netlify.app, with continuous deployment linked to the GitHub repo so every push to `master` redeploys automatically.

## Known constraints / non-goals

- No global leaderboard — high score is local to each browser.
- No build pipeline by design — if the project ever needs bundling (e.g. splitting into modules, adding a sprite atlas), that's a deliberate scope change, not an oversight.
- No automated test suite — the game is small enough that manual verification (play-testing in a real browser) is the practical check; see [EXTENDING.md](EXTENDING.md) if you're adding logic complex enough to warrant tests.
