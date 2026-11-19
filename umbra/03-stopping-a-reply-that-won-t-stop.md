# 3. Stopping a reply that won't stop

**Symptom.** Saying "stop" during a long reply worked perhaps one time in four,
and needed repeating and raising my voice.

**The obvious approach, and why it failed.** "Stop" was a keyword in the same
spotting model as the wake phrase. Tuning its threshold down made it fire on
near-silence — the log caught it triggering at an audio level *below* the
speech floor, on nothing at all — without making it any more reliable when
actually spoken. It's one short syllable competing against the assistant's own
voice through a loudspeaker a metre away. The word simply isn't a strong enough
signal.

**Reframing it.** Once instrumented, the timing was damning. The one time it did
fire, it fired five seconds into a six-second reply. Mechanically a success.
Practically useless — it stopped her as she was finishing anyway.

The insight was that the *word* carries almost no information. There is nothing
else you'd be saying while the assistant is mid-sentence. So the trigger doesn't
need to be a word at all.

**What I built.** Barge-in: 300ms of sustained speech above an adapted noise
floor cuts the reply short. No model, no threshold on a phrase, no vocabulary.

The subtlety is the noise floor, because the assistant's own voice is in the
room. During a 1.5 second warm-up at the start of each reply the floor rises to
include her, so only something clearly louder counts as an interruption. That
produces an asymmetry I chose deliberately:

- On a headset her voice is echo-cancelled, the floor settles near silence, and
  barge-in triggers easily.
- Over the phone's loudspeaker the floor settles high and barge-in becomes hard.

That's the safe direction. A reply you can't interrupt is a nuisance. A reply
that interrupts *itself* on the speaker's first word is unusable.

**The lesson.** When something is hard to detect, question whether you're
detecting the right thing. The best fix here removed the detection problem
rather than solving it.

---

*Project: Umbra. Copied 19 Nov 2026 from [umbra-sub-os-showcase / CASE-STUDIES.md](https://github.com/Ciaran11221/umbra-sub-os-showcase/blob/main/CASE-STUDIES.md#3-stopping-a-reply-that-wont-stop) at commit 5940844. The showcase copy is kept as well; if they differ, the showcase is newer.*
