# 26. The lock I put on the door, and the window I left open

**Symptom.** None. Nothing was broken and nobody had reported anything. I'd
just finished wiring the new UI to the assistant and went looking for what I'd
got wrong.

**What I'd built, and why.** The assistant's server checks a shared secret on
every connection, as a request header. A browser page cannot set a header on
that kind of connection — a rule of the browser, not of my code — so the page
can never hold the secret or talk to the server directly. I put a small local
process in front of it: the process holds the secret, the page asks the
process. The page ends up with less access than my phone has, which was the
whole point.

**What I assumed.** That binding it to `127.0.0.1` was the security model.
Nothing outside the machine can reach it. I'd checked that — three addresses
this machine answers on, all refused.

**Why that was wrong.** `127.0.0.1` keeps other *machines* out. It does
nothing about the browser already running on the machine.

A page on any website can send a request to a local address. For most request
shapes the browser asks permission first, and my server doesn't answer that
question, so those are refused. But a POST with `Content-Type: text/plain` is
what the standard calls a *simple request*: no permission step, sent straight
through. The attacking page can't read the reply — which is the part that makes
it feel safe. It doesn't need the reply. The call has already happened.

```
POST /call   Origin: https://evil.example   Content-Type: text/plain
  -> executed, "Nothing on your calendar for today."
POST /call   Host: attacker.example
  -> executed, "You haven't taken any notes this session."
POST /host   {"control":"wake"}   no credential of any kind
  -> would have run
```

That last one is why this is here. `wake` opens the microphone. A page in a
background tab could have turned on the mic.

**What I changed.** Three checks, each closing a different route. A custom
header, which cannot be set on a simple request — so anything cross-origin has
to ask permission first, and that question is refused. An origin check. And a
host check, which closes the trick where an attacker's hostname is made to
resolve to `127.0.0.1` so their page counts as local.

That stopped websites and not much else: a custom header is trivial to send
from any program on the machine. So the process now mints a fresh secret each
time it launches and hands it to the page in the URL *fragment* — the part
after the `#`, which browsers never send to a server. It reaches no request
line, no log, no referrer. The page reads it and strips it out of the address
bar, so a screenshot of the window doesn't carry it. It isn't saved anywhere
and it dies with the process.

I re-ran all three attacks afterwards and added a fourth, a guessed key. All
four refused; the real page unaffected.

**The part worth keeping.** The process existed *because* the page must not
hold the secret. I'd thought hard about that boundary and not at all about the
door I'd just built to stand behind it. Moving a secret somewhere safer creates
a new thing that holds a secret, and that new thing needs its own answer to
"who is allowed to ask".

Three further findings came out of the same hour, and I wrote them down instead
of quietly fixing them, because two of them were judgement calls rather than
bugs. One turned out to predate all of this: a delete needed the spoken
confirmation phrase when it arrived as speech and didn't when it arrived as
parameters — the same act, protected differently depending on which door it came
through. The test I wrote for the fix fails against the old code, and its
failure output shows the ungated path reaching the real calendar, which is
precisely what the guard exists to prevent.

---

*Project: Umbra. Copied 19 Nov 2026 from [umbra-sub-os-showcase / CASE-STUDIES.md](https://github.com/Ciaran11221/umbra-sub-os-showcase/blob/main/CASE-STUDIES.md#26-the-lock-i-put-on-the-door-and-the-window-i-left-open) at commit 5940844. The showcase copy is kept as well; if they differ, the showcase is newer.*
