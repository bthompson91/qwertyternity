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

Raycasting (DDA, one ray per character column, fisheye-corrected), 8×8 ASCII wall tiles per wall type, ASCII billboard sprites with a per-column depth buffer, floor/ceiling ASCII gradients, and a minimap — all drawn as text on a canvas, 104 columns × 42 rows.
