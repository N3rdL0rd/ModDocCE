---
sidebar_position: 4
---

# Using crashlink to manipulate game code

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

> **Note:** The `[extras]` group adds `tqdm` for progress bars when parsing large files, `dill` for faster save/load via pickling, and `pygments` for syntax-highlighted decompiler output.

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

## The crashlink shell

Once loaded, you'll see a `crashlink>` prompt. Type `help` to see all available commands. Some of the most useful ones are covered below.

### Navigating functions

- **`funcs`** – Lists every function and native in the bytecode. By default, standard-library functions are hidden; pass `funcs std` to include them.
- **`fn <findex>`** (alias `f`) – Disassembles a single function by its index.
- **`fnn <name>`** – Finds and prints a function by its fully-qualified name.
- **`entry`** – Shows the bytecode entrypoint.
- **`infile <file.hx>`** – Lists all functions that originate from a given source file (requires debug info).

Example session:

```
crashlink> funcs
f@22 static Clazz.main () -> Void (from Clazz.hx)
f@23 Clazz.method (Clazz) -> I32 (from Clazz.hx)
crashlink> fn 22
f@22 static Clazz.main () -> Void (from Clazz.hx)
Reg types:
  0. Void

Ops:
  0. Ret             {'ret': 0}                                       return
```

### Exploring types and data

- **`types`** – Lists every type in the bytecode (`Obj`, `Fun`, `Enum`, etc.).
- **`type <tIndex>`** (alias `t`) – Prints detailed information about a type.
- **`obj <tIndex>`** (alias `object`) – Shows a class's fields, instance methods (protos), and static methods (bindings).
- **`enum <tIndex>`** – Prints an enum's constructs and parameter types.
- **`virt <tIndex>`** – Prints a virtual type's fields.
- **`strings`** (alias `strs`) – Dumps the string table.
- **`string <idx>`** (alias `s`) – Prints a single string by index.
- **`ss <query>`** (alias `search`) – Searches for a substring in the string table.
- **`int <idx>`** (alias `i`) – Prints an integer constant by index.
- **`floats`** – Lists all float constants.
- **`global <gIndex>`** (alias `g`) – Shows a global variable and its initialized constant value, if any.

### Cross-references

- **`xref <fIndex>`** – Finds every function that calls the given function.
- **`strref <string_idx>`** (alias `sref`) – Finds every opcode that references a string directly or via a global variable initialized to that string.

### Decompilation and control flow

- **`decomp <fIndex>`** (aliases `decompile`, `dec`, `pseudo`) – Decompiles a function to Haxe-like pseudocode. If `pygments` is installed, the output is syntax-highlighted.
- **`class <tIndex>`** (aliases `cls`, `c`) – Decompiles an entire class (all methods) to pseudocode.
- **`cfg <fIndex>`** – Builds a control flow graph (CFG) for a function, renders it to PNG via Graphviz, and opens it in your default image viewer.
- **`ir <fIndex>`** – Prints the intermediate representation (IR) of a function in object notation.

> **Note:** Graphviz must be installed for `cfg` to work. Install it through your package manager (`graphviz` on most systems).

### Modifying bytecode

- **`patch <fIndex>`** (alias `edit`) – Opens a function's disassembly in an editor (tkinter, nano, or Notepad). Edit the opcodes below the separator line, save, and crashlink will re-assemble the function in place.
- **`setstring <idx> <text>`** – Overwrites a string in the constant pool.
- **`save <path>`** – Serialises the (possibly modified) bytecode back to disk.

### Utility commands

- **`info`** – Prints a quick overview of the bytecode (version, counts of strings/functions/types/etc., debug info presence).
- **`verify`** (alias `check`) – Runs basic sanity checks on the loaded bytecode.
- **`nativelibs`** (alias `libs`) – Lists the native `.hdll` libraries the bytecode depends on.
- **`debugfiles`** – Lists all source files recorded in debug info.
- **`apidocs <path>`** – Generates Markdown API docs for every class, inferred from the bytecode.
- **`mkdocs <path> [site name]`** – Generates a full MkDocs + Material site from the inferred API docs.
- **`stub <path>`** – Recreates the original Haxe source file tree as empty stubs (requires debug info).
- **`pickle <path>`** (alias `pkl`) – Saves the in-memory `Bytecode` object with `dill` for much faster subsequent loads.
- **`offset <hex>`** – Dumps the raw bytecode section at a given file offset.
- **`op <opcode>`** – Prints documentation for a given opcode (e.g. `op Call`).
- **`wiki`** – Opens the ModDocCE HashLink bytecode page in your browser.
- **`repl`** – Drops into a Python REPL with `code`, `disasm`, and `decomp` already imported.
- **`interp [fIndex]`** (alias `run`) – Runs the bytecode in crashlink's integrated interpreter (experimental).
- **`hlc <output.c>`** – Transpiles the loaded bytecode to crashlink cHL/C code.

## Python API

crashlink is designed to be used as a library as well as a CLI tool. Every CLI feature is backed by the Python API in `crashlink/`.

### Loading bytecode

```py
from crashlink import *

# From a file (searches for HLB magic automatically)
code = Bytecode.from_path("path/to/file.hl")

# From bytes
code = Bytecode.from_bytes(raw_bytes)

# Create empty bytecode
empty = Bytecode.create_empty()
```

### Disassembling

```py
# Disassemble a function by its fIndex
func = code.fn(22)          # Returns Function or Native
print(disasm.func(code, func))

# Or by iteration
for func in code.functions:
    print(disasm.func_header(code, func))
```

### Decompiling

```py
from crashlink import decomp, pseudo

func = code.fn(22)
ir = decomp.IRFunction(code, func)
ir.print()                  # Print IR

print(pseudo(ir))           # Print Haxe-like pseudocode

# Decompile an entire class
cls_type = code.types[some_index]
if isinstance(cls_type.definition, Obj):
    ir_cls = decomp.IRClass(code, cls_type.definition)
    print(ir_cls.pseudo())
```

### Control flow graphs

```py
from crashlink import decomp

func = code.fn(22)
cfg = decomp.CFGraph(func)
cfg.build()
dot = cfg.graph(code)       # DOT graphviz source
print(dot)
```

### Patching and saving

```py
# Modify a function's opcodes from assembly text
new_ops = disasm.from_asm("""
Ret {'ret': 0}
""")
func.ops = new_ops

# Save back to disk
with open("patched.hl", "wb") as f:
    f.write(code.serialise())
```

### Working with strings and constants

```py
# Read string
print(code.strings.value[42])

# Modify string
code.strings.value[42] = "New text"

# Find string references
for func in code.functions:
    for op in func.ops:
        for k, v in op.df.items():
            if isinstance(v, strRef) and v.value == 42:
                print(f"Found in f@{func.findex.value}")
```

### Assembly format

crashlink includes a textual assembly format in `crashlink/asm.py`. You can assemble it programmatically:

```py
from crashlink.asm import AsmFile

asm = AsmFile.from_path("my_code.asm")
bytecode = asm.assemble()
with open("out.hl", "wb") as f:
    f.write(bytecode.serialise())
```

Or from the CLI:

```bash
crashlink -a my_code.asm -o out.hl
```
