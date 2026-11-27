# 2. The cheaper model that passed the made-up cases and failed the real ones

**In one line:** I tested whether a cheaper AI model could run the chat that hands work out to other agents; it scored 36 of 36 on invented cases and failed on real ones.

| | |
|---|---|
| **Where** | The orchestrating chat: the one that reads a task, splits it into steps and decides which model gets each step |
| **Result** | Real cases: 86 of 90 fact checks, on the pass line, but 2 tasks routed too cheaply, so it failed. The stronger model stayed in charge |
| **Topic** | Testing, Measurement, Design decision |

## The problem

My setup uses three sizes of model: Opus (largest, most expensive), Sonnet (middle) and Haiku (smallest). The orchestrating chat ran on Opus, and most of my cost was there. If Sonnet could make the same routing calls, that cost would drop.

## What I did before looking at any result

I wrote the routing rules down and froze them, so they could not be fitted to the test afterwards. I set the pass mark before a single case was drawn: at least 95% of fact checks right, and **0** tasks sent to a cheaper route than the reference. A route that is too cheap is the dangerous error: a small model gets work it cannot do, and nobody notices until later.

I drew 15 real tasks from my own past handoff notes. A separate Opus agent filled in the reference answers first. A fresh Sonnet agent then filled in the same 6 facts per task (does it change files, does it add new logic, and so on) without seeing them.

## What the evidence said

```
  set                 fact checks      routes
  15 real tasks       86 of 90         2 cheaper than the reference
  6 made-up tasks     36 of 36         6 of 6 match
```

86 of 90 is 95.6%, just over the line. But 2 routes were cheaper than the reference, and the mark was 0, so it failed.

All 4 wrong fact checks were the same question in 2 tasks: does writing results into a plan file count as only reporting? Sonnet said yes, which made the task look read-only and cheap. My ruling was no: it changes a file.

## The part worth noticing

The 6 made-up cases, one per route type, scored perfectly. Invented examples are clean; real tasks have the ambiguous edges where the mistakes live. Had I tested only on invented cases, I would have switched.

## What I changed

The orchestrator stays on Opus. The written routing rules still decide which model gets each handed-out step, and every hand-off is logged so the rules can be judged on real use: at 30 logged hand-offs or on 11 Dec 2026, whichever comes first.

## What I am not claiming

15 real cases is a small test. For 13 of them the routes could not be scored independently, so the route result rests on the 2 failures plus the made-up set.

## What I took from it

Set the pass mark before the test, and test on real cases; made-up ones flatter.

---

*Project: Claude-Setup, the private repo that holds my AI coding setup and the tools that measure it. Written 27 Nov 2026 from the repo's research notes and routing brief.*
