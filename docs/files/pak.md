---
sidebar_position: 1
---

# PAK files

PAK files are checksum-verified packages that contain a full filesystem structure inside them - they store all assets for the game. PAK files can be layered on top of each other - the game will first search for `res.pak`, then `res1.pak`, then `res2.pak` (etc.) while loading PAKs - PAKs are incrementally stacked on top of each other during the loading process - conflicts are ignored, and the PAK with the highest priority is used.

## Format

:::tip

If you're a REHex user, you can download a fully commented out example PAK file from [here](https://n3rdl0rd.github.io/alivecells/stamptool/example.pak.zip)

:::

### File

| Size (bytes) | Name           | Description                                        | Struct            |
|--------------|----------------|----------------------------------------------------|-------------------|
| Variable     | Header         | File magic, version, data sizes, stamp             | [Header](#header) |
| Variable     | Root directory | Main directory entry, contains all data in the PAK | [Entry](#entry)   |
| 4            | DATA marker    | Marks the start of file content data               | [DATA marker](#data-marker) |
| Variable     | File contents  | Actual file data for all files in the PAK          | Raw binary data   |

### Header

| Size (bytes) | Name        | Description                                                                   | Struct             | Condition            |
|--------------|-------------|-------------------------------------------------------------------------------|--------------------|----------------------|
| 3            | Magic       | "PAK"                                                                         | ASCII-encoded text | None                 |
| 1            | Version     | 0x00 or 0x01 - v0 doesn't support stamping, v1 does                           | Unsigned byte      | None                 |
| 4            | Header size | Size of the header - at v0, guaranteed to be >16, at v1, guaranteed to be >80 | Signed 32-bit int  | None                 |
| 4            | Data size   | Size of the data                                                              | Signed 32-bit int  | None                 |
| 64           | Stamp       | Signature based on git commit - see [here](#stamps)                           | ASCII-encoded text | Game >=v35, PAK >=v1 |

### Entry

An Entry can either be a directory or a file, determined by its `Flags`.

| Size (bytes) | Name          | Description                               | Struct        |
| :----------- | :------------ | :---------------------------------------- | :------------ |
| 1            | Name length   | The number of characters in the name.     | Unsigned byte |
| Variable     | Name          | The name of the entry.                    | ASCII text    |
| 1            | Flags         | Bit flags that determine the entry type.  | Unsigned byte |

The `Flags` field is a bitmask that specifies the properties of the entry:

* **`&0x01`**: If this bit is set, the entry is a **directory**.
* **`&0x02`**: If this bit is set, the `Position` of the file data is represented as a 64-bit long integer.

:::note

The `0x02` flag cannot generally be found in Dead Cells on any platforms - but it exists here for completeness of the format. Dead Cells also does not handle deserialising this flag, but other Heaps games (such as Wartales) generate such PAKs.

:::

---

#### If the entry is a directory

| Size (bytes) | Name        | Description                                       | Struct            |
| :----------- | :---------- | :------------------------------------------------ | :---------------- |
| 4            | Entry count | The number of entries inside this directory.      | Signed 32-bit int |
| Variable     | Entries     | A recursive list of `Entry` structures.           | [Entry](#entry)   |

---

#### If the entry is a file

| Size (bytes) | Name     | Description                                                                                             | Struct                         |
| :----------- | :------- | :------------------------------------------------------------------------------------------------------ | :----------------------------- |
| 4 or 8       | Position | The offset of the file data from the start of the data section. It is a 64-bit `long` if the `&0x02` flag is set; otherwise, it is a 32-bit `int`. | Signed 32-bit int or 64-bit long |
| 4            | Size     | The size of the file data in bytes.                                                                     | Signed 32-bit int              |
| 4            | Checksum | The Adler32 checksum of the file data.                                                                  | Signed 32-bit int              |

### DATA marker

| Size (bytes) | Name   | Description                           | Struct              |
|--------------|--------|---------------------------------------|---------------------|
| 4            | Marker | ASCII "DATA" to mark start of content | ASCII-encoded text  |

The DATA marker serves as a separator between the header/directory information and the actual file contents. It helps in clearly delineating the structure of the PAK file and can be used as a reference point when reading or writing PAK files.

## Stamps

In v35+ of the game (v1 of the PAK format), PAK files now include "stamps". A stamp is a 64-character (ASCII) digest of a SHA256 hash, generated with this formula (written in pseudocode):

```txt
ASCII digest(
    SHA256(
        UTF-8 encode("Dc02&0hQC#G0:"),
        UTF-8 encode(Current git commit - short hash)
    )
)
```

Due to the deterministic nature of SHA256 and the short length of the commit hash, it is possible to easily brute-force the short hash from a given stamp - an implementation of this exists in Alive Cells' [main script](https://github.com/N3rdL0rd/alivecells/blob/main/alivecells.py), as the `commitbrute` subcommand. You can also calculate or extract a stamp from a PAK file or from a short hash using the online [stamp calculator](https://n3rdl0rd.github.io/alivecells/stamptool/).
