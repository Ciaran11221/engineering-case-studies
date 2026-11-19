# 21. The detector that failed its own gate, and the branch that closed cheaply

**Symptom.** I trained two voices from the same pipeline. The female one I
rate at ninety-five percent and happily ship. The male one sounds robotic on
held sounds — measurably so: nine point nine percent of his voiced runs sit
flat, against the human actor's one point seven. The female voice measures
zero point seven at the same setting. Fourteen times worse, same pipeline.

**What I assumed.** Days earlier I'd fixed the female voice, and the fix was a
good one. Her actor has very distinct emotional registers — upbeat and chirpy
normally, low and almost whispered when the scene is grim — and a
single-speaker model has no emotion input, so it blends them. Splitting the
training data into registers and retraining with each as a separate speaker
gave a voice I preferred by a large margin.

So: do the same for male. My working theory was that his actor has a distinct
range too, just used less often.

**Why that needed a different test.** The clustering tool I'd used on the
female data falsifies on *bimodality* — it's built to catch two comparably
sized modes. A register used in three percent of lines isn't bimodal. It's a
tail. Run that tool on the male data and it could have returned "no registers"
for data holding exactly what I was looking for, and I'd have believed it.

So I wrote a second measurement: component weights rather than just the best
cluster count, one-sided tail shares rather than symmetric spread, and the
outlier lines themselves printed with their text.

**What it found.** Female's minority register is 828 clips, eight point seven
percent, separated by seven standard deviations, and stable at every cluster
count from three to six. Male's genuinely-voiced equivalent is **four clips**.
All of them screaming. Two are the same take.

The register fix that won for the female voice is not available for the male
one. There is barely a second register present to separate.

**Where I got it wrong.** The outlier list also held ten lines that measured
twenty-six to thirty-one semitones above his median pitch — nearly two and a
half octaves — on flat conversational dialogue like "Let's go." and "Thanks,
Rogue." The pitch tracker's own confidence on those was 0.01. I concluded they
were tracker failures on ordinary speech, wrote it up, and moved on.

Then I listened to them. They weren't ordinary speech. Some were shouts or
cries of pain. One had another voice echoing in the background. The tracker
hadn't failed on nothing — it had locked onto a second voice. My ranking was
right and my explanation of it was wrong, and only the ear caught that.

**The gate that saved the next day's work.** Background voice in the training
data is a much more interesting problem than a mislabelled outlier, because
nothing filters for it. My dataset builder drops clips for bad text, bad
duration and read errors. Contamination isn't checked at all, for either
voice. Every affected clip is training the model.

I had an echo detector from earlier that week — a cepstral method, after two
earlier approaches failed. Before sweeping it across both datasets I wrote
the pass condition into the script: the clips I'd heard echo in must score
above the clips I hadn't, with a standardised separation of at least 0.8.

It came back at **minus 0.32**. The suspect clips scored slightly *lower* than
the clean baseline. The script printed the failure and skipped the sweep
rather than producing numbers off a detector that had just failed its own
test.

That first test was weak and the weakness was mine — I'd lumped echo clips and
shouting clips into one group, which destroys the contrast a two-sample test
needs. So I got a clean label instead: of those ten clips, exactly one had the
echo. Re-scored, that clip ranked **seventh of ten**, below the unflagged
baseline. The detector is blind to it. It had been validated on two clips of
the other voice, and it looks fitted to them.

**The part that mattered more than the detector.** One clip in ten. That group
was the ten most extreme outliers in a set of 7,703. If only one of *those*
carries background voice, contamination is rare rather than systemic — and
rare contamination cannot explain an artifact that shows up across ordinary
speech.

So the branch closed. Total cost: one listening pass and a detector run that
refused to finish. The version of this where I trust the detector, sweep both
datasets, get a plausible-looking number and start filtering training data on
it is a day's work built on an instrument that cannot see the thing it's
pointed at.

**What's left standing.** Neither the register theory nor the contamination
theory survived. What did survive is duller and better supported: the male
voice is simply far less periodic. Voicing confidence 0.058 against her 0.219.
Half his non-silent frames against a quarter of hers. Harmonic-to-residual
ratio nearly eight decibels lower. And both sets of recordings are clean —
fifty-one decibels signal-to-noise, noise floor at minus seventy-four — so
this is how the actor speaks, not how he was recorded. A model that predicts
pitch has half as much pitch to learn from, and defaults to holding a note.

It also means every number I have understates the problem: the flatness
detector works by pitch-tracking, so it is blind to unvoiced sounds, and
unvoiced is precisely where this voice lives. The fourteen-times gap is a
floor, not a measurement.

I haven't proven the mechanism. That needs a retrain, and the check that would
kill it — comparing how the two datasets were extracted, in case the
difference is my pipeline rather than his delivery — hasn't been run yet. It's
written down as the next step, with what result would void the finding.

---

*Project: Umbra. Copied 19 Nov 2026 from [umbra-sub-os-showcase / CASE-STUDIES.md](https://github.com/Ciaran11221/umbra-sub-os-showcase/blob/main/CASE-STUDIES.md#21-the-detector-that-failed-its-own-gate-and-the-branch-that-closed-cheaply) at commit 5940844. The showcase copy is kept as well; if they differ, the showcase is newer.*
