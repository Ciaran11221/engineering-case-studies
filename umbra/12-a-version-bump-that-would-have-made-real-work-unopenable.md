# 12. A version bump that would have made real work unopenable

**Symptom.** None yet. This one was caught in design, which is the only
place it could have been caught cheaply.

**The setup.** Two pieces of state storage had grown up independently for
the same feature: one holding a planning tool's own session, the other
holding the handoff between four separate pipeline stages. They
overlapped, they didn't reference each other, and they disagreed about
something real — one of them stored a progress count on disk, the other
deliberately recomputed it on every read so it could never drift from the
data it summarises. Merging them meant changing the file format, which
meant incrementing the schema version integer stamped into every file.

**What I checked before writing the plan.** What the existing version
policy actually does on a mismatch. It's strict, and deliberately
asymmetric: a *temporary* session at an unrecognised version is discarded
with a message naming both versions; a *saved* session is refused
outright, named, and **left on disk untouched**. That asymmetry is right —
throwing away something a person explicitly chose to keep is worse than
declining to open it.

Then what was actually on disk:

```
2026-09-07 202012060642  saved=True  status=ready
"Logs are sent to many different sources, softwares and pcs with
 different operating systems, need aggregation system"
```

One saved session. Real work, not test data — a problem I'd been planning
through the day before.

**Why that mattered.** Incrementing the version without a migration path
would have sent that file straight down the "refuse to open" branch,
permanently, from every surface including the phone. Nothing would have
been deleted. Nothing would have failed in the test suite. The file would
simply have stopped opening — correctly, according to a policy working
exactly as designed.

**What I specified.** A read-time migration lane: a registry of known
upgrades applied in memory only, so an older file is brought up to date as
it's read and the file on disk is never touched. Only a version with no
registered upgrade still hits discard-or-refuse. The migration widens what
counts as readable; it doesn't weaken what happens to something genuinely
unreadable.

The plan was written constraints-first, with the migration step ordered
explicitly *before* the version bump and an instruction not to reorder
them, because in the other order the destructive window is real. Each step
carried its own falsifier — what to measure, what it must read, and what
it means if it reads something else. For this one: open the real saved
session after the change, confirm its text is intact, and hash the file
before and after to prove the migration never wrote.

**Then I reviewed the implementation, and found two things the plan hadn't
protected against.** Reading the diff rather than the summary is what
turned these up; neither is visible from a written report of the work,
because the report is produced by whatever did the work.

The first was a silent failure waiting for the *next* version bump. The
migration registry mapped an old version straight to its upgrade function,
and the loading code assumed every registered function landed on the
current version. True while there's one migration. Add a second, and a
file at the oldest version runs only the first hop and is then accepted as
fully current — remaining upgrade skipped, no error, no warning. I proved
it rather than arguing it: set the version constant forward by one, fed it
an old file, and watched it load clean when it should have refused. The
fix makes each migration declare the version it upgrades *to* and walks
the chain, so a file that can't reach the current version falls through to
the loud path instead of the quiet one.

The second was worse, because it was an absence. The migration lane — the
most dangerous path in the whole change, the reason the plan was ordered
the way it was — had **no test at all**. It looked covered. There were
several tests about schema mismatches. But every one of them wrote a file
at the *current* version and then incremented the constant, which
exercises the refuse-and-discard branch and never once reaches the upgrade
branch. A green suite, over the exact code whose failure would have made
my own saved work unopenable.

**The lesson.** Two, and the second is the one I'd want to be asked about.

A version number is a promise to files that already exist. The cost of
breaking it isn't paid by the code you're writing — it's paid by data
someone already chose to keep, on a path no failing test will ever point
at, because from the code's point of view, refusing to open a file it
doesn't understand *is* success.

And a test suite tells you which branches were exercised, never which were
skipped. Several tests named the schema policy, and it was easy —
correct-looking, even — to read that cluster as coverage. What made the
gap visible wasn't reading the tests. It was asking which specific input
reaches this specific line, and finding that nothing in the suite ever
produced one.

---

*Project: Umbra. Copied 19 Nov 2026 from [umbra-sub-os-showcase / CASE-STUDIES.md](https://github.com/Ciaran11221/umbra-sub-os-showcase/blob/main/CASE-STUDIES.md#12-a-version-bump-that-would-have-made-real-work-unopenable) at commit 5940844. The showcase copy is kept as well; if they differ, the showcase is newer.*
