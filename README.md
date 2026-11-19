# Engineering case studies

Problems from my own builds that took real work. Each one follows the same shape: what it looked like, what I assumed, what the evidence actually said, and what I changed.

## Umbra

Problems from this project that took real work — twenty-seven recent ones in
detail, six earlier ones briefly. Each follows the same shape: what it looked like,
what I assumed, what the evidence actually said, and what I changed.

Four of them (11, 12, 13 and 27) were caught by reading the diff rather than the
summary of the work. A summary is written by whatever produced the change, so
it cannot check itself. All four are included because of what that review
caught — and 13 because half of what it caught was wrong in my own plan.

The pattern across all of them is the same, and it's the point of this
document.

Umbra's full project showcase: https://github.com/Ciaran11221/umbra-sub-os-showcase

| # | Case study | In one line | Headline number | Topic |
|---|---|---|---|---|
| 1 | [The log that only recorded successes](umbra/01-the-log-that-only-recorded-successes.md) | Wake phrase failed; the log could not show misses; the cause was a headset microphone. | Zero detections | Debugging, Measurement |
| 2 | [Three speculative fixes that each broke working code](umbra/02-three-speculative-fixes-that-each-broke-working-code.md) | Three plausible changes made without evidence each broke working code and were reverted. | — | Debugging |
| 3 | [Stopping a reply that won't stop](umbra/03-stopping-a-reply-that-won-t-stop.md) | Saying stop rarely worked; I replaced the keyword with detecting sustained speech. | one time in four | Design decision |
| 4 | [A microphone that flapped](umbra/04-a-microphone-that-flapped.md) | Headset microphone switching several times a minute cut spoken phrases in half. | — | Debugging |
| 5 | [The answer it already had](umbra/05-the-answer-it-already-had.md) | The assistant denied remembering a fact it held; recall questions now use a direct lookup. | — | Design decision |
| 6 | [The magic packet that couldn't leave the house](umbra/06-the-magic-packet-that-couldn-t-leave-the-house.md) | Wake-on-LAN failed away from home; a second always-on device now relays the packet. | — | Debugging, Design decision |
| 7 | [A safeguard against duplicates that quietly ate real data](umbra/07-a-safeguard-against-duplicates-that-quietly-ate-real-data.md) | A duplicate check silently dropped voice-logged readings that the assistant confirmed as saved. | 3bpm | Debugging |
| 8 | [Three failures that were obvious to a person and invisible to the system](umbra/08-three-failures-that-were-obvious-to-a-person-and-invisible-to-the-system.md) | A demo failed: the code never arrived, and the error message blamed the network. | forty-nine characters | Debugging |
| 9 | [The claim in a comment that closed a phone's connection](umbra/09-the-claim-in-a-comment-that-closed-a-phone-s-connection.md) | A comment claimed a handler filtered fields; it did not, and the connection closed. | — | Debugging, Testing |
| 10 | [The test that would have failed a working component](umbra/10-the-test-that-would-have-failed-a-working-component.md) | My acceptance test for a judge component would have failed a component scoring perfectly. | 1 of 8 | Testing, Measurement |
| 11 | [The document that was wrong about its own repository](umbra/11-the-document-that-was-wrong-about-its-own-repository.md) | A handoff document marked verified said three components did not exist; all three did. | 17 KB | Measurement |
| 12 | [A version bump that would have made real work unopenable](umbra/12-a-version-bump-that-would-have-made-real-work-unopenable.md) | A version bump would have made a saved session unopenable; review found an untested migration. | One saved session | Code review, Testing |
| 13 | [The one-word default that would have left the machine unstartable](umbra/13-the-one-word-default-that-would-have-left-the-machine-unstartable.md) | A timer default could stop the app restarting; found by reading the diff, not tests. | ~121 seconds | Code review, Testing |
| 14 | [Earlier problems, more briefly](umbra/14-earlier-problems-more-briefly.md) | Six earlier problems: silent startup crash, wake lag, lost first word, and others. | — | Debugging |
| 15 | [The theory that felt right and the number that settled it](umbra/15-the-theory-that-felt-right-and-the-number-that-settled-it.md) | A claimed fourteen-second pause was disproved by the eight-second listening cap in the configuration. | fourteen seconds | Debugging, Measurement |
| 16 | [The hotkey that also meant undo](umbra/16-the-hotkey-that-also-meant-undo.md) | Ctrl+Z toggled conversation mode; I replaced the library's hotkey matching with raw key codes. | — | Debugging, Testing |
| 17 | [I attacked my own voice authentication, and it lost](umbra/17-i-attacked-my-own-voice-authentication-and-it-lost.md) | A clone of my voice scored higher than my own real speech against my voiceprint. | 73 cloned utterances | Security, Testing |
| 18 | [Three instruments, all wrong, and the ear that caught them](umbra/18-three-instruments-all-wrong-and-the-ear-that-caught-them.md) | Three measuring instruments gave wrong results; listening caught each one. | Fourteen times worse | Measurement, Testing |
| 19 | [The judge that found nothing, and the worse model that scored better](umbra/19-the-judge-that-found-nothing-and-the-worse-model-that-scored-better.md) | A local model found a known deprecated call in none of the test runs. | 0/19 | Testing, Measurement |
| 20 | [The consistency feature that would have introduced an inconsistency](umbra/20-the-consistency-feature-that-would-have-introduced-an-inconsistency.md) | Review of a voice-clip feature found it could mix two voices in one reply. | twelve of twenty-six | Code review |
| 21 | [The detector that failed its own gate, and the branch that closed cheaply](umbra/21-the-detector-that-failed-its-own-gate-and-the-branch-that-closed-cheaply.md) | My echo detector failed its own pass condition, so I closed that theory cheaply. | minus 0.32 | Measurement |
| 22 | [Asking one question invited a hedge, so I asked eight](umbra/22-asking-one-question-invited-a-hedge-so-i-asked-eight.md) | One open question about ambiguity gave hedged answers; eight yes/no questions gave usable ones. | eight times | Design decision |
| 23 | [More training data changed the voice and did not improve it](umbra/23-more-training-data-changed-the-voice-and-did-not-improve-it.md) | More training data changed a synthetic voice but blind listening showed no improvement. | 1.12 times | Testing, Measurement |
| 24 | [Every experiment I ran changed two things at once](umbra/24-every-experiment-i-ran-changed-two-things-at-once.md) | A watch app failed to install because of a placeholder app ID, not the package. | 1000001 | Debugging |
| 25 | [Three real fixes for a problem none of them caused](umbra/25-three-real-fixes-for-a-problem-none-of-them-caused.md) | Three real rendering fixes did not help; the image had been re-encoded in transfer. | 39% of the pixels | Debugging, Measurement |
| 26 | [The lock I put on the door, and the window I left open](umbra/26-the-lock-i-put-on-the-door-and-the-window-i-left-open.md) | A local-only server could still be called by any website open in the browser. | — | Security |
| 27 | [A delete safeguard that deleted](umbra/27-a-delete-safeguard-that-deleted.md) | A recycle-bin backup path deleted files; the existing test never ran that path. | 3 files in, 1 file left | Testing, Debugging |
| 28 | [A week of missing heart rate, and the page I never turned](umbra/28-a-week-of-missing-heart-rate-and-the-page-i-never-turned.md) | Heart rate stopped arriving for a week; my app read only the first page. | 56 readings | Debugging, Measurement |

[The thread running through all of these](umbra/the-thread-running-through-all-of-these.md)
