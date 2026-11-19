# 9. The claim in a comment that closed a phone's connection

**Symptom.** I'd just redesigned how the architecture jar (see [case study
8](08-three-failures-that-were-obvious-to-a-person-and-invisible-to-the-system.md))
stores its in-progress work, restarted the desktop app to pick up the
change, then opened the corresponding screen on my phone. It came back with
"couldn't reach core, EOF" and the connection closed immediately.

**What I assumed, and wrote down as fact.** While writing the redesign, I
added an internal field to the data one function returns, and documented it
with a claim about the code that consumes it: that a specific handler "only
forwards a few named fields," so the extra field would never actually leave
the process. I never opened that handler to check. It read as obviously
true from the calling code alone.

**Why that was wrong.** The handler forwards the whole structure wholesale,
with no filtering at all. The extra field I'd added held a filesystem path
object, not a plain string — and the moment a real request tried to
serialize that structure to JSON to send over the wire, it failed outright.
The error handling around that failure closed the socket immediately,
which is exactly the "EOF" the phone saw.

**What the log actually said**, once I checked it instead of the comment
I'd written:

```
TypeError: Object of type WindowsPath is not JSON serializable
when serializing dict item '_folder'
when serializing dict item 'diagrams'
```

Two lines that state the entire bug outright, sitting untouched in the log
the whole time it mattered.

**What I built.** Moved the actual filesystem lookup into a function that
nothing outside the module ever sees the return value of — the data handed
back to any caller now only ever carries plain, wire-safe fields. Added a
test that does the one thing that would have caught this before it
shipped: takes the real function's real output and runs it through the
same serializer the network layer uses, asserting it doesn't raise.

**The lesson.** A comment that states how another piece of code behaves is
a claim about that code, not a fact available from the file you're
actually editing. It needed exactly one grep before being written down,
and it shipped anyway because that grep felt unnecessary — the calling
code's own shape seemed to make the claim obviously true. It didn't cost a
debugging session to catch. It cost a live connection, closing, the moment
it mattered.

---

*Project: Umbra. Copied 19 Nov 2026 from [umbra-sub-os-showcase / CASE-STUDIES.md](https://github.com/Ciaran11221/umbra-sub-os-showcase/blob/main/CASE-STUDIES.md#9-the-claim-in-a-comment-that-closed-a-phones-connection) at commit 5940844. The showcase copy is kept as well; if they differ, the showcase is newer.*
