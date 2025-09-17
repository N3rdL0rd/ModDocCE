---
sidebar_position: 2
---

# PAK File Preprocessing

Peculiarly, on mobile builds of Dead Cells, the `res.pak` seems to be instead split into many gradual load PAKs: `res.pak`, `res1.pak`, `res2.pak`, etc. In many cases, it also seems that the first PAK is missing a stamp, which, on game versions >=v35, may cause it to be skipped during the graduated load process. This has yet to be investigated further, but it's worth looking into, as it may show promise for PAK modification.

On the Netflix Android builds, both `res.pak` and `res1.pak` are stamped and V1.
