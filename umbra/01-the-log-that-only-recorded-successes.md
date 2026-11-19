# 1. The log that only recorded successes

**In one line:** the phone's wake phrase kept failing, and the log said it was fine, because the log could only record successes.

| | |
|---|---|
| **Where** | Phone app, voice wake-up |
| **Result** | 25 missed wake phrases found, 0 heard; fixed by changing how the phone listens on a headset |
| **Topic** | Debugging, Measurement |

## The problem

I had to say "Hey Phone" 3 or 4 times before it answered, sometimes shouting.

## What I assumed

The detection threshold was too high, so the fix was to lower it.

## What the evidence said

- The log showed 2 clean detections in the time I tested. That looked like it was working.
- A failed detection writes nothing. The wake-word model either reports a match or says nothing. It gives no score.
- So 5 failures then 1 success leaves exactly the same log as 1 success on the first try. The log could not show the problem I was looking for.

## How I measured it

The model would not report near-misses, so I measured the sound myself. The phone now logs every burst of speech that ends without a match:

```
Speech heard but nothing matched: peak=0.1282 floor=0.0002 length=500ms
Speech heard but nothing matched: peak=0.1338 floor=0.0011 length=500ms
```

| Measurement | Value |
|---|---|
| Bursts of speech heard | 25 |
| My voice compared with background noise | about 100 times louder (peaks 0.08 to 0.17, background about 0.001) |
| Detections | 0 |
| Same phrase heard correctly by the desktop, on its own mic, same room | 7 times |

My voice was arriving loud and clear. The threshold was not the problem.

## The cause

The microphone. It was a Bluetooth headset, and its mic sends only the lower half of the sound frequencies. The wake-word model was trained on full-range speech (16 kHz recordings), so it could not use that input.

## The fix

On a headset, the phone now uses full speech recognition to listen for the wake phrase, which handles that mic fine. The phone's own mic keeps the wake-word model, because it is cheap enough to run all day.

## What I took from it

Before trusting a log to answer a question, check that it can record the answer.

---

*Project: Umbra. Rewritten for easier reading from the original in [umbra-sub-os-showcase / CASE-STUDIES.md](https://github.com/Ciaran11221/umbra-sub-os-showcase/blob/main/CASE-STUDIES.md#1-the-log-that-only-recorded-successes) (commit 5940844). Same facts and numbers.*
