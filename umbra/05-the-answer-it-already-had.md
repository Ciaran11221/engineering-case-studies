# 5. The answer it already had

**Symptom.** Mid-conversation, I mentioned a friend's name and a fact about
them in passing — never said "remember this," just said it. A few turns
later I asked whether it had caught that. It told me, flatly, that it had no
record of the conversation at all.

**What I assumed.** That this was a memory gap — the fact simply hadn't been
saved, and the honest answer was "no."

**Why that was wrong.** It had been saved. The fact was sitting in the exact
context every reply already has access to, at the moment the denial was
generated. The model wasn't missing information. It had the right context in
front of it and produced a fluent, confident, wrong answer anyway.

**What I built.** A question shaped like "did you remember X" or "what did I
tell you about X" is now answered by a direct, deterministic lookup against
what's actually stored, before the model is ever asked. Not a better prompt
asking it to please check its context properly — removing the step where it
gets to guess.

**The lesson.** Having the right information available isn't the same as
using it. For anything that has to be *right* rather than *plausible*, the
fix isn't phrasing the question better. It's not asking a model the question
at all.

---

*Project: Umbra. Copied 19 Nov 2026 from [umbra-sub-os-showcase / CASE-STUDIES.md](https://github.com/Ciaran11221/umbra-sub-os-showcase/blob/main/CASE-STUDIES.md#5-the-answer-it-already-had) at commit 5940844. The showcase copy is kept as well; if they differ, the showcase is newer.*
