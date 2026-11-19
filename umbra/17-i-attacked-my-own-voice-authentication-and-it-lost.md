# 17. I attacked my own voice authentication, and it lost

**What I'd built.** My assistant identifies me by voice. A speaker-embedding
model turns each utterance into a vector, compares it against a stored
voiceprint that adapts over time, and a similarity score decides whether the
speaker is me. It worked. It had worked for weeks.

**What I assumed.** That the threshold was the hard part. I'd spent real effort
calibrating it — including finding, from actual recordings rather than
guesswork, that my own clean speech scored 0.611–0.714 against my matured
profile, which meant an earlier threshold of 0.80 had been quietly stalling
the adaptation it was supposed to drive. Fixing that felt like the work.

**Why that was wrong.** I'd only ever tested it with *me*. Every measurement
asked "does it recognise Ciaran?" and none asked "does it recognise *only*
Ciaran?" So I cloned my own voice and pointed it at my own front door.

73 cloned utterances, scored against my real stored voiceprint:

    median similarity   0.756
    range               0.662 - 0.802
    my own real clean speech, for comparison    0.611 - 0.714 (mean 0.653)

**The clone's *worst* score beat my own *median*.** Every one of the 73 passed,
100%, with room to spare — not a marginal defeat where a tighter threshold
would help. There is no cut-off that admits me and rejects this, because the
clone sits *above* me. My system rated the fake as more me than I am.

**The part I didn't expect.** The same clone was measurably *worse* at being
understood. Feeding both through the speech recogniser:

    exact transcription, real speech   53/71 = 75%
    exact transcription, clone         27/73 = 37%

Half as intelligible, and still a total win against identity. The two
properties are unrelated — it was good enough to defeat the check asking "is
this him?" while being far worse at the check asking "what did he say?" I had
assumed a convincing clone would be convincing at both.

**What I changed, and what I deliberately didn't.** The obvious response is a
tighter threshold. The data says that cannot work, so I didn't pretend
otherwise.

Instead the voice match was demoted from a decision to a signal, and a third
state was added between "trust silently" and "do nothing": in a narrow
borderline band — above background noise, below my own real floor — the
assistant asks a spoken arithmetic question and logs the answer.

Three constraints on it, all deliberate:

- **It never blocks.** It cannot: the clone scores in the trusted range, so a
  hard gate here would inconvenience me without stopping the attack it was
  built for. A gate that can be walked around is worse than an honest signal,
  because it invites you to rely on it.
- **It fires in a band, not on a score.** Below the band it isn't an attempt to
  talk to the assistant at all; above it, the match is within my own normal
  range and challenging it would be noise.
- **It has a cooldown.** A cold, or a bad microphone day, would otherwise mean
  an arithmetic question on every single wake.

**What this does not do.** It does not stop a competent clone — nothing in this
system does. It raises the cost of a casual one and creates a logged record
where previously there was none. That's the honest description, and it is much
less than "voice authentication, secured."

**The lesson.** A defence that has only been tested by its legitimate user has
not been tested. Every measurement I'd taken was a *recognition* measurement;
not one was an *impersonation* measurement, and the system passed all of the
first kind right up until the moment the second kind was run.

The uncomfortable part is how cheap the attack was. No rented compute, no
specialist knowledge — an evening, a consumer GPU, and enough clean audio.
Anyone building voice-as-identity today should assume their threshold has
already lost and design for what remains true afterwards. Mine now logs
instead of promising.

---

*Project: Umbra. Copied 19 Nov 2026 from [umbra-sub-os-showcase / CASE-STUDIES.md](https://github.com/Ciaran11221/umbra-sub-os-showcase/blob/main/CASE-STUDIES.md#17-i-attacked-my-own-voice-authentication-and-it-lost) at commit 5940844. The showcase copy is kept as well; if they differ, the showcase is newer.*
