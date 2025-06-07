---
sidebar_position: 3
---

# Save Files

This is Dead Cells' specific binary save file format. It stores multiple serialised objects in a compressed format along with hashes to prevent corruption or tampering.

:::note

All this information was obtained from `f@34927 static tool.$Save.genSave (User, Bool) -> haxe.io.Bytes` - if you'd like to help out, try looking at this function yourself!

:::

:::warning

This is not complete! We still need to figure out how hxbit actually stores its data, and until then, save files can only be partially modified.

:::

A working example of reading and writing a Dead Cells save file is available in `alivecells` as [`savetool.py`](https://github.com/N3rdL0rd/alivecells/blob/main/savetool.py).

## Format

### File

| Size (bytes) | Name           | Struct |
|--------------|----------------|-------|
| 59           | Header         | [Header](#header) |
| Variable     | Payload        | [Payload](#payload) |

### Header

| Size (bytes) | Name           | Description                                        | Struct            |
|--------------|----------------|----------------------------------------------------|-------------------|
| 4            | Magic          | `0xDEADCE11`                                       | None              |
| 1            | Version        | Current format version, usually `1`                | Unsigned 8-bit integer |
| 20           | Checksum       | SHA-1 raw hex digest of the file's contents, assuming these 20 bytes are 0x0 while taking the hash | None |
| 20           | Git hash       | Current long commit hash of the game, as a hex digest | None |
| 10            | Build date     | Build date of the game version creating this save file | UTF-8 string |
| 4            | Flags          | Flags and metadata about the save file - mostly unknown | Flags |

### Flags

| Bit | Value (Dec) | Value (Hex) | Name                | Type            | Description                                                                                                                                                             |
| :-- | :---------- | :---------- | :------------------ | :-------------- | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 0   | 1           | `0x1`       | `S_User`            | **Data Chunk**      | If set, a chunk containing persistent user progress (unlocks, cells, gold) exists.                                                                                      |
| 1   | 2           | `0x2`       | `S_Game`            | **Data Chunk**      | If set, a chunk containing an in-progress run exists. If clear, the save was made from the main menu.                                                                   |
| 2   | 4           | `0x4`       | `S_UserAndGameData` | **Data Chunk**      | If set, a chunk containing general game metadata exists.                                                                                                                |
| 3   | 8           | `0x8`       | `S_Date`            | **Data Chunk**      | If set, a chunk containing the 8-byte save timestamp exists.                                                                                                            |
| 4   | 16          | `0x10`      | `S_Experimental`    | *Feature Flag*    | If set, the save was made while the game's experimental/beta features were enabled.                                                                                     |
| 5   | 32          | `0x20`      | `S_UsesMods`        | *Feature Flag*    | If set, the save was made with mods active. This is determined by checking if `user.activeMods` is populated.                                                            |
| 6   | 64          | `0x40`      | `S_HaveLore`        | *Feature Flag*    | If set, indicates the save supports lore rooms. This flag is unconditionally set on all modern saves.                                                                   |
| 7   | 128         | `0x80`      | `S_VersionNumber`   | **Data Chunk**      | If set, a chunk containing the 4-byte float game version exists.                                                                                                        |
| 8   | 256         | `0x100`     | `S_DLCMask`         | **Data Chunk**      | If set, a chunk containing the 4-byte integer bitmask of installed DLCs exists.                                                                                         |
| 9-31| -           | -           | *Unused*            | -               | These bits are currently unused but reserved for future expansion (which is unlikely). They are always `0`.                                                               |

### Payload

The payload is a Zlib-compressed blob that contains the actual save file - some of them are [hxbit](https://github.com/HeapsIO/hxbit) serialised data, and others are just raw bits and pieces. They follow the order of the data chunk flags that were set previously - basically, if a data chunk flag was `1`, then the chunk is present in the save file. The payload is compressed with Zlib level 9 (maximum) compression. To manipulate it, read it as follows:

- Decompress the payload (it has headers, so any stock zlib implementation can handle it)
- Figure out how many data chunks are enabled
- For each of those data chunks that's enabled, read a [Chunk](#chunk) from the payload blob, seeking forward in it as you do.

### Chunk

| Size (bytes) | Name           | Description                                        | Struct            |
|--------------|----------------|----------------------------------------------------|-------------------|
| 4            | Size           | Size of the contents                               | Unsigned 32-bit integer |
| Variable     | Contents       | Raw bytes of the save file chunk contents          | Varies, see [Chunk Formats](#chunk-formats) |

## Chunk Formats

Depending on what kind of data chunk you're reading, the format of its chunk in the save file changes. Here's a basic map:

| Bit | Name | Type |
|----|-----|-----|
| 0 | `S_User` | [hxbit Data](#hxbit-data) |
| 1 | `S_Game` | [hxbit Data](#hxbit-data) |
| 2 | `S_UserAndGameData` | [hxbit Data](#hxbit-data) |
| 3 | `S_Date` | [Date Chunk](#date-chunk) |
| 7 | `S_VersionNumber` | [VersionNumber Chunk](#versionnumber-chunk) |

### Date Chunk

| Size (bytes) | Name           | Description                                        | Struct            |
|--------------|----------------|----------------------------------------------------|-------------------|
| 8            | Date           | Date of save file creation as a unix timestamp     | Double            |

### VersionNumber Chunk

| Size (bytes) | Name           | Description                                        | Struct            |
|--------------|----------------|----------------------------------------------------|-------------------|
| 4            | Version        | Major game version for this save file, eg `34` or `35` | 32-bit float  |

## hxbit Data

This data is serialised with help from [hxbit](https://github.com/HeapsIO/hxbit) - a Haxe library that enables serialisation of arbitrary objects to binary representations. This has yet to be reversed, but if it were able to be, it would be the key to actually modifying save files.