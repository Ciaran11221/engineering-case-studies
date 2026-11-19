# 19. The judge that found nothing, and the worse model that scored better

**What I was about to build.** A scanner that walks my own source, asks a local
model whether each file uses a deprecated API, and reports. I'd planned it
properly: constraints first, prior art researched, a phased build order, and a
falsifier written for each phase. The first phase was a timing run — how long
does one full pass over 358 files cost? — because that number decides whether
the thing needs a cache.

Before running it, I planted a positive control.

**The control.** My own code had a clean example: `asyncio.get_event_loop()`,
deprecated in favour of `get_running_loop()`. I checked that against the
interpreter rather than trusting my memory of it — on Python 3.14.6 a bare call
raises `RuntimeError: There is no current event loop`. I also checked all four
call sites and found every one inside an `async def`, where a running loop
exists and nothing raises. So: a genuine deprecated idiom that is not a live
bug. Unambiguous, and with nothing at stake to distract from a miss.

Then I handed the model a 40-line excerpt with that call at its midpoint.

**The result.**

```
  free-text prompt      0/13
  structured JSON       0/6
                       -----
                        0/19
```

Adding the target Python version didn't help. Naming the API family didn't
help. Two independent prompt designs, nineteen runs, zero detections.

**It wasn't ignorance, which is the interesting part.** Asked the bare
question with no code-review wrapper — "is `asyncio.get_event_loop()`
deprecated?" — the same model answers correctly and identically three times out
of three. It has the fact. It won't apply the fact to code put in front of it.
In review framing it states the deprecation and then talks itself out of it:

> "The asyncio.get_event_loop() method is deprecated as of Python 3.12, but in
> the context of the snippet, it is not used in a deprecated manner."

That's worse than not knowing, because the rationalisation reads plausibly. A
human skimming a report full of those would accept them.

**The finding that mattered more than the miss.** I asked what to use instead.
It named a replacement, three times out of three — and that replacement is
*itself* deprecated, which I confirmed by executing it and reading the warning
come back. A scanner that correctly found the defect and recommended that fix
would have been actively harmful. My standing rule that these tools report and
never auto-fix had been a policy preference up to that point. It became a
measured requirement.

**Then I scored it wrong. Twice.**

The first time, I counted a detection by checking whether the API name appeared
anywhere in the reply, and reported partial successes of 1/3 and 2/3 — "naming
the API family helps a bit." Reading the full replies showed every one of those
was the model *excusing* the call rather than flagging it. A mention and a
finding are identical to a substring match. The true score was zero, and I had
been about to write up a prompt improvement that didn't exist.

The second time pointed at a decision, which made it worse. I tried a smaller
8B model on the same control:

```
  8B    4/4
  14B   0/19
```

Read as a score, that says use the smaller, cheaper, faster model. Read the
justifications and it inverts. One claimed a different function was deprecated
— I checked, it isn't; no warning, no marker in its docs. One echoed the
function signature back at me. One restated my own prompt's wording. It answers
"outdated: true" and manufactures a reason. My own dependency notes already
described that model as "a padding filter, not a discriminator" from an
unrelated benchmark months earlier. This was the same property showing up on a
new task, and a naive score would have had me ship it.

**What it changed.** The timing phase was the wrong first step and I'd have run
it. A scanner that finds nothing produces the *cheapest* possible verdict — ten
tokens instead of a paragraph — so the performance number would have come back
flattering, produced by the exact failure it couldn't see.

So a new phase went in front of it: a ground-truth corpus, with a kill gate. If
detection can't clear both bars — above the floor on known-deprecated code, and
*silent on known-clean code* — the tool doesn't get built at all.

The corpus verifies its own ground truth rather than asserting it. Every entry
executes its trigger and confirms the warning or the raise actually happens on
this interpreter. Nothing in it is true because a blog post said so, which
matters because that is precisely how the 8B got its false positive.

**One more, on the ruler itself.** Reviewing the finished corpus, I found an
entry in the wrong half. `typing.List` had been filed as clean, on the
reasoning that it emits no runtime warning and has no removal date — both true,
and both verified. But Python's own docstring for it reads "Deprecated alias to
list." It's deprecated by the language; it just never got a warning wired up,
because that would break an enormous amount of working code.

Filed as clean, it would have scored a *correct* judge as a false positive. And
the cause was my own specification: I'd required ground truth to be provable by
a warning or an exception, which cannot express "documented deprecation with no
runtime signal." There was nowhere correct to put it.

**What I'm not claiming.** All of this rests on one positive control, one API,
on one machine. It is not a detection rate and it would be dishonest to present
it as one — replacing it with something that is, is the entire point of the
phase I added. Whether any of it generalises past this single API is unknown.
The scanner doesn't exist. What exists is a plan, a timing harness that has
never run a full pass, and a corpus to measure against.

Four times in one sitting, an automated score gave me the opposite of the
truth, and every single time the thing that caught it was reading the raw
output by hand. The fourth one was my check *for this exact problem*: it
returned a confident zero, because the field I was parsing held prose rather
than a resolvable path, so every lookup failed silently and I read "nothing
resolved" as "nothing matched." I only noticed because zero contradicted a
measurement I'd already taken by hand ten minutes earlier.

That's the whole lesson, and it isn't about models. Build the ruler before you
trust the measurement, and check the ruler too.

---

*Project: Umbra. Copied 19 Nov 2026 from [umbra-sub-os-showcase / CASE-STUDIES.md](https://github.com/Ciaran11221/umbra-sub-os-showcase/blob/main/CASE-STUDIES.md#19-the-judge-that-found-nothing-and-the-worse-model-that-scored-better) at commit 5940844. The showcase copy is kept as well; if they differ, the showcase is newer.*
