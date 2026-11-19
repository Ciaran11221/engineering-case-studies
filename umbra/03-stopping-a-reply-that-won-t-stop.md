# 3. Stopping a reply that won't stop

**In one line:** saying "stop" during a long reply worked about 1 time in 4, so I replaced the spoken word with a simple rule: sustained speech cuts the reply off.

| | |
|---|---|
| **Where** | Phone app, interrupting the assistant's spoken reply |
| **Result** | "Stop" fired 5 seconds into a 6-second reply, too late to matter; replaced by detection of 300ms of sustained speech |
| **Topic** | Debugging, Design decision |

## The problem

Saying "stop" during a long reply worked perhaps 1 time in 4. I had to repeat it and raise my voice.

## What I assumed

"Stop" was a keyword in the same keyword-spotting model as the wake phrase, so I assumed the fix was tuning its threshold (the score it needs before it fires).

## What the evidence said

- Lowering the threshold made it fire on near-silence. The log caught it triggering at an audio level below the speech floor, on nothing at all.
- It did not become any more reliable when I actually said the word.
- "Stop" is 1 short syllable, competing against the assistant's own voice coming from a loudspeaker a metre away. The word is not a strong enough signal.

## How I measured it

I added timing to the log. The one time it did fire, it fired 5 seconds into a 6-second reply. It counted as a success, but it stopped her as she was finishing anyway.

## The cause

I was detecting the wrong thing. While the assistant is mid-sentence, there is nothing else I would be saying, so the word carries almost no information. The trigger does not need to be a word at all.

## The fix

Barge-in (interrupting a reply by speaking over it): 300ms of sustained speech above an adapted noise floor cuts the reply short. It uses no model, no phrase threshold and no vocabulary.

The noise floor is the tricky part, because the assistant's own voice is in the room. During a 1.5-second warm-up at the start of each reply, the floor rises to include her voice, so only something clearly louder counts as an interruption. I chose the resulting difference on purpose:

- On a headset her voice is echo-cancelled, the floor settles near silence, and barge-in triggers easily.
- Over the phone's loudspeaker the floor settles high and barge-in becomes hard.

That is the safe direction. A reply you cannot interrupt is a nuisance. A reply that interrupts itself on the speaker's first word is unusable.

## What I took from it

When something is hard to detect, ask whether you are detecting the right thing. The best fix here removed the detection problem instead of solving it.

---

*Project: Umbra. Rewritten for easier reading from the original in [umbra-sub-os-showcase / CASE-STUDIES.md](https://github.com/Ciaran11221/umbra-sub-os-showcase/blob/main/CASE-STUDIES.md#3-stopping-a-reply-that-wont-stop) (commit 5940844). Same facts and numbers.*
