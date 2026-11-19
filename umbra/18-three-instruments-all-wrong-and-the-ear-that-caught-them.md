# 18. Three instruments, all wrong, and the ear that caught them

**Symptom.** I'd trained a text-to-speech voice and it sounded off. My note at
the time: the tempo is all over the place, words drag or get spoken too quickly.

**What I assumed.** The model's timing was unstable, and an earlier training
checkpoint would be steadier. I had sixteen checkpoints saved, so I set up a
sweep to find the best one.

**Why that was wrong.** The benchmark I was measuring against was a single
render, and this synthesiser is nondeterministic — the same text gives different
audio every call. I knew that. I'd written it down as a warning to myself. I
still built a benchmark that took one draw per line.

Measured properly, across repeated draws, the model's timing variance was
0.303–0.315. The one render I'd been judging read 0.360. When I re-rendered the
same model at the same settings and listened again, my verdict was "absolutely
perfect."

The checkpoint sweep, a test of the sentence splitter, and an intelligibility
scan had all run against an unlucky roll of the dice. Knowing a process is
random isn't enough — the measurement has to average over it, or it will
quietly hand you one sample and let you call it a finding.

**The second instrument.** With tempo settled, the real defect surfaced: some
sounds hung on one note, which reads as robotic. I built a detector for it —
find the voiced runs, measure how flat the pitch stays — and swept the one
setting that plausibly affected it.

It reported the artifact going from 12.8% of runs to 0.0%. Fixed.

The ear said "a 10 to 20 percent improvement." Not fixed.

The metric was the problem. "Flat" was a threshold: a run 89% flat scored
identically to one 10% flat, so a change that nudged runs just under the line
registered as a total cure. Re-scored continuously, the same three settings told
a different story — the fix worked by making *all* pitch less steady, pushing a
second property 14 points further from human than where it started. It was a
trade, not a fix, and the binary version of the metric had hidden exactly the
half that made it a trade.

**The third instrument.** I then ran a listening A/B between the two settings.
Twice. On a line whose defect turned out to be something else entirely — words
merging together, which that setting cannot affect at all. Two rounds of
careful listening, on a test that could not have answered the question it was
asked. Re-run on a line that actually exhibited the artifact, with three
confounds deliberately removed, it gave a clean answer in one pass.

**What actually worked.** I had two trained voices, and one of them sounded
fine. Seventeen script lines existed in both source sets — identical words. So I
rendered both models on those lines at identical settings, and measured the same
way.

    voice A     0.7% of voiced runs held flat        reference recording: 0.6%
    voice B     9.9%

Fourteen times worse, same setting, same words. That one run ended the
question: the defect was in the model, not in any knob, and the setting I'd
spent days tuning had never been the cause. A control that differs in exactly
one variable answered in a few minutes what a week of sweeps hadn't.

The same comparison then killed my other theory. I'd assumed the two defects —
held notes and merged words — shared a cause. Measuring syllable pulses showed
voice A merging *more* than voice B, while still being the one I preferred. Two
unrelated problems, and one of them wasn't worth fixing.

**The lesson.** Every instrument I built in this session was wrong in a
different way: one sampled a random process once, one thresholded a continuous
property, one tested a line that couldn't exhibit the effect. Each was wrong
*confidently* — they all produced clean, plausible numbers.

What caught all three was the same thing: a human saying the number didn't match
what they heard. I'd been treating the ear as the thing to be replaced by
measurement. It was actually the only check the measurements had. Now the
numbers rank candidates and the ear decides, and when they disagree I debug the
instrument first.

---

*Project: Umbra. Copied 19 Nov 2026 from [umbra-sub-os-showcase / CASE-STUDIES.md](https://github.com/Ciaran11221/umbra-sub-os-showcase/blob/main/CASE-STUDIES.md#18-three-instruments-all-wrong-and-the-ear-that-caught-them) at commit 5940844. The showcase copy is kept as well; if they differ, the showcase is newer.*
