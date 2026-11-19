# 12. A version bump that would have made real work unopenable

**In one line:** I was about to change a file format's version number, and checked first what that would do to the one saved session on disk.

| | |
|---|---|
| **Where** | Session storage for the planning pipeline |
| **Result** | Caught in design. Review of the finished code then found 2 more problems the plan had missed, including a migration path with 0 tests |
| **Topic** | Design decision, Testing, Code review |

## The problem

Nothing had failed yet. This one was caught in design, which is the only place it could have been caught cheaply.

Two pieces of state storage had grown up separately for the same feature. One held a planning tool's own session. The other held the handoff between 4 separate pipeline stages. They overlapped and did not refer to each other. They also disagreed on one thing: one stored a progress count on disk, and the other recomputed it on every read so it could never drift from the data it summarises. Merging them meant changing the file format, which meant raising the schema version, the integer stamped into every file.

## What I assumed

That raising the version number was a safe, ordinary step. Before writing the plan, I checked what the existing version policy does on a mismatch.

## What the evidence said

- The policy is strict and deliberately lopsided. A temporary session at an unknown version is discarded, with a message naming both versions.
- A saved session at an unknown version is refused, named, and left on disk untouched. That is right: throwing away something a person chose to keep is worse than declining to open it.
- On disk there was one saved session. It was real work, a problem I had been planning through the day before.

```
2026-09-07 202012060642  saved=True  status=ready
"Logs are sent to many different sources, softwares and pcs with
 different operating systems, need aggregation system"
```

Raising the version with no migration path would have sent that file down the "refuse to open" branch for good, from every surface including the phone. Nothing would have been deleted, and no test would have failed. The file would simply have stopped opening, which is the policy working as designed.

## The fix

I specified a read-time migration lane. It is a registry of known upgrades, applied in memory only, so an older file is brought up to date as it is read and the file on disk is never written. A version with no registered upgrade still gets discarded or refused. The lane widens what counts as readable and does not weaken what happens to something genuinely unreadable.

The plan was written constraints first. The migration step was ordered before the version bump, with an instruction not to reorder them, because in the other order the destructive window is real. Each step had its own falsifier, meaning what to measure, what it must read, and what it means if it reads something else. For this step: open the real saved session after the change, confirm its text is intact, and hash the file before and after to prove the migration never wrote to it.

## What the review found

I then reviewed the implementation by reading the diff, not the summary, because a summary is produced by whatever did the work. It turned up 2 things the plan had not protected against.

**1. A silent failure waiting for the next version bump.** The migration registry mapped an old version straight to its upgrade function, and the loading code assumed every function landed on the current version. That holds while there is 1 migration. Add a 2nd, and a file at the oldest version runs only the first step and is then accepted as fully current. The rest of the upgrade is skipped, with no error and no warning. I proved it: I moved the version constant forward by 1, fed it an old file, and watched it load clean when it should have refused. The fix makes each migration declare the version it upgrades to, and walks the chain. A file that cannot reach the current version now falls through to the loud path.

**2. A test gap.** The migration lane, the most dangerous path in the change, had no test at all. It looked covered, because several tests were about schema mismatches. But each one wrote a file at the current version and then raised the constant. That exercises the refuse-and-discard branch and never reaches the upgrade branch. So the suite was green over the exact code whose failure would have made my own saved work unopenable.

## What I took from it

A version number is a promise to files that already exist, and the cost of breaking it falls on data someone chose to keep, on a path no failing test points at. A test suite also shows which branches ran, never which were skipped, so the useful question is which specific input reaches which specific line.

---

*Project: Umbra. Rewritten for easier reading from the original in [umbra-sub-os-showcase / CASE-STUDIES.md](https://github.com/Ciaran11221/umbra-sub-os-showcase/blob/main/CASE-STUDIES.md#12-a-version-bump-that-would-have-made-real-work-unopenable) (commit 5940844). Same facts and numbers.*
