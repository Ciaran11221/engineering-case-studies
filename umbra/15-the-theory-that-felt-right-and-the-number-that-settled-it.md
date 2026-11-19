# 15. The theory that felt right and the number that settled it

**Symptom.** A multi-turn voice exchange — the assistant reads out a short
list, asks which one I meant, I name one — lost all memory of the list
between the question and my answer. It replied as if I'd said the name out
of nowhere, with nothing to attach it to.

**The first explanation, and why I didn't accept it.** The claim handed to
me was that I'd paused about fourteen seconds before answering, and that
the assistant's wake claim had quietly expired somewhere in that gap. It's
a plausible story — voice systems do lose the floor to silence — and it
was stated with real confidence. I hadn't paused. I answered right after
the assistant finished talking, and I said so.

**What checking it actually looked like.** Rather than argue from memory
against memory, the fix was to find a number in the system that didn't
depend on either of us being right. The component doing the listening has
a hard configured cap on how long it will wait for speech to start — eight
seconds. The gap in the log was closer to fourteen. Eight from fourteen
leaves six seconds that could not have been spent listening for me at all,
which meant they were spent somewhere else: the assistant was still
synthesizing and speaking a four-sentence reply before it ever started
listening.

**What that reframed.** The bug wasn't "the user paused too long." It was
that a long spoken reply eats into the same fifteen-second window the
system uses to hold the floor for an answer, and the code only renewed
that hold *after* getting a reply back — never while it was still talking,
never while it was still listening. A short answer never showed the
problem. A four-item list read out loud, every time, did.

**The part worth naming.** This exact class of bug had already been found
and fixed once before, three weeks earlier — and the comment describing
that fix was still sitting in the file, describing almost the same
symptom. The first fix renewed the hold after a successful answer came
back, which covers a *fast* back-and-forth. It never covered a *single*
reply that was slow to say out loud. Not a new bug. The same bug,
incompletely closed the first time, waiting for a case that happened to
exercise the part that wasn't covered.

**What changed as a result.** The hold now gets renewed the moment
listening starts, not only after it returns — covering a slow reply's own
talking time the same way the first fix covered slow turnarounds between
turns. And every reply now logs how long it took to say, specifically so
the next time something looks like a fourteen-second silence, the log can
answer whether it actually was one instead of two people trading guesses
about it.

**The lesson.** The theory that gets offered with confidence is not
automatically the theory that's correct, including from a tool that just
watched the whole exchange happen. The fix here wasn't "trust the second
guess more" — it was finding the one number already sitting in the
configuration that neither guess needed to win the argument to settle.

---

*Project: Umbra. Copied 19 Nov 2026 from [umbra-sub-os-showcase / CASE-STUDIES.md](https://github.com/Ciaran11221/umbra-sub-os-showcase/blob/main/CASE-STUDIES.md#15-the-theory-that-felt-right-and-the-number-that-settled-it) at commit 5940844. The showcase copy is kept as well; if they differ, the showcase is newer.*
