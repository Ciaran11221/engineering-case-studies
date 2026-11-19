# 4. A microphone that flapped

**Symptom.** After adding Bluetooth headset support, wake detection got worse
rather than better — which was the opposite of expected, since the headset mic
is closer to the speaker's mouth.

**What the log showed.** Not a detection problem at first. A routing one:

```
Listening through Bluetooth headset
Wake word detection started (idle)
...
Back on the phone mic
Listening through Bluetooth headset
```

Capture was bouncing between the headset and the phone, several times a minute.
Each bounce tore down the audio pipeline and built a new one with a fresh model
stream — so any phrase spoken across a bounce was cut in half and could never
match.

**The cause.** I'd registered a callback for audio devices appearing and
disappearing, and acted on every event immediately. But a Bluetooth link drops
out of the device list *during its own state transitions*. My code read that as
"headset unplugged", switched to the phone mic, saw it reappear, and switched
back — a feedback loop of my own making.

**What I changed.** One pending routing change at a time, with a settle period
before believing the device list. Cancel any change already queued when a new
event arrives.

**The related mistake.** In the same work I labelled the device
`Bluetooth headset (classic, narrowband mic)` in the log. Classic Bluetooth runs
at 8kHz *or* 16kHz depending on what the headset negotiates — I never measured
which. Later I read that log line back as evidence for a narrowband diagnosis,
having written the word "narrowband" into it myself. The log now prints the
device's actual reported sample rates.

**The lesson.** Be careful what you assert in a log. Anything you write there
will eventually be read back as evidence, including by you, and a label that
encodes an assumption is worse than no label at all.

---

*Project: Umbra. Copied 19 Nov 2026 from [umbra-sub-os-showcase / CASE-STUDIES.md](https://github.com/Ciaran11221/umbra-sub-os-showcase/blob/main/CASE-STUDIES.md#4-a-microphone-that-flapped) at commit 5940844. The showcase copy is kept as well; if they differ, the showcase is newer.*
