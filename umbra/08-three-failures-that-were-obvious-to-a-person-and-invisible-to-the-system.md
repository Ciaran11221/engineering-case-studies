# 8. Three failures that were obvious to a person and invisible to the system

**In one line:** a friend gave me a real C# bug to analyse, the assistant gave a confident wrong answer, and the logs showed 3 separate failures that were each a 1-second diagnosis for a person and invisible to the system.

| | |
|---|---|
| **Where** | Architecture jar (a jar is one self-contained feature module of the assistant), dictation input, error messages |
| **Result** | 49 characters of input arrived instead of the code; a paid reply of nearly 2,000 characters was thrown away; parsing and error messages fixed, code analysis still not built |
| **Topic** | Debugging, Design decision |

## The problem

A friend gave me a real C# problem, an infinite loop he could not place. I passed it to the assistant in front of him. What came back was confident, plausible and wrong. In his words, it would have sent him round in circles. It was the first time I had put the assistant in front of someone who had not watched me build it, and it was embarrassing.

## What I assumed

That the model was the weak link: its reasoning was not good enough for real debugging and I would need a better one.

## What the evidence said

The model never got the chance to reason. Reading the logs afterwards, I found 3 separate failures stacked on top of each other. None was about the quality of the analysis.

| Failure | What happened |
|---|---|
| The code never arrived | The whole payload that reached the system was 49 characters: a spoken summary of the problem, not the problem. The input was dictation, which captured what my friend said about his bug, not its source. The model was asked to find an infinite loop in code it had never seen. It had no way to say "you haven't shown me anything", so it answered anyway |
| It went to the wrong tool | I sent it to the architecture jar, which turns a spoken description of a system into a design diagram. It is not a debugger. Even with the full source pasted in, it would have replied with a system architecture, because that is all it does. I had not built a code-analysis tool to route to |
| The error message was wrong | The model did answer, with nearly 2,000 characters, billed to my account. Our own JSON parsing (reading the reply's structured format) threw it away, because the reply had a literal newline inside a string value. That is invalid JSON by the letter of the spec and easy to recover in practice. The failure was then shown to the user as "couldn't reach Claude". The only diagnostic signal pointed away from the real fault: a working network, a successful paid call, and a message saying the network was down |

## How it looks from the 2 sides

To a person, each is a 1-second diagnosis: you did not send him the code, that is the diagram tool, the network is fine because it charged you.

To the system, all 3 were invisible, because at every layer something completed successfully. Dictation produced valid text, the jar received a well-formed request, and the API call returned 200. Nothing had the context to notice that the whole exchange was meaningless. Nothing in the chain was positioned to know what it was missing.

## The fix

1. **Parsing.** I fixed this first, since it was throwing away work I had already paid for. A reply whose only fault is unescaped control characters (such as that newline) is now re-parsed leniently and used. Structural damage, such as truncation or genuinely broken syntax, is still rejected rather than repaired. A half-guessed design would render as an authoritative-looking wrong diagram, which is worse than an honest failure.
2. **Error messages.** The failure reason now travels with the failure instead of being guessed where the call is made. The 4 ways generation can fail are unreachable, no JSON at all, unparseable, and no diagram in the reply. Each now says which one happened. A test asserts that no failure other than a network failure ever mentions reaching Claude.

## What I have not built yet

A real code-analysis tool, separate from the architecture one, and a way to get actual source onto the phone that is not dictation. Those 2 would have saved that demo. The fixes above only mean it would have failed honestly instead of confidently. I would rather be clear about that than describe this as solved.

## What I took from it

An error message that names the wrong cause is worse than one that names no cause, because it moves someone's attention away from the fault. A system that cannot tell "I have the information" from "I have 49 characters and no code" will answer both with the same confidence, so the confidence tells you nothing. What helps is building the places where a component is allowed to say it has not been given enough to work with.

---

*Project: Umbra. Rewritten for easier reading from the original in [umbra-sub-os-showcase / CASE-STUDIES.md](https://github.com/Ciaran11221/umbra-sub-os-showcase/blob/main/CASE-STUDIES.md#8-three-failures-that-were-obvious-to-a-person-and-invisible-to-the-system) (commit 5940844). Same facts and numbers.*
