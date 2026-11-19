# 14. Earlier problems, more briefly

**In one line:** six smaller problems from earlier in the project, each with its cause and fix.

| | |
|---|---|
| **Where** | Desktop app and voice assistant, across several areas |
| **Result** | 6 problems found and fixed |
| **Topic** | Debugging, Testing, Design decision |

## A silent startup crash with no error output

**Problem:** the desktop app runs without a console window, through `pythonw.exe`. In that mode `sys.stdout` (the place a program prints to) is `None`, not just non-interactive. That broke the web server's default logging setup and produced 0 lines of diagnostic output. The app simply died.

**Cause and fix:** I put explicit file logging around every startup step to find it, then made the code handle the `None` stream. When there is no output at all, suspect the output mechanism before the code it was meant to describe. This is the same lesson as case study 1, from the other direction.

## A multi-second lag after every wake phrase

**Problem:** the assistant lagged for several seconds after every wake phrase. The symptom looked like slow speech processing.

**Cause and fix:** the cause was name lookup. On Windows, resolving `localhost` tries IPv6 first and falls back to IPv4, and that fallback got much worse when a VPN mesh's virtual network interfaces were active. I connected to `127.0.0.1` directly. It was a one-line change that took far longer to find than to make.

## Regex triggers could not cope with real speech

**Problem:** a command like "meeting Friday 2:32pm calendar" has the trigger word at the end and the date and time in the middle. That broke matching on fixed phrase patterns (regex) completely.

**Cause and fix:** instead of adding patterns indefinitely, I switched mid-project to AI intent classification with slot extraction. That means the model works out what the command is and pulls out the details, whatever the wording or word order. The regex layer stays as a fallback for when the model is unavailable, which turned out to be the right place for it.

## Losing the first word of every command

**Problem:** "Take note, buy milk" kept being transcribed as "note, buy milk".

**Cause and fix:** recording start was racing against when I actually began speaking. I now record continuously into a short rolling buffer and add it to the front of every capture, so the audio always begins slightly before the trigger. You cannot react fast enough to a sound you have not heard yet. The only fix is to already be recording.

## A cancel that did not cancel everything

**Problem:** while a skill (one feature module of the assistant) is mid-conversation, it keeps its own private state, such as waiting on a yes or no, a name change, or a delete confirmation. A fresh wake phrase cleared the top-level state but left that private state alone. A confirmation I had walked away from could come back later, and the assistant got stuck re-asking a question I had already abandoned.

**Cause and fix:** every piece of state like this now gets its own explicit clear, wired into the same interrupt path. I had to learn this a second time, for a 4th piece of state, a day after fixing it for the first 3.

## A fix for quiet speech that made quiet speech untestable

**Problem:** speech detected below a certain volume is boosted automatically before anything else sees it, so a mumbled command still gets a chance. That meant any test simulating a quiet or distant voice was quietly undone, because the same boost fired during the test. Turning the volume down and checking how the system handled it could never answer the question.

**Fix:** testing that condition now has to bypass the boost explicitly.

---

*Project: Umbra. Rewritten for easier reading from the original in [umbra-sub-os-showcase / CASE-STUDIES.md](https://github.com/Ciaran11221/umbra-sub-os-showcase/blob/main/CASE-STUDIES.md#14-earlier-problems-more-briefly) (commit 5940844). Same facts and numbers.*
