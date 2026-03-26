# Reaction Timer

A visual reaction time test built as a single self-contained HTML file. Measures how fast you respond to a visual stimulus across 5 rounds, then evaluates your performance against published human reaction time benchmarks.

![Three.js](https://img.shields.io/badge/Three.js-r128-black)
![Zero Dependencies](https://img.shields.io/badge/dependencies-0-brightgreen)
![Single File](https://img.shields.io/badge/architecture-single%20file-blue)

## What It Does

1. You click to start a round
2. A random delay (2–5 seconds) builds anticipation
3. The screen flashes **!!!** — click as fast as you can
4. After 5 valid rounds, you get a performance rating

Clicking during the wait period registers a **false start** — it doesn't count toward your 5 rounds but is logged in the session history.

### Performance Tiers

| Rating | Average Response |
|---|---|
| Excellent | < 180 ms |
| Fast | < 230 ms |
| Average | < 300 ms |
| Below Average | < 400 ms |
| Slow | ≥ 400 ms |

These thresholds are ~20ms more generous than lab-grade benchmarks to account for browser input pipeline latency (event loop, compositor, display vsync) that the user cannot control.

## Demo

Deploy the single `index.html` file to any static host. No build step, no server, no configuration.

## Technical Decisions

**Single file architecture** — all HTML, CSS, and JavaScript live in one `index.html`. Three.js is loaded via CDN. This makes deployment trivial (drag and drop to any static host) but means the file is ~1000 lines. This is a deliberate tradeoff: portability over modularity.

**`performance.now()` for all timing** — not `Date.now()`. The Performance API provides a monotonic clock with sub-millisecond precision, unaffected by system clock adjustments. For a tool that measures sub-second human responses, this precision matters.

**Three.js r128 via CDN** — renders a fullscreen wireframe icosahedron with orbiting cubes as a background layer. The 3D scene responds to game state through color-lerped transitions (idle → waiting → stimulus → result). The visual layer and game logic are fully separated into independent IIFE modules that communicate through a single callback.

**CSS custom properties for all design tokens** — no hardcoded color values in rules. Every color, font, radius, and transition duration is defined once in `:root`.

**Explicit state machine** — game phases (`idle`, `waiting`, `stimulus`, `false-start`, `result`, `complete`) are managed through a single `AppState` object. No scattered globals, no ambiguous states.

## Layout

- **Desktop (>768px):** Semi-transparent side panel on the right, 3D scene centered in the remaining viewport
- **Mobile (≤768px):** 3D scene in the top half, controls in a bottom sheet with a soft gradient fade at the boundary

## Project Structure

```
.
└── index.html    ← the entire application
```

That's it. No `package.json`, no `node_modules`, no build artifacts.

## Deploy

### GitHub Pages

```bash
git init
git add index.html
git commit -m "initial commit"
git remote add origin <your-repo-url>
git push -u origin main
```

Then enable GitHub Pages in your repo settings (Settings → Pages → Source: main branch).

### Any Static Host

Upload `index.html` to Netlify, Vercel, Cloudflare Pages, S3, or any web server. It runs as-is with zero configuration.

## Browser Support

Works in all modern browsers (Chrome, Firefox, Safari, Edge). Requires WebGL for the Three.js background — if WebGL is unavailable, the 3D scene won't render but the game logic functions normally.

`backdrop-filter` is used for panel blur effects. Unsupported in some older browsers — the panel falls back to a solid background, which is functional if less polished.

## Known Limitations

- **Browser input latency is unmeasurable.** The app measures from stimulus render to click event, but the browser's own input pipeline adds 5–30ms that varies by device, browser, and system load. Your true neural reaction time is faster than what this tool reports.
- **No persistent storage.** Results are lost on page refresh. This is intentional — the app is a quick test, not a longitudinal tracker.
- **Touch devices have inherent click delay.** The `click` event on mobile has more latency than `pointerdown`. Results on touch devices will be systematically slower than on desktop with a mouse.
- **Three.js r128 is outdated.** Pinned to this version per the original spec constraints. A production project should use a current release.
