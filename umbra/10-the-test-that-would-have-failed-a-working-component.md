# 10. The test that would have failed a working component

**Symptom.** None yet — that's the point. I was designing a planning tool whose
core component is a small local model that judges whether a given question is
worth asking about a problem. Nothing was built. Following my own rule for this
project, I wrote the acceptance test for that component before writing the
component: state in advance what result would prove it isn't good enough.

**What I assumed.** I wrote it down like this: *if the judge marks more
questions irrelevant than relevant on a hand-labelled set, the model is too
weak.* The reasoning felt solid. A model that can't see why a question matters
will reject it, the pool of questions shrinks, and the coverage figure I planned
to show myself goes up. It fails in the flattering direction, and it fails
silently — exactly the thing worth writing a test for.

**Why that was wrong.** The model runs locally and costs nothing per call, so
before building anything I built a set of fourteen questions instead — six that
genuinely mattered for a worked example, eight that didn't — and put them past
it one at a time.

It agreed with me on all fourteen.

It also returned eight "not relevant" against six "relevant", which trips my
test. My own acceptance criterion would have condemned a component that had
just scored a hundred percent.

The ratio isn't a property of the model. It's a property of the set I handed it,
and I chose to make most of those questions irrelevant. Worse, the design
deliberately generates far more candidate questions than it needs so that most
get filtered away — which means a *correct* judge will normally return more
rejections than keeps. The test fires hardest exactly when the thing works.

**What the corrected test caught.** I replaced the ratio with the number that
actually matters: how many genuinely relevant questions get wrongly thrown away.
That's the only error I can't see, because a discarded question never appears on
screen at all.

Then I made the set harder — adding questions *adjacent* to a relevant one but
asking something subtly different — and ran it three times:

```
FALSE DROPS (relevant judged NOT)  1 of 8
    DROPPED: Who has to approve a schema change on the order service?
FALSE KEEPS  3 of 12
    obvious    0/4      plausible  0/4      near-miss  3/4
UNSTABLE across 3 runs at temp 0   0 of 20
```

It threw away "who has to approve a schema change" on every run, not as noise.
That's one of six fixed slots the whole design is built around, and it is
near-word-for-word the example my own design document uses to illustrate a
*high-value* question. The original test would have passed this run. The
corrected one failed it on the first attempt.

The rest of that table matters too: perfect on obviously irrelevant questions,
perfect on plausible-but-wrong ones, three of four wrong on the near-misses.
It's a padding filter, not a judge — and the boundary is the only place a judge
would earn its keep.

**The fix I proposed, and what happened to it.** I thought the model might do
better if it had to state what each question eliminates before voting. Testing
that took ten minutes and killed it twice. Letting the model apply the rule made
it worse — wrong keeps went from three in twelve to ten in twelve — and on every
near-miss it wrote `"eliminates": "nothing"` and then voted the question
relevant anyway. So I tried having my code make the call from that field instead
of trusting the vote. Told it was only reporting and not deciding, the model
wrote "nothing" for all sixteen questions, including the four that genuinely
mattered.

**What I changed.** The judge no longer decides the coverage figure — it
annotates, and I can overrule it. The six fixed slots are never sent to it at
all, so the failure that fired cannot recur. Four claims in the design document
that measurement had made false were corrected in place rather than left
standing, and the failed fix was written up with its numbers as a
do-not-retry — a good idea that doesn't work gets re-proposed in a month
otherwise.

**The lesson.** A test is code, and it can be wrong in the same way the thing it
tests can be. Mine wasn't sloppy; it encoded a specific and reasonable
prediction about which direction the failure would come from, and the prediction
was wrong. Every earlier case study here is about checking a claim before acting
on it. This one is a level up: check that the instrument measuring the claim
measures what you think it does. It was catchable for the same reason as all the
others — running it cost three minutes, and I ran it before building on top of
it.

---

*Project: Umbra. Copied 19 Nov 2026 from [umbra-sub-os-showcase / CASE-STUDIES.md](https://github.com/Ciaran11221/umbra-sub-os-showcase/blob/main/CASE-STUDIES.md#10-the-test-that-would-have-failed-a-working-component) at commit 5940844. The showcase copy is kept as well; if they differ, the showcase is newer.*
