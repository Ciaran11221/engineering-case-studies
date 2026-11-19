# 1. The log that only recorded successes

**Symptom.** The wake phrase was unreliable. I'd say "Hey Phone" three or four
times before it answered, sometimes shouting.

**What I assumed.** The detection threshold was too high. Obvious fix: lower it.

**Why that was wrong.** I checked the log first and found two clean detections
in the window I'd tested. That looked like the wake word working fine — so I
concluded the problem was elsewhere.

It wasn't. A failed detection writes *nothing*. The keyword spotter reports a
match or stays silent; there's no confidence score in its result type. So five
failures followed by one success and a single success on the first try produce
byte-identical logs. I'd read a log that was structurally incapable of showing
me the thing I was investigating, and drawn a confident conclusion from it.

**What I built.** Since the model wouldn't tell me about near-misses, I measured
the audio myself. The capture loop now tracks a running noise floor and logs
every burst of speech that ended without a match:

```
Speech heard but nothing matched: peak=0.1282 floor=0.0002 length=500ms
Speech heard but nothing matched: peak=0.1338 floor=0.0011 length=500ms
```

**What it showed.** Twenty-five bursts. Peaks of 0.08–0.17 against a floor
around 0.001 — a signal a hundred times above the noise, in a quiet room, for
half a second each. Zero detections.

That killed the threshold theory outright. My voice was arriving in excellent
condition and the model was matching nothing.

**The corroboration.** In the same period the desktop, listening on its own
microphone, transcribed the same speech correctly seven times:

```
wake-check heard: 'hey phone.'  -> Wake phrase for the phone -- staying quiet
```

Two independent systems, same room, same voice. One understood it every time;
the other never did. The difference was the microphone — a Bluetooth headset
whose narrowband capture leaves the upper half of every audio frame empty, which
is exactly the input a model trained on 16kHz speech cannot use.

**What I changed.** On a headset, idle listening now goes through full speech
recognition instead of the keyword model — which handles that microphone
without difficulty. The keyword model stays for the phone's own mic, where it's
cheap enough to run all day.

**The lesson.** Absence of evidence in a log is not evidence of absence. Before
trusting a log to answer a question, check that it's capable of recording the
answer.

---

*Project: Umbra. Copied 19 Nov 2026 from [umbra-sub-os-showcase / CASE-STUDIES.md](https://github.com/Ciaran11221/umbra-sub-os-showcase/blob/main/CASE-STUDIES.md#1-the-log-that-only-recorded-successes) at commit 5940844. The showcase copy is kept as well; if they differ, the showcase is newer.*
