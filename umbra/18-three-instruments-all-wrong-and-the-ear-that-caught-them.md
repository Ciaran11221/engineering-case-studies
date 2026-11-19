# 18. Three instruments, all wrong, and the ear that caught them

**In one line:** three measuring tools I built each gave a clean, wrong answer about a speech voice, and only my own listening caught them.

| | |
|---|---|
| **Where** | Text-to-speech voice training |
| **Result** | The held-note defect was 14 times worse in one model (9.9% against 0.7% of voiced runs), found by a one-variable comparison |
| **Topic** | Measurement, Debugging, Testing |

## The problem

I had trained a text-to-speech voice and it sounded off. My note at the time said the tempo was all over the place, with words dragging or spoken too quickly.

## What I assumed

That the model's timing was unstable and an earlier training checkpoint (a saved snapshot of the model) would be steadier. I had 16 checkpoints saved, so I set up a sweep to find the best one.

## What the evidence said

**Instrument 1: one draw from a random process.** The benchmark I measured against was a single render. This synthesiser is nondeterministic, meaning the same text gives different audio on every call. I knew that and had written it down as a warning to myself. I still built a benchmark that took 1 draw per line.

Measured properly, across repeated draws, the model's timing variance was 0.303 to 0.315. The one render I had been judging read 0.360. When I re-rendered the same model at the same settings and listened again, my verdict was "absolutely perfect." The checkpoint sweep, a test of the sentence splitter and an intelligibility scan had all run against an unlucky roll of the dice. Knowing a process is random is not enough. The measurement has to average over it, or it hands you one sample and lets you call it a finding.

**Instrument 2: a threshold on a continuous property.** With tempo settled, the real defect showed up: some sounds hung on one note, which sounds robotic. I built a detector that finds the voiced runs and measures how flat the pitch stays, and I swept the 1 setting that plausibly affected it. It reported the artifact falling from 12.8% of runs to 0.0%. My ear said a 10 to 20 percent improvement, so not fixed.

"Flat" was a threshold. A run that was 89% flat scored the same as one that was 10% flat, so a change that nudged runs just under the line looked like a total cure. Re-scored continuously, the same 3 settings told a different story. The fix made all pitch less steady, which pushed a 2nd property 14 points further from human than where it started. It was a trade, and the binary version of the metric hid the half that made it a trade.

**Instrument 3: a test that could not answer its question.** I ran a listening A/B between the 2 settings, twice, on a line whose defect was something else entirely: words merging together, which that setting cannot affect. That was 2 rounds of careful listening on a test that could not answer the question. Re-run on a line that actually showed the artifact, with 3 confounds (other differences between the 2 clips) removed on purpose, it gave a clean answer in 1 pass.

## How I measured it

I had 2 trained voices, and 1 of them sounded fine. 17 script lines existed in both source sets, with identical words. I rendered both models on those lines at identical settings and measured the same way.

```
voice A     0.7% of voiced runs held flat        reference recording: 0.6%
voice B     9.9%
```

| Measurement | Value |
|---|---|
| Voice A, voiced runs held flat | 0.7% (reference recording 0.6%) |
| Voice B, voiced runs held flat | 9.9% |
| Ratio | 14 times worse |
| Lines compared | 17 |

## The cause

The defect was in the model, not in any setting. The setting I had spent days tuning was never the cause. A control that differs in exactly one variable answered in a few minutes what a week of sweeps had not.

The same comparison also ended my other theory, that the 2 defects (held notes and merged words) shared a cause. Measuring syllable pulses showed voice A merging more than voice B, while still being the one I preferred. They were 2 unrelated problems, and 1 of them was not worth fixing.

## The fix

The numbers now rank candidates and my ear decides. When they disagree, I debug the instrument first.

## What I took from it

Each instrument was wrong in a different way, and each gave clean, plausible numbers. What caught all 3 was a human saying the number did not match what they heard. I had been treating the ear as something to replace with measurement. It was the only check the measurements had.

---

*Project: Umbra. Rewritten for easier reading from the original in [umbra-sub-os-showcase / CASE-STUDIES.md](https://github.com/Ciaran11221/umbra-sub-os-showcase/blob/main/CASE-STUDIES.md#18-three-instruments-all-wrong-and-the-ear-that-caught-them) (commit 5940844). Same facts and numbers.*
