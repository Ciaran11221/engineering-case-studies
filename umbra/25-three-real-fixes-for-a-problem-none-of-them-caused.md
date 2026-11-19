# 25. Three real fixes for a problem none of them caused

**Symptom.** I'd built a desktop UI for the assistant, with artwork I'd
generated as its centrepiece. Side by side with the original file, the page
looked soft and hazy. Same image, visibly worse.

**What I assumed.** That the browser was mangling it — that the page was doing
something to the picture an image viewer wasn't. That turned out to be true
three separate times, and none of them was the reason.

**What I found, fixed, and still had the problem.**

The first was arithmetic. The image is 1672px wide and the layout was drawing
it at 3000 — a 1.8x upscale. Measured in the live page, it held together to 28%
zoom and the browser smeared it above that. I sized the layout to the source
instead.

The second was one line of CSS. `will-change: transform` on the container,
which I'd added for smooth panning. It promotes the element to its own
composited layer, and the browser then rasterises that layer *once* at the
current scale and stretches the bitmap as you zoom. So the picture went soft
before the source ran out of pixels. That line reads as a performance hint and
behaves as a resolution cap.

The third was a blend mode. The art is neon on opaque black, and I'd used
`mix-blend-mode: screen` so the black would disappear against the page. By then
I'd made the page pure black, where screen-on-black changes nothing — so the
blend was buying nothing and still forcing a separate compositing path.

All three were real. All three are still fixed. The picture was still wrong.

**What the evidence actually said.** I went to compare the two again and
noticed the copy I was comparing *against* looked blurry too. It was
byte-identical to the one I'd been building with. So I compared that against
the file on my own disk:

```
on disk       1672 x 940    805 KB   PNG
what I had    1672 x 940     81 KB   WebP

identical pixels   61.2%
mean error         0.71 / 255
max error          90
PSNR               39.8 dB
```

Same dimensions. Ten times smaller. The copy I'd been working from had gone
through a transfer step that re-encoded it, and 39% of the pixels came out
different. Thin bright filaments on black is exactly where that shows.

**What I changed.** Read the original off disk and built the page's artwork
from that, losslessly. Nothing lossy is in the chain now.

**The part worth keeping.** I spent two rounds fixing the renderer before I
checked the thing being rendered. Every fix was correct, and that is what made
it convincing — each one produced a visible improvement, so each one felt like
progress toward the answer. Comparing the file against its original took one
command and would have gone first if I'd treated "the input might be damaged"
as a hypothesis worth killing rather than an assumption not worth stating.

It's the same shape as everything else here, with one difference: the wrong
theory kept paying out small, real dividends. That is much harder to abandon
than a theory that simply fails.

---

*Project: Umbra. Copied 19 Nov 2026 from [umbra-sub-os-showcase / CASE-STUDIES.md](https://github.com/Ciaran11221/umbra-sub-os-showcase/blob/main/CASE-STUDIES.md#25-three-real-fixes-for-a-problem-none-of-them-caused) at commit 5940844. The showcase copy is kept as well; if they differ, the showcase is newer.*
