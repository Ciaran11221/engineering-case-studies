# 24. Every experiment I ran changed two things at once

**In one line:** a watch app would not install, and I spent hours testing the wrong cause because each test build also changed the app ID.

| | |
|---|---|
| **Where** | Watch app |
| **Result** | Every failed build used app ID 1000001, and the same build installed with another ID. Fixed by changing 1 number in 1 configuration file |
| **Topic** | Debugging |

## The problem

A small app I had written for my watch would not install. The build tool compiled it and uploaded it to the vendor's servers without complaint. Then the phone app, which is what pushes an app onto the watch, refused it every time with 4 words: *failed to unzip the package*.

## What I assumed

That the package was malformed. There were several candidates:

- The messaging library I depended on was pinned to a version that did not exist.
- The app icon was a quarter of the size the watch requires.
- The app declared it targeted an older version of the watch's software than the watch actually ran.

Any of those could plausibly produce a corrupt archive.

## What the evidence said

I checked the package first. I rebuilt it clean, opened it and walked every layer: the outer archive, the per-device archive inside it, and the 2 archives inside that. Every entry was intact, every checksum passed, and the device manifest named my exact watch. Then I built a stripped-down app with one text label and nothing else, and it installed on the first attempt. So the watch, my account and the tooling were fine. Whatever was wrong was in my project.

That is where I started bisecting (removing one suspect at a time), and where I went wrong for several hours.

Each test meant building a variant with one suspect removed and installing it alongside the ones already on the watch. Two apps cannot share an app ID, the number that identifies an app to the vendor's systems, so every variant needed a new one. I was not testing the app ID. It was a chore I did to make the test possible.

My first variant removed the messaging library and it installed. I read that as proof the library was the problem. It proved nothing, because I had changed the library and the app ID in the same build. I then spent 3 more rounds on conclusions stacked on that one: that the library's compiled output was unparseable, that there was a size ceiling on that half of the app, and that the 2 halves of the app were fine alone but conflicted together. All 3 were false, and each cost a rebuild, an install and a scan.

The results stopped fitting when 2 builds that should have contradicted each other both worked.

## How I measured it

I put every build I had made into one table with all its measurements side by side. The column I had never looked at explained everything.

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

Every failure has the same app ID. Nothing else ever produced that error, across 2 completely different codebases. The last row is the exact build that had been failing all evening, byte for byte, with only the ID changed, and it installed.

## The cause

`1000001` was a placeholder someone typed months earlier. The vendor's own project templates ship `20000` and `20001` as examples, and nobody had ever registered ours. It appears to collide with something on their side. The error message says nothing about that.

## The fix

I changed 1 number in 1 configuration file. The app installed, and the watch reached the rest of the system as soon as I pressed the button:

```
16:22:45.560  POST /find_device  200   find target=phone  -> "Buzzing your phone"
16:22:50.549  POST /find_device  200   stop               -> "Stopping."
```

2 other things came out of this and were worth keeping:

- The messaging library had been pinned to a version that has never been published, so that dependency had never once installed. Nobody noticed, because the build does not need it present.
- The app's name on the watch comes from a translation file compiled into the package, not from the configuration file where I had put it. So 7 successive installs all reached the watch correctly, and none could be identified on it. They showed as blank rows.

## What I took from it

A control is only a control if it differs in one thing, and mine differed in 2 because I had counted the app ID as setup, not as a variable. The fix is to write down every field that differs between the control and the test before running it, including the ones changed for logistical reasons. With that list, the second column of the table would have been obvious on the first experiment instead of the eighth.

---

*Project: Umbra. Rewritten for easier reading from the original in [umbra-sub-os-showcase / CASE-STUDIES.md](https://github.com/Ciaran11221/umbra-sub-os-showcase/blob/main/CASE-STUDIES.md#24-every-experiment-i-ran-changed-two-things-at-once) (commit 5940844). Same facts and numbers.*
