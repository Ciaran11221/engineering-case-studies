# 8. Three failures that were obvious to a person and invisible to the system

**Symptom.** A friend gave me a real C# problem — an infinite loop he
couldn't place. I passed it to the assistant in front of him. What came
back was confident, plausible, and wrong: an analysis that, in his words,
would have sent him round in circles. This was the first time I'd put the
thing in front of someone who hadn't watched me build it, and it was
embarrassing.

**What I assumed.** The model was the weak link. The reasoning wasn't good
enough for real debugging, and I'd need a better one.

**Why that was wrong.** The model never got the chance to reason. Reading
the logs afterward, three separate failures had stacked, and none of them
were about analysis quality.

First, *the code never arrived*. The entire payload that reached the
system was forty-nine characters: a spoken summary of the problem, not the
problem. The input path was dictation, and dictation captured what my
friend said about his bug rather than the source of it. The model was
asked to find an infinite loop in code it had never seen, and — having no
way to say "you haven't shown me anything" — did what these models do with
a plausible-sounding request: it answered anyway.

Second, *it went to the wrong tool*. I'd sent it to the architecture jar,
which exists to turn a spoken description of a system into a design
diagram. It is not a debugger and was never meant to be. Even with the
full source pasted in perfectly, it would have replied with a system
architecture, because that is the only thing it does. There was no
code-analysis tool to route to; I hadn't built one.

Third, and worst, *the error message lied*. The model did answer — nearly
two thousand characters, billed to my account. Our own JSON parsing threw
it away, because the reply contained a literal newline inside a string
value, which is invalid JSON by the letter of the spec and trivially
recoverable in practice. The failure was then reported to the user as
"couldn't reach Claude." So the one diagnostic signal available pointed
directly away from the actual fault: a working network, a successful paid
call, and a message telling us the network was down.

The thing that stayed with me is how each of these looks from the two
sides. To a person, every one is a one-second diagnosis. *You didn't send
him the code.* *That's the diagram tool.* *The network's fine — it charged
you.* To the system, all three were structurally invisible, because at
every layer something completed successfully: dictation produced valid
text, the jar received a well-formed request, the API call returned 200.
Nothing anywhere had the context to notice that the whole exchange was
meaningless. A simple problem for me was a hard problem for it, and not
because the reasoning was weak — because nothing in the chain was
positioned to know what it was missing.

**What I built.** The parsing fix first, since it was throwing away work
I'd already paid for: a reply whose only fault is unescaped control
characters is now re-parsed leniently and used. Structural damage —
truncation, genuinely broken syntax — is still rejected rather than
"repaired", because a half-guessed design renders as an
authoritative-looking wrong diagram, which is a worse outcome than an
honest failure.

Then the error messages. The failure reason now travels with the failure
instead of being guessed at the call site, so the four distinct ways
generation can fail — unreachable, no JSON at all, unparseable, no diagram
in the reply — each say which one happened. There's a test asserting that
no non-network failure ever mentions reaching Claude again.

**What I haven't built yet.** A real code-analysis tool, separate from the
architecture one, and a way to get actual source onto the phone that isn't
dictation. Those two are what would genuinely have saved that demo; the
fixes above only mean it would have failed honestly instead of
confidently. I'd rather be clear about that than describe this as solved.

**The lesson.** An error message that names the wrong cause is worse than
one that names no cause, because it spends someone's attention actively
moving them away from the fault. And a system that cannot distinguish
"I have the information" from "I have forty-nine characters and no code"
will answer both with the same confidence — which means the confidence
carries no information at all. The fix for that isn't a better model. It's
building the places where a component is allowed to say it hasn't been
given enough to work with.

---

*Project: Umbra. Copied 19 Nov 2026 from [umbra-sub-os-showcase / CASE-STUDIES.md](https://github.com/Ciaran11221/umbra-sub-os-showcase/blob/main/CASE-STUDIES.md#8-three-failures-that-were-obvious-to-a-person-and-invisible-to-the-system) at commit 5940844. The showcase copy is kept as well; if they differ, the showcase is newer.*
