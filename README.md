# Neon Void

An endless neon sci-fi space shooter that runs in the browser and installs as a Progressive Web App (PWA). No framework, no build step, no external assets: everything is drawn with Canvas 2D.

## Files

| File | Role |
| --- | --- |
| `index.html` | The whole game: HTML, CSS and JavaScript in one file |
| `sw.js` | Service worker: caches the game so it works offline |
| `manifest.webmanifest` | PWA metadata (name, colors, icons) |
| `icon-192.png`, `icon-512.png` | App icons |

## Run it

Service workers only work over HTTPS or on `localhost`, so do not open the file with `file://` if you want offline mode.

**Locally**

```bash
cd neon-void-pwa
python3 -m http.server 8000
# open http://localhost:8000
```

**Online**: upload the 5 files, in the same folder, to any static host (GitHub Pages, Netlify, Cloudflare Pages, etc.).

The game itself also runs by simply double-clicking `index.html`; only the offline and install features need a server.

## Install and offline play

1. Open the game once while online so the service worker caches it.
2. Chrome, Edge and Android show an "Install Neon Void" banner in the game. On iPhone and iPad, use Share, then "Add to Home Screen".
3. After that, the game launches and plays without a connection.

When you change any file, bump `CACHE_VERSION` in `sw.js` (for example `neon-void-v3`). Otherwise players keep the old cached version.

## Controls

| Action | Desktop | Mobile |
| --- | --- | --- |
| Move | Arrows or WASD (ZQSD on AZERTY) | Drag your finger, the ship follows |
| Fire | Automatic, or hold Space if auto-fire is off | Automatic |
| Pause | P or Esc, or the pause button | Pause button |
| Start | Enter on the menu | Play button |

Auto-fire can be switched on or off in the main menu. The game also pauses by itself when the tab loses focus.

## Gameplay

- Your ship starts with 3 HP (max 5). A hit gives you a short invulnerability window.
- The starfield scrolls endlessly and waves spawn procedurally, faster and denser as your score grows (a new level every 300 points).
- The best score is saved in `localStorage`.

**Enemies**

| Enemy | Behavior | Unlocks at |
| --- | --- | --- |
| Drone | Falls in a straight line, sometimes in formations | Start |
| Sniper | Hovers near the top, strafes and aims at you | 200 points |
| Kamikaze | Homes in on your ship | 500 points |
| Mini-boss | Zigzags across the top and fires a fan of bullets; more HP each level | Every 40 seconds |

**Bonuses** (dropped by enemies, always dropped by bosses)

| Icon | Effect |
| --- | --- |
| S | Shield: absorbs one hit, lasts 6 seconds |
| M | Double shot, then triple shot if you grab another, for 9 seconds |
| + | Heals 1 HP |

## Balancing the game

All tuning values are grouped in the `CFG` object at the top of the script in `index.html`: player speed and HP, fire rate, spawn delay, difficulty curve, enemy HP, speed, score and unlock thresholds, bonus durations and drop chance, shake strength, and the particle limit.

## Code structure

`index.html` is organized in numbered sections:

1. **Config**: the `CFG` object
2. **Setup**: canvas, helpers, `localStorage`, service worker registration, install banner
3. **Game state**: player, enemy, bullet and bonus arrays, the particle pool, stars
4. **Input**: keyboard and pointer (touch/mouse) handling
5. **UI / screens**: menu, how to play, pause, game over
6. **Entities**: spawning functions and the particle `burst`
7. **Systems**: player update, spawning, enemy behaviors, collisions, scoring
8. **Rendering**: neon polygons, stars, HUD, screen shake
9. **Main loop**: `requestAnimationFrame` with a clamped delta time
10. **Init**

Performance notes: particles come from a fixed pool of 400 reused objects, and the shapes are simple polygons drawn with glow.

## Adding a new enemy

1. Add an entry to `CFG.enemies` (HP, speed, size, score, color, unlock, weight).
2. Add its polygon to `SHAPES`.
3. Add a `case` for it in `updateEnemy`.
4. Add its name to the type lists in `updateSpawning` if it should spawn in regular waves.

## Troubleshooting

- **No install banner**: the browser only offers it over HTTPS (or `localhost`) once the manifest and service worker load correctly. Check the Application tab in your browser dev tools.
- **Old version keeps showing**: bump `CACHE_VERSION` in `sw.js`, or use "Clear site data" in dev tools.
- **Best score lost**: private browsing and some privacy settings block `localStorage`; the game still runs without it.
