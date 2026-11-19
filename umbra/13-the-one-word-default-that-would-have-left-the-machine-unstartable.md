# 13. The one-word default that would have left the machine unstartable

**In one line:** a timer in a self-healing feature would have kept the app running after the user quit it, and blocked the helper that restarts the app.

| | |
|---|---|
| **Where** | The desktop app's startup and restart code |
| **Result** | 1 word fixed (`daemon=True`). 5 of 6 planned real-system checks passed; the sixth could not be run and is reported as open |
| **Topic** | Code review, Testing, Design decision |

## The problem

I had specified a self-healing feature that records which startups actually worked, so a broken one can be compared with the last good one. A startup only counts as good if it survives a 120-second soak window. Without that, a crash loop stamps a fresh "known good" on every cycle, and the record ends up pointing at the very state that is crashing. The implementation was a timer:

```python
threading.Timer(SOAK_SECONDS, promote_if_soaked).start()
```

That line is faithful to the plan. The plan was wrong. Nothing failed: 68 new tests for the module passed, and the full suite of 56 was clean. I caught it by reading the diff.

## What I assumed

That the timer was harmless background work.

## What the evidence said

I checked 4 properties, none interesting alone:

1. A timer created on the main thread is a non-daemon thread, which means the program waits for it at exit. I confirmed this by running it, not by trusting my memory of the documentation.
2. Every other thread in that file (the web server, the voice loop, the startup watcher) is explicitly `daemon=True`. This was the only one that was not.
3. Nothing in the file forces the process to exit past a lingering thread. There is no `os._exit` or `sys.exit`.
4. The single-instance lock is a named Windows mutex (a lock the operating system hands out by name). Its handle is stored and never released, and Windows frees it only when the process exits.

## The cause

Together, those 4 meant that quitting the app inside the soak window left the process alive, holding the lock, for the rest of the 120 seconds. The relauncher is the detached helper that brings the app back after a restart. It waits 60 seconds for that lock and then gives up.

So a restart in the first minute after a startup would have left the machine with the app down, the tray icon gone, and the one process that could restart it having timed out. The restart design exists to prevent exactly that: it starts the relauncher before stopping anything, so there is always a way back. The way back was there, but it could not get in. It was worst during rapid restarts, which is the workflow the feature's own safety rule was written to protect.

No test would have found it. The defect is in the interaction between a thread's default flag, the lifetime of an operating-system handle, and a timeout in a different file. It only appears when a person quits at the wrong moment, so there is no single unit that fails.

## The fix

Set `daemon=True`. That is safe because the promotion function already refuses to run unless a startup is still in progress, so a process on its way out has nothing to promote.

## What the review found about my own plan

Four findings came out of that review. 2 of them were faults in my plan, not in the code written from it.

I had specified that the always-on health line should report "N settings files changed" and "N commits since". Both are ordinary daily activity, so the line would have been non-empty most days on a machine with nothing wrong. That is the failure the step I wrote opens by warning about, 3 paragraphs above the mistake.

The falsifier I attached to it asked "is it silent when nothing has changed?" The question that mattered was "is it silent when something has changed but nothing is wrong?" The implementation answered my question correctly, and passed.

## How I measured the fix

A fix reviewed on a diff is still a claim until it runs on a real system. So I ran a real desktop app, with real process kills and a real socket, and no mocks. 5 of 6 planned checks passed with numbers I measured:

| Check | Value |
|---|---|
| Independent startups reaching "healthy" | 2, at about 121 seconds against a 120-second target |
| Real kills fired mid-write against the live snapshot-writing function | 40, with 0 corrupt files |
| Restore | Identical byte for byte, verified by hash, and touched nothing else on disk |
| Crash-loop refusal | Fired for real, with the actual system dialog on screen; a manual restart stayed unblocked at the same time |
| Unauthenticated write request over the live network socket | Refused, and confirmed by hash to change nothing |

The sixth check was whether quitting inside the 120-second window actually avoids the hang the fix exists to prevent. I could not complete it. I had no tool for driving the native desktop UI, so I could not click the system tray's own menu. I tried the nearest safe substitute, a graceful process-close signal. It reached the target process, confirmed against the running log and the durable state file, but produced no response at all. So even the workaround did not exercise the code path in question.

I reported that plainly and did not round a near-miss up to a pass. The one property left open is written down as open, and is not folded into "verified" because 4 other properties near it were.

## What I took from it

A falsifier that only tests the case its author had in mind is not a falsifier. Reviewing generated code catches what the generator got wrong, and reviewing it against a specification you wrote yourself catches what you got wrong, which is easier to miss because the code agrees with you. A fix that passes review still owes a real run, and when a planned check needs a tool you do not have, say which one and why, instead of substituting a weaker check.

---

*Project: Umbra. Rewritten for easier reading from the original in [umbra-sub-os-showcase / CASE-STUDIES.md](https://github.com/Ciaran11221/umbra-sub-os-showcase/blob/main/CASE-STUDIES.md#13-the-one-word-default-that-would-have-left-the-machine-unstartable) (commit 5940844). Same facts and numbers.*
