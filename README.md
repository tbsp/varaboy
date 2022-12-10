# Varaboy

Varaboy is a Game Boy emulator written in uxntal for the Varvara system.

## Special Thanks

- Devine Lu Linvega (and collaborators) for [uxn](https://100r.co/site/uxn.html) and [Varvara](https://wiki.xxiivv.com/site/varvara.html), the reason this exists at all
- Andrew Richards for [uxn32](https://github.com/randrew/uxn32), without which debugging this would have been a nightmare
- Calindro for [Emulicious](https://emulicious.net/) and all his guidance on this silly project
- binji for his writeup on [POKEGB](https://binji.github.io/posts/pokegb/), and the emulator code itself
- Everyone that's contributed to the [Pandocs](https://gbdev.io/pandocs/)
- Nintendo for making the Game Boy, which has consumed far too much of my life

## Features

- ROM/MBC1/MBC3/MBC5 games (only basic ROM/RAM banking support, extended bank bits not handled, no RTC for MBC3)
- Variable frameskip (default: 3), adjustable with number keys 0-9
- .theme file support
- No sound support

## Stability

Note that due to the incomplete MBC implementations and SRAM not being repacked unless you quit with the escape key I **strongly discourage** playing any game seriously with this emulator at this time. Games will almost certainly break, and save files will almost certainly be lost/corrupted.

## Usage

This emulator requires a [UXN emulator](https://100r.co/site/uxn.html) to run. The name of the Game Boy ROM to run is provided on the command line, for example:

```
uxnemu varaboy.rom tetris.gb
```

Note: Since [uxngb](https://github.com/tbsp/uxngb) (my UXN VM written for Game Boy) only supports up to 8KiB UXN ROMs, it's not possible to run this Game Boy emulator inside uxngb. It is entirely possible to run UXN ROMs 8KiB or smaller inside uxngb inside varaboy though!

## Screenshots

![cpu_instrs](https://user-images.githubusercontent.com/10489588/206598200-41defefa-eca2-4bd9-82ec-38a91720051f.png)
![dmg_acid2](https://user-images.githubusercontent.com/10489588/206787383-e8560e41-8bd5-411e-a122-65590a3451d3.png)
![sml](https://user-images.githubusercontent.com/10489588/206598205-648f7803-ef93-48bf-941c-fd6364377f26.png)
![tetris](https://user-images.githubusercontent.com/10489588/206598212-73f4ea28-b395-48c3-8b4e-1f3fa53ce066.png)
![megaman](https://user-images.githubusercontent.com/10489588/206598220-17d24d65-aad9-40ff-99aa-296304af2537.png)
![sml2](https://user-images.githubusercontent.com/10489588/206598226-99623113-ee35-4cfb-9c0d-18776744a003.png)
![megaman5](https://user-images.githubusercontent.com/10489588/206598232-b11c6275-c0e3-44f0-872e-68553066a54c.png)
![ffl3](https://user-images.githubusercontent.com/10489588/206787415-c6e668c9-e108-400f-b57e-59eb941aa9f0.png)
![shocklobster](https://user-images.githubusercontent.com/10489588/206598263-3e502dc6-36a7-42fd-a832-41fba7d70fa4.png)
![fruitpursuit](https://user-images.githubusercontent.com/10489588/206598268-7ce553ee-7a00-46e8-a54f-bbced9f2336c.png)
![deathplanet](https://user-images.githubusercontent.com/10489588/206598279-2072e63b-05da-40fb-a9f0-cc11933b387c.png)
![uxn_screen](https://user-images.githubusercontent.com/10489588/206768329-aad8e609-1f79-458d-9824-f29eb0f9bdbe.png)

## Compatibility

Note that unless otherwise noted, only a few minutes of testing per game was performed.

| Game | Notes |
| --- | --- |
| Tetris | Playable |
| Super Mario Land | Playable |
| The Legend of Zelda: Link's Awakening | Playable |
| Mega Man: Dr. Wily's Revenge | Payable |
| Mega Man V | Playable |
| Mario's Picross | Playable |
| Final Fantasy Legend III | Playable |
| Wario Land II | Playable |
| Super Mario Land 2: Six Golden Coins | Map corrupt, levels playable |
| Donkey Kong | Freezes after first level, visual glitches in cutscenes |
| Dr. Mario | Freezes when you try to start the game |
| [Shock Lobster](https://tbsp.itch.io/shock-lobster) (Homebrew) | Playable |
| [Fruit Pursuit Beta](https://tbsp.itch.io/fruit-pursuit) (Homebrew) | Playable |
| Adjustris (Homebrew) | Playable |
| [uxngb](https://github.com/tbsp/uxngb) (Homebrew) | Playable |
| [Death Planet](https://makrill.itch.io/death-planet) (Homebrew) | Playable |
| [Libbet](https://github.com/pinobatch/libbet) (Homebrew) | Playable |
| [Geometrix](https://github.com/AntonioND/geometrix) (Homebrew) | Playable |
| [Sam Mallard](https://snorpung.itch.io/sam-mallard-gb) (Homebrew) | Hangs on startup |
| [Quartet](https://makrill.itch.io/quartet) (Homebrew) | Works, but RNG is broken |

## How it works

Both the Game Boy and UXN use a 16bit address space ($0000-$ffff). The Game Boy has a large region of "echo RAM" from $e000 to $fdff, which mirrors the contents of WRAM (c000~ddff) and was considered off-limits for Game Boy software by Nintendo.

Varaboy starts at the UXN entry point ($0100, the same as the Game Boy entry point), sets up some basic stuff, reads the GB ROM header to load the appropriate MBC handler code, and then jumps to the main runtime code inside echo RAM. As long as we can fit all UXN runtime code inside echo RAM the Game Boy code is able to access the rest of memory using native addresses, which I find super fun!

Note that this means we don't have access to the UXN zero page, which contains the Game Boy RST and interrupt vectors.

## Performance

This emulator is *very slow*. On a Ryzen 5600X running in [uxnemu](https://sr.ht/~rabbits/uxn/) some games could almost be considered playable with a frameskip setting of 3 or so. Performance in [uxn32](https://github.com/randrew/uxn32) is much worse, making it very hard to get inputs to register at all (perhaps due to differences in how vectors are handled). A i5-540M with a frameskip of 9 isn't close to playable. Performance on the Nintendo DS UXN VM is even worse, which isn't surprising.

I've sped up instruction dispatch by using jump tables, which in certain cases "wastes" as much as 126 bytes for the ~64 "ld r8,r8" instructions, but overall I believe the performance gain is worth it. I've also tried to pre-calculate as much as possible in the PPU scanline renderer to reduce redundant calculations as I'm not considering mid-scanline register writes. Background/window tiles are cached for reuse for up to 8 pixels, which provides a slight performance gain, though the presence of that code also slows things down a bit, so the net gain isn't huge. In addition, several common operations (ticks, reads, etc) have been converted to macros for speed over size, though the gains are minor.

Save files are unpacked into a file per bank on startup for faster access during SRAM banking. The individual bank files are repacked on shutdown if you quit by pressing the Escape key. Quitting by closing the VM any other way will not properly write SRAM contents back to the SAV file. Without file seeking, games which use lots of ROM banks (and bank often) could also suffer a notable performance hit which could be reduced by unpacking ROM banks in a similar manner.

The only "big" idea I have to speed things up right now is:
- Write 8 full rows of pixels to a buffer of 20 2bpp UXN tiles and draw them to screen with two .Screen/sprite (auto) writes instead of 1280 .Screen/pixel writes. It's unclear if the extra VM instructions to juggle the buffering would be worth the reduction in .Screen calls though, and the benefits may vary by VM implementation.

In addition, I'm still very new at writing uxntal, so there are likely a whole bunch of smart optimizations which could be done to speed things up. Anything that could speed up the main CPU and PPU loops would likely yield huge speed benefits.

## Accuracy

- Passes [blargg's cpu_instrs tests](https://github.com/retrio/gb-test-roms)
- Fails [blargg's instr_timing test](https://github.com/retrio/gb-test-roms), possibly due to a flawed timer implementation which mis-measures instruction timing
- Passes most of Matt Currie's [dmg-acid2](https://github.com/mattcurrie/dmg-acid2), except for sprite x priority and 10 spr/line limit (which are not implemented)
- Fails most other test ROMs
- Still manages to run a surprising number of commercial/homebrew games, despite the above!
