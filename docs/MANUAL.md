# ShadowDash — Player Manual

## Objective

Survive as long as possible. Your score climbs automatically the longer you run and the faster you're going, plus bonuses for collecting orbs and clearing Shadow Gates. You have **3 lives** — lose them all and the run ends.

## Controls

| Action | Keyboard | Touch (mobile) |
|---|---|---|
| Jump | `Space` or `↑` | **JUMP** button |
| Duck | `↓` (hold) | **DUCK** button (hold) |
| Dash | `Shift` | **DASH** button |
| Pause / Resume | `P` | — |
| Start / Restart | `Space` or `Enter` on menu/game-over screen | Tap the game area on the menu screen |
| Mute | — | 🔊 button, top-right of the game |

On touch devices the on-screen controls appear automatically; on desktop, keyboard hints are shown on the start screen.

## What you'll run into

- **Spikes** (red triangles, ground level) — jump over them.
- **Wraiths** (purple bobbing orbs with eyes, head height) — duck under them.
- **Shadow Gates** (pulsing violet rectangles, full height) — these **cannot be jumped or ducked**. You must be dashing when you pass through one, or it counts as a hit.
- **Orbs** (small cyan glowing spheres, floating at two heights) — fly through them to collect. Each orb adds to your score and charges your dash meter.

## The Dash

Dash is the core mechanic and your only way through a Shadow Gate:

- Press **Shift** (or the **DASH** button) to dash. It costs **34%** of your dash meter and lasts **0.35 seconds**.
- While dashing you move roughly **2.4×** faster, leave a glowing trail, and are briefly invulnerable — you can also dash straight through spikes and wraiths unharmed, not just gates.
- The dash meter starts at 40%, regenerates slowly on its own (**3% per second**), and refills faster by collecting orbs (**+20%** each). You need at least 34% banked before you can dash again.
- The meter fill bar glows gold once you have enough charge to dash.

**Strategy:** don't hoard your dash only for gates — it's also a panic button through a mistimed jump or duck. But since gates are mandatory dash checks, always try to keep at least one dash charge (34%+) banked before a gate comes into view.

## Lives and hits

- You start with 3 hearts, shown top-right.
- Getting hit by a spike, wraith, or an un-dashed Shadow Gate costs one life and gives you ~1.4 seconds of invulnerability (the player sprite flashes red).
- Lose all 3 lives and the run ends, showing your final score and whether it beat your saved best.

## Difficulty curve

The longer you survive, the faster the world scrolls and the more often obstacles spawn — both scale continuously with distance traveled, so later runs demand tighter reflexes and better dash timing than early ones.

## Scoring & high scores

- Score increases continuously based on your speed, plus **+8** per orb collected and **+15** per Shadow Gate successfully dashed through.
- Your best score is saved locally in your browser (`localStorage`) and persists between visits — it's per-browser, not a global leaderboard.

## Tips

1. Duck early — the duck hitbox shrinks the moment you start ducking, so tap it a beat before a wraith reaches you.
2. Time your dash for the gate, not on top of it — starting the dash slightly before you reach the gate ensures you're already in the invulnerable dashing state when you hit the collision zone.
3. Detour toward high orbs when it's safe — they're worth both score and dash charge, and dash charge is what keeps gates from being free hits.
4. When in doubt, dash — its brief invulnerability can bail you out of a bad jump/duck read on anything, not just gates.
