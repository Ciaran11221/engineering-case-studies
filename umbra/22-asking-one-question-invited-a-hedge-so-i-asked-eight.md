# 22. Asking one question invited a hedge, so I asked eight

**In one line:** asking a model one open question about whether a request was unclear got vague answers, so I split it into 8 yes-or-no questions.

| | |
|---|---|
| **Where** | Coding pipeline, the planner tool |
| **Result** | One open question became 8 yes-or-no questions, about a third of a second per call, under 3 seconds for a full pass |
| **Topic** | Design decision |

## The problem

The first tool in my coding pipeline is a planner. You describe a problem in your own words, and it works out what is known and what still has to be decided. To do that it first has to decide one thing: whether the request is underspecified, and if so in what way.

I asked a model exactly that. The answers were agreeable, hedged and different every time. For example: "There may be some ambiguity around scope, though the intent seems reasonably clear." That sentence is true of every request ever written. It commits to nothing, so nothing later in the pipeline can act on it.

## What I assumed

That the prompt quality was the problem. I rewrote the instruction several times, with sharper wording, an explicit output format and a worked example. The prose improved and the decision did not. The model still would not say a plain no, and would not say a plain yes twice running on the same input.

## What the evidence said

- Better wording did not fix it.
- I had asked one open question with an unbounded answer space, to a model trained to sound helpful. "There may be some ambiguity" reads like insight, so an open question invites it.
- I had built the hedge into the shape of the question and was trying to fix it in the wording.

## The fix

I stopped asking whether the problem is ambiguous. I started asking whether it has one specific kind of ambiguity, 8 times, once per kind. Each question can only be answered yes or no, with a reason.

```
overloaded_term       one word means genuinely different things by
                      domain, and nothing says which
referential           nothing anywhere names the thing being discussed
scope                 the what is named, the which-one is not
vague_quantifier      "fast", "a few users", with no number anywhere
missing_environment   the fix would differ by OS, runtime or version,
                      and none is stated
symptom_vs_cause      an observed effect with several possible causes
unmeasurable_success  "more reliable", with no metric or baseline
unrecognised_term     a term I cannot confidently resolve at all
```

A yes-or-no question has no diplomatic middle, so the pressure that produced "there may be some" has nowhere to go.

Two of the 8 classes needed definitions written against how a model gets them wrong:

- **referential** kept firing on any sentence containing "it". The definition now says to test for the thing the pronoun points back to, not for the pronoun itself. If any specific named thing appears anywhere in the statement, the class does not apply, even if the statement also says "it". Using a pronoun with something to refer back to is ordinary English.
- **unrecognised_term** asks the model to admit it does not know something. Its definition ends: *answering honestly that you do not recognise something is the correct answer here, not a failure.* Saying so changed the behaviour where sharpening the question had not.

## How I measured it

8 questions instead of 1 is 8 times the calls. The judge runs on a local model on my own machine, so the cost of one more question is zero and the only budget is time.

| Measurement | Value |
|---|---|
| Time per call | about a third of a second |
| Full 8-question pass | under 3 seconds |

On a per-call bill I would have kept trying to fix the wording. The cost model made this idea available.

## What it deliberately does not do

A finding is metadata attached to the round. It never halts anything by itself. A flag is something a human can proceed past. When a finding should escalate into something that stops a round is a separate design question I have not answered, so there is no halt path in the module at all.

## What I would still challenge

Which classes run at which difficulty level is a judgement call, not a measurement. Easy checks none and hard checks all 8, and both of those are structural. The middle tier is 5 classes I picked as the ones a careful reader would catch on a single read-through, and I have not tested that grouping against anything. It is the weakest claim in the design and is labelled as such in the source.

## What I took from it

I had to change the shape of the question, because rewording it could not remove the room to hedge.

---

*Project: Umbra. Rewritten for easier reading from the original in [umbra-sub-os-showcase / CASE-STUDIES.md](https://github.com/Ciaran11221/umbra-sub-os-showcase/blob/main/CASE-STUDIES.md#22-asking-one-question-invited-a-hedge-so-i-asked-eight) (commit 5940844). Same facts and numbers.*
