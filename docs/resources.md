---
sidebar_position: 6
---

# Resources

:::info Heads up!

This page is still very much a work-in-progress. As you can see, I was too lazy to even finish writing the hlmod section. I've almost certainly 

:::

Here's a smattering of related modding resources that can be of use:

## Bytecode and code modding

- [DCDocs](https://n3rdl0rd.github.io/dcdocs/) by N3rdL0rd
  - Generated API documentation from all the types and functions in the Dead Cells bytecode from Steam, for use as a modding reference
- [crashlink](https://n3rdl0rd.github.io/crashlink/) by N3rdL0rd
  - Command-line Hashlink bytecode utility

## Dead Cells-specific Files

- [alivecells](https://github.com/N3rdL0rd/alivecells) by N3rdL0rd
  - A collection of tools for working with Dead Cells' files that include:
  - [PAKTool Extended](https://github.com/N3rdL0rd/alivecells/releases/latest)
    - A modified .NET PAKTool based on the one in the base game's ModTools, but with [v1 (stamped) PAK](/files/pak#stamps) support
  - [stamptool](https://n3rdl0rd.github.io/alivecells/stamptool)
    - A tool for calculating and extracting v1 PAK stamps
  - [savetool](https://n3rdl0rd.github.io/alivecells/savetool/web)
    - A utility that allows reading, extracting, and writing game save files, alongside conversion between game platforms (e.g. mobile -> PC)
- [SaveAlchemist](https://labare.eu/DeadCells/SaveAlchemist) by LAbare
  - A port of savetool to the browser

## Modding Frameworks & Mods

:::info Mod labels

Mods are labeled with:

- [U] if they are unstable (e.g. frequent crashes, some things don't work)
- [S] if they are stable (e.g. infrequent crashes occur, operates well the majority of the time)
- [M] if they are mature (e.g. the mod has existed a while, is used by a large number of people, and does not crash frequently)
- [?] if they are untested by the authors of ModDocCe

Some DCCM mods are bundled for distribution on the Steam Workshop, and if they are, a workshop link will be appended.

:::

### Dead Cells Core Modding (DCCM)

[Docs](https://dead-cells-core-modding.github.io/docs/docs/) - [GitHub org](https://github.com/dead-cells-core-modding/)

The first real modloader and modding framework for a Hashlink game, or, for that matter, for Dead Cells. Written by HKLab in C#, runs on the .NET VM.

- [S] [Dead Cells Multiplayer Mod](https://github.com/vaiserYT/DeadCellsMultiplayerMod) by VaiserYT
  - Adds networked multiplayer through a client-server architecture and synchronized game states and RNG
- [S] [Dead Cells Archipelago](https://github.com/Maxlamenace572/DeadCellsArchipelago) by Maxlamenace572
  - [Archipelago](https://archipelago.gg/) port
- [M] [MoreSettingsMod](https://github.com/ChiuYi0912/DCCMMods) by ChiuYi
  - Adds a variety of extra tweakable settings, primarily visual
- [M] [DebugMod](https://github.com/ChiuYi0912/DCCMMods) by ChiuYi ([Workshop](https://steamcommunity.com/sharedfiles/filedetails/?id=3660397838))
  - Adds a variety of additional debug options
- [?] [Playable Collector](https://steamcommunity.com/sharedfiles/filedetails/?id=3722311186) by Somebody once told me (Workshop)
  - Adds the option to spawn and play as the Collector
- [M] [Debug Console Enabler](https://github.com/DreamBoxSpy/DebugConsole) by HKLab
  - Enables the game's built-in debug console
- [M] [Pop Damage Mod](https://github.com/ChiuYi0912/DCCMMods) by ChiuYi
  - Adds options to make all damage indicators use some of the special crit effects introduced in some DLCs

There are more that I haven't had a chance to list here yet, so let me know if I've missed a mod you'd like to see on this list.

### hlmod

Todo! Bug n3rdl0rd about this in the Discord if you want this to get written sooner.