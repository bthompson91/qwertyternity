# QWERTYTERNITY

An Eviternity-style raycaster running entirely in the browser. No build step, no libraries — one `index.html` (plus a Google Fonts link for the HUD typeface). It started life as QWERTYSTEIN (a Wolfenstein-style raycaster); it is now QWERTYTERNITY — Doom, but bigger, in the spirit of Eviternity: epic scale, grand halls, new monsters, and a heavenly Astral episode. Walls are Doom-style 64×64 texture slices, enemies are drawn sprites, and the floors have Doom-style sector heights (raised platforms, a boss dais). And it looks the part now: every episode has its own scrolling sky (the Astral episode's is a pink cloudy heaven), the walls carry glowing teal inlay and gold runes, and every floor is an outdoor-first map — you spawn in a big octagonal plaza with diagonal walls, glowing pillars and trees, with indoor rooms attached. The human enemies are knights (plate armour, great helms, crossbows) and the dogs are lions.

Play it live: https://bthompson91.github.io/qwertyternity/

## How to play

Open `index.html` in a browser, click, press Enter. Or play live at https://bthompson91.github.io/qwertyternity/

- **WASD** move, **mouse** look (click to lock), **← →** turn, **mouse Y / ↑ ↓** look up/down, **Space / J** jump
- **Click / Ctrl** shoot (Ctrl is the keyboard trigger; click is the mouse trigger)
- **1–8** weapons: Fist, Pistol, Shotgun, Super Shotgun, Chaingun, Rocket Launcher, Plasma Gun, BFG9000
- **F** meat hook (Super Shotgun), **Shift** run, **Q** dash, **T** throw a bomb
- **E** use (doors, pushwalls, elevator) / glory kill, **P** pause, **M** mute, **N** new run (on death)

> **Ctrl+W warning:** Chrome reserves Ctrl+W to close the tab and a page can't stop it — so don't hold Ctrl to shoot while pressing W to move forward, or the tab closes. Shoot with the mouse click while moving, or use Ctrl only when you're not pressing W.

### Phone

On a touchscreen you get on-screen controls: a **MOVE** virtual stick (left pad), a **LOOK** pad (drag to turn and look up/down, tap to fire), and FIRE / DASH / BOMB / JUMP / WEAPON / HOOK / PAUSE buttons. You can also touch the game itself: drag the left half to move, drag the right half to look, tap the right half to fire.

### Joypad

Yes — any standard gamepad works via the browser Gamepad API, no install needed. Plug in an Xbox, PlayStation, Switch Pro or 8BitDo pad (USB or Bluetooth) and press a button; the game shows its name in the HUD.

- Left stick: move · Right stick: look (push up/down to look up/down)
- **RT or A**: fire (hold) · **X**: meat hook (Super Shotgun) · **B**: dash · **Y**: bomb
- **LB / RB**: change weapon · **Start**: pause · **Back/Select**: mute
- Upgrade screens: **LB / RB** to pick, **A** to confirm · Title / death screens: **A** to start a new run

## Procedural levels

Every floor is generated from the run seed: you spawn outdoors in a big octagonal **plaza** (16–22 × 12–16 tiles, its corners chamfered off with 45° diagonal walls, 3–5 trees, glowing pillars, a platform), with up to 16 indoor rooms attached by corridors. The plaza has a 2.5-tall open sky above it; indoors the ceiling is flat and dark. Enemy count, health and damage scale with depth. The exit portal is always in the room farthest from your spawn.

Floors run in **Eviternity episodes**, one theme per 4 floors: **Episode 1 — UAC Tech Base** (floors 1–4), **Episode 2 — Hell's Gate** (5–8), **Episode 3 — Gothic** (9–12), **Episode 4 — Astral** (13–16, the heavenly finale, with its own bright gold-and-white textures and an original heroic track), then the cycle repeats. A boss spawns every third floor: **the Knight Lord** on floors 3, 9, 15… and **the Annihilator** — Eviternity's big Baron — on every 6th floor.

## Roguelike progression

- **Permadeath.** Die and the run is gone — new seed, new castle.
- **XP and levels.** Kills give XP. On level-up, pick 1 of 3 upgrades (max HP, damage, fire rate, speed, armor, lifesteal, crit, bombs, dash recharge, regen, XP gain, ammo).
- **Floor-clear upgrade.** Clearing a floor (reaching the exit portal) also offers 1 of 3 upgrades before you descend.
- **Loot.** Health, ammo, treasure (+score), bombs and keys drop from rooms and dead enemies.
- **Best run** (deepest floor, score, kills) is kept in `localStorage`.

## Tech

Raycasting (DDA, one ray per character column, fisheye-corrected) drawn straight to a canvas. Walls are Doom-style 64×64 texture slices — brick, stone, wood, metal, vault, door, pushwall — one 1px-wide slice per ray column, pre-baked in 4 distance-shade levels and drawn nearest-neighbour so they stay crunchy. (The earlier ojread/ascii-fps stretched-character walls shimmered: the dithered glyphs moiré'd as they scaled and fractional column widths left seams, so they were replaced with real textures.) Above the walls, the ceiling is a scrolling sky panorama — a 2048×256 texture per episode, gradient + soft clouds, that pans with your view and sits on the horizon: dark blue-grey for the tech base, blood-red for Hell, grey cloud for Gothic, and a pink cloudy heaven for Astral. The floor is a distance-shaded band with a whisper of speckle grain. Floors have Doom sector heights: 1–2 raised platforms per floor (a 0.5 step you walk straight up) in tall rooms, and a dais under the boss on every 3rd floor — your eye rides the floor you're standing on, walls take their base and ceiling from the room you're in, and the floor is redrawn in strips where the height changes so the step faces read as dark risers. Enemies are drawn 3D-style billboard sprites (muted colours, hard dark outline, slit eyes under heavy brows, rifle aimed at you), clipped per column against the wall depth buffer, with HP bars and a "!" when staggered; guns are Doom-style drawn viewmodel sprites with a muzzle flash. The scene is rendered to an offscreen canvas larger than the view (overscan) and view-bob just pans the window inside it, so bobbing never exposes a black border. The HUD is drawn in Bungee Tint. The minimap is a rotating radar (player facing is up) with fog of war, top-left, half the old size.

## Wolfenstein 3D logic

The gameplay logic is ported from the real Wolfenstein 3D source (id Software, 1992; source released 1995, `WOLFSRC/` in id-Software/wolf3d). The code here is an original JS implementation of those mechanics — the design, constants and state names are Wolf3D's, not a verbatim copy.

Ported, with file references in the code:

- **Maps** (`WL_DEF.H`, `WL_GAME.C`): 64×64 grid (`MAPSIZE 64`), tile codes for walls, doors (90–101), pushwalls (98), elevator (21), exit (99), ambush (106).
- **Enemies** (`WL_ACT2.C`): **Knight** 25 HP / 100 pts, **Knight Captain** 50 / 400, **Dark Knight** 100 / 500, **Lion** 1 / 200, **Hedge Knight** 55 / 700, **the Knight Lord** 850 / 5000; drops (clip +4, the Knight Lord drops the gold key). Eviternity monsters: **Nightmare Demon** (from floor 2, 40 HP — a shadowy spectre that flickers in and out of sight, fast, bites), **Astral Cacodemon** (from floor 5, 120 HP — a floating red sphere that spits a spread of 3 fireballs), **the Annihilator** (every 6th floor, 1200 HP — a bulkier horned Baron that hurls green hellfire).
- **Enemy AI** (`WL_STATE.C`): `T_Stand` / `T_Path` / `T_Chase` / `T_Shoot` / `T_DogChase` / `T_Bite` state machine, `SightPlayer` reaction delays, `FirstSighting` speed multipliers, `CheckLine`, attack chance `(tics<<4)/dist`, shoot hit-chance and damage tables by distance, pain state (10 tics, dogs have none), damage doubled on unaware enemies.
- **Weapons** (`WL_AGENT.C`): `GunAttack` hitscan pick + damage by distance (`dist<2: Rnd/4, dist<4: Rnd/6`), `KnifeAttack` (1.5 tiles, `Rnd>>4`), `ClipMove` wall sliding, `Cmd_Use` on **E**.
- **Doors** (`WL_ACT1.C`): sliding doors, open ~3–4 s (`OPENTICS 300`), re-open if blocked, locks 1–4 needing keys, enemies open them too.
- **Pushwalls** (`WL_ACT1.C`): press E facing a pushwall (tile 98) and it slides 2 tiles, revealing a secret alcove with treasure.
- **Scoring / pickups** (`WL_AGENT.C`): treasure cross 100, chalice 500, bible 1000, crown 5000; first aid +25, food +10, clip +8 (cap 99); every 40,000 points (`EXTRAPOINTS`) grants an extra life, which revives you at 50 HP.

Deliberate differences from Wolf3D: levels are procedurally generated instead of hand-made, runs are roguelike (permadeath, XP levels, upgrade picks between floors), and the screen is a canvas raycaster instead of textured walls. Enemy guns are nerfed from Wolf3D's tables (hit chance and damage roughly halved) — the originals were far too strong for this game's pacing. The chaingun/SMG fire faster than Wolf3D's 24-tic shots — that's the roguelike talking.

## Doom movement (linuxdoom-1.10)

Movement is Doom's, ported from the real source (`id-Software/DOOM`,
`g_game.c` / `p_user.c` / `p_mobj.c` / `p_local.h`), with 1 tile = 128
Doom units. Fixed 35 Hz tics: thrust, then move, then friction 0.90625
per tic, hard stop below STOPSPEED. Terminal speeds: walk 2.28 tiles/s,
run 4.56, strafe 2.19 / 3.65. It ramps (50% in 0.2s, 90% in 0.67s) and
slides ~1.26 tiles after a run stop, instead of the old instant
3.4 / 5.1 tiles/s. Keyboard turning is Doom's too: 61.5 deg/s for the
first 6 tics held, then 123 deg/s walk, 246.1 run. View bob follows
`P_CalcHeight` (speed^2 capped at MAXBOB 16 units, 0.57s cycle, up to
+/-8 units) but is rendered at ~40% amplitude through the overscan
(see Tech) — the old full bob was too much and exposed black borders.
Weapon bob on the 1.83s `A_WeaponReady` cycle. Dash is a
momentum impulse, not a speed multiplier. Player radius stays Wolf3D's
0.34 tiles — the maps are Wolfenstein-scale.

## Verticality

Mouse Y, ArrowUp/ArrowDown, the touch LOOK pad and the gamepad right
stick pitch the view up and down (about 23 degrees each way) by
moving the horizon; **Space / J** jumps with simple gravity (about
0.4 tiles high), which shifts the walls and floor through the eye
height. And the floors themselves have height, Doom-style: each floor
has 1–2 raised platforms (a 0.5 step you walk straight up, no jumping
needed) in tall rooms with 1.5 ceilings, and a dais under the boss on
every 3rd floor. Your eye rides the floor you're standing on, walls
take their base and ceiling from the room you're in, and the step
faces are drawn as dark risers. And every floor has an outdoor **plaza** — a huge open octagon with a 2.5 ceiling (the sky), the tallest space in the game — with indoor rooms attached. There are no stairs or multi-storey floors — it's Doom's sector heights, not a second storey.

## Unicode + ASCII art

Walls are 64×64 textures (brick courses, stone blocks, wood planks,
metal panels with rivets, bevelled vault blocks, a sliding door with a
barred window, a pushwall with a carved statue face), tinted from the
floor theme's palette, drawn one 1px-wide slice per ray column and
nearest-neighbour so they stay crunchy. Each Eviternity episode has
its own OTEX-style textures: the tech base uses clean panels with a
glowing trim, Hell and Gothic use a pointed-arch gothic texture and a
pillar texture — dark stone columns with glowing teal inlay and a gold
rune — and the Astral episode uses bright marble with gold veins and a
gold rune. The door is a glowing gold portal. The plaza on every
floor has four glowing pillars at its corners, and the Astral and
Gothic rooms have trees. Enemies and guns are drawn
sprites, not ASCII: knights in plate armour with great helms and crossbows (plain steel, gold-trimmed captain, black Dark Knight, green Hedge Knight, gold Knight Lord), a lion in profile with a big mane and a tail tuft — plus Eviternity's
own monsters: a shadowy Nightmare Demon, a floating Astral Cacodemon,
and the horned Annihilator — and Doom-style
viewmodels for all eight weapons. Items (first aid, food, clip,
shells, rockets, cells, cross, chalice, bible, crown, gold key, bomb,
elevator exit) are pixel-art icons rasterised from the old ASCII sprites.

## Doom Eternal glory kills

Damage builds a stagger meter. At 50% of an enemy's max HP in one
stretch (the meter decays when you're not hitting them) the enemy
staggers for 3 seconds: frozen, defenceless, flashing gold with a
"!" over its head, and the HUD shows "STAGGERED — press E for GLORY KILL".
Walk up and press E in melee range to execute: a 0.9s locked animation
with a big " GLORY KILL " banner and a per-type execution line
(SNAPPED HIS NECK / HELMET CRUSH / RIBCAGE RIP / SKULL SPLIT /
PUNTED INTO NEXT WEEK), +15 HP, a food drop at the corpse, +100 bonus
score on top of the kill, and 1.5s invulnerability — Eternal's
execution i-frames. The Knight Lord can't be staggered. Glory kills are
the health economy, like Eternal: the counter is in the HUD.

## Doom weapons (all 8 originals) + Eternal meat hook

The old Knife / Pistol / SMG / Shotgun / Chaingun set is replaced by
all eight original Doom weapons, with Doom's exact damage tables
(step x (1 + rndT()%n)), Doom's tic rates and Doom's four ammo types
with Doom's caps (bullets 200, shells 50, rockets 50, cells 300).
Doom weapons have no magazines and no reload — they fire straight
from the reserve, and semi-auto weapons (fist, pistol, shotgun, SSG,
rocket, BFG) need a fresh press per shot while the chaingun and
plasma gun fire while held.

- **1 Fist** — 2-20, melee, no ammo.
- **2 Pistol** — 5-15, 1 bullet.
- **3 Shotgun** — 7 pellets x 5-15, 1 shell.
- **4 Super Shotgun** (Doom 2) — 20 pellets x 5-15, 2 shells.
- **5 Chaingun** — 5-15, 1 bullet, auto.
- **6 Rocket Launcher** — projectile, 20-160 direct + blast up to
  128 falling off over 2 tiles, 1 rocket.
- **7 Plasma Gun** — projectile, 5-40, 1 cell, auto.
- **8 BFG9000** — projectile, ball impact 100-800 + 40 invisible
  tracers at 15 each in a 45-degree cone out to 12 tiles, 40 cells.

Ammo pickups use Doom's values: clip +10 bullets, shell box +4,
rocket box +5, cell pack +20. SS enemies drop shells. Starting ammo:
50 bullets, 16 shells, 10 rockets, 80 cells.

**Meat hook (Doom Eternal Super Shotgun mod)** — press **F**
(gamepad X, HOOK button) with the Super Shotgun equipped: a chain
latches onto the nearest enemy within 12 tiles in your facing cone
and pulls you to it at 14 tiles/s, dealing 20 damage and setting it
burning (Flaming Hook mastery: a burning enemy that dies drops +15
armor). 3s cooldown, 2s if it misses. The chain is drawn as dots
across the screen while you're latched.

Deliberate differences from Doom: no rocket self-damage (close
rockets don't hurt you); SSG rate is 1.0s not Doom's ~1.5s pump
cycle; rocket blast radius is 2 tiles not Doom's 1; each BFG tracer
hits an enemy once (in Doom all 40 can pile onto one target).

## MIDI music and sound effects

Four original compositions, embedded as base64 Standard MIDI Files
and played by a tiny SMF parser + Web Audio General-MIDI-ish synth
written for the game — no audio files, no libraries, no network.
- **Title** — 80 BPM, A minor, string pads + piano arpeggio.
- **Combat** — 150 BPM, D minor, overdriven-guitar riff, synth bass,
  kick/snare/hats. Plays on normal floors.
- **Boss** — 170 BPM, E minor, faster riff, syncopated kick. Plays on
  every 3rd floor (boss floors).
- **Astral** — 140 BPM, D major, a bright heroic loop (lead, driving
  bass, pad, drums) written for the Astral episode. Plays on floors
  13–16 (and every 4th episode after that).
Music starts on the first key/click (browsers need a gesture for
audio), switches per floor, and stops on death. M mutes everything.

Sound effects are MIDI note sequences through the same synth:
shoot, shotgun, hit, hurt, pickup, level-up fanfare, death descent,
exit fanfare, glory-kill power chord, meat-hook riff, explosion.
The old `beep()` calls still work — frequency is converted to the
nearest MIDI note. Known limit: browsers throttle timers in a hidden
tab, so music can stutter if you switch away mid-run.

## Fullscreen + Doom/Wolf HUD

- **Fullscreen:** a [ FULLSCREEN ] button under the game (or F11) — when the browser is fullscreen the
  canvas resizes to fill the screen, keeping the game's 1080x600
  aspect (letterboxed) and scaling the character cells with it
  (up to 2x device pixel ratio for sharpness). Leaving fullscreen
  restores the in-page 1080x600 canvas. The page chrome (title, help
  text) hides in fullscreen; touch controls overlay the bottom.
  Clicking the game locks the mouse pointer (a click that re-locks
  does not fire); double-clicking no longer toggles fullscreen — it
  used to flip in and out of fullscreen every time you double-clicked
  to shoot.
- **Bottom HUD:** a Doom/Wolf3D-style status bar along the bottom of
  the canvas, drawn in Bungee Tint (Google Fonts) — Doom's face in
  the middle (healthy, hurt, low, critical, just hit, glory kill,
  dead), HEALTH and ARMOR with big numerals and bars on the left,
  weapon + ammo in the middle-left, SCORE / KILLS / GOLD / FLOOR /
  level / lives / glory kills / keys / ammo totals on the right, the
  current song (COMBAT / BOSS / TITLE) or MUTED, and an XP bar along
  the very bottom edge. The face stays monospace (Bungee Tint has no
  box-drawing glyphs). The HUD never bobs — only the 3D view moves.
