# 7. A safeguard against duplicates that quietly ate real data

**Symptom.** Testing a new health-tracking feature by voice — "log my
resting heart rate as 68" — the assistant said "Logged" every time. The
stored history disagreed: some logs simply weren't there.

**What I assumed.** The storage layer was fine; this had to be a routing or
parsing bug further up, since the confirmation was clearly firing.

**Why that was wrong.** The confirmation firing was exactly the problem —
it meant the code path completed successfully and still didn't write
anything. The feature has two sources of the same kind of reading: you can
say a number out loud, and (a separate later addition) a watch can sync one
in automatically. Automatic syncs re-send the same reading on every sync by
design — the sync has no reliable "only what's new" cursor, so it re-reads
a trailing window every time and leans on deduplication to make that safe.
That dedup keyed on timestamp at second resolution. A voice log stamped in
the same second as anything already stored — trivially easy, since "log my
resting heart rate as 68" takes under a second to say and process — got
silently treated as a repeat of something that didn't exist yet, and
dropped. The safeguard built for one intent (don't re-store what a sync
already sent) was firing against a completely different intent (a human
just told you a fresh number) that happened to share a data shape.

**What I built.** Split what had been one function into two that name the
intent instead of inferring it: a voice log always stores, full stop — you
said it once, it happens once, no dedup logic gets a vote. A synced reading
still deduplicates on timestamp, because that's the exact case the
safeguard exists for. Same storage underneath; the difference is which
guarantee the caller is asking for.

**The related bug, same commit.** The anomaly detector that flags an
unusual reading had a second version of the identical mistake at a
different layer. It compared a new reading to your history using one
threshold — more than two standard deviations from your own average — with
no floor under it. Tested against a baseline that happened to cluster
tightly, a 3bpm swing tripped it. That's not a finding; day-to-day
heart rate moves more than that doing nothing in particular. A detector
that flags ordinary noise trains you to stop trusting it, which is exactly
backwards for something meant to catch the one reading that matters. Fixed
by giving each measurement category its own noise floor — normal day-to-day
movement for *that* metric — and requiring a reading to clear both the
statistical test and the noise floor before it counts as unusual. Checked
against both a tight and a noisy synthetic history afterward: the 3bpm
swing is now ignored, a 13bpm one is still caught.

**The lesson.** A safeguard is built against one specific danger, and it
doesn't know that when a second, different intent starts sending it
data that happens to look the same shape. "Working as designed" and
"correct for this caller" are different claims — the first one doesn't
prove the second.

---

*Project: Umbra. Copied 19 Nov 2026 from [umbra-sub-os-showcase / CASE-STUDIES.md](https://github.com/Ciaran11221/umbra-sub-os-showcase/blob/main/CASE-STUDIES.md#7-a-safeguard-against-duplicates-that-quietly-ate-real-data) at commit 5940844. The showcase copy is kept as well; if they differ, the showcase is newer.*
