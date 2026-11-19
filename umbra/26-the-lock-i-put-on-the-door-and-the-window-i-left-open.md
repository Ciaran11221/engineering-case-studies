# 26. The lock I put on the door, and the window I left open

**In one line:** I reviewed my own security work on the new UI and found that any website open in the browser could have sent commands to the assistant, including one that turns on the microphone.

| | |
|---|---|
| **Where** | Desktop UI, the local process that sits between the page and the assistant's server |
| **Result** | 3 attacks shown (2 ran, 1 would have run), 3 checks added, and all 4 attacks refused on re-test (the 3 plus a guessed key) |
| **Topic** | Security, Code review |

## The problem

There was no symptom. Nothing was broken and nobody had reported anything. I had just finished wiring the new UI to the assistant and went looking for what I had got wrong.

## What I assumed

The assistant's server checks a shared secret on every connection, as a request header. A browser page cannot set a header on that kind of connection. That is a rule of the browser, not of my code. So the page can never hold the secret or talk to the server directly. I put a small local process in front of the server. The process holds the secret and the page asks the process. The page ends up with less access than my phone has, which was the whole point.

I assumed that binding the process to `127.0.0.1` (the machine's own address) was the security model, because nothing outside the machine can reach it. I had checked that: 3 addresses this machine answers on, all refused.

## What the evidence said

`127.0.0.1` keeps other machines out. It does nothing about the browser already running on the machine.

A page on any website can send a request to a local address. For most request shapes the browser asks permission first, and my server does not answer that question, so those are refused. But a POST with `Content-Type: text/plain` is what the web standard calls a simple request. It has no permission step and is sent straight through. The attacking page cannot read the reply, which makes it feel safe. It does not need the reply, because the call has already happened.

## How I measured it

```
POST /call   Origin: https://evil.example   Content-Type: text/plain
  -> executed, "Nothing on your calendar for today."
POST /call   Host: attacker.example
  -> executed, "You haven't taken any notes this session."
POST /host   {"control":"wake"}   no credential of any kind
  -> would have run
```

The last one is why this is here. `wake` opens the microphone. A page in a background tab could have turned on the mic.

## The fix

3 checks, each closing a different route:

- **A custom header.** It cannot be set on a simple request, so anything from another website has to ask permission first, and that question is refused.
- **An origin check.** The process looks at which site the request came from.
- **A host check.** It closes the trick where an attacker's hostname is made to resolve to `127.0.0.1` so their page counts as local.

That stopped websites and not much else, because a custom header is trivial to send from any program on the machine. So the process now creates a fresh secret each time it launches and hands it to the page in the URL fragment. That is the part after the `#`, which browsers never send to a server. It reaches no request line, no log and no referrer. The page reads it and strips it from the address bar, so a screenshot of the window does not carry it. It is not saved anywhere and it dies with the process.

I re-ran all 3 attacks afterwards and added a 4th, a guessed key. All 4 were refused, and the real page was unaffected.

3 further findings came out of the same hour. I wrote them down instead of quietly fixing them, because 2 of them were judgement calls and not bugs. One predated all of this. A delete needed the spoken confirmation phrase when it arrived as speech and did not when it arrived as parameters, so the same act was protected differently depending on which door it came through. The test I wrote for that fix fails against the old code, and its failure output shows the unprotected path reaching the real calendar, which is what the guard exists to prevent.

## What I took from it

The process existed because the page must not hold the secret. I had thought hard about that boundary and not at all about the door I had just built to stand behind it. Moving a secret somewhere safer creates a new thing that holds a secret, and that thing needs its own answer to "who is allowed to ask".

---

*Project: Umbra. Rewritten for easier reading from the original in [umbra-sub-os-showcase / CASE-STUDIES.md](https://github.com/Ciaran11221/umbra-sub-os-showcase/blob/main/CASE-STUDIES.md#26-the-lock-i-put-on-the-door-and-the-window-i-left-open) (commit 5940844). Same facts and numbers.*
