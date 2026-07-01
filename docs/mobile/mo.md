---
sidebar_position: 4
---

# MO Files (Translations)

MO files are GNU gettext Machine Object files used for language translations. The mobile builds of Dead Cells use the standard MO binary format but may additionally encrypt the string contents.

## Format

### File

| Size (bytes) | Name                | Description                              | Struct            |
|-------------|---------------------|------------------------------------------|-------------------|
| 4           | Magic               | `0x950412de` (LE) or `0xde120495` (BE)   | Unsigned int      |
| 4           | Revision            | Format revision, `0x00`                  | Unsigned int      |
| 4           | Number of strings   | Total msgid/msgstr pairs                 | Unsigned int      |
| 4           | Original table off  | Offset to the msgid descriptor table     | Unsigned int      |
| 4           | Translation table off | Offset to the msgstr descriptor table  | Unsigned int      |
| 4           | Hash table size     | Size of the hash table                   | Unsigned int      |
| 4           | Hash table offset   | Offset to the hash table                 | Unsigned int      |
| Variable    | String descriptors  | Two parallel tables of length/offset pairs, one for msgid, one for msgstr | Array of [Descriptor](#string-descriptor) |
| Variable    | String data         | Raw string bytes for all msgid and msgstr entries | Raw binary data   |

### String Descriptor

Each string in the MO file is described by an 8-byte entry in either the original or translation table:

| Size (bytes) | Name   | Description                              | Struct       |
|-------------|--------|------------------------------------------|--------------|
| 4           | Length | Length of the string in bytes            | Unsigned int |
| 4           | Offset | Offset of the string data from file start | Unsigned int |

---

## Mobile Encryption

Some mobile builds encrypt MO string data — confirmed as far back as Google Play v3.5.9 (every MO in `res4.pak` was scrambled) and it goes even earlier. The analysis below used v3.5.9's `libnative-lib.so`. The MO header (magic, offsets, string counts) is left untouched and remains valid; only the raw bytes of each msgid and msgstr are encrypted.

:::note

Function names and addresses below are from the v3.5.9 `libnative-lib.so`. Older versions use the same algorithm but may have different symbol names.

:::

### Where It's Implemented

On the Haxe side, `Lang_getTexts` calls the native class `libs_data_GetText`, which loads MO files through `libs_data_MoReader`. The relevant functions:

| Function                            | Address        | Role                                |
|-------------------------------------|---------------|-------------------------------------|
| `libs_data_MoReader_parse`          | `0x022347f0`  | Reads MO header and decrypts strings |
| `libs_data_MoReader_init`           | `0x02234704`  | Precomputes the 384-byte XOR key    |
| `libs_data_MoReader_fib8`           | `0x022346d0`  | Computes fib(n) mod 256             |
| `libs_data_MoReader_decrypt`        | `0x02234d94`  | XOR decrypts a single string        |
| `libs_data_MoReader_getString`      | `0x02234e54`  | Retrieves a decrypted string by index |
| `libs_data_MoReader_getOriginalString` | `0x02234d88` | Retrieves a decrypted msgid        |
| `libs_data_MoReader_getTranslatedString` | `0x02234d7c` | Retrieves a decrypted msgstr     |

### Key Generation (`init`)

`init` allocates a table of 384 32-bit integers and fills it by calling `fib8(i)` for `i = 0..383`. The key length (`0x180` = 384) is loaded from a config field at offset `0x3c`, and the finished table pointer is stored in the `MoReader` object at offset `0x40`.

Each key entry is stored as a full 32-bit integer, but only the low byte is ever used — the thread id modulo 256 step truncates it to `0..255`.

The `fib8` routine itself is straightforward:
- If `n < 1`, returns 0
- Otherwise iterates n times with `a=0, b=1`, computing `a, b = b, a+b`
- Returns `b & 0xff`

### Per-String Decryption (`decrypt`)

The `decrypt` function operates on a single string in-place. The string is passed as a buffer object with length at `[buf + 8]` and data pointer at `[buf + 0x10]`.

For each byte position `i` from 0 to `len-1`:
1. Compute `key_idx = (len ^ i) % 384` via `eor` + `sdiv` + `msub`
2. Read the 32-bit key value at `key_table + key_idx * 4`
3. XOR the cipher byte with the key byte: `plain[i] = cipher[i] ^ key_table[key_idx]`

Because XOR is symmetric, the same operation performs both encryption and decryption.

### Algorithm Summary

| Property          | Value                                           |
|-------------------|-------------------------------------------------|
| Cipher            | XOR stream                                      |
| Key stream        | Fibonacci sequence modulo 256, 384 bytes        |
| Key length        | 384 (Pisano period of Fibonacci mod 256)        |
| Key index         | `(string_length ^ byte_position) % 384`          |
| Operation         | `plaintext[i] = ciphertext[i] ^ key[(len ^ i) % 384]` |

The key stream is generated once — the first 384 terms of the Fibonacci sequence (starting from 0), each taken modulo 256. The same key is reused for every string in the file, with the index formula `(len ^ pos) % 384` ensuring strings of different lengths use different portions of the key.
