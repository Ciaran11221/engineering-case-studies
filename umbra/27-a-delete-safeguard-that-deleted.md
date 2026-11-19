# 27. A delete safeguard that deleted

A recycle bin's move step had a backup path for folders Windows won't move.
With one file held open, that backup path erased the other files from both
places: 3 files in, 1 file left.

The existing test replaced the move step with a stand-in, so the backup path
never ran.

The fix removed the backup path entirely. A move is now all-or-nothing, and a
new test holds a file open and checks that every file survives.

---

*Project: Umbra. Copied 19 Nov 2026 from [umbra-sub-os-showcase / CASE-STUDIES.md](https://github.com/Ciaran11221/umbra-sub-os-showcase/blob/main/CASE-STUDIES.md#27-a-delete-safeguard-that-deleted) at commit 5940844. The showcase copy is kept as well; if they differ, the showcase is newer.*
