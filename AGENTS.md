# AGENTS.md — Perimeter Defense

A single-file, fake Grafana dashboard with a playable tower-defense game in the
panel gutters. Everything (HTML, CSS, JS, synthesized audio) lives in
`index.html`. There is no build step, no package manager, and no external
assets.

## Run

Open `index.html` directly, or serve it:

```bash
python3 -m http.server 8000
# http://localhost:8000
```

## How the game works

- The dashboard is a 12-column CSS grid of fake, non-interactive Grafana panels.
- Enemies spawn at the left edge, follow a BFS flow field through the gaps
  between panels, and breach at the red right edge (game over).
- Structures can **only be built inside panel bodies**. Left click a panel to
  place the selected structure; `1` = laser turret, `2` = crystal generator,
  `3` = mega laser.
- There is one fabricator slot: turrets and generators take **3 seconds**,
  mega lasers take **9 seconds**, and only one build runs at a time.
- Turrets consume capacitor charge continuously (`1.5/s` each) plus a per-shot
  cost. Normal turrets cost `3` per shot; mega lasers cost `12` and have double
  range, 4x damage, and half the fire rate. Generators supply `9/s` each.
  Starting charge is 80/100. At 0 charge turrets stop firing until charge
  recovers. The “Reactor capacitor” panel is live and shows charge, generator
  count, base draw, and base net rate.
- Score / wave / hostile counters and the “Defense grid” list are real; the
  rest of the dashboard widgets are intentionally fake.

## Architecture (all in `index.html`)

- `#game` canvas (z-index below panels): perimeter, spawn markers, enemies,
  particles, floating score text.
- `#fx` canvas (above panels, below top bar): turrets, generators, beams,
  construction sites, ghost preview, and all click/mouse input.
- Panels are `pointer-events: none`; the `#fx` canvas handles input. Do not
  change this layering without moving input handling too.
- `readPanelRects()` / `buildField()` recompute panel rectangles and the
  pathfinding field on resize. Enemies use `sampleFlow()` to steer.
- `canPlace()` enforces: inside a panel, at least `GEN.radius + 4` (18px) from
  panel edges, and `STRUCT_SPACING` (32px) from other structures/sites.
- `startBuild()` / `updateBuild()` implement the single fabricator build (3s;
  9s for mega lasers) and create a turret, mega laser, or generator on
  completion.
- `updatePower()` applies generation/draw; `updateTowers()` spends each tower's
  `shotCost` before firing.
- Audio is synthesized with the Web Audio API in `ensureAudio()` /
  `playLaser()`. Browsers require a gesture, so the context is created/resumed
  on `pointerdown`; the speaker button in the top bar mutes everything.

## Tuning constants

Search for these names to rebalance:

- `TYPES` — enemy hp/speed/score per class.
- `TOWER` — normal turret range, fire rate, damage, radius.
- `MEGA` — mega laser multipliers (2x range, 4x damage, 2x fire interval, 4x
  shot cost, 3x build time) and size/color.
- `GEN`, `STRUCT_SPACING` — structure size and placement spacing.
- `POWER` — max/start charge, turret draw, shot cost, generator output, base
  build time.
- `DIFFICULTY` — post-wave-3 ramp: 2x enemy count (cap 100), extra HP/speed
  per wave, faster spawns, and brute/runner composition caps. Waves 1-3
  deliberately match the original tuning.
- `makeWaveSpecs()` / `pickType()` — wave size and enemy composition.
- `startWave()` / boot + `resetGame()` spawn grace (currently 6s).

## Conventions and cautions

- Keep the game single-file and dependency-free.
- Any structure that must be visible on top of fake panels must be drawn on
  `fctx` (`drawTowers` handles both turret kinds, `drawGenerators`, `drawSite`,
  `drawBeams`, `drawGhost`, `drawEffects`), not the lower `ctx`.
- Pathfinding is rebuilt from live DOM rects; avoid layout changes that move
  panels without triggering resize.
- Fake widgets should remain non-interactive (no drag/resize/edit features).

## Validation

There is no committed test suite. Before finishing changes:

```bash
# Extract the inline script and syntax-check it
python3 - <<'EOF'
import re
src = open('index.html').read()
js = re.search(r'<script>(.*)</script>', src, re.S).group(1)
open('/tmp/game.js', 'w').write(js)
EOF
node --check /tmp/game.js
```

Then play it in a browser: place a turret, mega laser, and generator, verify
the 3s/9s builds, power drain/recharge, no placement in gutters, both laser
audio variants, mute toggle, and the breach/restart flow.
