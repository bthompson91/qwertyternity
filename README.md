# QWERTYSTEIN

A Wolfenstein-style raycaster rendered in **dense ASCII art**, running entirely in the browser. No build step, no assets, no libraries — one `index.html`.

Play it live: https://bthompson91.github.io/qwertystein/

## How to play

Open `index.html` in a browser, click, press Enter. Or play live at https://bthompson91.github.io/qwertystein/

- **WASD** move, **mouse** look (click to lock), **← →** turn
- **Click / Space** shoot
- **1–5** weapons: Knife, Pistol, SMG, Shotgun, Chaingun
- **R** reload, **Shift** run, **Q** dash, **T** throw a bomb
- **P** pause, **M** mute, **N** new run (on death)

### Phone

On a touchscreen you get on-screen controls: a **MOVE** virtual stick (left pad), a **LOOK** pad (drag to turn, tap to fire), and FIRE / DASH / BOMB / WEAPON / RELOAD / PAUSE buttons. You can also touch the game itself: drag the left half to move, drag the right half to look, tap the right half to fire.

### Joypad

Yes — any standard gamepad works via the browser Gamepad API, no install needed. Plug in an Xbox, PlayStation, Switch Pro or 8BitDo pad (USB or Bluetooth) and press a button; the game shows its name in the HUD.

- Left stick: move · Right stick: look
- **RT or A**: fire (hold) · **X**: reload · **B**: dash · **Y**: bomb
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

Raycasting (DDA, one ray per character column, fisheye-corrected), 8×8 ASCII wall tiles per wall type, ASCII billboard sprites with a per-column depth buffer, floor/ceiling ASCII gradients, and a minimap — all drawn as text on a canvas, 180 columns × 75 rows (13,500 cells, 3.09× the original 104×42), with run-length blitting so a frame is ~1,700 text draws instead of 13,500.

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

Deliberate differences from Wolf3D: levels are procedurally generated instead of hand-made, runs are roguelike (permadeath, XP levels, upgrade picks between floors), and the screen is dense ASCII instead of textured walls. The chaingun/SMG fire faster than Wolf3D's 24-tic shots — that's the roguelike talking.
