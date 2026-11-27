# 1. The cost report that counted copied calls twice

**In one line:** the tool I built to measure what my AI coding agents cost was 21% too high over 30 days, because some chats carry copies of other chats' calls.

| | |
|---|---|
| **Where** | `token_report.py`, a script that adds up token use and cost from the AI coding tool's own session logs |
| **Result** | 30 days: $490.65 reported, $405.38 when each call is counted once. After the fix, a 7-day reading matched an independent count to within 1 call in 1,341 |
| **Topic** | Measurement, Testing |

## The problem

I work with AI coding agents (Claude Code), and I wanted to know where the cost goes: which model, which kind of work, how much is the agent re-reading a long conversation. So I wrote a report that reads the session logs the tool keeps on disk and adds up every model call. A token is the unit these models are billed in, roughly three-quarters of a word. Every decision I made about my setup afterwards leaned on that report's numbers.

## What I assumed

That one log file holds one chat's calls, so removing duplicates inside each file was enough.

## What the evidence said

While checking an outside review of my setup, I checked the report itself. Starting a new chat from part of an old one copies the old chat's calls into the new file. My report counted each copy again.

```
  window    call rows   repeats   as reported   counted once   too high by
  3 days      1,135        65       $121.86        $116.85        4.3%
  7 days      3,047        65       $401.05        $396.04        1.3%
  30 days     3,504       482       $490.65        $405.38       21.0%
```

Before calling it a double count I tried to disprove it: if the repeats were separate calls, something would differ between copies. Nothing did. Every copy had the same usage, the same timestamp, the same request id and the same model.

## Why the tests had not caught it

The report's self-test passed 10 of 10. It used two files, but never put the same call in both, so it could not see the case that broke it.

## The fix

Duplicates are now removed across every file, and each call is kept in the chat where it first happened. Calls are picked by their own timestamp, not by the file's date. The new tests fail on the old code. After the fix, a 7-day reading came to 1,340 calls and $131.20; an independent one-off count over the same window came to 1,341 calls and $131.27.

## What I am not claiming

The dollar figures are API list prices, a stand-in for what my subscription actually uses. The 30-day baseline I had taken the day before was probably inflated the same way, by about 24% on a rebuild, and some older chats had been deleted, so it is a sample, not a total.

## What I took from it

Measure the measuring tool against a second count before trusting a decision to it.

---

*Project: Claude-Setup, the private repo that holds my AI coding setup and the tools that measure it. Written 27 Nov 2026 from the repo's work log and review notes.*
