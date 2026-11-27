# QWERTYSTEIN

A Wolfenstein-style raycaster running entirely in the browser. No build step, no libraries — one `index.html` (plus a Google Fonts link for the HUD typeface). Walls are rendered the ojread/ascii-fps way: one ASCII character per ray column, stretched to the wall's full height at its regular width, so wall tops and bottoms are pixel-smooth. Enemies and guns are drawn sprites, Doom-style.

Play it live: https://bthompson91.github.io/qwertystein/

## How to play

Open `index.html` in a browser, click, press Enter. Or play live at https://bthompson91.github.io/qwertystein/

- **WASD** move, **mouse** look (click to lock), **← →** turn, **mouse Y / ↑ ↓** look up/down, **J** jump
- **Click / Space** shoot
- **1–8** weapons: Fist, Pistol, Shotgun, Super Shotgun, Chaingun, Rocket Launcher, Plasma Gun, BFG9000
- **F** meat hook (Super Shotgun), **Shift** run, **Q** dash, **T** throw a bomb
- **E** use (doors, pushwalls, elevator) / glory kill, **P** pause, **M** mute, **N** new run (on death)

### Phone

On a touchscreen you get on-screen controls: a **MOVE** virtual stick (left pad), a **LOOK** pad (drag to turn and look up/down, tap to fire), and FIRE / DASH / BOMB / JUMP / WEAPON / HOOK / PAUSE buttons. You can also touch the game itself: drag the left half to move, drag the right half to look, tap the right half to fire.

### Joypad

Yes — any standard gamepad works via the browser Gamepad API, no install needed. Plug in an Xbox, PlayStation, Switch Pro or 8BitDo pad (USB or Bluetooth) and press a button; the game shows its name in the HUD.

- Left stick: move · Right stick: look (push up/down to look up/down)
- **RT or A**: fire (hold) · **X**: meat hook (Super Shotgun) · **B**: dash · **Y**: bomb
- **LB / RB**: change weapon · **Start**: pause · **Back/Select**: mute
- Upgrade screens: **LB / RB** to pick, **A** to confirm · Title / death screens: **A** to start a new run

## Procedural levels

Every floor is generated from the run seed: 8–10 rooms connected by corridors, on a 36×36 grid, with a new theme every floor (Castle, Crypt, Bunker, Vault). Enemy count, health and damage scale with depth. A boss spawns every third floor. The exit portal is always in the room farthest from your spawn.

## Roguelike progression

- **Permadeath.** Die and the run is gone — new seed, new castle.
- **XP and levels.** Kills give XP. On level-up, pick 1 of 3 upgrades (max HP, damage, fire rate, speed, armor, lifesteal, crit, bombs, dash recharge, regen, XP gain, ammo).
- **Floor-clear upgrade.** Clearing a floor (reaching the exit portal) also offers 1 of 3 upgrades before you descend.
- **Loot.** Health, ammo, treasure (+score), bombs and keys drop from rooms and dead enemies.
- **Best run** (deepest floor, score, kills) is kept in `localStorage`.

## Tech

Raycasting (DDA, one ray per character column, fisheye-corrected) drawn straight to a canvas. Walls use the ojread/ascii-fps trick — one ASCII character per column, pre-rendered to a small tile and stretched to the wall's full height, coloured from the floor theme's palette and distance-shaded. Floor and ceiling are flat-shaded bands. Enemies are drawn 3D-style billboard sprites (shaded figures that always face you, clipped per column against the wall depth buffer, with HP bars and a "!" when staggered); guns are Doom-style drawn viewmodel sprites with a muzzle flash. The scene is rendered to an offscreen canvas larger than the view (overscan) and view-bob just pans the window inside it, so bobbing never exposes a black border. The HUD is drawn in Bungee Tint. The minimap is a rotating radar (player facing is up) with fog of war, top-left, half the old size.

## Wolfenstein 3D logic

The gameplay logic is ported from the real Wolfenstein 3D source (id Software, 1992; source released 1995, `WOLFSRC/` in id-Software/wolf3d). The code here is an original JS implementation of those mechanics — the design, constants and state names are Wolf3D's, not a verbatim copy.

Ported, with file references in the code:

- **Maps** (`WL_DEF.H`, `WL_GAME.C`): 64×64 grid (`MAPSIZE 64`), tile codes for walls, doors (90–101), pushwalls (98), elevator (21), exit (99), ambush (106).
- **Enemies** (`WL_ACT2.C`): guard 25 HP / 100 pts, officer 50 / 400, SS 100 / 500, dog 1 / 200, mutant 55 / 700, Hans Grosse 850 / 5000; drops (clip +4, Hans drops the gold key).
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
moving the horizon; **J** jumps with simple gravity (about 0.4 tiles
high), which shifts the walls and floor through the eye height. The
renderer is a full 2.5D raycaster, so pitch and jump are
horizon/eye-height effects, not a new map axis — there are no stairs
or multi-storey floors.

## Unicode + ASCII art

Walls are one ASCII character per column (▓ brick, █ stone, ▒ wood,
█ metal, ▓ vault, █ door, ▒ pushwall), stretched to the wall's full
height and coloured from the floor theme's palette with distance
shading — the ojread/ascii-fps trick, so wall edges are pixel-smooth
instead of stepping in character rows. Enemies and guns are drawn
sprites, not ASCII: shaded 3D-style figures for guard, officer, SS,
dog, mutant and Hans Grosse, and Doom-style viewmodels for all eight
weapons. Items (first aid, food, clip, shells, rockets, cells, cross,
chalice, bible, crown, gold key, bomb, elevator exit) are pixel-art
icons rasterised from the old ASCII sprites.

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
execution i-frames. Hans Grosse can't be staggered. Glory kills are
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

Three original compositions, embedded as base64 Standard MIDI Files
and played by a tiny SMF parser + Web Audio General-MIDI-ish synth
written for the game — no audio files, no libraries, no network.
- **Title** — 80 BPM, A minor, string pads + piano arpeggio.
- **Combat** — 150 BPM, D minor, overdriven-guitar riff, synth bass,
  kick/snare/hats. Plays on normal floors.
- **Boss** — 170 BPM, E minor, faster riff, syncopated kick. Plays on
  every 3rd floor (boss floors).
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
