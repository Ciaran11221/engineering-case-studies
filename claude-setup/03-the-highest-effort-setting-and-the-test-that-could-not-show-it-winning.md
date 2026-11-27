# 3. The highest effort setting, and the test that could not show it winning

**In one line:** the AI model's maximum thinking setting gave the same result as one level lower on 3 of 3 tasks, at 2.2 times the cost, and the test itself turned out unable to show max winning.

| | |
|---|---|
| **Where** | The effort setting: how much the model is allowed to think before it answers. Higher means more thinking tokens, which are billed |
| **Result** | Same result on 3 of 3 pairs; $6.17 at max against $2.78 one level lower; 3.8 times the thinking |
| **Topic** | Measurement, Testing, Design decision |

## The problem

I had been running most chats at the highest effort level. Across my logs, the thinking those calls did controlled about half of what they cost. I wanted to know when max was actually the right pick.

My definition of efficient: what it costs to finish the task, compared with one level lower for the same result.

## What I did

I took 3 real past tasks: a critique of a design, a review of a code change, and a diagnosis from logs. For each, I wrote down the known correct answers, the "core points", before any run. The same brief then went to the same model, with no tools, once at max and once one level lower.

```
  task                    core points (lower / max)   cost (lower / max)
  design critique          3 of 3 / 3 of 3             $0.99 / $2.11
  code review              6 of 6 / 5 of 6             $1.16 / $2.41
  log diagnosis            5 of 5 / 5 of 5             $0.63 / $1.65
                                                      -----------------
                                                       $2.78 / $6.17
```

A repeat max run of the code review scored 6 of 6, so a 1-point gap between runs is noise, not evidence. Same result on all 3, at 2.2 times the cost and 3.8 times the thinking tokens.

## The flaw I found when writing it up

The lower level got full marks on all 3 tasks. A test where the cheaper option already scores 100% cannot show the expensive one doing better. So these 3 pairs show max was not needed on these tasks. They do not show what max does on a task the lower level gets partly wrong. The next test needs a brief the lower level can miss.

## What I changed

Max is now used only when I ask for it, or when the same task has already failed one level lower. Open design work no longer defaults to max.

## What I am not claiming

3 pairs are examples, not a rule. Dollar figures are API list prices, a stand-in for subscription use. Max is not shown useless; it is shown unneeded on 3 tasks.

## What I took from it

Before reading a test's result, check that the test could have given the other answer.

---

*Project: Claude-Setup, the private repo that holds my AI coding setup and the tools that measure it. Written 27 Nov 2026 from the repo's research notes (model and effort section).*
