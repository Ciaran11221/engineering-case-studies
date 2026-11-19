# 14. Earlier problems, more briefly

Six from earlier in the project. Same shape, less space.

**A silent startup crash with no error output.** The desktop app runs without a
console window via `pythonw.exe`, whose `sys.stdout` isn't merely
non-interactive — it's `None`. That broke the web server's default logging
setup in a way that produced *zero* diagnostic output: the app simply died.
Root-caused by instrumenting every startup step with explicit file logging,
then handling the `None` stream defensively. The lesson is the same one as case
study 1 from the other direction — when there's no output at all, suspect the
output mechanism before the code it was meant to be describing.

**A multi-second lag after every wake phrase.** Traced to name resolution:
resolving `localhost` on Windows tries IPv6 first and falls back to IPv4, and
that fallback got substantially worse with a VPN mesh's virtual interfaces
active. Fixed by connecting to `127.0.0.1` explicitly. A one-line change that
took far longer to find than to make, because the symptom looked like slow
speech processing and the cause was in the network stack.

**Regex triggers couldn't cope with real speech.** Commands like "meeting Friday
2:32pm calendar" — trigger word at the end, date and time in the middle — broke
phrase-anchored matching completely. Rather than accumulate patterns
indefinitely, this prompted a mid-project pivot to AI intent classification with
slot extraction, which handles arbitrary phrasing and word order without
enumerating it. The regex layer survives as a fallback for when the model is
unavailable, which turned out to be the right place for it.

**Losing the first word of every command.** "Take note, buy milk" consistently
transcribed as "note, buy milk". The recording start was racing against when I
actually began speaking. Fixed with a continuously-recording ring buffer
prepended to every capture, so the audio always begins slightly before the
trigger. You cannot react fast enough to a sound you haven't heard yet — the
only fix is to already have been recording.

**A cancel that didn't cancel everything.** A skill mid-conversation keeps its
own private state — waiting on a yes/no, a name change, a delete
confirmation. Interrupting with a fresh wake cleared the top-level
bookkeeping but left that private state untouched, so a confirmation you'd
walked away from could resurface later and the assistant would get stuck
re-asking a question you'd already abandoned, forever. Every piece of state
like this now gets its own explicit clear, wired into the same interrupt
path — a lesson that had to be relearned a second time, for a fourth piece of
state, a day after fixing it for the first three.

**A fix for quiet speech that made quiet speech untestable.** Detected speech
below a certain volume gets automatically boosted before anything downstream
sees it — reasonable, since a mumbled command still deserves a chance. It
also meant that any test simulating a quiet or distant voice got quietly
un-simulated: the same boost fired during the test, so "how does this handle
quiet speech" could never actually be answered by turning the volume down and
checking. Testing that specific condition now has to explicitly bypass the
boost it exists to prove is doing its job.

---

*Project: Umbra. Copied 19 Nov 2026 from [umbra-sub-os-showcase / CASE-STUDIES.md](https://github.com/Ciaran11221/umbra-sub-os-showcase/blob/main/CASE-STUDIES.md#14-earlier-problems-more-briefly) at commit 5940844. The showcase copy is kept as well; if they differ, the showcase is newer.*
