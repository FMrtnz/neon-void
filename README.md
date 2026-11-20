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
| Bomb | B (when the circle is full) | Tap the glowing circle (bottom right) |
| Mute | M or the speaker button | Speaker button |
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

**Bonuses** (dropped by enemies, dropped by enemies, always dropped by bosses)

| Icon | Effect |
| --- | --- |
| S | Shield: absorbs one hit, lasts 10 seconds |
| M | Double shot, then triple shot if you grab another, for 15 seconds |
| + | Heals 1 HP |

## Sound

All audio is synthesized in the browser with the Web Audio API, so there are no audio files to load and it works offline.

- **Effects**: shots (higher pitch with double/triple shot), explosions (small, medium, boss), player hit, shield break, one distinct sound per bonus, level up, boss siren, low-HP alarm, game over jingle and menu clicks.
- **Music**: a synthwave loop (bass, arpeggio, pad) that gets drums at level 3 and a faster tempo during boss fights. It stops on pause and game over.
- **Settings**: the AUDIO menu has separate music and effects volumes plus a mute switch. Press **M** or tap the speaker button in game. Settings are saved in `localStorage`.
- **Vibration**: a short haptic pulse when you are hit (Android only).
- Browsers block audio until the first tap or click, so sound starts when you press Play.

To change a sound, edit the `SFX` table in the `Sound` module. To change the music, edit `CHORDS` and `scheduleStep`.

## Bomb

The bomb is not a pickup: it recharges as you destroy enemies. The circle in the bottom-right corner fills up (15 kills; a boss counts for 8). When it is full it glows, and you can use it with **B** or by tapping it.

- Only one bomb can be stored per charge. The circle stays full until you use it, then starts again from empty.
- A bomb wipes all enemy bullets, destroys every normal enemy on screen and deals 12 damage to a boss.
- Enemies destroyed by the bomb itself do not recharge it.
- Tune it in `CFG.bomb`: `killsToCharge`, `bossCharge`, `bossDamage`.

## HP display and icons

HP is drawn as glowing neon hearts in the top-left corner: filled hearts are your remaining HP, dim outlines are empty slots (you can heal back up to the maximum), and they pulse red when only 1 HP is left.

The mute icon is an inline SVG speaker, so it looks the same on every device (no emoji).

## Balancing the game

All tuning values are grouped in the `CFG` object at the top of the script in `index.html`: player speed and HP, fire rate, spawn delay, difficulty curve, enemy HP, speed, score and unlock thresholds, bonus durations and drop chance, shake strength, and the particle limit.

## Code structure

`index.html` is organized in numbered sections:

1. **Config**: the `CFG` object
2. **Setup**: canvas, helpers, `localStorage`, service worker registration, install banner
   - **2b. Sound**: the `Sound` module (Web Audio API)
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
- **No sound on iPhone / iPad**: turn off the silent switch (the ringer switch on the side), make sure the volume is up, and update iOS to 16.4 or later. If you installed the app before an update, delete it from the home screen and install it again.
- **Best score lost**: private browsing and some privacy settings block `localStorage`; the game still runs without it.
