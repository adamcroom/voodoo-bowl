# Voodoo Bowl — Bytecode-Accurate Corrections

Full disassembly of `Football.class` revealed 38 differences between the HTML5 recreation and the original Java applet. All fixes listed below, grouped by category.

## 6A. Game Logic Fixes

- [x] **F01 — Initial density is 0.15, not 0.11**: `newGame()` sets `density = 0.15`. Density multiplier (1.1x) applies after each TD but there is NO cap — density grows without limit. Our code caps at 0.15 and starts at 0.11.
- [x] **F02 — Respawn only spawns zombies + refs**: `newDown()` spawns `(remaining * vcells * density)` zombies (cell=10) and `(remaining * vcells * 0.02)` refs (cell=6+rnd(2)). NO tombstones or coffins in respawn. Tombstones only appear when a ref is hit. Our code spawns 70/15/10/5 mix.
- [x] **F03 — Protective zombies spawn adjacent to QB**: After each down, 3 zombies are placed at `(qbX+1, qbY-1)`, `(qbX+1, qbY)`, `(qbX+1, qbY+1)` — a wall just ahead of the QB. Our code doesn't do this.
- [x] **F04 — Yards-to-go allows negative yardage**: Original does `toFirstDown = toFirstDown - (qbX - scrimage)` with NO `Math.max(0, ...)` protection. Sacks SHOULD cost yards. We incorrectly prevent this.
- [x] **F05 — Game over triggers on down==4 (not down>4)**: Original checks `if (down == 4 && toFirstDown > 0)` for turnover. Our code checks `down > 4`.
- [x] **F06 — Ref collision: QB moves INTO ref cell**: In original, the QB position updates BEFORE the ref check. So QB moves onto the ref's cell, ref turns to tombstone at that cell. QB now stands on the tombstone (tombstone only blocks future moves INTO it). Our code places tombstone at QB's old position.
- [x] **F07 — Coffin is cell value 4, not 19-20**: Original `COFFIN=4` is a single 60x60 sprite. It's never placed on the field during gameplay — it's only drawn on the QB at game over. `L_ENDZONE=19, R_ENDZONE=20` are endzone strip sprites. Our code treats 19-20 as coffin obstacles.
- [x] **F08 — No coffin obstacles on field**: Coffins are never spawned as field obstacles in the original. Remove coffin spawning from respawn and initial placement.
- [x] **F09 — Zombie frame animation**: Zombies cycle through frames 8-12 as they move. When moving left and not a ref, zombies change: `if (cell != 12) { even cell -> cell+1, odd -> cell-1 }`. Same for right movement but reversed direction `(8+rnd(2))` or `(10+rnd(2))`.
- [x] **F10 — Refs move on the field**: Original `moveDefense` moves both zombies (8-12) AND refs (6-7). If defender is left of QB: 50% chase chance (`random(2)==0`). Otherwise: 20% random move (`random(5)==0`). Our code only moves zombies.

## 6B. Sack/Tackle Rendering Fixes

- [x] **F11 — Sack sprites are field cells, not QB overlays**: In original, sack creates a FIELD CELL (15-18) at the old position showing the tackle direction. These are rendered as normal sprites during field drawing, NOT as an overlay on the QB. The QB sprite is simply NOT drawn when sacked.
- [x] **F12 — Sack direction in keyDown (QB walks into zombie)**: `field[zombie_pos] = 0` (clear zombie). Then based on direction: east→SACKE(17) at old QB pos, west→SACKW(18) at zombie pos, south→SACKS(16) at old QB pos, north→SACKN(15) at old QB pos.
- [x] **F13 — Sack direction in moveDefense (zombie walks into QB)**: Same directional logic but from the zombie's perspective. `field[old_zombie] = 0`. Then: east→SACKW(18), west→SACKE(17), south→SACKN(15), north→SACKS(16).
- [x] **F14 — Sack sprites are NOT explode1/explode2**: The real sack sprites from the sprite sheet: SACKN=(240,0,40,80), SACKS=(280,0,40,80) — both 40x80 (tall). SACKE=(240,80,80,40), SACKW=(240,120,80,40) — both 80x40 (wide). Need to re-extract these from images.gif.
- [x] **F15 — Sack cells cleared on game over**: During rendering, when `gameOver` is true and a cell is 15-18 (sack), it's cleared to 0. This cleans up sack debris after the game ends.

## 6C. Visual/Sprite Fixes

