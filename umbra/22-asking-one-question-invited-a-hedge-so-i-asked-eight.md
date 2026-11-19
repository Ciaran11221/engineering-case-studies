# 22. Asking one question invited a hedge, so I asked eight

**The problem.** The first tool in my coding pipeline is a planner: you
describe a problem in your own words and it works out what is actually known
versus what still has to be decided. To do that it has to answer one question
first — is this request underspecified, and if so, how?

I asked a model exactly that. The answers were useless in a specific,
recognisable way: agreeable, hedged, and different every time. "There may be
some ambiguity around scope, though the intent seems reasonably clear."
That sentence is true of every request ever written. It commits to nothing,
and nothing downstream can act on it.

**What I assumed.** Prompt quality. I rewrote the instruction several times —
sharper wording, an explicit output schema, a worked example. It improved the
prose and not the decision. The model still would not say a plain no, and
still would not say a plain yes twice running on the same input.

**Why that was wrong.** The problem was not how I was asking. It was that I
had asked one open question with an unbounded answer space, to a system whose
training rewards sounding helpful. An open question is an invitation to
produce something that reads like insight, and "there may be some ambiguity"
reads like insight. I had built the hedge into the shape of the question and
was trying to fix it in the wording.

**What I changed.** I stopped asking whether the problem is ambiguous, and
started asking whether it exhibits one specific kind of ambiguity — eight
times, once per kind, each answerable only yes or no with a reason.

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

A binary question has no room for a hedge. There is no diplomatic middle
between yes and no, so the pressure that produced "there may be some" has
nowhere to go.

**The part that took longer than the decomposition.** Two of the eight
classes turned out to need definitions written specifically against how a
model gets them wrong, and those are the most useful lines in the file.

*referential* kept firing on any sentence containing "it". So the definition
now says: test for the antecedent, not for the pronoun — if any specific named
thing appears anywhere in the statement, this class does not apply, even if the
statement also says "it". Using a pronoun with something to refer back to is
ordinary English, not ambiguity.

*unrecognised_term* is the one I am most pleased with, because it asks the
model to admit it does not know something — the exact thing the agreeableness
pressure exists to prevent. Its definition ends: *answering honestly that you
do not recognise something is the correct answer here, not a failure.* Saying
so explicitly changed the behaviour where sharpening the question had not.

**Why this was affordable.** Eight questions instead of one is eight times the
calls, which on a metered API would have been an argument against doing it at
all. The judge runs on a local model on my own machine, so the marginal cost
of a question is zero and the only budget is time — measured at about a third
of a second per call, so a full eight-class pass finishes in under three
seconds. The decomposition is not a clever idea that happened to be
affordable; it is an idea the cost model made available. On a per-call bill I
would have kept trying to fix the wording.

**What it deliberately does not do.** A finding is metadata attached to the
round. It never halts anything by itself. A flag is something a human can
proceed past, and when a finding should escalate into something that does stop
a round is a separate design question I have not answered — so there is no
halt path in the module at all, rather than a halt path I would have had to
guess the rules for.

**What I would still challenge.** Which classes run at which difficulty level
is a judgement call, not a measurement. Easy checks none and hard checks all
eight — both structural — but the middle tier is five classes I picked as "the
ones a careful reader would catch on a single read-through", and I have not
tested that grouping against anything. It is the weakest claim in the design
and it is labelled as such in the source.

---

*Project: Umbra. Copied 19 Nov 2026 from [umbra-sub-os-showcase / CASE-STUDIES.md](https://github.com/Ciaran11221/umbra-sub-os-showcase/blob/main/CASE-STUDIES.md#22-asking-one-question-invited-a-hedge-so-i-asked-eight) at commit 5940844. The showcase copy is kept as well; if they differ, the showcase is newer.*
