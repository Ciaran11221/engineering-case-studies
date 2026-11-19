# 28. A week of missing heart rate, and the page I never turned

**In one line:** heart-rate data stopped reaching my assistant for a week because my phone app read only the first page of results from the health store.

| | |
|---|---|
| **Where** | Phone app, watch data sync |
| **Result** | 56 readings sent every sync before the fix, 70 after, and the newest pulse reading went from 7 days old to today |
| **Topic** | Debugging |

## The problem

My assistant tracks vitals from my watch. The watch app writes to Android's shared health store (Health Connect). My phone app reads from it every 2 hours and sends daily figures to the assistant. For a week, heart rate, heart-rate variability and resting heart rate had stopped arriving. Blood oxygen still arrived every day.

## What I assumed

The morning's write-up blamed the watch app. My phone app's read permissions were all granted, so the break had to be upstream. That was reasoning from what I could see in my own code, and I had not looked at the store itself.

## What the evidence said

When I did look, Health Connect's own heart-rate screen held readings from the watch app up to an hour old. The data was there and my app was not getting it.

## The cause

A request to Health Connect returns one page of results: 1,000 records, oldest first, plus a token you pass back to get the next page. My app took the first page and never asked for the next. For a while a month of data fitted on one page, so it worked. Once it did not, the newest days were never read. Every sync sent the same 56 readings, and the assistant logged each one as a duplicate, "0 new". That looks exactly like a quiet week, not a broken connection.

## How I measured it

The first sync after the fix:

```
heart rate                  1,485 records over 2 pages
heart-rate variability      4,808 records over 5 pages
readings sent               70   (was 56)
new readings stored         13
newest pulse                today (was 7 days old)
```

## The fix

I made the app follow the token to every page. The same trace turned up 3 more things, each fixed and checked on a real sync the same morning:

- **Blood oxygen froze at its first value each day.** The phone sends one daily average, stamped at midnight, and re-sends it as the day goes on. The assistant kept the first copy and threw the rest away as duplicates, because for this type it discarded the sample count it would have needed to tell them apart. After the fix, the stored value matched what the phone sent.
- **Nothing had warned me for a week.** The existing check asked whether anything had arrived recently, and blood oxygen kept arriving. I replaced it with a check per type. It puts a notification on my phone when the set of stale types changes, not every 2 hours. Its first run flagged exactly the 2 types still missing.
- **The watch app had stopped exporting 2 types.** Resting heart rate and heart-rate variability were last written on 21 Sep, though every setting was on. I now work out resting heart rate myself from the heart-rate readings. To choose how, I logged my figure next to the watch app's for every day that had both. Only 1 day did: watch app 67, mine 67. One day is a sanity check, not a validation, and it is written down that way.

## What I took from it

My first diagnosis was built from the code I owned, with permissions granted and sync jobs reporting success, and all of it was true. The question was whether the data existed where my code was reading from, and the only way to answer that was to look at the source, not at my side of the connection.

---

*Project: Umbra. Rewritten for easier reading from the original in [umbra-sub-os-showcase / CASE-STUDIES.md](https://github.com/Ciaran11221/umbra-sub-os-showcase/blob/main/CASE-STUDIES.md#28-a-week-of-missing-heart-rate-and-the-page-i-never-turned) (commit 5940844). Same facts and numbers.*
