# 13. The one-word default that would have left the machine unstartable

**Symptom.** None. Every test passed — 68 new ones for the module, and the
full suite of 56 clean. This one was caught by reading the diff.

**The setup.** I'd specified a self-healing feature: record which boots
actually worked, so a broken one can be diagnosed against the last good
one. A boot only counts as good if it survives a soak window — 120
seconds — because otherwise a crash loop stamps a fresh "known good" every
cycle and the record ends up pointing at the very state doing the
crashing. The implementation was a timer:

```python
threading.Timer(SOAK_SECONDS, promote_if_soaked).start()
```

That line is faithful to the plan. The plan was wrong.

**What I checked.** Four properties, none of them interesting alone:

1. `threading.Timer` inherits its daemon flag from the creating thread.
   Created on the main thread, it is **non-daemon** — so the interpreter
   waits for it at exit. Confirmed by running it, rather than trusting my
   memory of the documentation.
2. Every *other* thread in that file — the web server, the voice loop, the
   startup watcher — is explicitly `daemon=True`. This was the only one
   that wasn't.
3. There is no `os._exit` or `sys.exit` anywhere in the file, so nothing
   forces the process down past a lingering thread.
4. The single-instance lock is a named Windows mutex whose handle is
   stored and never released. Windows frees it **when the process exits**.

**What they meant together.** Quitting the app inside the soak window left
the process alive, holding the mutex, for the rest of the 120 seconds. The
relauncher — the detached helper that brings the app back after a restart —
waits 60 seconds for that mutex and then gives up.

So a restart in the first minute of a boot would have left the machine with
the app down, the tray icon already gone, and the one process that could
have restarted it having quietly timed out. That is the precise failure the
restart design exists to prevent: it spawns the relauncher *before* stopping
anything, so there is always a way back. The way back was still there. It
just couldn't get in.

Worst of all during rapid restarts — which is exactly the workflow the
feature's own safety rule had been written to protect.

**The fix is `daemon=True`.** Safe, because the promotion function already
refuses to run unless a boot is still in progress, so a process on its way
out has nothing to promote.

**Why no test would have found it.** The defect isn't in a function. It
lives in the interaction between a thread's default flag, an OS handle's
lifetime, and a timeout in a different file — and it only appears when a
person quits at the wrong moment. There is no unit that fails.

**The part I'd rather leave out, which is why it's here.** Four findings came
out of that review. Two of them were faults in *my plan*, not in the code
written from it.

I had specified that the always-on health line should report "N settings
files changed" and "N commits since". Both are ordinary daily activity — the
line would have been non-empty most days, on a machine with nothing wrong
with it. That is the exact failure the step I wrote opens by warning about,
three paragraphs above the mistake.

And the falsifier I attached to it — the measurement meant to prove the step
correct — asked *"is it silent when nothing has changed?"* The question that
mattered was *"is it silent when something has changed but nothing is
wrong?"* The implementation answered my question correctly, and passed.

**What I take from it.** A falsifier that only tests the case its author had
in mind is not a falsifier. Reviewing generated code catches what the
generator got wrong; reviewing it against a specification you wrote yourself
catches what *you* got wrong — and that second one is easier to miss,
precisely because the code agrees with you.

**Part two: verifying the fix, and the one thing I couldn't.** A fix
reviewed on a diff is still a claim until it runs against a real system. So
I ran it — a real desktop app, real process kills, a real socket, no mocks.

Five of six planned checks passed with numbers I actually measured, not
inferred: two independent boots reached the "healthy" state at ~121 seconds
against a 120-second target; forty real kills against the live snapshot
-writing function, fired mid-write, produced zero corrupt files; a restore
came back byte-for-byte identical, verified by hash rather than a glance,
and touched nothing else on disk; a crash-loop refusal fired for real — the
actual system dialog, on screen — while a manual restart stayed provably
unblocked at the same time; an unauthenticated write request sent over the
live network socket was refused and confirmed, again by hash, to change
nothing.

The sixth check — does quitting inside that 120-second window actually
avoid the hang the fix exists to prevent — I could not complete. There was
no tool available to me for driving the native desktop UI: no way to click
the system tray's own menu. I tried the nearest safe substitute, a
graceful process-close signal, and it reached the target process (confirmed
against the running log and the durable state file) but produced no
response at all. Even the "safe" workaround didn't exercise the code path
in question.

I reported that plainly rather than rounding a near-miss up to a pass. The
one property left open is written down as open, not folded into "verified"
because four other properties near it were.

**What I take from it, the second time.** A fix that passes review still
owes you a real run before "done" means anything. And when a planned check
turns out to need a tool you don't have, the honest move is to say exactly
which one and why — not to substitute a weaker check and let the report
imply it covered the same ground.

---

*Project: Umbra. Copied 19 Nov 2026 from [umbra-sub-os-showcase / CASE-STUDIES.md](https://github.com/Ciaran11221/umbra-sub-os-showcase/blob/main/CASE-STUDIES.md#13-the-one-word-default-that-would-have-left-the-machine-unstartable) at commit 5940844. The showcase copy is kept as well; if they differ, the showcase is newer.*
