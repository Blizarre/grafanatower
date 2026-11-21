# Perimeter Defense

A fake Grafana dashboard that hides a playable tower-defense game in the gaps
between its panels. Enemies spawn on the left, weave through the dashboard
gutters, and breach the red perimeter on the right if you fail to stop them.

Everything is self-contained in `index.html` — HTML, CSS, JavaScript, and
synthesized audio. No build step, no dependencies, no external assets.

## Run

Open `index.html` directly in a browser, or serve the folder:

```bash
python3 -m http.server 8000
# http://localhost:8000
```

A desktop browser is recommended. Sound unlocks on your first click due to
browser autoplay rules.

## How to play

- Click a panel to build there. Structures can only be placed inside panel
  bodies, not in the corridors. A colored ghost preview shows a valid placement;
  red means blocked or too close to another structure.
- Use the fabricator bar in the bottom-left, or press `1` / `2`, to choose what
  to build:
  - **Laser turret** — fires at nearby hostiles. Costs 1.5 capacitor charge per
    second while online, plus 3 charge per shot.
  - **Crystal generator** — glowing cyan crystal that supplies 9 charge per
    second.
- Each structure takes **3 seconds** to construct, and the fabricator can build
  only one structure at a time. A progress ring and countdown appear on the
  map; the bar shows the remaining time.
- The **Reactor capacitor** starts at 80/100. If it empties, turrets go offline
  until charge recovers. Watch the gauge and its base draw / net readouts, and
  add generators before adding more guns.
- Enemies that reach the right edge end the run. Waves get larger and tougher;
  your score and kill count are tracked in the stat panels.
- The speaker button in the top bar mutes the laser audio.

## Tips

- One generator roughly sustains one continuously firing turret. Build supply
  before a long fight, not during it.
- Turrets only reach corridors near their panel edge — check the range circle
  in the placement preview.
- Use the 6-second grace period at the start to get your first turret and
  generator online.
- Turrets in the narrow stat panels near the top can cover both the corridor
  above and below the panel.

## Under the hood

- The dashboard is a CSS grid of nine fake, non-interactive Grafana panels.
- `#game` (behind the panels) draws enemies, particles, and the perimeter.
- `#fx` (above the panels) draws turrets, generators, beams, construction
  sites, and the placement ghost, and handles all input.
- Enemy movement uses a BFS flow field rebuilt from live panel rectangles on
  resize, so they always navigate the current gutter layout.
- Audio is synthesized with the Web Audio API: layered saw/body/noise layers,
  saturation, and compression. No audio files.
- All gameplay constants (`TYPES`, `TOWER`, `GEN`, `POWER`, wave composition,
  build time, spawn grace) are near the top of the script in `index.html`.

See `AGENTS.md` for architecture notes and the validation workflow.
