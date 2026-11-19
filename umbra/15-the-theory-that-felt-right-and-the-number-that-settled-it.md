# 15. The theory that felt right and the number that settled it

**In one line:** the assistant forgot the list it had just read out, and a number already in the settings showed the confident explanation was wrong.

| | |
|---|---|
| **Where** | Voice assistant, multi-turn conversation |
| **Result** | 6 seconds of the gap were the assistant still talking, not me pausing; the fix renews the hold when listening starts |
| **Topic** | Debugging, Measurement |

## The problem

In a voice exchange, the assistant reads out a short list and asks which one I meant, and I name one. It lost all memory of the list between the question and my answer. It replied as if I had said the name out of nowhere, with nothing to attach it to.

## What I assumed

I did not assume anything myself. The explanation I was handed, with real confidence, was that I had paused about 14 seconds before answering, and that the assistant's hold on the conversation (its "wake claim") had expired in that gap. It is a plausible story, since voice systems do lose the floor to silence. But I had not paused. I answered right after the assistant finished talking, and I said so.

## What the evidence said

- The component that listens has a hard configured limit on how long it waits for speech to start: 8 seconds.
- The gap in the log was close to 14 seconds.
- 14 minus 8 leaves 6 seconds that could not have been spent listening for me. They were spent elsewhere: the assistant was still synthesizing and speaking a 4-sentence reply before it started listening.

## How I measured it

I arbitrated with a number that did not depend on either of us being right, and not by arguing from memory against memory.

| Measurement | Value |
|---|---|
| Gap in the log | about 14 seconds |
| Configured wait for speech to start | 8 seconds |
| Time not spent listening for me | 6 seconds |
| Length of the reply being spoken in that time | 4 sentences |

## The cause

A long spoken reply eats into the same 15-second window the system uses to hold the floor for an answer. The code only renewed that hold after getting a reply back, never while it was still talking and never while it was listening. A short answer never showed the problem. A 4-item list read out loud showed it every time.

This class of bug had been found and fixed once before, 3 weeks earlier. The comment describing that fix was still in the file, describing almost the same symptom. The first fix renewed the hold after a successful answer came back, which covers a fast back-and-forth. It never covered a single reply that is slow to say out loud. It was the same bug, closed incompletely the first time.

## The fix

The hold is now renewed the moment listening starts, not only after it returns. That covers a slow reply's own talking time the same way the first fix covered slow turnarounds between turns. Every reply also logs how long it took to say, so the next time something looks like a 14-second silence, the log can show whether it was one, instead of 2 people trading guesses about it.

## What I took from it

The theory offered with confidence is not automatically the correct one, even from a tool that watched the whole exchange. What worked was to find the one number already in the configuration that settled it without either guess having to win.

---

*Project: Umbra. Rewritten for easier reading from the original in [umbra-sub-os-showcase / CASE-STUDIES.md](https://github.com/Ciaran11221/umbra-sub-os-showcase/blob/main/CASE-STUDIES.md#15-the-theory-that-felt-right-and-the-number-that-settled-it) (commit 5940844). Same facts and numbers.*
