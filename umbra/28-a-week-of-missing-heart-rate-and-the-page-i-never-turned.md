# 28. A week of missing heart rate, and the page I never turned

**Symptom.** My assistant tracks vitals from my watch: the watch app writes to
Android's shared health store (Health Connect), my phone app reads from it
every two hours and sends daily figures to the assistant. For a week, heart
rate, heart-rate variability and resting heart rate had stopped arriving.
Blood oxygen still arrived, every day.

**The first diagnosis, and why it was wrong.** The morning's write-up blamed
the watch app: my phone app's read permissions were all granted, so the
break had to be upstream. That was reasoning from what I could see in my own
code. I hadn't looked at the store itself. When I did — Health Connect's own
heart-rate screen — it held readings from the watch app up to an hour old.
The data was there. My app wasn't getting it.

**The cause.** A request to Health Connect returns one page of results: 1,000
records, oldest first, plus a token you pass back to get the next page. My
app took the first page and never asked for the next. For a while a month of
data fitted on one page, so it worked. Once it didn't, the newest days were
simply never read. Every sync sent the same 56 readings, and the assistant
logged each one as a duplicate — "0 new" — which looks exactly like a quiet
week, not a broken connection.

The first sync after the fix:

```
heart rate                  1,485 records over 2 pages
heart-rate variability      4,808 records over 5 pages
readings sent               70   (was 56)
new readings stored         13
newest pulse                today (was 7 days old)
```

**Three more things the same trace turned up.** Each fixed, each checked on a
real sync the same morning:

- **Blood oxygen froze at its first value each day.** The phone sends one
  daily average, stamped at midnight, and re-sends it as the day goes on. The
  assistant kept the first copy and threw the rest away as duplicates,
  because for this type it discarded the sample count it would have needed to
  tell them apart. After the fix, the stored value matched what the phone sent.
- **Nothing had warned me for a week.** The existing check asked "has
  anything arrived recently?" — and blood oxygen kept arriving. I replaced it
  with a check per type, which puts a notification on my phone when the set of
  stale types changes, not every two hours. Its first run flagged exactly the
  two types still missing.
- **The watch app really had stopped exporting two types.** Resting heart
  rate and heart-rate variability: last written on 21 Sep, though every
  setting was on. I now work out resting heart rate myself from the heart-rate
  readings. To choose how, I logged my figure next to the watch app's for
  every day that had both. Only one day did: watch app 67, mine 67. One day
  is a sanity check, not a validation, and it's written down that way.

**The part worth keeping.** The first diagnosis was built from the code I
owned — permissions granted, sync jobs reporting success — and all of it was
true. None of it was the question. The question was whether the data existed
where my code was reading from, and the only way to answer that was to look
at the source, not at my side of the connection.

---

*Project: Umbra. Copied 19 Nov 2026 from [umbra-sub-os-showcase / CASE-STUDIES.md](https://github.com/Ciaran11221/umbra-sub-os-showcase/blob/main/CASE-STUDIES.md#28-a-week-of-missing-heart-rate-and-the-page-i-never-turned) at commit 5940844. The showcase copy is kept as well; if they differ, the showcase is newer.*
