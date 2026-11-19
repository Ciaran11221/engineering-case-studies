# 16. The hotkey that also meant undo

**In one line:** Ctrl+Z started toggling conversation mode, and the cause was in the keyboard library, not my code.

| | |
|---|---|
| **Where** | Desktop app, global hotkey for conversation mode |
| **Result** | Replaced the library's hotkey matching with my own; the test for it caught a 2nd bug (repeat firing) before it shipped |
| **Topic** | Debugging, Testing |

## The problem

Ctrl+Z started toggling conversation mode on and off. The build was already confirmed to be wired to a different combination, Ctrl+Shift+C.

## What I assumed

That the two hotkeys were colliding somewhere in my own code, perhaps a leftover registration or a stale handler. I expected that to be a 5-minute search.

## What the evidence said

- There was no second registration. Searching the whole codebase for Ctrl+Z, undo, or anything similar in a hotkey context found nothing. The only hotkey registered anywhere was the correct one.
- So whatever matched Ctrl+Z was in the library underneath my code.
- The hotkey library (`pynput`) does not document how it decides a combination is pressed, so I read its installed source. It turns each keystroke into a character, based on the modifiers held at that instant, and compares that character with the combination's own parsed form. Held modifiers change what a keystroke turns into. That is normal keyboard behaviour, the same thing that makes Shift+2 produce `@`.
- With 2 modifiers held together, that translation produced the same character for a different physical key than the one pressed.
- I could not pin down the exact trigger for that collision with confidence, and I said so instead of overclaiming it.
- The same source holds a field that does not depend on the unreliable part: the raw virtual-key code, which identifies the physical key. It is reported on every key event, because the operating system produces it before any modifier-aware translation happens.

## The cause

The library's character translation. Under 2 held modifiers, the translated character matched the hotkey for a different key.

## The fix

I replaced the library's built-in hotkey matcher with a small state machine of my own. It tracks Ctrl, Shift and the letter key as 3 independent true-or-false values, updated from raw key-down and key-up events. It matches the letter by its virtual-key code, not its translated character. The toggle fires only when the letter goes down while both modifiers are already down. The modifiers alone never fire it.

## How I measured it

Writing the test for that fix caught a 2nd, unrelated bug before it shipped. My first version matched on whether the right keys were currently held, a state and not an event. On Windows, a held key resends its key-down at the system's repeat rate. So 1 physical hold would have re-fired the toggle at every repeat interval for as long as the key stayed down, turning conversation mode on, off, on, off, dozens of times from 1 keypress. The test that held the combination for a while surfaced it. Testing the original collision would not have.

## What I took from it

Documentation was not enough for something this central, and reading the library's installed source found the one field immune to the failure. A fix is also not finished when the reported bug stops, because the test that proves it should try the thing nobody reported yet, such as holding the key down.

---

*Project: Umbra. Rewritten for easier reading from the original in [umbra-sub-os-showcase / CASE-STUDIES.md](https://github.com/Ciaran11221/umbra-sub-os-showcase/blob/main/CASE-STUDIES.md#16-the-hotkey-that-also-meant-undo) (commit 5940844). Same facts and numbers.*
