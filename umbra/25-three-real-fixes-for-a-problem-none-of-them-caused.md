# 25. Three real fixes for a problem none of them caused

**In one line:** the artwork on my desktop UI looked blurry, I fixed 3 real rendering problems, and the cause was that the copy of the image I was working from had been re-compressed.

| | |
|---|---|
| **Where** | Desktop UI, the main artwork |
| **Result** | The copy I used was 81 KB against 805 KB on disk, with 39% of its pixels different. Fixed by building from the original file |
| **Topic** | Debugging, Measurement |

## The problem

I had built a desktop UI for the assistant, with artwork I had generated as its centrepiece. Side by side with the original file, the page looked soft and hazy. It was the same image and visibly worse.

## What I assumed

That the browser was mangling the picture, and that the page was doing something to it that an image viewer was not. That turned out to be true 3 separate times, and none of them was the reason.

## What I found, fixed, and still had the problem

- **Arithmetic.** The image is 1672px wide and the layout was drawing it at 3000, a 1.8x upscale. Measured in the live page, it held together to 28% zoom and the browser smeared it above that. I sized the layout to the source instead.
- **One line of CSS (the page styling language).** I had added `will-change: transform` to the container for smooth panning. It promotes the element to its own layer, and the browser then draws that layer once at the current scale and stretches the bitmap as you zoom. So the picture went soft before the source ran out of pixels. The line reads as a performance hint and behaves as a resolution cap.
- **A blend mode.** The art is neon on opaque black, and I had used `mix-blend-mode: screen` so the black would disappear against the page. By then I had made the page pure black, where that blend changes nothing. It bought nothing and still forced a separate drawing path.

All 3 were real and are still fixed. The picture was still wrong.

## What the evidence said

I went to compare the two again and noticed that the copy I was comparing against looked blurry too. It was byte-identical to the one I had been building with. So I compared it against the file on my own disk.

## How I measured it

```
on disk       1672 x 940    805 KB   PNG
what I had    1672 x 940     81 KB   WebP

identical pixels   61.2%
mean error         0.71 / 255
max error          90
PSNR               39.8 dB
```

| Measurement | Value |
|---|---|
| Dimensions | 1672 x 940, the same |
| File size | 805 KB against 81 KB, about 10 times smaller |
| Pixels that came out different | 39% |

## The cause

The copy I had been working from had gone through a transfer step that re-encoded it, and 39% of the pixels came out different. Thin bright filaments on black is exactly where that shows.

## The fix

I read the original off disk and built the page's artwork from that, losslessly. Nothing lossy is in the chain now.

## What I took from it

I spent 2 rounds fixing the renderer before I checked the thing being rendered. Every fix was correct, which made it convincing, because each one produced a visible improvement. Comparing the file against its original took 1 command and should have gone first. A wrong theory that keeps paying out small real gains is much harder to abandon than one that simply fails.

---

*Project: Umbra. Rewritten for easier reading from the original in [umbra-sub-os-showcase / CASE-STUDIES.md](https://github.com/Ciaran11221/umbra-sub-os-showcase/blob/main/CASE-STUDIES.md#25-three-real-fixes-for-a-problem-none-of-them-caused) (commit 5940844). Same facts and numbers.*
