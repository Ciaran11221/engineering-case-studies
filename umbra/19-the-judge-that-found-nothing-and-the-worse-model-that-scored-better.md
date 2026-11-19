# 19. The judge that found nothing, and the worse model that scored better

**In one line:** before building a code scanner, I tested its model on a known case. It found 0 of 19, and my own scoring misled me twice.

| | |
|---|---|
| **Where** | A planned scanner that asks a local AI model whether each source file uses a deprecated API (a feature the language has marked as outdated) |
| **Result** | 0 of 19 detections on a known case; the scanner was not built, and a ground-truth test set with a stop condition was added first |
| **Topic** | Measurement, Testing, Design decision |

## The problem

I was about to build a scanner that walks my own source, asks a local model whether each file uses a deprecated API, and reports. I had planned it properly: constraints first, prior art researched, a phased build order, and a falsifier (a measurement that would prove the phase wrong) for each phase. The first phase was a timing run, to see what 1 full pass over 358 files costs, because that number decides whether the scanner needs a cache.

Before running it, I planted a positive control, a case where I already know the right answer.

## What I assumed

That the model would find a clear deprecated call if I handed it one.

## What the evidence said

My own code had a clean example: `asyncio.get_event_loop()`, deprecated in favour of `get_running_loop()`. I checked this against the interpreter and not my memory. On Python 3.14.6, a bare call raises `RuntimeError: There is no current event loop`. I also checked all 4 call sites and found each inside an `async def`, where a running loop exists and nothing raises. So it is a genuine deprecated idiom that is not a live bug, with nothing at stake to distract from a miss.

I gave the model a 40-line excerpt with that call at its midpoint.

```
  free-text prompt      0/13
  structured JSON       0/6
                       -----
                        0/19
```

Adding the target Python version did not help. Naming the API family did not help. 2 independent prompt designs and 19 runs gave 0 detections.

**It was not ignorance.** Asked the bare question with no code-review wrapper, "is `asyncio.get_event_loop()` deprecated?", the same model answered correctly and identically 3 times out of 3. It has the fact but will not apply it to code in front of it. In review framing it states the deprecation and then talks itself out of it:

> "The asyncio.get_event_loop() method is deprecated as of Python 3.12, but in the context of the snippet, it is not used in a deprecated manner."

That is worse than not knowing, because the excuse reads plausibly. A human skimming a report full of those would accept them.

**Its suggested replacement was itself deprecated.** I asked what to use instead. It named a replacement 3 times out of 3, and that replacement is also deprecated. I confirmed it by running it and reading the warning that came back. A scanner that found the defect and recommended that fix would have done harm. My standing rule that these tools report and never auto-fix had been a policy preference. It became a measured requirement.

## How I measured it, and how I scored it wrong twice

**First wrong score.** I counted a detection when the API name appeared anywhere in the reply, and I reported partial successes of 1/3 and 2/3, as if naming the API family helped a bit. Reading the full replies showed every one of those was the model excusing the call, not flagging it. A mention and a finding look identical to a substring match. The true score was 0, and I had been about to write up a prompt improvement that did not exist.

**Second wrong score.** This one pointed at a decision, which made it worse. I tried a smaller 8B model (8 billion parameters) on the same control:

```
  8B    4/4
  14B   0/19
```

Read as a score, that says to use the smaller, cheaper, faster model. Read the justifications and it flips. One claimed a different function was deprecated, and I checked: it is not, with no warning and no marker in its docs. One echoed the function signature back at me. One restated my own prompt's wording. It answers "outdated: true" and makes up a reason. My dependency notes already described that model as "a padding filter, not a discriminator" from an unrelated benchmark months earlier. This was the same property on a new task, and a naive score would have had me ship it.

| Run | Score | What the raw replies showed |
|---|---|---|
| Free-text prompt, 14B model | 0/13 | Talked itself out of the deprecation |
| Structured JSON, 14B model | 0/6 | Same |
| Substring score, 1st attempt | 1/3 and 2/3 | Every one was an excuse, true score 0 |
| 8B model | 4/4 | Wrong or echoed reasons, not detections |

## The cause

Four times in one sitting, an automated score gave me the opposite of the truth. The 4th was my check for this exact problem. It returned a confident 0 because the field I was parsing held prose and not a resolvable path. Every lookup failed silently, and I read "nothing resolved" as "nothing matched." I only noticed because the 0 contradicted a measurement I had taken by hand 10 minutes earlier.

Each time, reading the raw output by hand is what caught it.

## The fix

The timing phase was the wrong first step, and I would have run it. A scanner that finds nothing produces the cheapest possible verdict, about 10 tokens and not a paragraph, so the timing number would have come back flattering, produced by the failure it could not see.

So I put a new phase in front of it: a ground-truth corpus (a set of known cases) with a kill gate. If detection cannot clear both bars, above the floor on known-deprecated code and silent on known-clean code, the tool does not get built at all. The corpus checks its own ground truth. Every entry runs its trigger and confirms the warning or error really happens on this interpreter. Nothing in it is true because a blog post said so, which is how the 8B model got its false positive.

**One more, on the test set itself.** Reviewing the finished corpus, I found an entry in the wrong half. `typing.List` had been filed as clean, because it emits no runtime warning and has no removal date, and I had verified both. But Python's own docstring for it reads "Deprecated alias to list." It is deprecated by the language, and it never got a warning wired up because that would break a huge amount of working code. Filed as clean, it would have scored a correct judge as a false positive. The cause was my own specification: I required ground truth to be provable by a warning or an exception, which cannot express a documented deprecation with no runtime signal. There was nowhere correct to put it.

## What I am not claiming

All of this rests on 1 positive control, 1 API, on 1 machine. It is not a detection rate, and presenting it as one would be dishonest. Replacing it with a real rate is the whole point of the phase I added. Whether any of it holds beyond this single API is unknown. The scanner does not exist. What exists is a plan, a timing harness that has never run a full pass, and a corpus to measure against.

## What I took from it

Build the ruler before you trust the measurement, and check the ruler too.

---

*Project: Umbra. Rewritten for easier reading from the original in [umbra-sub-os-showcase / CASE-STUDIES.md](https://github.com/Ciaran11221/umbra-sub-os-showcase/blob/main/CASE-STUDIES.md#19-the-judge-that-found-nothing-and-the-worse-model-that-scored-better) (commit 5940844). Same facts and numbers.*
