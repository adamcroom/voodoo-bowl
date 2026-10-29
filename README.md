# Voodoo Bowl

A faithful HTML5 recreation of the original **Voodoo Bowl** Java applet — a zombie football game from the late 1990s web!

You play as a quarterback navigating a field full of zombies, referees, and tombstones. Score touchdowns, dodge the undead, and survive the 60-second clock.

## Play

Open `voodoo_bowl.html` in any modern browser. No install, no server, no dependencies.

**[Play it on GitHub Pages](https://adamcroom.github.io/voodoo-bowl/voodoo_bowl.html)**

## Controls

| Action | Keyboard | Mobile |
|--------|----------|--------|
| Move right | Arrow Right / l | Right button |
| Move left | Arrow Left / j | Left button |
| Move up | Arrow Up / i | Up button |
| Move down | Arrow Down / k | Down button |

As in the original applet, the letter keys are lowercase only (Caps Lock off), any key restarts after a game over, and the game shows "click here to play" whenever it doesn't have keyboard focus.

## How to Play

- Move the QB down the field to score touchdowns (7 pts each)
- Hit referees to turn them into tombstones (+2 pts)
- Avoid zombies — they'll sack you and cost yards
- Tombstones block your path permanently
- Get 10 yards for a first down, or lose possession after 4th down
- Difficulty increases after each touchdown — more zombies spawn

## History

Patrick Chan was a member of the Java platform team at Sun Microsystems and co-author of *The Java Developers Almanac*. He won the Duke Award at JavaOne in 1998. His personal site at xeo.com showcased Java applet work he had built for clients including Sun, JavaSoft, Oracle, Yahoo, and the Democratic National Convention.

One of those projects, listed on xeo.com as "Voodoo Football," was a promotional applet commissioned by Sun and the House of Blues for Super Bowl XXXI, held January 26, 1997 in New Orleans. The House of Blues sponsored the halftime show that year. Chan's own description: "a football game where you try to get a touchdown while avoiding zombies and hitting referees." The game was hosted on the House of Blues website at hob.com/football/, where it appeared in the site's "Interactive" games section alongside a puzzle, a blues guitar app, and a jukebox. It was built in Java 1.1 and packaged as a single class file (`Football.class`) inside a Windows Cabinet archive (`football.cab`), with a 500x280 sprite sheet and seven `.au` sound effects.

After the promotion ended, the game appeared on javagame.net under the name Voodoo Bowl and later on coffeebreakarcade.com, where it remained available for years. The Wayback Machine archived the javagame.net page between 2001 and 2023. The original `.cab` file and both asset files — `images.gif` and `bg.gif` — were preserved in the Internet Archive.

In 2026, the `.class` file (11,904 bytes, Java 1.0) was disassembled from bytecode, recovering the complete game logic: field layout, defender AI, the downs system, the 60-second clock, scoring rules, and the pixel coordinates of every sprite crop. The xeo.com portfolio page, also preserved in the Wayback Machine, confirmed Chan as the developer.

This repository is a reconstruction of that game in HTML5 Canvas, built from the disassembled bytecode and the original sprite assets.

### Sources

- [Patrick Chan — Wikipedia](https://en.wikipedia.org/wiki/Patrick_Peter_Chan)
- [xeo.com portfolio (May 1997) — Internet Archive](https://web.archive.org/web/19970513005208/http://www.xeo.com/)
- [Voodoo Bowl on hob.com (May 1998) — Internet Archive](https://web.archive.org/web/19980508141838/http://hob.com/football/)
- [Voodoo Bowl rules on hob.com (May 1998) — Internet Archive](https://web.archive.org/web/19980508151421/http://hob.com/football/play.html)
- [House of Blues Interactive section (Dec 1997) — Internet Archive](https://web.archive.org/web/19971212051114/http://www.hob.com/vanilla/)
- [Voodoo Bowl on javagame.net — Internet Archive](https://web.archive.org/web/20081119144728/http://www.javagame.net/games/voodoo)

## About the Reconstruction

The game is a single, self-contained HTML file. Every sprite is cropped from `images.gif` at the coordinates in the original `init()`, and the sound effects are decoded from the original `.au` files — everything is embedded as base64 data URIs directly in the HTML. No external files needed.

### What's in the single file

- **45 sprites** (31 game images, 10 digits, 4 down markers) cropped from the original 500x280 sprite sheet
- **6 sound effects** decoded losslessly from 8 kHz µ-law Sun AU to PCM WAV (the seventh, `-crowdsoft`, is loaded by the original but never played)
- **Game logic** reverse-engineered from Java bytecode disassembly
- **HUD** rendered from the original `bg.gif` background strip

### Accuracy

The recreation is based on a complete disassembly of `Football.class` (11,904 bytes, Java 1.0). The game script is a method-by-method port of that bytecode, using the original field names and constants. Two audit passes (38 items, then 33 more) are documented in `CHANGELOG.md`, covering:

- Spawn density, defender AI, and difficulty progression
- Directional sack sprites and tackle mechanics
- Viewport scrolling formulas and QB animation frames
- Endzone rendering, yard line placement, and score display
- Audio trigger mapping and timing
- Timer behavior during sack/touchdown pauses

## Tech

- Single HTML file (~400KB)
- One HTML5 Canvas, 568x389, laid out like the applet: a 568x286 field back buffer (field at offset 4,6) above the 560x103 `bg.gif` HUD
- No frameworks, no build tools, no external assets
- 333ms game tick (`setTimeout`, like the original's `Thread.sleep(333)`); redraws are requested the way AWT `repaint()` was
- Works offline — just save the file

## License

This is a fan recreation of an original work. All original sprite art and sound effects belong to their respective creators.
