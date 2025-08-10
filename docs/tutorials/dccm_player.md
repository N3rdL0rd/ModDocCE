---
sidebar_position: 3
---

# Dynamic Core Modding with DCCM (For Players)

:::info

This is a vastly simplified tutorial intended for non-technical people who want to play Dead Cells with DCCM mods. If you want to develop mods, read the [similar tutorial for developers](/tutorials/dccm_dev)

:::

## Installing

Open up your game directory in your favorite file explorer. Then, go download the [latest release of DCCM](https://github.com/dead-cells-core-modding/core/releases).

Make a subdirectory in the root of your game directory - call it `coremod`. Unzip the contents of the DCCM release to that directory. It should look like:

```txt
<DeadCellsGameRoot>
├─ coremod
│  ├─ core
│  │  ├─ native
│  │  │  └─ …
│  │  ├─ mdk
│  │  │  ├─ install.ps1
│  │  │  ├─ uninstall.ps1
│  │  │  └─ …
│  │  ├─ host
│  │  │  ├─ startup
│  │  │  │  ├─ DeadCellsModding.exe
│  │  │  │  └─ …
│  │  │  └─ …
│  │  └─ …
│  └─ …
├─ deadcells.exe
├─ deadcells_gl.exe
└─ …
```

## Testing

Now, let's test and make sure it's functional. Find `coremod/core/host/startup/DeadCellsModding.exe` and run it to launch the game.

:::tip

When launching the game from now on, if you run `deadcells.exe`, the vanilla game will launch. Running the launcher in `coremod/core/host/startup/` is the only way to launch the game modded.

:::

When you launch the game, you should see a new menu item labeled "About Core Modding". If you don't, then DCCM wasn't installed correctly.

![Core modding](dccm/menu.png)

## Installing Mods

Follow [the official docs](https://dead-cells-core-modding.github.io/docs-en/docs/tutorial/install-mods/).
