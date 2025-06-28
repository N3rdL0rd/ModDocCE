---
sidebar_position: 1
---

# Unpacking and Repacking Game Files

This guide serves to provide a very basic guide to unpacking and repacking the game's asset files (contained in `res.pak`).

## Prerequisites

- A computer (running Windows for this guide, although the alivecells ModTools can be compiled with any dotnet toolchain)
- An Internet connection
- Some level of comfort with a terminal (eg. ability to run a script with arguments, understanding of absolute vs. relative paths)

## What is a PAK file?

A PAK file is a bundle of files - not unlike a GNU `tar` archive or a ZIP file. Dead Cells loads these files at runtime to get access to the game's assets.

:::note

On a technical side of things, the only thing that differentiates a PAK file from a `tar` archive or another archive format is that it indexes the offsets of all files within the header section, which means the game can load the entire contained filesystem tree into memory much more quickly than having to fully load the entire archive into memory.

:::

### PAK File Versions

The PAK file format has seen two major versions in Dead Cells:

`v1` PAKs contain a game-version-specific "stamp", which serves to verify the version of the game that the PAK is loading on and to prevent direct modification of the game files. However, this wasn't done securely and is easily bypassed. These were first introduced in v35, and don't apply to official mod PAKs, just the main game `res.pak`. These also can't be generated with the official `PAKTool.exe` included in the game's `ModTools`.

`v0` PAKs don't contain a stamp and instead are just normal archives. These were present as the main `res.pak` prior to v35, and as official supported mod PAKs.

## Getting a Better PAKTool

Since the game's official PAKTool can't handle v1 PAKs, we need to use a better version of the PAKTool. This is provided by `alivecells`. [Download it here](https://github.com/N3rdL0rd/alivecells/releases/tag/paktool-v0.1a) and unzip it somewhere.

## Unpacking Your Game Files

In order to unpack the game files, we need to identify three things:

- Where the PAKTool is
- Where the game's `res.pak` is
- Where we want the extracted files to go

The first two should be obvious - one is wherever you unzipped the file from the last step, and the other is usually `C:\Program Files (x86)\Steam\steamapps\common\Dead Cells\res.pak`. The destination folder can be whatever you choose, but usually it'll be `C:\Program Files (x86)\Steam\steamapps\common\Dead Cells\unpacked`. Let's assume your PAKTool is at `C:\Users\You\Downloads\PAKTool-alivecells-0.1a\PAKTool.exe`. Open up a terminal and run:

```cmd
C:\Users\You\Downloads\PAKTool-alivecells-0.1a\PAKTool.exe expand "C:\Program Files (x86)\Steam\steamapps\common\Dead Cells\unpacked" "C:\Program Files (x86)\Steam\steamapps\common\Dead Cells\res.pak"
```

Then, look in `C:\Program Files (x86)\Steam\steamapps\common\Dead Cells\unpacked` to find your unpacked game files! You can make any changes you want to these files, then proceed to:

## Repacking Your Game Files

If you're on v35 or higher, please follow this next step. Otherwise, skip it and move on to [Repacking without a Stamp](#repacking-without-a-stamp).

### Getting a Stamp

In order to repack game files and have the game read from them past v35, you must get a *stamp*, a unique signature of a specific version of the game. There are a number of ways to get this value, but this guide only covers one.

First, go to [this site](https://n3rdl0rd.github.io/alivecells/stamptool/). Then, drag and drop (or browse for) your game's `res.pak` and click `Calculate`. This should spit out a stamp, which is a random stream of hexadecimal characters. For the latest version of the game (at the time of writing), the stamp is `0022228129b0973a12d14548434b3741debcd3a38734f1e0dd1f3b3f7acdd91c`.

### Repacking with a Stamp

This is for v35+. If you're on a version &lt;v35, then instead use [Repacking without a Stamp](#repacking-without-a-stamp).

Again, assuming the same paths as before, open a terminal and run:

:::warning

This will overwrite the existing res.pak with your modifications. If you ever want to revert these changes easily, you can just verify the game's integrity from Steam.

:::

:::danger

Make sure you replace `<YOUR STAMP HERE>` in the command with your stamp that you found earlier! If you don't do this, a default will be used, which probably won't load correctly.

:::

```cmd
C:\Users\You\Downloads\PAKTool-alivecells-0.1a\PAKTool.exe collapsev1 "C:\Program Files (x86)\Steam\steamapps\common\Dead Cells\unpacked" "C:\Program Files (x86)\Steam\steamapps\common\Dead Cells\res.pak" -s <YOUR STAMP HERE>
```

### Repacking without a Stamp

:::warning

This will overwrite the existing res.pak with your modifications. If you ever want to revert these changes easily, you can just verify the game's integrity from Steam.

:::

Again, assuming the same paths as before, open a terminal and run:

```cmd
C:\Users\You\Downloads\PAKTool-alivecells-0.1a\PAKTool.exe collapse "C:\Program Files (x86)\Steam\steamapps\common\Dead Cells\unpacked" "C:\Program Files (x86)\Steam\steamapps\common\Dead Cells\res.pak"
```

## That's all, folks!

Relaunch your game and you should see your asset updates. Have fun!