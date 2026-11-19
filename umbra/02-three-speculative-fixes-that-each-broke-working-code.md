# 2. Three speculative fixes that each broke working code

Over a few sessions I made three changes on plausible reasoning, without
evidence. All three broke functionality that had been working.

**`recognizer.cancel()` to free the microphone.** The theory: the speech
recogniser was holding the mic and blocking wake detection during playback.
Reasonable. The log later disproved it — the detector restarts happily
mid-reply. Meanwhile cancelling left the recogniser unable to hear anything
afterwards: every wake produced "Yes?" and then silence.

**Shortening the endpointing timeouts.** The theory: ending the turn faster
would make it feel snappier. What actually happens is there's a natural pause
between the acknowledgement finishing and you starting to speak, and the
recogniser counted that as the end of the turn. Every command came back empty.

**Switching the audio source to `VOICE_COMMUNICATION`.** The theory: its echo
cancellation would help detect "stop" over the speaker. That processing is
built for phone calls — heavy suppression and gain control — and it degrades
exactly the signal a keyword model needs. Stop detection got worse, never
better.

**What I changed.** All three reverted. More usefully, each one is now a
comment in the source at the exact place someone would try it again:

> **NOTE: do not set the silence-length extras here.** Shortening them to end
> the turn faster looked reasonable and broke recognition outright... The
> platform defaults account for that gap; leave them alone.

**The lesson.** A negative result is worth as much as a positive one and is
forgotten twice as fast. Codify it where the next person — usually me — will
trip over it.

---

*Project: Umbra. Copied 19 Nov 2026 from [umbra-sub-os-showcase / CASE-STUDIES.md](https://github.com/Ciaran11221/umbra-sub-os-showcase/blob/main/CASE-STUDIES.md#2-three-speculative-fixes-that-each-broke-working-code) at commit 5940844. The showcase copy is kept as well; if they differ, the showcase is newer.*
