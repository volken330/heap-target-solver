# Heap target solver

Give an address and the page lists every way it finds to fill OoT's actor heap (NTSC 1.2) exactly up to it, simplest first. After any listed option, the next allocation in the stated size range lands at that address.

## Publish

`index.html` is self-contained. Push it to a repository and enable GitHub Pages (Settings → Pages → deploy from `main`, root). Or drop it into an existing Pages site as `target-solver/index.html`.

## Starting point (two options)

- **Option 1: a RAM dump** (Dolphin Wii VC MEM1 or raw RDRAM). The solver can also remove actors already in the scene. The page reads:
  - the heap blocks, starting from the arena node at 0x801DB2C0;
  - the live actors, from `play->actorCtx` lists at 0x801CA990;
  - which actor code is loaded, from `gActorOverlayTable` at 0x800E8B70;
  - Link's age, items, ammo, bottles, songs and magic, from `gSaveContext` at 0x8011AC80.

  It shows the heap, and offers the things already in the scene that Link can make go away:
  - **Pots and crates:** break them. Their fixed drop is worked out the way `Item_DropCollectible` does it.
  - **Bushes:** cut type-1 ones. Bushes with random drops are listed but not used.
  - **Enemies:** kill them. The page ignores their drops and death effects.
  - **Items on the ground:** collect them.
- **Option 2: where free memory starts.** The solver can only add actors Link makes himself. Free memory is taken to start there and continue upward, with no item code loaded. You pick child or adult.

## End point

The address to fill the heap up to.

## What Link can make

The page starts every usable item checked, and you can turn any of them off.

| Group | Items |
|---|---|
| Explosives | bomb; bombchu (exploding spawns a separate explosion actor) |
| Shooting | slingshot seed; arrow (+2 tables); fire, ice and light arrows (+ their effect actor); Deku nut (on a hit, spawns a flash actor) |
| Tools | boomerang (either age); hookshot |
| Bottles | bugs (3 actors; catching one refills the bottle); fish; fairy; blue fire (19 actors) |
| Magic | Din's Fire; Nayru's Love; Farore's Wind (+ a table); quick and charged magic spin |
| Songs | Zelda's Lullaby, Saria's, Epona's, Sun's Song, Song of Time, Song of Storms |

Sizes, code sizes and code type (normal, persistent or absolute) come from the heap simulator's NTSC 1.2 export. Allocation follows `__osMalloc`, `__osMallocR` and `__osFree` exactly. An actor's code loads with its first instance. Normal code is freed with its last instance; persistent code stays. Absolute code shares one 0x24E0 space, allocated once.

## Steps and options

- **Steps:** making something, something made earlier going away, or breaking, killing or collecting something in the scene. There's no timing: anything can go away at any later step, or stay to the end.
- **Rules kept:**
  - at most 3 live bombs, bombchus and explosions (`z_player.c`);
  - one drawn or held item (seed, arrow, hookshot) at a time;
  - bottles need their contents.
- **Ranking:** options are ranked by number of steps, then everyday items over bottles, magic and songs, then the widest landing range.
- **Merging:** the same steps in a different order count as one option. Interchangeable steps, such as songs with the same footprint, are shown as alternatives.
- **Hidden options:** an option where a bigger gap below would catch everything that fits at the address isn't shown.
- **Limits:** the search stops at 8 steps, 20 options or about 20 seconds.
