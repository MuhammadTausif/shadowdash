# ShadowDash — Extending the Game

Everything lives in one file, [`index.html`](../index.html), inside a single `<script>` IIFE. This doc points to exactly where to make common changes. Section headers below match the `// ---------- Section ----------` comments in the source, so search for them.

## Map of the file

| Section | What it owns |
|---|---|
| Canvas setup | The fixed 900×420 logical resolution and DPI scaling |
| Audio | `beep()` primitive and the `sfx` object (all sound effects) |
| Game constants | Tunable numbers: gravity, jump strength, speeds, dash cost/regen |
| State | `player` object, obstacle/orb/particle arrays, score/best |
| Input | Keyboard + touch event wiring, and the `doJump`/`doDuckStart`/`doDash` action functions |
| Spawning | `spawnObstacle()`, `spawnOrb()`, `spawnDust()`, `spawnBurst()` |
| Collision | `playerBox()`, `rectsOverlap()`, `hitPlayer()`, `endGame()` |
| Update | `update(dt)` — the per-frame simulation step |
| Draw | `draw()` and `drawPlayerShape()` — everything rendered to canvas |
| Main loop | `loop(ts)` — the `requestAnimationFrame` driver |

## Recipe: add a new obstacle type

1. **Spawn it.** In `spawnObstacle()`, add a new probability band and push an object with a new `type` string, plus whatever fields you need (position, size, animation phase):
   ```js
   function spawnObstacle(){
     const roll = Math.random();
     let type;
     if(roll < 0.30) type = 'spike';
     else if(roll < 0.55) type = 'wraith';
     else if(roll < 0.80) type = 'gate';
     else type = 'saw'; // new type

     // ...existing branches...
     else if(type === 'saw'){
       obstacles.push({ type, x: W+40, y: GROUND_Y-20, w: 30, h: 30, spin: 0 });
     }
   }
   ```
   Keep the probabilities summing to 1 across all branches.

2. **Animate it (optional).** In the obstacle-update loop inside `update(dt)`, add a branch alongside the existing `wraith`/`gate` animation updates (e.g. `if(o.type==='saw'){ o.spin += dt*10; }`).

3. **Resolve collision behavior.** Still inside `update(dt)`'s obstacle loop, the `rectsOverlap(pBox, obox)` block is where hit rules live. Obstacles default to "hit unless dashing" (the `else` branch under `if(player.dashing)`). If your new obstacle should have a different rule (e.g. always damaging, or duckable only), add an explicit `if(o.type==='saw'){ ... }` branch there rather than falling through to the default.

4. **Draw it.** In `draw()`, inside the `for(const o of obstacles)` loop, add an `else if(o.type==='saw'){ ... }` branch using canvas 2D calls. Match the existing style: a solid fill color, a `shadowColor`/`shadowBlur` glow in the same hue, `ctx.save()`/`ctx.restore()` around any `ctx.translate()`.

5. **Clean-up is automatic** — the `if(o.x + o.w < -20){ obstacles.splice(i,1); continue; }` line at the top of the loop already removes any obstacle type once it scrolls off-screen; you don't need to touch it.

## Recipe: add a new power-up (like the dash orb, but different effect)

Orbs are the simplest template. Copy the pattern:
1. Add a `spawnPowerup()` function modeled on `spawnOrb()`, pushing to a new array (e.g. `let shields = [];`) initialized alongside `obstacles`/`orbs` and reset in `resetGame()`.
2. Call it from a timer in `update(dt)`, similar to `orbTimer`.
3. Add a collision-and-collect block modeled on the orb loop in `update(dt)` — on pickup, mutate player state (e.g. `player.shielded = true; player.shieldTime = 5;`) instead of `dashMeter`.
4. Consume/expire the effect somewhere in `update(dt)` (tick down `shieldTime`, clear the flag at 0).
5. Draw it in `draw()`, and reflect the active effect on the player sprite or HUD if relevant.
6. If it needs a HUD indicator, add a small element to the `.hud` block in the HTML and update it in `updateHud()`.

