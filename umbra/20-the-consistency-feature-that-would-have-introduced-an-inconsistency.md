# 20. The consistency feature that would have introduced an inconsistency

**Symptom.** Umbra says a handful of fixed lines constantly — "Noted.",
"Still with you.", "Conversation mode on." In one three-and-a-half hour
window, twelve of twenty-six spoken replies were repeats of five lines. Every
one of them was a fresh synthesis, and this synthesiser is nondeterministic,
so each was a fresh roll of the dice. A bad roll is the likeliest explanation
for a warped reply I'd heard that morning.

**What I planned.** The phone already solved this: it plays a known-good
pre-rendered take for its fixed lines. The desktop didn't. So — render the
same clips for the desktop, and have the speech function check for a match
before it touches the synthesiser. I wrote the constraints out, and what came
back was a clean implementation, a new test file, and a passing suite.

**What the review found.** I review the diff, not the summary — a summary is
written by whatever did the work, so it cannot check itself. Four defects, and
the first one is the reason the rule exists.

The voice system has a cloned-voice engine that runs on a local server. When
that server is unhealthy, the speech function already handles it: it
substitutes a plain fallback voice, sentence by sentence, so a dead server
means a slightly different voice rather than silence.

The new clip lookup didn't know about any of that. It keyed off the voice's
*name*, which doesn't change when the fallback kicks in. So with the server
down, "Noted." would play from the genuinely cloned pre-rendered take, and the
very next sentence would come out in the generic fallback voice.

Two different voices, in one reply. The feature existed to remove exactly that
inconsistency. And the server being unhealthy isn't a rare edge case — I'm in
the middle of removing that engine entirely, so unhealthy is about to be its
permanent state.

**The second one was quieter and I nearly missed it.** If a clip file was
listed in the manifest but couldn't be decoded — a truncated file from a
render that got killed partway, which is a live risk because I run renders at
below-normal priority so they don't disturb anything — the code logged the
error and returned. Nothing was spoken at all, with a working synthesiser
sitting right there.

Every other failure mode in the feature fell through to synthesis: no clip
set, an unreadable manifest, a line with no match. The pull request body even
said unreadable clip sets fall through. This one didn't, and the test file
covered a missing *manifest* rather than a bad *clip*, so nothing caught it.

The fix was structural rather than a patch: separate reading the file from
playing it. A read failure is recoverable — synthesise instead. A failure
*during* playback deliberately stays fatal, because audio has already started
and re-speaking the line would repeat half of it.

**What I'd claim less of.** A third finding was a type check — the manifest
loader promised in its own docstring never to raise, and a corrupted file that
still parsed as valid JSON would have raised straight out of the speech
function. Real, cheap to fix, low odds.

The fourth I reported as not currently reachable, and said so in the commit.
A stereo clip would have played at half speed, because the clip path skipped a
reshape the synthesis path does two hundred lines above it. I checked every
clip on disk — all 487 are mono, and both renderers only emit mono. It's a
guard against a future regression, not a bug anyone would hit today. Writing
it up as a live defect would have been the easier and less useful thing to do.

**The thread.** The agent's summary was accurate about what it built. It was
accurate about what it tested. It was confidently wrong about one behaviour it
claimed — the fall-through — and completely silent about the voice-mixing
case, because nothing in its own change looked wrong in isolation. The defect
lived in the interaction between new code and a fallback path two hundred
lines away. No summary written by the author can surface that, however honest
the author is.

---

*Project: Umbra. Copied 19 Nov 2026 from [umbra-sub-os-showcase / CASE-STUDIES.md](https://github.com/Ciaran11221/umbra-sub-os-showcase/blob/main/CASE-STUDIES.md#20-the-consistency-feature-that-would-have-introduced-an-inconsistency) at commit 5940844. The showcase copy is kept as well; if they differ, the showcase is newer.*
