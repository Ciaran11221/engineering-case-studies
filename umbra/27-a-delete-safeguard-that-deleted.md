# 27. A delete safeguard that deleted

**In one line:** the backup path in a recycle bin's move step erased files when one file was held open.

| | |
|---|---|
| **Where** | Recycle bin, the move step |
| **Result** | 3 files in, 1 file left. The fix removed the backup path |
| **Topic** | Testing, Debugging |

## The problem

A recycle bin's move step had a backup path for folders Windows will not move. With one file held open, that backup path erased the other files from both places. 3 files went in and 1 was left.

## What the evidence said

The existing test replaced the move step with a stand-in, so the backup path never ran.

## The fix

I removed the backup path entirely. A move is now all-or-nothing. A new test holds a file open and checks that every file survives.

---

*Project: Umbra. Rewritten for easier reading from the original in [umbra-sub-os-showcase / CASE-STUDIES.md](https://github.com/Ciaran11221/umbra-sub-os-showcase/blob/main/CASE-STUDIES.md#27-a-delete-safeguard-that-deleted) (commit 5940844). Same facts and numbers.*
