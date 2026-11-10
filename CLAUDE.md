# CLAUDE.md

This file gives Claude Code guidance for working in this repository.

## Overview

Galactic Defender is a wave-based HTML5 canvas space shooter, live at https://galacticdefender.sala.company. The player moves a ship (WASD/arrows, or a virtual joystick on touch), fires (Space, or the touch fire button) and clears waves of basic, zigzag and heavy enemies, with a boss every 5th wave. Destroyed enemies can drop power-ups (weapon level, shield). The entire game is one self-contained `index.html` (inline CSS and inline vanilla JS, class-based entities). There is no framework, no build step, no dependencies and no automated tests.

## Commands

No package.json, build, lint or test tooling. Open `index.html` directly in a browser, or serve the folder:

```bash
python3 -m http.server 8000
```

## Architecture

The repo contains `index.html`, `README.md`, `LICENSE` and `CNAME`, plus two QA reports from earlier manual testing (`galactic_defender_comprehensive_test_report.pdf`, `space_shooter_retest_report.docx`). The reports are not used by the game.

`index.html` layout, top to bottom:
- `<style>`: neon/cyan UI, `#touchControls`, `#joystick`, breakpoints at `max-width: 768px` and `480px`.
- DOM: `#gameCanvas`; HUD (`#score`, `#lives`, `#level`, `#wave`, `#enemies`, `#powerIndicator`); `#startScreen` and `#gameOverScreen` (buttons call `startGame()`, `restartGame()`, `showStartScreen()` through inline `onclick`); touch fire button `#shootBtn`; `#joystick` / `#joystickKnob`.
- Main `<script>`:
  - `resizeCanvas()` keeps a 3:2 canvas, capped at 1200x800, and re-runs on resize.
  - Global state (`gameState`, `gameRunning`, score/lives/level/wave, entity arrays).
  - Sound: `initAudio()` / `playSound()` synthesize tones with the Web Audio API. The AudioContext is created in `startGame()`, on a user gesture.
  - Classes: `Player`, `Bullet`, `Enemy` (types `basic`/`zigzag`/`heavy`/`boss`, worth 100/150/200/500 points), `PowerUp`, `Particle`, `Star`.
  - `createWave()` / `updateWaveSpawning()` handle wave composition (a boss when `wave % 5 === 0`). `checkCollision()` is circular collision detection.
  - `gameLoop()` is driven by `requestAnimationFrame` and stops when `gameRunning` is false.
  - Input: keyboard map `keys`, joystick touch handlers, and `touchmove` `preventDefault` on `#gameContainer` to block page scrolling. Touch controls are shown only when `'ontouchstart' in window`.
- A "Created by MiniMax Agent" floating badge (its own `<style>`, `#minimax-floating-ball` and script, which opens agent.minimax.io with `noopener`). It is attribution from the tool that generated the game.
- The GoatCounter snippet comes last.

There is no localStorage use (no saved high scores), no network data and no service worker.

## Deployment

No `.github/workflows` directory and no `.nojekyll`. The site is published by GitHub Pages' branch-based deploy (configured in the repo settings, not in this repo), serving `main` as-is. The custom domain comes from `CNAME` (`galacticdefender.sala.company`). **Treat every push to `main` as a live deploy.** Because the whole branch is served, the PDF/DOCX reports are publicly reachable as well.

## Conventions and gotchas

- Keep the GoatCounter analytics snippet at the end of `index.html` intact (endpoint `https://salacompany.goatcounter.com/count`, skipped on localhost, paths prefixed with `location.host`).
- Keep the MIT copyright header in `index.html`. External links should keep `noopener`.
- The README's feature and "performance" claims came from the generated project. Verify behavior in code rather than trusting the README.
- Physics and timers are per-frame (e.g. `waveDelay = 180` means "3 seconds at 60fps"), so game speed depends on the display refresh rate.
- English only, LTR. When changing UI, test mobile at both breakpoints and confirm the joystick and fire button still work and the page doesn't scroll.
