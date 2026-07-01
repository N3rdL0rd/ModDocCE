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

Some mobile builds encrypt MO string data,  as far back as Google Play v3.5.9 (every MO in `res4.pak` was scrambled) and it goes even earlier. The MO header (magic, offsets, string counts) is left untouched and remains valid; only the raw bytes of each msgid and msgstr are encrypted.

### Algorithm

The cipher is an XOR stream keyed by a 384-byte table, where `plaintext[i] = ciphertext[i] ^ key[(len ^ i) % 384]` for a string of length `len`. The key table holds the first 384 terms of the Fibonacci sequence (starting from 0), each taken modulo 256, and is generated once. The same key is reused for every string in the file — the index formula `(len ^ i) % 384` ensures strings of different lengths draw from different portions of the key even at the same byte position. Because XOR is symmetric, the same operation performs both encryption and decryption.

```python
def make_key():
    key = []
    a, b = 0, 1
    for _ in range(384):
        key.append(b & 0xff)
        a, b = b, a + b
    return key

def crypt(data: bytes, key: list[int]) -> bytes:
    length = len(data)
    return bytes(b ^ key[(length ^ i) % 384] for i, b in enumerate(data))
```
