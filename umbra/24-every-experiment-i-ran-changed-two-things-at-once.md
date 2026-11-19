# 24. Every experiment I ran changed two things at once

**Symptom.** A small app I'd written for my watch wouldn't install. The build
tool compiled it without complaint and uploaded it to the vendor's servers
without complaint. Then the phone app, which is what actually pushes an app
onto the watch, refused it every time with four words: *failed to unzip the
package*.

**What I assumed.** That the package was malformed — that something in what I
was producing was structurally wrong. There was no shortage of candidates. The
messaging library I depended on was pinned to a version that didn't exist. The
app icon was a quarter of the size the watch requires. The app declared it
targeted an older version of the watch's software than the watch actually ran.
Any of those could plausibly produce a corrupt archive.

**Why that was wrong.** I checked the package first. I rebuilt it clean, opened
it, and walked every layer: the outer archive, the per-device archive inside it,
and the two archives inside that. Every entry was intact, every checksum passed,
and the device manifest named my exact watch. Then I built a stripped-down app —
one text label, nothing else — and that one installed on the first attempt. So
the watch was fine, my account was fine, and the tooling was fine. Whatever was
wrong was in my project.

That's where I started bisecting, and that's where I went wrong for several
hours.

Each test meant building a variant with one suspect thing removed, installing it
alongside the ones already on the watch, and seeing whether it worked. But two
apps can't share an app ID — the number that identifies an app to the vendor's
systems — so **every variant I built needed a new one**. The app ID wasn't a
thing I was testing. It was a chore I did to make the test possible.

My first variant removed the messaging library, and it installed. I read that as
proof the library was the problem. It wasn't proof of anything: I had changed the
library *and* the app ID in the same build. I then spent three further rounds on
conclusions stacked on that one — that the library's compiled output was
unparseable, that there was a size ceiling on how large that half of the app
could be, and that the two halves of the app were fine alone but conflicted
together. All three were false, and each one cost a rebuild, an install and a
scan.

The results stopped fitting when two builds that should have contradicted each
other both worked. I put every build I'd made into a table with all its
measurements side by side, and the column I had never once looked at was the
one that explained everything:

```
appId     app-side bytes   watch-side bytes   result
1000002             none                851   installed
1000005              152                962   installed
1000008           17,696                962   installed
1000009           14,677                962   installed
1000010              150             27,378   installed
1000001           14,677             27,383   FAILED
1000001           21,935                967   FAILED
1000003              926                990   installed
```

Every failure is the same app ID. Nothing else ever produced that error, across
two completely different codebases. And the last row is the one that settles it:
that's the exact build that had been failing all evening, byte for byte, with
only the ID changed. It installed.

`1000001` was a placeholder someone had typed months earlier — the vendor's own
project templates ship `20000` and `20001` as examples, and nobody had ever
registered ours. It appears to collide with something on their side. The error
message says nothing about that, and never would.

**What I changed.** One number in one configuration file. The app installed, and
the watch reached the rest of the system in the time it took to press the button:

```
16:22:45.560  POST /find_device  200   find target=phone  -> "Buzzing your phone"
16:22:50.549  POST /find_device  200   stop               -> "Stopping."
```

Two other things came out of the wreckage and were worth keeping. The messaging
library had been pinned to a version that has never been published, so that
dependency had never once installed successfully — nobody noticed, because the
build doesn't need it present. And the app's name on the watch comes from a
translation file compiled into the package, not from the configuration file where
I'd put it, so seven successive installs had all reached the watch correctly and
none of them could be identified on it. They showed as blank rows.

**The lesson, and it isn't the app ID.** A control is only a control if it
differs in one thing. Mine differed in two, and one of them was invisible to me
because I'd classified it as setup rather than as a variable. I had the
disproving evidence in hand from the first experiment onward — every build's app
ID was right there in its configuration — and I didn't look, because I wasn't
asking that question.

The fix isn't "be more careful". It's to write down every field that differs
between the control and the test *before* running it, including the ones you
changed for logistical reasons. If I'd written that list, the second column of
this table would have been obvious on the first experiment instead of the eighth.

---

*Project: Umbra. Copied 19 Nov 2026 from [umbra-sub-os-showcase / CASE-STUDIES.md](https://github.com/Ciaran11221/umbra-sub-os-showcase/blob/main/CASE-STUDIES.md#24-every-experiment-i-ran-changed-two-things-at-once) at commit 5940844. The showcase copy is kept as well; if they differ, the showcase is newer.*