- [x] **F16 — Endzones are SPRITES, not programmatic fills**: `L_ENDZONE` = `cropImage(0, 0, 40, 280)` and `R_ENDZONE` = `cropImage(40, 0, 40, 280)`. These are 40x280 full-height strips from the sprite sheet. Should be drawn as images, not colored rectangles with diagonal stripes.
- [x] **F17 — Endzone rendering**: Left endzone drawn when `viewPt.x == 0` at `(fieldOffset.x, fieldOffset.y)`. Right endzone drawn when `viewPt.x == hcells - viewwidth` at `(fieldOffset.x + (viewwidth-1)*cellsize, fieldOffset.y)`. Each is exactly 1 cell wide.
- [x] **F18 — EYE sprite is 80x80, not 130x50**: `cropImage(120, 200, 80, 80)`. Positioned at midfield: `x = fieldOffset.x + hcells*cellsize/2 - viewPt.x*cellsize - 40`, `y = fieldOffset.y + vcells*cellsize/2 - 40`. Need to re-extract.
- [x] **F19 — GAMEOVER is a 60x60 sprite**: `cropImage(320, 60, 60, 60)`. Drawn at QB position (with 10px margin adjustments) when `gameOver && pauseTime == 0`. NOT canvas text.
- [x] **F20 — COFFIN sprite drawn at QB on game over**: `images[4]` (60x60) drawn at QB position when game over. Coffin drawn first, then GAMEOVER sprite on top. Both flash: GAMEOVER only shows when `tick % 5 != 0`.
- [x] **F21 — Yard lines skip endzones**: Original draws `LINE` (images[21]) for each visible cell, but only when `viewPt.x + i > 1` AND `viewPt.x + i < hcells - 1`. This skips the first 2 cells and last cell. Our code draws them everywhere.
- [x] **F22 — Scrimmage MARKER sprite**: Original uses a 12x12 `MARKER` sprite (`cropImage(320, 120, 12, 12)`) drawn at the first-down line position, just below the field. Our code uses a dashed yellow line.
- [x] **F23 — CLICKHERE background is fillRoundRect(119, 37, 10, 10)**: Original uses `fillRoundRect` with specific dimensions: width=119, height=37, arcW=10, arcH=10. Centered on field. The click sprite is offset +10px inside the rect.
- [x] **F24 — CLICKHERE shown when game over OR lost focus**: The click-here prompt appears both on game over AND when the applet loses focus (`!hasFocus`). Not just game over + start screen.
- [x] **F25 — Grass sprites are from the sprite sheet center, not left column**: `GRASS1=cropImage(200,160,40,40)`, `GRASS2=cropImage(200,200,40,40)`, `GRASS3=cropImage(200,240,40,40)`. These are 40x40 sprites from x=200. Our grass was extracted from the left column.
- [x] **F26 — BLANK sprite for leading zeros**: `images[28] = cropImage(400, 200, 14, 20)` — a blank 14x20 sprite used instead of leading zeros in TIME, SCORE, and YARDS displays.
- [x] **F27 — Score is 3 digits (hundreds/tens/ones)**: Score display uses 3 positions at 14px spacing. Leading digits show BLANK when zero.

## 6D. Audio Fixes

- [x] **F28 — Crowd noise is random ambient**: In the `run()` loop, `random(12)==0` triggers `audioClips[5]` (CROWDLOUD) each tick. That's ~1/12 chance per 333ms tick = roughly every 4 seconds on average. NOT a one-shot at game start.
- [x] **F29 — Game over plays YELL, not whistle**: Original plays `audioClips[6]` (YELL) on game over. Our code plays whistle.
- [x] **F30 — Touchdown plays SCORE sound**: Original plays `audioClips[3]` on touchdown, which maps to the "score" audio name. Not "touchdown" + "crowdloud".
- [x] **F31 — Audio name mapping**: `[0]=tackle, [1]=hike, [2]=yell, [3]=touchdown, [4]=-crowdsoft, [5]=crowdloud, [6]=whistle`. The indices don't match our sound trigger names.

## 6E. Movement/Viewport Fixes

- [x] **F32 — QB starts at x=-1 (off-field)**: `qbPt = Point(-1, vcells/2)`. First key press moves QB onto the field at x=0.
- [x] **F33 — Viewport scroll logic differs**: Moving left: viewport scrolls when `viewPt.x > 0 AND qbX < hcells - viewwidth*3/4`. Moving right: viewport scrolls when `qbX >= viewwidth/4 AND viewPt.x < hcells - viewwidth`.
- [x] **F34 — QB frame animation**: Frames use images[23+qbFrame]. Up/Down set qbFrame=4. Left alternates 2↔3. Right alternates 0↔1. Our code cycles 0-4 on every move.
- [x] **F35 — qbDepth animation**: On new down (not TD), `qbDepth = cellsize - 3 = 37`. Each tick, `qbDepth = max(0, qbDepth - 7)`. QB is drawn at `y + qbDepth` — a "dropping in from above" animation. Our code may not handle this correctly.
- [x] **F36 — Defenders move even during game over**: In `run()`, `moveDefense()` is called when `gameOver` is true OR `startTime > 0`. Zombies keep moving after the game ends. Our code stops them.

## 6F. Rendering Pipeline

- [x] **F37 — Double-buffered rendering**: Original draws to `bbuf` (back buffer), then copies to screen. Field is drawn to bbuf, HUD is drawn directly. Our canvas approach is equivalent but the draw order matters.
- [x] **F38 — Negative cell values are zombie depth animation**: When a cell value is negative, it represents a spawning zombie. Rendered as `images[12]` (ZOMBIE5) at `y - cellValue` offset. Each tick, the value increases by 3; once >= 0, it becomes zombie (cell=12). This creates a "rising from the ground" animation.
