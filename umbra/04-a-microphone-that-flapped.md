# 4. A microphone that flapped

**In one line:** after I added Bluetooth headset support, wake detection got worse, because my own code kept switching between the headset and the phone mic.

| | |
|---|---|
| **Where** | Phone app, choosing which microphone to listen on |
| **Result** | Audio capture was switching between headset and phone several times a minute; fixed with a settle period before acting on device changes |
| **Topic** | Debugging, Measurement |

## The problem

After I added Bluetooth headset support, wake detection got worse. I expected the opposite, since a headset mic is closer to the speaker's mouth.

## What I assumed

That it was a detection problem.

## What the evidence said

The log showed a routing problem, meaning which device the audio came from:

```
Listening through Bluetooth headset
Wake word detection started (idle)
...
Back on the phone mic
Listening through Bluetooth headset
```

Capture was bouncing between the headset and the phone, several times a minute. Each bounce tore down the audio pipeline and built a new one with a fresh model stream. A phrase spoken across a bounce was cut in half and could never match.

## The cause

I had registered a callback for audio devices appearing and disappearing, and I acted on every event immediately. A Bluetooth link drops out of the device list during its own state changes. My code read that as "headset unplugged", switched to the phone mic, saw the headset reappear, and switched back. I had built a feedback loop myself.

## The fix

One pending routing change at a time, with a settle period before believing the device list. If a new event arrives, any change already queued is cancelled.

## A related mistake

In the same work I labelled the device `Bluetooth headset (classic, narrowband mic)` in the log. Classic Bluetooth runs at 8kHz or 16kHz depending on what the headset negotiates, and I never measured which one this was. Later I read that log line back as evidence for a narrowband diagnosis, having written the word "narrowband" into it myself. The log now prints the device's actual reported sample rates.

## What I took from it

Be careful what you assert in a log. It will eventually be read back as evidence, including by you, and a label that encodes an assumption is worse than no label.

---

*Project: Umbra. Rewritten for easier reading from the original in [umbra-sub-os-showcase / CASE-STUDIES.md](https://github.com/Ciaran11221/umbra-sub-os-showcase/blob/main/CASE-STUDIES.md#4-a-microphone-that-flapped) (commit 5940844). Same facts and numbers.*
