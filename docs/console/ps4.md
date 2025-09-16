# PS4/5

Dead Cells on the PS4/5 is shipped as a signed `PKG`. Inside of the PKG, there are native interfaces for all trophies (achievements), alongside a SELF (Signed ELF) binary that acts as the main entry point (`eboot.bin`) and the obligatory `res.pak`, which seems to be V0 but could possibly be V1.

## `eboot.bin`

This file, as previously mentioned, is a signed ELF that contains the HL/C compilation of the game. Sadly, this ELF does not contain DWARF symbols, or any other means to recover information so it's most likely impossible to see what's changed for PS4/5 in specific.

## `res.pak`

Mostly unremarkable, you can unpack this with any standard PAKTool or its forks.
