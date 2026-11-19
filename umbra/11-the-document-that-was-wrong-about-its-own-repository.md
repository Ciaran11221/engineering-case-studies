# 11. The document that was wrong about its own repository

**In one line:** a handoff note said three parts of a feature did not exist, and all three did.

| | |
|---|---|
| **Where** | A handoff document (the note I write at the end of a working session so the next session does not start cold) |
| **Result** | 3 rows saying "does not exist" were wrong; 2 of the 3 options in the open decision answered a question that no longer existed |
| **Topic** | Debugging, Measurement, Code review |

## The problem

The code had no fault. The fault was in a handoff document. One section was headed "where things actually are, verified not assumed". It listed which parts of a feature existed and which did not. Three of its rows said a component "does not exist". An architecture decision had been built on those three rows, and it was waiting for me to decide.

## What I assumed

I assumed that a section labelled "verified not assumed" had been verified. It named specific files and specific handler functions (the pieces of code that answer each network request), and it read like something checked, not remembered. The decision it described offered three ways forward. That only makes sense if those three things are really missing.

## What the evidence said

- The module that saves a session to disk existed, at 17 KB.
- All 6 network handlers the note called missing were in the main server file, at line numbers I could point at.
- The phone screen the note called an unbuilt placeholder was written and compiled. The file that supposedly still held the placeholder had a comment saying it had been replaced.
- All of it had been merged the previous evening at 20:12. The code was in the working directory of the session that wrote "does not exist", the whole time it was writing it.

## How I measured it

One git command shows whether a commit is already part of the current branch:

```
git merge-base --is-ancestor 31f64f5 HEAD
```

It confirmed the branch was cut after that merge. The claim "this module does not exist" is disproved by `ls` (the command that lists files). That takes 1 second, and nobody ran it, because the sentence was already written down in a section that said it had been checked.

| Check | Result |
|---|---|
| Session-persistence module on disk | 17 KB |
| Network handlers claimed missing, found in the main server file | 6 of 6 |
| Rows saying "does not exist" that were wrong | 3 |
| Options in the decision that no longer applied | 2 of 3 |

## The cause

The document recorded what someone believed at one moment. It started going out of date the moment it was written, and it could not look at the repository itself.

## The fix

Two of the three options were already answered. "Build this whole component first" had been done the previous evening. "Leave it as a script-only tool and put it on the phone later" was already on the phone. The real problem underneath, two overlapping stores for the same state, was not described anywhere, because the document could not see it.

The first fix was not code. I put the correction at the top of the new plan, with the measurement next to each claim: file size, line numbers, the ancestry command and its output. That way the next reader can re-run them instead of trusting me, the way I had trusted the last document. I struck the stale rows through in place and did not delete them, and I noted which half of each had been right. One row was partly correct, and quietly rewriting it would have lost that.

## What I took from it

A tool returning nothing is not evidence of absence until you know the tool could have seen it, and a document is weaker still, because it records a belief and starts going out of date at once. This was the fourth time in this project that a confident "it is absent" came from a stale or blind check. The others were a service reported down that was serving fine, a crash diagnosed that had never happened, and a branch reported missing that existed but had not been fetched. So my working notes now require naming the single check that would disprove a diagnosis, and running that one.

---

*Project: Umbra. Rewritten for easier reading from the original in [umbra-sub-os-showcase / CASE-STUDIES.md](https://github.com/Ciaran11221/umbra-sub-os-showcase/blob/main/CASE-STUDIES.md#11-the-document-that-was-wrong-about-its-own-repository) (commit 5940844). Same facts and numbers.*
