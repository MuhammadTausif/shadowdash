# ShadowDash

*Outrun the dark. Dash through the gates.*

**ShadowDash** is a single-page, single-file browser game — an endless runner built entirely with HTML5 Canvas and vanilla JavaScript, no build step, no dependencies, no external assets. Jump spikes, duck wraiths, and burn your dash meter to phase through pulsing Shadow Gates that otherwise end your run.

**▶ Play it live:** https://shadowdash-651.netlify.app

![ShadowDash](https://img.shields.io/badge/status-live-7c5cff) ![No dependencies](https://img.shields.io/badge/dependencies-none-3ce7ff) ![Single file](https://img.shields.io/badge/build%20step-none-ffd166)

---

## Quick start

There is nothing to install or build. The entire game lives in one file: [`index.html`](index.html).

```bash
git clone https://github.com/MuhammadTausif/shadowdash.git
cd shadowdash
# just open it
start index.html      # Windows
open index.html        # macOS
xdg-open index.html    # Linux
```

Or serve it locally if you prefer a URL instead of a `file://` path:

```bash
python -m http.server 8080
# then visit http://localhost:8080
```

## Documentation

| Doc | What's in it |
|---|---|
| [docs/MANUAL.md](docs/MANUAL.md) | How to play — controls, obstacles, scoring, strategy tips |
| [docs/DEVELOPMENT.md](docs/DEVELOPMENT.md) | How the game was built — architecture, tech choices, deploy pipeline |
| [docs/EXTENDING.md](docs/EXTENDING.md) | How to add new obstacles, power-ups, mechanics, or reskin it |

## Features

- Canvas-based endless runner with jump, duck, and dash mechanics
- A signature **dash** mechanic: charge a meter by collecting orbs, then dash to phase through obstacles and pass through Shadow Gates (which otherwise block you completely)
- Increasing difficulty over time (speed and spawn rate scale with distance)
- 3-life system with hit invulnerability and visual flash feedback
- Local high-score persistence (`localStorage`, no backend)
- Procedural sound effects via the Web Audio API — no audio files
- Parallax starfield/mountain background, particle bursts, dash trail
- Full keyboard controls plus on-screen touch controls for mobile
- Menu, pause, and game-over overlays; mute toggle

## Tech stack

Plain HTML + CSS + JavaScript. No frameworks, no bundler, no package.json. Rendering is `<canvas>` 2D context; audio is the native `AudioContext`/oscillator API. See [docs/DEVELOPMENT.md](docs/DEVELOPMENT.md) for the full breakdown.

## Deployment

The site auto-deploys to Netlify from the `master` branch of this repo. To deploy your own copy:

```bash
npx netlify-cli login
npx netlify-cli deploy --prod --dir=.
```

## Contributing / extending

Want to add a new obstacle, power-up, or visual theme? [docs/EXTENDING.md](docs/EXTENDING.md) walks through the game loop and shows exactly where to hook in new features.

## License

No license file has been added yet — treat this as all-rights-reserved by default until one is added. Open an issue if you'd like a specific license applied.
