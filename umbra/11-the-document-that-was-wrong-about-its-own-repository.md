# 11. The document that was wrong about its own repository

**Symptom.** None in the code. The failure was in a handoff document — the
kind I write at the end of a working session so the next one doesn't start
cold. This one carried a section headed "where things actually are,
verified not assumed", listing which parts of a feature existed and which
didn't. Three of its rows said a component "does not exist". An
architectural decision had been built on top of those three rows, and was
sitting open, waiting on me.

**What I assumed.** That a section explicitly labelled "verified not
assumed" had been verified. It named specific files and specific handler
functions, in the tone of something checked rather than remembered. The
decision it framed offered three ways forward, and that framing only makes
sense if those three things are genuinely missing.

**Why that was wrong.** All three existed. The session-persistence module
was on disk at 17 KB. All six network handlers it claimed were missing
were in the main server file, at line numbers I could point at. The phone
screen it described as an unbuilt placeholder was written and compiled,
and the file that supposedly still held the placeholder had a comment
saying it had been replaced.

They had been merged the previous evening at 20:12. One command settles
when:

```
git merge-base --is-ancestor 31f64f5 HEAD
```

It confirms the branch was cut *after* that merge. The code was sitting in
the working directory of the session that wrote "does not exist", the
entire time it was writing it.

**What that cost.** Of the three paths the decision offered, one was
"build this whole component first" — already done, the previous evening.
Another was "leave it as a script-only tool and put it on the phone
later" — it was on the phone already. Two of three options were answers to
a question that had stopped being a question. Meanwhile the real problem
underneath, two overlapping stores for the same state, wasn't described
anywhere, because the document couldn't see it from where it thought it
was standing.

**What I built.** Not code, at first. The correction went into the new
plan as its own opening section, with the measurement beside each claim —
file size, line numbers, the ancestry command and its output — so the next
reader can re-run them instead of trusting me the way I'd trusted the last
document. The stale rows were struck through in place rather than deleted,
each annotated with which half of it had actually been right. One row was
partly correct, and quietly rewriting it would have destroyed that
distinction.

Then the check that made the difference. The claim "this module does not
exist" is disproved by `ls`. That is a one-second command, and nobody ran
it, because the sentence was already written down in a section that said
it had been.

**The lesson.** A tool returning nothing isn't evidence of absence until
you know the tool could have seen it — and a *document* returning nothing
is weaker evidence still, because a document can't see anything at all. It
records what someone believed at a moment, and it starts decaying the
instant it's written. "Verified" in a heading is a claim about the past,
not a property of the present.

This is the fourth time in this project that a confident statement of
absence turned out to be a stale or structurally blind check. The others:
a service reported down that was serving fine the whole time, a crash
diagnosed that had never happened, and a branch reported missing that
existed but hadn't been fetched. That repetition is why my working notes
for this project now require naming the single check that would
*disprove* a diagnosis, and running that one — not the check that would
confirm it.

---

*Project: Umbra. Copied 19 Nov 2026 from [umbra-sub-os-showcase / CASE-STUDIES.md](https://github.com/Ciaran11221/umbra-sub-os-showcase/blob/main/CASE-STUDIES.md#11-the-document-that-was-wrong-about-its-own-repository) at commit 5940844. The showcase copy is kept as well; if they differ, the showcase is newer.*
