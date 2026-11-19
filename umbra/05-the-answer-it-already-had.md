# 5. The answer it already had

**In one line:** the assistant told me it had no record of something I had said, when that fact was in the context it was reading, so I made that kind of question skip the model and look the answer up directly.

| | |
|---|---|
| **Where** | Assistant's conversation memory |
| **Result** | Questions like "did you remember X" are now answered by a direct lookup of what is stored, before the model is asked |
| **Topic** | Debugging, Design decision |

## The problem

Mid-conversation I mentioned a friend's name and a fact about them in passing. I never said "remember this". A few turns later I asked whether it had caught that. It told me, flatly, that it had no record of the conversation at all.

## What I assumed

That this was a memory gap: the fact had not been saved, so "no" was the honest answer.

## What the evidence said

- The fact had been saved.
- It was sitting in the context every reply already has access to, at the moment the denial was generated.
- The model had the right context in front of it and still produced a fluent, confident, wrong answer.

## The cause

The model had the information and did not use it.

## The fix

A question shaped like "did you remember X" or "what did I tell you about X" is now answered by a direct lookup against what is actually stored, before the model is asked. A better prompt would still have left the model to guess, so this kind of question no longer goes to it.

## What I took from it

Having the right information available does not mean the model uses it. When an answer has to be right and not just plausible, phrasing the question better will not fix it. Do not ask a model the question at all.

---

*Project: Umbra. Rewritten for easier reading from the original in [umbra-sub-os-showcase / CASE-STUDIES.md](https://github.com/Ciaran11221/umbra-sub-os-showcase/blob/main/CASE-STUDIES.md#5-the-answer-it-already-had) (commit 5940844). Same facts and numbers.*
