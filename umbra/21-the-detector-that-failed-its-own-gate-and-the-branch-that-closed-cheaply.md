# 21. The detector that failed its own gate, and the branch that closed cheaply

**In one line:** I tested two theories for why one synthetic voice sounds robotic, and my echo detector failed the pass condition I had written for it, so I dropped that line of work after one listening pass.

| | |
|---|---|
| **Where** | Voice training (synthetic voices) |
| **Result** | Detector scored -0.32 against a required 0.8, and the work stopped. Both theories were dropped |
| **Topic** | Measurement, Debugging |

## The problem

I trained two voices from the same pipeline. I rate the female one at 95% and ship it. The male one sounds robotic on held sounds. 9.9% of his voiced runs (stretches of the audio where the pitch can be tracked) sit flat, against 1.7% for the human actor. The female voice measures 0.7% at the same setting. That is 14 times worse from the same pipeline.

## What I assumed

Days earlier I had fixed the female voice. Her actor has very distinct emotional registers: upbeat and chirpy normally, low and almost whispered when the scene is grim. A single-speaker model has no emotion input, so it blends them. I split the training data into registers and retrained with each as a separate speaker, and I preferred the result by a large margin.

My working theory was that the male actor has a distinct range too, just used less often, so the same fix would work.

## What the evidence said

- **The first tool could not test this.** The clustering tool I used on the female data looks for two groups of similar size. A register used in 3% of lines is a thin tail, not a second group. The tool could have said "no registers" for data that held exactly what I was looking for.
- **So I wrote a second measurement.** It reports the weight of each cluster, the share in one tail of the distribution, and prints the outlier lines with their text.
- **Female:** her minority register is 828 clips, 8.7% of the data. It sits 7 standard deviations away and is stable at every cluster count from 3 to 6.
- **Male:** the equivalent is 4 clips, all screaming, and 2 are the same take. The register fix is not available for him.

## Where I got it wrong

The outlier list also held 10 lines measured 26 to 31 semitones above his median pitch, nearly 2.5 octaves, on flat dialogue like "Let's go." and "Thanks, Rogue." The pitch tracker's own confidence on those was 0.01. I concluded the tracker had failed on ordinary speech, wrote that up and moved on.

Then I listened to them. They were not ordinary speech. Some were shouts or cries of pain, and one had another voice echoing in the background. The tracker had locked onto a second voice. My ranking was right and my explanation was wrong, and only my ear caught that.

## How I measured it

Background voice in the training data matters more than a mislabelled outlier, because nothing filters for it. My dataset builder drops clips for bad text, bad duration and read errors. It does not check for contamination, for either voice.

I had an echo detector from earlier that week, built after 2 earlier approaches failed. Before running it over both datasets, I wrote the pass condition into the script. The clips I had heard echo must score above the clips I had not, with a standardised separation of at least 0.8.

| Measurement | Value |
|---|---|
| Required separation | at least 0.8 |
| First result (echo and shouting clips grouped) | -0.32, so the script skipped the sweep |
| Clips with echo among the 10 outliers | 1 |
| That clip's rank among the 10 | 7th, below the unflagged baseline |
| Size of the set the 10 came from | 7,703 clips |

The suspect clips scored slightly lower than the clean baseline. The script printed the failure and skipped the sweep, instead of producing numbers from a detector that had just failed its own test.

That first test was weak, and the weakness was mine. I had grouped echo clips and shouting clips together, which destroys the contrast a two-sample test needs. So I got a clean label. Exactly 1 of the 10 clips had the echo, and re-scored it ranked 7th of 10. The detector is blind to it. It had been validated on 2 clips of the other voice and looks fitted to them.

## The cause

I have not proven the cause. Both theories failed.

- **Registers:** there is barely a second register in the male data.
- **Contamination:** 1 clip in 10 among the most extreme outliers of 7,703 means contamination is rare, and rare contamination cannot explain an artifact across ordinary speech.

What survived is duller and better supported: the male voice is far less periodic (it has less steady, repeating pitch).

| Measure | Male | Female |
|---|---|---|
| Voicing confidence | 0.058 | 0.219 |
| Non-silent frames that are voiced | half | a quarter |
| Harmonic-to-residual ratio | nearly 8 dB lower | |

Both sets of recordings are clean, at 51 dB signal-to-noise with a noise floor at -74 dB. So this is how the actor speaks, not how he was recorded. A model that predicts pitch has half as much pitch to learn from and defaults to holding a note.

Every number I have understates the problem. The flatness detector works by pitch tracking, so it cannot see unvoiced sounds, and unvoiced is where this voice lives. The 14-times gap is a floor, not a measurement.

## The fix

The branch closed. The cost was 1 listening pass and a detector run that refused to finish. If I had trusted the detector, swept both datasets and started filtering training data on a plausible number, that would have been a day's work built on an instrument that cannot see what it is pointed at.

Not done: the mechanism needs a retrain to prove. The check that would void it, comparing how the 2 datasets were extracted in case the difference is my pipeline rather than his delivery, has not been run. It is written down as the next step, with what result would void the finding.

## What I took from it

Writing the pass condition before the run meant the detector's failure stopped me from building on it.

---

*Project: Umbra. Rewritten for easier reading from the original in [umbra-sub-os-showcase / CASE-STUDIES.md](https://github.com/Ciaran11221/umbra-sub-os-showcase/blob/main/CASE-STUDIES.md#21-the-detector-that-failed-its-own-gate-and-the-branch-that-closed-cheaply) (commit 5940844). Same facts and numbers.*
