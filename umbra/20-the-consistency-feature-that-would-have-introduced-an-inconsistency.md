# 20. The consistency feature that would have introduced an inconsistency

**In one line:** I added pre-recorded clips so fixed phrases would always sound the same, and the code review found the feature could make a single reply speak in two different voices.

| | |
|---|---|
| **Where** | Desktop voice output |
| **Result** | 4 defects found in review: 2 real bugs, 1 real but low odds, 1 not reachable today |
| **Topic** | Code review, Design decision |

## The problem

Umbra says a handful of fixed lines all the time, such as "Noted.", "Still with you." and "Conversation mode on." In one 3.5 hour window, 12 of 26 spoken replies were repeats of 5 lines. Each one was generated fresh by the speech engine, and that engine gives a slightly different result every time. A bad result was the likeliest explanation for a warped reply I had heard that morning.

## What I assumed

The phone app already solved this. It plays a known-good pre-recorded take for its fixed lines, and the desktop did not. So the plan was to record the same clips for the desktop and have the speech function look for a match before it uses the speech engine. I wrote out the constraints. What came back was a clean implementation, a new test file and a passing test suite.

## What the review found

I review the code change itself, not the summary of it. A summary is written by whatever did the work, so it cannot check itself. The review found 4 defects.

**1. Two voices in one reply.** The voice system has a cloned-voice engine that runs on a local server. When that server is unhealthy, the speech function already switches to a plain fallback voice, sentence by sentence. A dead server then means a slightly different voice, not silence.

The new clip lookup did not know about this. It looked clips up by the voice's name, and the name does not change when the fallback starts. With the server down, "Noted." would play from the cloned-voice recording, and the next sentence would come out in the fallback voice. That is the exact inconsistency the feature was meant to remove. The unhealthy state is also not rare, because I am in the middle of removing that engine, so unhealthy is about to be permanent.

**2. Silence instead of speech.** If a clip was listed in the manifest (the list of available clips) but could not be decoded, the code logged the error and returned, so nothing was spoken at all, even with a working speech engine available. A truncated file from a render that got stopped part way is a live risk, because I run renders at below-normal priority so they do not disturb other work.

Every other failure in the feature fell back to the speech engine: no clip set, an unreadable manifest, a line with no match. The pull request description even said unreadable clip sets fall back. This case did not, and the tests covered a missing manifest, not a bad clip.

**3. A type check.** The manifest loader promised in its own description never to raise an error. A corrupted file that still parsed as valid JSON would have raised one straight out of the speech function. It is real and cheap to fix, with low odds.

**4. Not currently reachable.** A stereo clip would have played at half speed, because the clip path skipped a reshape step that the synthesis path does 200 lines above it. I checked every clip on disk. All 487 are mono, and both renderers only produce mono. It is a guard against a future regression, not a bug anyone would hit today. I said so in the commit and did not write it up as a live defect.

## The fix

For defect 2 I separated reading the file from playing it. A read failure can be recovered from, so the speech engine speaks the line instead. A failure during playback stays fatal on purpose, because audio has already started and speaking the line again would repeat half of it.

## What I took from it

The agent's summary was accurate about what it built and tested, but wrong about one behaviour (the fall-back) and silent about the voice mixing, because the defect sat in how new code interacted with a fallback path 200 lines away, and no summary written by the author can show that.

---

*Project: Umbra. Rewritten for easier reading from the original in [umbra-sub-os-showcase / CASE-STUDIES.md](https://github.com/Ciaran11221/umbra-sub-os-showcase/blob/main/CASE-STUDIES.md#20-the-consistency-feature-that-would-have-introduced-an-inconsistency) (commit 5940844). Same facts and numbers.*
