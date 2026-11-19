# 16. The hotkey that also meant undo

**Symptom.** Ctrl+Z started toggling conversation mode on and off, on a build
I'd already confirmed was wired to a different combination — Ctrl+Shift+C.

**What I assumed.** That the two hotkeys were simply colliding somewhere in my
own code — maybe a leftover registration, maybe a stale handler. That would
be a five-minute grep.

**Why that was wrong.** There was no second registration. Grepping the whole
codebase for Ctrl+Z, undo, or anything z-shaped in a hotkey context found
nothing. The only hotkey registered anywhere was the correct one. Whatever
was matching Ctrl+Z wasn't in my code at all — which meant it was in the
library underneath it.

The hotkey library (`pynput`) doesn't document how it decides a combination
is pressed, so I read its actual installed source rather than guess from its
API surface. It matches a hotkey by translating each keystroke into a
*character*, based on whatever modifiers are held at that instant, and
comparing that character against the combination's own parsed form. Held
modifiers change what a keystroke translates to — that's normal keyboard
behavior, the same mechanism that makes Shift+2 produce `@`. Under two
modifiers held together, that translation turned out to produce the same
character for a different physical key than the one actually pressed. I
couldn't pin down the exact trigger condition for that collision with
confidence, and said so rather than overclaim it.

What mattered more than the exact trigger was finding something in the same
source that didn't depend on the unreliable part: the raw, untranslated
virtual-key code. It's reported on every key event regardless of what the
character translation decides, precisely because the OS layer that produces
it runs before any modifier-aware translation happens.

**What I built.** Replaced the library's built-in hotkey matcher with a small
state machine of my own: track Ctrl, Shift, and the letter key as three
independent booleans, updated from raw key-down/key-up events, matching the
letter by its virtual-key code instead of its translated character. The
toggle fires only on the letter's own transition into "pressed while both
modifiers are already down" — not on the modifiers alone.

**The part worth naming.** Writing the test for that fix caught a second,
unrelated bug before it ever shipped. My first version matched on whether the
right keys were currently *held* — a level, not an event. Held keys on
Windows resend their own key-down at the OS's repeat rate, so a single
physical hold would have re-fired the toggle every repeat interval for as
long as the key stayed down — turning conversation mode on, off, on, off,
dozens of times, from one keypress. The test that exercised "hold the
combination for a while" is what surfaced it; nothing about testing the
original collision would have.

**The lesson.** For behavior this central to a fix, the library's
documentation wasn't enough — reading its actual installed source was what
turned up the one field immune to the failure I was fixing. And a fix isn't
finished when it stops reproducing the bug that was reported; the test
written to prove it also needs to try the thing nobody reported yet, like
just holding the key down.

---

*Project: Umbra. Copied 19 Nov 2026 from [umbra-sub-os-showcase / CASE-STUDIES.md](https://github.com/Ciaran11221/umbra-sub-os-showcase/blob/main/CASE-STUDIES.md#16-the-hotkey-that-also-meant-undo) at commit 5940844. The showcase copy is kept as well; if they differ, the showcase is newer.*
