---
sidebar_position: 2
---

# Using crashlink

crashlink is a new HashLink bytecode Swiss Army knife that allows you to easily (and programatically) load, save, modify, patch, decompile, and otherwise poke at compiled Haxe in HashLink. This page serves as a brief overview of how to install, run, and use crashlink intuitively on your computer.

## Prerequisites

- A computer
- An Internet connection
- A Python installation (3.10+), or [`uv`](https://docs.astral.sh/uv/getting-started/installation/)

## Installing

**If you're using `uv`:** (Highly recommended)

```bash
uv tool install crashlink[extras]
# or, for ToT
# uv tool install git+https://github.com/N3rdL0rd/crashlink[extras]
```

**Otherwise:**

```bash
pip install crashlink[extras]
```

## Loading the game

**On Windows:**

```bat
crashlink "C:\Path\To\Dead Cells\deadcells.exe"
```

**On \*nix and MacOS:**

```bash
crashlink "/path/to/Dead Cells/hlboot.dat"
```

Loading the game into memory will take a minute or so. Then, you'll be dropped into the crashlink shell.