## Recipe: tune difficulty

All difficulty knobs are constants or single formulas — no scattered magic numbers:

- **Base numbers:** the `// ---------- Game constants ----------` block (`GRAVITY`, `JUMP_VEL`, `BASE_SPEED`, `DASH_SPEED_MULT`, `DASH_DURATION`, `DASH_COST`, `ORB_CHARGE`, `METER_REGEN`).
- **Ramp over time:** in `update(dt)`:
  ```js
  speed = BASE_SPEED + Math.min(340, distance*0.02);
  spawnInterval = Math.max(0.55, 1.15 - distance*0.00012);
  ```
  Raise the `Math.min`/`Math.max` caps to let it get harder for longer, or change the multipliers (`0.02`, `0.00012`) to change how *fast* it ramps.
- **Obstacle mix:** the probability thresholds in `spawnObstacle()` (currently 38% spike / 30% wraith / 32% gate).

## Recipe: change scoring

Score is accumulated in three places, all in `update(dt)`:
- Passive: `score += dt * (speed/10);` — continuous, tied to current speed
- Orb pickup: `score += 8;` in the orb collision block
- Gate clear: `score += 15;` in the gate collision block

Change the constants directly, or add new scoring events following the same `score += N;` pattern at the point where the event is detected.

## Recipe: reskin / retheme

- **Colors:** CSS custom properties at the top of `<style>` (`--bg-top`, `--bg-bot`, `--accent`, `--accent2`, `--danger`, `--gold`, `--text`) drive the UI chrome. Canvas draw colors are separate (hardcoded hex/rgba strings inside `draw()`) since canvas can't read CSS variables directly — update both if you want a consistent palette swap.
- **Player sprite:** `drawPlayerShape()` — currently a rounded rect with a glow and two eye dots. Replace the `ctx.roundRect(...)` fill with any canvas path (or an `Image`/sprite draw call) as long as you preserve the `x, y, w, h` sizing contract so collision (`playerBox()`) still lines up visually.
- **Sound palette:** every effect in the `sfx` object is one or two `beep(freq, dur, type, vol)` calls — change frequencies/waveforms (`'sine' | 'square' | 'sawtooth' | 'triangle'`) to restyle the audio without adding any files.

## Recipe: add a real (server-side) leaderboard

Currently `best` is read/written only via `localStorage.getItem/setItem('shadowdash_best', ...)` in two places (initial load, and inside `endGame()`). To go global:
1. Stand up a small API (any backend) with a `POST /scores` and `GET /scores/top`.
2. In `endGame()`, alongside the existing `localStorage.setItem(...)` call, `fetch()` the score to your API (consider debouncing/guarding against spam submissions).
3. Add a "Leaderboard" overlay (same pattern as `#menuOverlay`/`#gameOverOverlay`) that fetches and renders `GET /scores/top` on demand.
4. This is the one extension that pushes the project past "single static file" — at that point consider whether the backend needs its own auth/rate-limiting before shipping it publicly.

## Testing your changes

There's no automated test suite (see [DEVELOPMENT.md](DEVELOPMENT.md#known-constraints--non-goals)) — the practical loop is:
1. Serve the folder locally (`python -m http.server`) and open it in a real browser.
2. Play through: hit every obstacle type at least once, collect orbs, trigger a dash through a gate, deplete all 3 lives, and confirm the game-over/high-score flow.
3. Check the browser console for errors — canvas/audio code fails silently in a lot of cases (e.g. a typo in a new obstacle's `draw()` branch just won't render, it won't throw).
4. If you're comfortable with it, `chromium-cli`/Playwright can automate the above for regression checks, but for a project this size manual play-testing is normally faster than writing and maintaining a driver script.

## Deploying your changes

```bash
git add -A
git commit -m "describe your change"
git push
```

If continuous deployment is linked (see the main [README](../README.md#deployment)), Netlify redeploys automatically. Otherwise, redeploy manually:

```bash
npx netlify-cli deploy --prod --dir=.
```
