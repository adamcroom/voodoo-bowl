# Voodoo Bowl - Java to HTML Conversion

## Project Overview
Converting the original **Voodoo Bowl** Java applet (`Football.class`) into a standalone HTML5/Canvas game (`voodoo_bowl.html`).

## Source Assets
- `Football.class` — Original compiled Java applet (no source available)
- `football.cab` — Cabinet archive for the applet
- `images.gif` — Sprite sheet (500x280) containing all game sprites
- `bg.gif` — Bottom HUD strip (560x103) with "Voodoo Bowl" logo, labels, controls
- `*.au` — 7 sound effect files (hike, whistle, yell, crowdloud, crowdsoft, tackle, touchdown)
- `.webp` screenshot — Reference of original game

## Current State
The game is complete and fully playable. All 38 bytecode-accurate corrections (F01–F38) have been implemented. README with verified history section is written. Ready for GitHub Pages deployment.

## Completed Work
All phases (1–6) complete. Phase 6 addressed 38 specific differences from the original Java bytecode, covering game logic, sack rendering, visual/sprite accuracy, audio mapping, movement/viewport, and rendering pipeline. See git history for details.

## Original Sprite Map (from bytecode)
```
Index  Name         cropImage(x, y, w, h)
  0    EYE          (120, 200, 80, 80)
  1    GRASS1       (200, 160, 40, 40)
  2    GRASS2       (200, 200, 40, 40)
  3    GRASS3       (200, 240, 40, 40)
  4    COFFIN       (320, 0, 60, 60)
  5    GAMEOVER     (320, 60, 60, 60)
  6    REFEREE1     (200, 80, 40, 40)
  7    REFEREE2     (200, 120, 40, 40)
  8    ZOMBIE1      (120, 0, 40, 40)
  9    ZOMBIE2      (120, 40, 40, 40)
 10    ZOMBIE3      (120, 80, 40, 40)
 11    ZOMBIE4      (120, 120, 40, 40)
 12    ZOMBIE5      (120, 160, 40, 40)
 13    TOMBSTONE1   (200, 0, 40, 40)
 14    TOMBSTONE2   (200, 40, 40, 40)
 15    SACKN        (240, 0, 40, 80)
 16    SACKS        (280, 0, 40, 80)
 17    SACKE        (240, 80, 80, 40)
 18    SACKW        (240, 120, 80, 40)
 19    L_ENDZONE    (0, 0, 40, 280)
 20    R_ENDZONE    (40, 0, 40, 280)
 21    LINE         (80, 0, 3, 280)
 22    TOUCHDOWN    (240, 220, 212, 59)
 23    QB1          (160, 0, 40, 40)
 24    QB2          (160, 40, 40, 40)
 25    QB3          (160, 80, 40, 40)
 26    QB4          (160, 120, 40, 40)
 27    QB5          (160, 160, 40, 40)
 28    BLANK        (400, 200, 14, 20)
 29    CLICKHERE    (240, 200, 99, 17)
 30    MARKER       (320, 120, 12, 12)

Numbers:  n0-n9 at (400, i*20, 14, 20)
Downs:    d0-d3 at (414, i*25, 33, 25)
```

## Original Audio Index Mapping
```
Index  Constant     File
  0    SACKED       tackle.au
  1    HIKE         hike.au
  2    HIT_REFEREE  yell.au
  3    SCORE        touchdown.au
  4    CROWDSOFT     -crowdsoft.au
  5    CROWDLOUD    crowdloud.au
  6    YELL         whistle.au
```

## Technical Notes
- Single self-contained HTML file with all sprites + audio embedded as base64
- Canvas: 568x280 (game) + 568x103 (HUD)
- `requestAnimationFrame` with 333ms tick interval
- bg.gif has transparent regions where numbers are drawn — requires black fill before redraw
- `.au` files converted with macOS `afconvert` to m4a/AAC format for web playback
