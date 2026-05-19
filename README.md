# Voodoo Bowl

A faithful HTML5 recreation of the original **Voodoo Bowl** Java applet — a zombie football game from the late 1990s web!

You play as a quarterback navigating a field full of zombies, referees, and tombstones. Score touchdowns, dodge the undead, and survive the 60-second clock.

## Play

Open `voodoo_bowl.html` in any modern browser. No install, no server, no dependencies.

**[Play it on GitHub Pages](https://adamcroom.github.io/voodoo-bowl/voodoo_bowl.html)**

## Controls

| Action | Keyboard | Mobile |
|--------|----------|--------|
| Move right | Arrow Right / L | Right button |
| Move left | Arrow Left / J | Left button |
| Move up | Arrow Up / I | Up button |
| Move down | Arrow Down / K | Down button |

## How to Play

- Move the QB down the field to score touchdowns (7 pts each)
- Hit referees to turn them into tombstones (+2 pts)
- Avoid zombies — they'll sack you and cost yards
- Tombstones block your path permanently
- Get 10 yards for a first down, or lose possession after 4th down
- Difficulty increases after each touchdown — more zombies spawn

## History

Patrick Chan was a member of the Java platform team at Sun Microsystems and co-author of *The Java Developers Almanac*. He won the Duke Award at JavaOne in 1998. His personal site at xeo.com showcased Java applet work he had built for clients including Sun, JavaSoft, Oracle, Yahoo, and the Democratic National Convention.

One of those projects, listed on xeo.com as "Voodoo Football," was a promotional applet commissioned by Sun and the House of Blues for the Super Bowl. Chan's own description: "a football game where you try to get a touchdown while avoiding zombies and hitting referees." The game was built in Java 1.1 and packaged as a single class file (`Football.class`) inside a Windows Cabinet archive (`football.cab`), with a 500x280 sprite sheet and seven `.au` sound effects.

After the promotion ended, the game appeared on javagame.net under the name Voodoo Bowl and later on coffeebreakarcade.com, where it remained available for years. The Wayback Machine archived the javagame.net page between 2001 and 2023. The original `.cab` file and both asset files — `images.gif` and `bg.gif` — were preserved in the Internet Archive.

In 2026, the `.class` file (11,904 bytes, Java 1.0) was disassembled from bytecode, recovering the complete game logic: field layout, defender AI, the downs system, the 60-second clock, scoring rules, and the pixel coordinates of every sprite crop. The xeo.com portfolio page, also preserved in the Wayback Machine, confirmed Chan as the developer.

This repository is a reconstruction of that game in HTML5 Canvas, built from the disassembled bytecode and the original sprite assets.

### Sources

- [Patrick Chan — Wikipedia](https://en.wikipedia.org/wiki/Patrick_Peter_Chan)
- [xeo.com portfolio (May 1997) — Internet Archive](https://web.archive.org/web/19970513005208/http://www.xeo.com/)
- [Voodoo Bowl on javagame.net — Internet Archive](https://web.archive.org/web/20081119144728/http://www.javagame.net/games/voodoo)

## About the Reconstruction

The game is a single, self-contained HTML file. All 30 original sprites are extracted from `images.gif` and all 7 sound effects are converted from `.au` format — everything is embedded as base64 data URIs directly in the HTML. No external files needed.

### What's in the single file

- **30 sprites** extracted from the original 500x280 sprite sheet
- **7 sound effects** converted from Sun AU to AAC/M4A
- **Game logic** reverse-engineered from Java bytecode disassembly
- **HUD** rendered from the original `bg.gif` background strip

### Accuracy

The recreation is based on a complete disassembly of `Football.class` (11,904 bytes, Java 1.0). 38 specific differences were identified and corrected to match the original, covering:

- Spawn density, defender AI, and difficulty progression
- Directional sack sprites and tackle mechanics
- Viewport scrolling formulas and QB animation frames
- Endzone rendering, yard line placement, and score display
- Audio trigger mapping and timing
- Timer behavior during sack/touchdown pauses

## Tech

- Single HTML file (~230KB)
- HTML5 Canvas (568x280 game + 568x103 HUD)
- No frameworks, no build tools, no external assets
- `requestAnimationFrame` game loop at 333ms tick interval
- Works offline — just save the file

## License

This is a fan recreation of an original work. All original sprite art and sound effects belong to their respective creators.
