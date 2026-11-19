# 17. I attacked my own voice authentication, and it lost

**In one line:** I cloned my own voice and played it to my assistant, and every one of 73 clips was accepted as me.

| | |
|---|---|
| **Where** | Voice assistant, speaker identification |
| **Result** | 73 of 73 cloned clips passed; the clone's lowest score (0.662) was above my own median |
| **Topic** | Security, Testing, Measurement |

## The problem

My assistant identifies me by voice. A speaker-embedding model turns each utterance into a vector (a list of numbers describing the voice). It compares that with a stored voiceprint that adapts over time, and a similarity score decides whether the speaker is me. It had worked for weeks.

## What I assumed

That the threshold was the hard part. I had spent real effort calibrating it. From actual recordings, not guesswork, I found that my own clean speech scored 0.611 to 0.714 against my matured profile. That meant an earlier threshold of 0.80 had been quietly stalling the adaptation it was meant to drive. Fixing that felt like the work.

## What the evidence said

I had only ever tested it with me. Every measurement asked "does it recognise Ciaran?" and none asked "does it recognise only Ciaran?" So I cloned my own voice and pointed it at my own front door.

- The clone's worst score beat my own median.
- All 73 passed, with room to spare. It was not a marginal defeat that a tighter threshold would fix.
- No cut-off admits me and rejects this, because the clone sits above me. My system rated the fake as more me than I am.

## How I measured it

73 cloned utterances, scored against my real stored voiceprint:

```
median similarity   0.756
range               0.662 - 0.802
my own real clean speech, for comparison    0.611 - 0.714 (mean 0.653)
```

The same clone was also worse at being understood. I fed both through the speech recogniser (the part that turns audio into text):

```
exact transcription, real speech   53/71 = 75%
exact transcription, clone         27/73 = 37%
```

| Measurement | Value |
|---|---|
| Clone clips scored | 73 |
| Clone clips that passed | 73 (100%) |
| Clone median / range | 0.756 / 0.662 to 0.802 |
| My real clean speech, range / mean | 0.611 to 0.714 / 0.653 |
| Exact transcription, real speech | 53 of 71 (75%) |
| Exact transcription, clone | 27 of 73 (37%) |

The clone was about half as intelligible and still won completely against identity. The two properties are unrelated. It was good enough to defeat the check "is this him?" and much worse at the check "what did he say?". I had assumed a convincing clone would be convincing at both.

## The cause

I had measured recognition and never impersonation. The threshold cannot separate me from a clone that scores above me.

## The fix

A tighter threshold cannot work, so I did not pretend otherwise. The voice match was changed from a decision into a signal, and a third state was added between "trust silently" and "do nothing". In a narrow borderline band, above background noise and below my own real floor, the assistant asks a spoken arithmetic question and logs the answer. It has 3 deliberate constraints:

- **It never blocks.** The clone scores in the trusted range, so a hard gate would inconvenience me without stopping the attack it was built for. A gate that can be walked around is worse than an honest signal, because it invites you to rely on it.
- **It fires in a band, not on a score.** Below the band it is not an attempt to talk to the assistant at all. Above it, the match is within my own normal range and challenging it would be noise.
- **It has a cooldown.** Otherwise a cold, or a bad microphone day, would mean an arithmetic question on every wake.

What this does not do: it does not stop a competent clone, and nothing in this system does. It raises the cost of a casual one and creates a logged record where there was none. That is much less than "voice authentication, secured."

## What I took from it

A defence that has only been tested by its legitimate user has not been tested. The attack was cheap: an evening, a consumer GPU and enough clean audio, with no rented compute or specialist knowledge. Anyone using voice as identity should assume their threshold has already lost and design for what remains true afterwards. Mine now logs instead of promising.

---

*Project: Umbra. Rewritten for easier reading from the original in [umbra-sub-os-showcase / CASE-STUDIES.md](https://github.com/Ciaran11221/umbra-sub-os-showcase/blob/main/CASE-STUDIES.md#17-i-attacked-my-own-voice-authentication-and-it-lost) (commit 5940844). Same facts and numbers.*
