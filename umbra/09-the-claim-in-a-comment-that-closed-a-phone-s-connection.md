# 9. The claim in a comment that closed a phone's connection

**In one line:** I wrote a comment claiming how other code behaved without checking, the claim was false, and the phone's connection to the desktop closed.

| | |
|---|---|
| **Where** | Architecture jar's storage of in-progress work (see [case study 8](08-three-failures-that-were-obvious-to-a-person-and-invisible-to-the-system.md)), and the phone connection |
| **Result** | Phone showed "couldn't reach core, EOF" and the connection closed; fixed, with a test that runs the real output through the network layer's serializer |
| **Topic** | Debugging, Testing |

## The problem

I had just redesigned how the architecture jar (one self-contained feature module of the assistant) stores its in-progress work. I restarted the desktop app to pick up the change, then opened the matching screen on my phone. It came back with "couldn't reach core, EOF" and the connection closed immediately.

## What I assumed

While writing the redesign, I added an internal field to the data 1 function returns. I documented it with a claim about the code that consumes it: that a specific handler "only forwards a few named fields", so the extra field would never leave the process. I never opened that handler to check. It read as obviously true from the calling code alone.

## What the evidence said

The handler forwards the whole structure with no filtering at all. The extra field held a filesystem path object, not a plain string. When a real request tried to convert that structure to JSON (the text format sent over the network), it failed. The error handling around that failure closed the connection immediately, which is the "EOF" the phone saw.

## How I measured it

Once I checked the log instead of the comment I had written, it said:

```
TypeError: Object of type WindowsPath is not JSON serializable
when serializing dict item '_folder'
when serializing dict item 'diagrams'
```

Those 2 lines state the whole bug, and they sat in the log the whole time it mattered.

## The cause

My comment was a claim about another piece of code, and I wrote it without looking at that code.

## The fix

I moved the filesystem lookup into a function whose return value nothing outside the module sees. The data handed back to any caller now carries only plain fields that are safe to send. I also added a test that would have caught this before it shipped: it takes the real function's real output, runs it through the same serializer the network layer uses, and asserts that it does not raise an error.

## What I took from it

A comment that states how other code behaves is a claim about that code, not a fact available from the file I am editing. It needed exactly 1 grep before I wrote it down, and it shipped anyway because the grep felt unnecessary. It cost a live connection closing at the moment it mattered.

---

*Project: Umbra. Rewritten for easier reading from the original in [umbra-sub-os-showcase / CASE-STUDIES.md](https://github.com/Ciaran11221/umbra-sub-os-showcase/blob/main/CASE-STUDIES.md#9-the-claim-in-a-comment-that-closed-a-phones-connection) (commit 5940844). Same facts and numbers.*
