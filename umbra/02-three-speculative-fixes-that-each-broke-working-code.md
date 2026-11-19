# 2. Three speculative fixes that each broke working code

**In one line:** I made 3 changes to the phone's voice code based on reasoning with no evidence, and all 3 broke things that had been working.

| | |
|---|---|
| **Where** | Phone app, voice listening |
| **Result** | 3 changes reverted; each is now a warning comment in the code where someone would try it again |
| **Topic** | Debugging, Design decision |

## The problem

Over a few sessions I made 3 changes on plausible reasoning, without checking any evidence first. Each one broke something that had worked.

## The three changes

| Change | What I assumed | What happened |
|---|---|---|
| Call `recognizer.cancel()` to free the microphone | The speech recogniser (the part that turns speech into text) was holding the mic and blocking wake-phrase detection during playback | The log later showed the detector restarts fine in the middle of a reply. Cancelling left the recogniser unable to hear anything afterwards: every wake phrase got a "Yes?" and then silence |
| Shorten the endpointing timeouts (the silence lengths that decide when I have finished speaking) | Ending the turn sooner would feel snappier | There is a natural pause between the assistant finishing its acknowledgement and me starting to speak. The recogniser counted that pause as the end of my turn, so every command came back empty |
| Switch the audio source to `VOICE_COMMUNICATION` (an Android mode meant for phone calls) | Its echo cancellation would help detect "stop" over the speaker | That mode applies heavy noise suppression and automatic volume control, which degrades the signal a keyword-spotting model needs. Stop detection got worse, never better |

## The fix

I reverted all 3. Each one is now a comment in the source, at the place where someone would try it again:

    NOTE: do not set the silence-length extras here. Shortening them to end
    the turn faster looked reasonable and broke recognition outright... The
    platform defaults account for that gap; leave them alone.

## What I took from it

A negative result is worth as much as a positive one, and it gets forgotten twice as fast, so write it down where the next person (usually me) will trip over it.

---

*Project: Umbra. Rewritten for easier reading from the original in [umbra-sub-os-showcase / CASE-STUDIES.md](https://github.com/Ciaran11221/umbra-sub-os-showcase/blob/main/CASE-STUDIES.md#2-three-speculative-fixes-that-each-broke-working-code) (commit 5940844). Same facts and numbers.*
