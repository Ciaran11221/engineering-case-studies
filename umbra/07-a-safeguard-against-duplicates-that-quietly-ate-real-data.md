# 7. A safeguard against duplicates that quietly ate real data

**In one line:** the assistant said "Logged" for health readings I spoke aloud, but some were never stored, because a duplicate filter built for watch syncs was throwing them away.

| | |
|---|---|
| **Where** | Health-tracking feature (voice logging, watch sync, unusual-reading detector) |
| **Result** | Voice logs now always store; the unusual-reading detector ignores a 3bpm swing and still catches a 13bpm one |
| **Topic** | Debugging, Testing |

## The problem

I was testing a new health-tracking feature by voice, saying "log my resting heart rate as 68". The assistant said "Logged" every time. The stored history disagreed: some logs were not there.

## What I assumed

That the storage was fine and this was a routing or parsing bug further up, since the confirmation was clearly firing.

## What the evidence said

- The confirmation firing was the clue. The code path finished successfully and still wrote nothing.
- The feature has 2 sources for the same kind of reading. I can say a number aloud, and a watch can sync one in automatically (a later addition).
- Watch syncs re-send the same reading every time, by design. The sync has no reliable "only what's new" marker, so it re-reads a trailing window each time and relies on deduplication to make that safe.

## The cause

The deduplication compared timestamps to the second. A voice log stamped in the same second as anything already stored was silently treated as a repeat of something that did not exist yet, and dropped. That is easy to hit, because saying "log my resting heart rate as 68" takes under a second to say and process.

The safeguard was built for 1 purpose (do not re-store what a sync already sent). It was firing against a different purpose (a person has just told me a fresh number) that happened to share the same data shape.

## The fix

I split what had been 1 function into 2 that name the intent instead of guessing it:

- A voice log always stores. You said it once, it happens once, and no deduplication gets a vote.
- A synced reading still deduplicates on timestamp, because that is the case the safeguard exists for.

The storage underneath is the same. The difference is which guarantee the caller asks for.

## A related bug, same commit

The detector that flags an unusual reading made the same kind of mistake at a different layer. It compared a new reading to your history with 1 threshold, more than 2 standard deviations from your own average (a measure of how much your readings normally vary), and no minimum size. Tested against a baseline that happened to cluster tightly, a 3bpm swing tripped it. That is not a finding, because day-to-day heart rate moves more than that doing nothing in particular. A detector that flags ordinary noise trains you to stop trusting it.

I fixed it by giving each measurement category its own noise floor, meaning the normal day-to-day movement for that metric. A reading now has to clear both the statistical test and the noise floor to count as unusual.

I checked it afterwards against both a tight and a noisy synthetic history. The 3bpm swing is now ignored and a 13bpm one is still caught.

## What I took from it

A safeguard is built against 1 specific danger. It does not know when a different intent starts sending it data of the same shape. "Working as designed" and "correct for this caller" are different claims, and the first does not prove the second.

---

*Project: Umbra. Rewritten for easier reading from the original in [umbra-sub-os-showcase / CASE-STUDIES.md](https://github.com/Ciaran11221/umbra-sub-os-showcase/blob/main/CASE-STUDIES.md#7-a-safeguard-against-duplicates-that-quietly-ate-real-data) (commit 5940844). Same facts and numbers.*
