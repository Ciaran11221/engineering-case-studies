# 10. The test that would have failed a working component

**In one line:** before building a planning tool, I wrote the test that would prove its small local model was not good enough, and the test itself turned out to be wrong.

| | |
|---|---|
| **Where** | Design of a planning tool, whose core component is a small local model that judges whether a question is worth asking |
| **Result** | My acceptance rule would have rejected a judge that scored 14 of 14; the replacement test caught 1 real failure on the first run |
| **Topic** | Testing, Measurement, Design decision |

## The problem

Nothing was built yet. I was designing a planning tool whose core component is a small local model that judges whether a given question is worth asking about a problem. Following my own rule for this project, I wrote the acceptance test before the component. The test states in advance what result would prove the model is not good enough.

## What I assumed

I wrote it down like this: if the judge marks more questions irrelevant than relevant on a hand-labelled set, the model is too weak. My reasoning was that a model that cannot see why a question matters will reject it, the pool of questions shrinks, and the coverage figure I planned to show myself goes up. That fails in the flattering direction and fails silently, which is the kind of failure worth writing a test for.

## What the evidence said

The model runs locally and costs nothing per call, so before building anything I made a set of 14 questions for a worked example: 6 that genuinely mattered and 8 that did not. I put them past the model one at a time.

- It agreed with my labels on all 14.
- It returned 8 "not relevant" against 6 "relevant", which trips my test. My acceptance rule would have condemned a component that had just scored 100 percent.
- The ratio is a property of the set I handed it, and I chose to make most of those questions irrelevant.
- The design also generates far more candidate questions than it needs, so that most get filtered away. A correct judge will normally return more rejections than keeps. The test fires hardest exactly when the thing works.

## How I measured it

I replaced the ratio with the number that matters: how many genuinely relevant questions are wrongly thrown away. That is the only error I cannot see, because a discarded question never appears on screen.

Then I made the set harder by adding questions next to a relevant one that ask something subtly different. I ran it 3 times:

```
FALSE DROPS (relevant judged NOT)  1 of 8
    DROPPED: Who has to approve a schema change on the order service?
FALSE KEEPS  3 of 12
    obvious    0/4      plausible  0/4      near-miss  3/4
UNSTABLE across 3 runs at temp 0   0 of 20
```

| Measurement | Value |
|---|---|
| Relevant questions wrongly dropped | 1 of 8, on every run, not as noise |
| Irrelevant questions wrongly kept | 3 of 12 |
| Obviously irrelevant questions wrongly kept | 0 of 4 |
| Plausible-but-wrong questions wrongly kept | 0 of 4 |
| Near-miss questions wrongly kept | 3 of 4 |
| Answers that changed between the 3 runs | 0 of 20 |

The dropped question, "who has to approve a schema change", is 1 of the 6 fixed slots the whole design is built around. It is nearly word for word the example my own design document uses for a high-value question. The original test would have passed this run. The corrected one failed it on the first attempt.

The judge was perfect on obviously irrelevant questions and on plausible-but-wrong ones, and wrong on 3 of 4 near-misses. So it filters out padding, but the boundary is the only place a judge would earn its keep.

## The fix I proposed, and what happened to it

I thought the model might do better if it had to state what each question eliminates before voting. Testing that took 10 minutes and ruled it out twice.

- Letting the model apply the rule made it worse. Wrong keeps went from 3 in 12 to 10 in 12. On every near-miss it wrote `"eliminates": "nothing"` and then voted the question relevant anyway.
- So I tried having my code make the call from that field instead of trusting the vote. Told it was only reporting and not deciding, the model wrote "nothing" for all 16 questions, including the 4 that genuinely mattered.

## What I changed

- The judge no longer decides the coverage figure. It annotates, and I can overrule it.
- The 6 fixed slots are never sent to it, so the failure that fired cannot recur.
- 4 claims in the design document that the measurements showed to be false were corrected in place.
- The failed fix was written up with its numbers as a do-not-retry, because a good-sounding idea that does not work gets proposed again a month later otherwise.

## What I took from it

A test is code, and it can be wrong in the same way the thing it tests can be. Mine encoded a reasonable prediction about which direction the failure would come from, and the prediction was wrong. The earlier case studies are about checking a claim before acting on it. This one is a level up: check that the instrument measuring the claim measures what you think it does. Running it cost 3 minutes, and I ran it before building on top of it.

---

*Project: Umbra. Rewritten for easier reading from the original in [umbra-sub-os-showcase / CASE-STUDIES.md](https://github.com/Ciaran11221/umbra-sub-os-showcase/blob/main/CASE-STUDIES.md#10-the-test-that-would-have-failed-a-working-component) (commit 5940844). Same facts and numbers.*
