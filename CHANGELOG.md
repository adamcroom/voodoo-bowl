# Voodoo Bowl — Bytecode-Accurate Corrections

Full disassembly of `Football.class` revealed 38 differences between the HTML5 recreation and the original Java applet, grouped by category below (sections 6A–6F). A second audit (October 2026) re-verified those items against the bytecode, corrected several entries, and added a new working list of open items (section 7, G01–G33).

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
- [x] **F12 — Sack direction in keyDown (QB walks into zombie)**: `field[zombie_pos] = 0` (clear zombie). Then based on direction: east→SACKE(17) at old QB pos, west→SACKW(18) at zombie pos, south→SACKS(16) at old QB pos, north→SACKN(15) at zombie pos. *(Corrected in audit: SACKN goes at the zombie's cell `(qbX, ny)`, not the old QB pos. Rule of thumb: every sack sprite is anchored at the top/left cell of the pair. Code already does this correctly.)*
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
- [x] **F24 — CLICKHERE shown when game over OR lost focus**: The click-here prompt appears both on game over AND when the applet loses focus (`!hasFocus`). Not just game over + start screen. *(Reopened in audit, then fixed by G21.)*
- [x] **F25 — Grass sprites are from the sprite sheet center, not left column**: `GRASS1=cropImage(200,160,40,40)`, `GRASS2=cropImage(200,200,40,40)`, `GRASS3=cropImage(200,240,40,40)`. These are 40x40 sprites from x=200. Our grass was extracted from the left column.
- [x] **F26 — BLANK sprite for leading zeros**: `images[28] = cropImage(400, 200, 14, 20)` — a blank 14x20 sprite used instead of leading zeros in TIME, SCORE, and YARDS displays.
- [x] **F27 — Score is 3 digits (hundreds/tens/ones)**: Score display uses 3 positions at 14px spacing. Leading digits show BLANK when zero.

## 6D. Audio Fixes

- [x] **F28 — Crowd noise is random ambient**: In the `run()` loop, `random(12)==0` triggers `audioClips[5]` (CROWDLOUD) each tick. That's ~1/12 chance per 333ms tick = roughly every 4 seconds on average. NOT a one-shot at game start.
- [x] **F29 — Game over plays YELL, not whistle**: Original plays `audioClips[6]` (YELL) on game over. Our code plays whistle. *(Reopened in audit, then fixed by G06.)*
- [x] **F30 — Touchdown plays SCORE sound**: Original plays `audioClips[3]` on touchdown, which maps to the "score" audio name. Not "touchdown" + "crowdloud".
- [x] **F31 — Audio name mapping**: `[0]=tackle, [1]=hike, [2]=whistle, [3]=touchdown, [4]=-crowdsoft, [5]=crowdloud, [6]=yell`. The indices don't match our sound trigger names. *(Corrected in audit: the `audioNames` array in `<init>` has whistle at [2] (HIT_REFEREE) and yell at [6] (game over); the earlier entry had them swapped. `-crowdsoft` [4] is loaded but never played.)*

## 6E. Movement/Viewport Fixes

- [x] **F32 — QB starts at x=-1 (off-field)**: `qbPt = Point(-1, vcells/2)`. First key press moves QB onto the field at x=0. *(Corrected in audit: -1 is only a sentinel. `newGame()` immediately calls `newDown()`, which sees `qbX < 0` and sets `qbX = 0`, so the QB starts at x=0 in the left endzone. Code already does this correctly.)*
- [x] **F33 — Viewport scroll logic differs**: Moving left: viewport scrolls when `viewPt.x > 0 AND qbX < hcells - viewwidth*3/4`. Moving right: viewport scrolls when `qbX >= viewwidth/4 AND viewPt.x < hcells - viewwidth`.
- [x] **F34 — QB frame animation**: Frames use images[23+qbFrame]. Up/Down set qbFrame=4. Left alternates 2↔3. Right alternates 0↔1. Our code cycles 0-4 on every move.
- [x] **F35 — qbDepth animation**: On new down (not TD), `qbDepth = cellsize - 3 = 37`. Each tick, `qbDepth = max(0, qbDepth - 7)`. QB is drawn at `y + qbDepth` — a "dropping in from above" animation. *(Corrected in audit: the sprite is offset downward and clipped to its cell, so it rises up out of the ground like a zombie, not drops from above. Code already does this correctly.)* Our code may not handle this correctly.
- [x] **F36 — Defenders move even during game over**: In `run()`, `moveDefense()` is called when `gameOver` is true OR `startTime > 0`. Zombies keep moving after the game ends. Our code stops them.

## 6F. Rendering Pipeline

- [x] **F37 — Double-buffered rendering**: Original draws to `bbuf` (back buffer), then copies to screen. Field is drawn to bbuf, HUD is drawn directly. Our canvas approach is equivalent but the draw order matters.
- [x] **F38 — Negative cell values are zombie depth animation**: When a cell value is negative, it represents a spawning zombie. Rendered as `images[12]` (ZOMBIE5) at `y - cellValue` offset. Each tick, the value increases by 3; once >= 0, it becomes zombie (cell=12). This creates a "rising from the ground" animation.

## 7. October 2026 Audit (all resolved)

Found by re-disassembling `Football.class` and comparing every embedded sprite pixel-by-pixel against the `cropImage()` coordinates in `init()`. Line numbers refer to `voodoo_bowl.html` at commit `863c263`. Ordered by impact within each group.

**Resolution:** rather than patching each item, the game script was rewritten as a method-by-method port of the disassembled bytecode (`newGame`, `newDown`, `stopPlay`, `keyDown`, `mouseDown`, `moveDefense` (both overloads), `run`, `update`), using the original field names and constants. Every sprite is now cropped fresh from `images.gif` at the `init()` coordinates and keyed by its bytecode constant name, `bg.gif` is embedded unscaled, and the six sounds the game plays are decoded losslessly from the µ-law `.au` files to PCM WAV. The game is a single 568x389 canvas (the 286px field back buffer above the 103px HUD). Line numbers below refer to the pre-rewrite file.

### 7A. High Impact

- [x] **G01 — First down of each game uses the wrong density**: `<init>` sets `density = 0.11`, not 0.0. `newGame()` calls `newDown()` *before* setting `density = 0.15`, so the first down of the first game spawns `(int)(39*7*0.11) = 30` zombie attempts, and the first down of every later game uses the previous game's final density (`0.15 * 1.1^TDs`). Our code forces density to 0 for that down and spawns no zombies (lines 159–165). Density must persist across `newGame()` calls.
- [x] **G02 — Zombie AI gating is restructured**: Original is two-stage. Outer (`moveDefense()`): if defender `x < qbX` and `random(2)==0` → move; otherwise if `random(5)==0` → move (the second check also applies to defenders behind the QB that failed the first roll). Inner (`moveDefense(c,x,y)`): if `!gameOver` and the defender is a zombie (8–12) and `random(2)==0` → chase, regardless of which side of the QB it is on; otherwise step in a random direction. Our code (lines 565–597): (a) zombies ahead of the QB never chase, (b) refs chase (they never do in the original), (c) odds behind the QB are 50% chase / 10% random instead of 30% / 30%.
- [x] **G03 — Rising-zombie spawn position is wrong**: Original `x = qbX + random(viewX + viewwidth - qbX)`. Our code adds an extra `G.viewX +` (line 552), so once the view scrolls, spawns land off the right of the screen where they never rise (only visible columns are updated).
- [x] **G04 — Defense runs when it shouldn't**: Original `run()` calls `moveDefense()` only when `pauseTime == 0`, `qbDepth == 0`, and (`gameOver` or `startTime > 0`). No zombies rise or spawn before the snap, during sack/TD pauses, or during the QB rise animation. Our loop calls `moveDefenders()` every tick while not game over (lines 694–698). Restructure the tick to: `if pauseTime>0 { if now>pauseTime newDown(); } else if qbDepth>0 { qbDepth=max(0,qbDepth-7) } else if gameOver || startTime>0 { moveDefense() }`, then crowd roll, then `tick++`.
- [x] **G05 — SACKS sprite was altered and is now an invalid PNG**: Commit `9d6f368` erased the tackled QB's legs from the bottom third of SACKS as "debris", but those pixels are real art in `images.gif` at `(280,0,40,80)`. The edit also left a bad zlib checksum, so the PNG fails strict decoders. Re-extract SACKS directly from `images.gif`. (The other 3 sack sprites match exactly.)
- [x] **G06 — Referee and game-over sounds are swapped**: Original `audioNames` is `[2]=whistle` (HIT_REFEREE) and `[6]=yell` (game over). Our code plays `yell` on a ref hit (line 496) and `whistle` on game over (line 652). Swap them. (See corrected F29/F31.)
- [x] **G07 — SACKW drawn one cell too far left**: Original draws every sack sprite at its own cell origin and lets it extend right/down. Our code draws SACKW at `px - CELL` (line 284). Draw at `px`.

### 7B. Medium Impact

- [x] **G08 — HUD background is stretched and digits are mispositioned**: Original draws `bg.gif` at its natural 560x103 at x=4. Our code stretches it to 568 wide (line 379). Digit positions in HUD-canvas coordinates should be: TIME `(89,34)` and `(103,34)`; SCORE `(75,54)`, `(89,54)`, `(103,54)`; DOWN sprite `(130,69)` (`ytdPt.y - 6`); YARDS starting at `(171,75)`. Spacing is 14px, not 15. Ours: time `(100,33)`, score `(100,53)`, down `(130,72)`, yards `(170,73)`.
- [x] **G09 — HUD number formatting**: Original uses BLANK (images[28]) for leading zeros at fixed positions. TIME is 2 positions (tens blank if 0). SCORE is 3 positions, right-aligned (hundreds/tens blank if 0). YARDS: if `scrimage + toFirstDown >= hcells-1` (goal to go) draw two BLANKs; otherwise if `toFirstDown > 9` draw a literal `1` then advance 14px, then draw `toFirstDown % 10`, then a BLANK after it. (Original quirk: 20+ yards displays as `1x`. Reproduce it.) Our `drawNumber()` left-aligns variable-length strings and never shows goal-to-go as blank.
- [x] **G10 — Clock rounds the wrong way**: Original shows `max(0, (timeLimit - used) / 1000)` with integer (floor) division, so it reads 59 immediately after the snap and 0 during the final second. Our code uses `Math.ceil` (line 387).
- [x] **G11 — Time-expired check must run before the touchdown reset**: In `newDown()` the game-over check (`down==4 && remaining>0 || usedTime > timeLimit`) comes first; if the game ends during a TD celebration, `touchdown` is cleared and the game is over. Our `endDown()` handles the TD branch first (lines 636–647), so a TD scored after time expires grants another down. Also use strict `usedTime > 60000`.
- [x] **G12 — QB row not reset after a touchdown**: Original `newDown()` always sets `qbY = vcells/2` before branching. Our TD branch leaves the QB in the row where it scored, and the protective 3-zombie wall spawns around that row.
- [x] **G13 — Walking into a rising zombie is a sack**: Original `keyDown` sacks when the target cell is 8–12 **or negative**. Ours only checks 8–12 (line 462), so the QB can step onto a rising zombie.
- [x] **G14 — Blocked moves must be no-ops**: Original computes the new position (clamped at edges); if it equals the current position, nothing happens at all. Clock start and HIKE only happen after the tombstone/sack checks pass, i.e. on a successful move. Our `doMove()` starts the clock and plays HIKE first (lines 446–449), and treats a wall-bump as a move onto its own cell (which can even "hit" a ref standing on the QB's cell for +2).
- [x] **G15 — Viewport scrolls one cell early**: Original tests the **pre-move** `qbX`: right scrolls if `qbX >= viewwidth/4` (3), left if `qbX < hcells - viewwidth*3/4` (30). Ours tests post-move `nx` (lines 502–512), so the QB rides in screen column 2 instead of 3.
- [x] **G16 — Draw order**: Original order into the back buffer: black fill → field color → endzones → eye (clipped to field) → yard lines → for each **row** y, for each column x: grass then field cell → QB / coffin+gameover → TOUCHDOWN → first-down MARKER → click-here. Ours draws the eye over the yard lines (line 418), the marker before the sprites (line 247), and cells column-by-column (changes how 80px sack sprites overlap neighbors).
- [x] **G17 — Don't erase grass under sacks**: Original never modifies `grassField` after `newGame()`. Our sack code zeroes grass cells permanently (lines 465–468, 606–609). Remove; correct overlap comes from G16's row-major order.
- [x] **G18 — Referees animate when they move**: Original toggles any moved defender's frame (`c != 12`: even→c+1, odd→c−1), so refs flip 6↔7. Ours only toggles zombies (line 623).
- [x] **G19 — Abort defense loop after a zombie sack**: In original `moveDefense()`, a sack sets `startTime = 0`; the next zombie/ref encountered then breaks out of the scan loop (remaining rising zombies are not advanced) and jumps to the spawn step. Ours continues scanning.

### 7C. Start, Focus and Input

- [x] **G20 — No "waiting" start screen**: Original has no start state: the clock and crowd noise run from load, the QB stands at x=0 and the first arrow key moves it. Click-here is not shown at startup if the applet has focus. Remove `G.waiting` and the start-screen click-here (lines 716–747), and let the tick loop run from load.
- [x] **G21 — Focus-loss overlay (completes F24)**: Track canvas `focus`/`blur` (or window focus) as `hasFocus`; draw click-here when `gameOver || !hasFocus`. Clicking only restarts when `gameOver` (it otherwise just grants focus).
- [x] **G22 — Any key restarts after game over**: Original `keyDown`: if `pauseTime > 0` ignore; set `qbDepth = 0`; if `gameOver` → `newGame()` on **any** key. Ours only accepts arrows/IJKL/Space/Enter.
- [x] **G23 — Any key cancels the QB rise animation**: `keyDown` sets `qbDepth = 0` before anything else.
- [x] **G24 — QB frame advances even when the move is blocked**: Frame changes (up/down → 4, left 2↔3, right 0↔1) happen in the key switch before collision checks, so they apply on wall/tombstone/zombie bumps too.
- [x] **G25 — `^` toggles noTouchMode (hidden cheat)**: Key code 94 toggles `noTouchMode`. While on: QB walking into a zombie just moves onto its cell, and zombies moving into the QB don't sack (they also don't move onto the QB's cell, because the cell isn't empty). Tombstones still block.

### 7D. Minor Accuracy

- [x] **G26 — Initial tombstone count and order**: `newGame()` loops `i < 280*0.02 (5.6)`, i.e. **6** placements, with no empty-cell check, and runs **after** `newDown()`, so tombstones can overwrite freshly spawned zombies/refs/wall cells. Ours places 5 first.
- [x] **G27 — Rising-zombie start value**: Original spawns at `-cellsize + 2 = -38` (13 ticks to rise). Ours uses `-40 + rnd(2)` (line 556).
- [x] **G28 — First-down marker range**: Draw only when `viewX < marker < viewX + viewwidth - 1` (screen columns 1–12). Ours also draws at column 13.
- [x] **G29 — Touchdown updates the HUD immediately**: `keyDown` sets `down = 1` and `toFirstDown = 10` the moment the TD is scored (scrimmage stays put, so yards shows blank/goal during the celebration). Ours waits until `endDown()`.
- [x] **G30 — Field color**: `bgColor = new Color(102,102,0)` = `#666600`, not `#666700`.
- [x] **G31 — Applet framing**: Original field is at `fieldOffset = (4,6)`: a 6px black band above the field and a 4px left margin, with the HUD at y=286. Ours draws the field at (0,0) and leaves 8px on the right.
- [x] **G32 — Game-over pause tick**: Original game over calls `stopPlay(0)` (sets `pauseTime = now`), so keys are ignored and the coffin appears one tick later. Ours sets `pauseTime = 0` immediately. Barely visible; include for completeness.
- [x] **G33 — Tick timing and audio latency**: Use a fixed 333ms timer (accumulator or `setInterval`) instead of rAF frame thresholds, which yields ~350ms ticks at 60Hz. The embedded sounds are 8kHz AAC with no edit list, so each clip likely starts 0.25–0.4s late. Decode the µ-law `.au` files to PCM WAV (lossless, ~2x size) instead.

### Cleanup (not accuracy)

- Unused sprites can be dropped from `SPRITES_B64`: `goalpost`, `explode1`, `explode2`, `qb_big1`, `qb_big2`, `qb_still`, `ball`, `endzone_L`/`endzone_R` (duplicates of `endzone_l`/`endzone_r`), plus the unused `crowdsoft` sound.
- Sprite keys are mislabeled (`qb*` = zombies, `def*` = QB, `tomb*` = referees, `ref*` = tombstones). Renaming them to the bytecode constant names (`ZOMBIE1`, `QB1`, `REFEREE1`, `TOMBSTONE1`, ...) would remove the swap comments.
