# 23. More training data changed the voice and did not improve it

**In one line:** I trained a synthetic voice on more hours of speech to improve it, and a blind listening test showed it changed the voice without making it better.

| | |
|---|---|
| **Where** | Voice training (synthetic voices) |
| **Result** | Gap between the models of 1.12 times their own noise, listening tally of 6-5, model not shipped |
| **Topic** | Testing, Measurement |

## The problem

I have 2 synthetic voices, built the same way from the same kind of source recordings. I rate one at about 95% and would ship it. The other sits around 65%. It is understandable and clearly the right character, but it has a roboty edge on individual words that I could not tune out.

## What I assumed

The obvious difference was volume. The good one was trained on 8.54 hours of speech and the poor one on 6.92. So the theory was that more hours would give a better voice.

Before spending an hour of GPU time, I wrote down the result that would prove me wrong:

> If the model trained on more data does not win a blind listening test, then the hours were a coincidence and not the cause, and the answer is to stop, not to go looking for even more data.

That sentence is the only reason this case study has an ending.

## Where the extra hours came from

An earlier cleanup pass had removed echoey recordings. It judged groups of files together, so if enough of a group was reverberant (echoey), the whole group went. That threw away 37% of this voice's material against 21% of the other's, and a lot of it was fine. Re-testing the discarded files one at a time against the same quality bar recovered 1,052 usable clips, which took the voice from 6.92 hours to 8.41.

**The trap inside that cost me a rebuild.** My first attempt applied the quality bar to every file one at a time, including the ones already in use. It rejected 4,804 files that were already passing, and made the training set smaller instead of larger.

The quality measure correlates with how long the clip is (+0.51). Clips of 1 to 2 seconds pass it 17% of the time and clips of 5 seconds pass 100% of the time. Applied per file, an "is this echoey" test is largely an "is this long" test. The original cleanup had only ever applied it to long lines, and I read straight past the word "long".

The rebuild now only adds. It tests the discarded pile and never re-judges anything already accepted. That is asserted in the code, so it fails loudly instead of quietly shrinking.

**Two failed runs before one finished.** The first ran out of memory partway through, with a game running on the same GPU. The second failed with nothing else running, which ruled out the easy explanation. The real cause was that the recovered clips were 3.3 times more likely to run over 8 seconds, and memory use scales with the longest clip in a batch. Capping the additions at the original set's own length limit fixed it.

## What the evidence said

The synthesiser gives a different result each time. Render the same word twice from the same model and you get 2 different files. So "the new model sounds different" means nothing unless the difference between the 2 models is bigger than a model's disagreement with itself.

| Measurement | Value |
|---|---|
| Renders per model per word | 5 |
| Words | 8 |
| Gap between the 2 models, relative to the gap within one model | 1.12 times |
| Threshold I wrote into the script before running it | 1.10 |

The comparison ignores length differences. The difference was real and small. That told me in advance that "I cannot tell them apart" would be a legitimate answer.

## How I measured it

**The first listening test was unusable.** I built it from full sentences, and I said:

> "the tempo of the words is wrong and thats fine for now but its making it hard to tell subtle differences. Can we use individual words side by side?"

That voice has uneven pacing at sentence length. Two renderings of the same sentence differ in their pauses before they differ in the voice, and the pauses are the loudest thing in the file. Rebuilding it as one word per file removed the problem, because there is no sentence and so no pacing to listen past.

**Round one.** 8 words, each played as A-B-A-B, with me not knowing which side was which. 6 were real comparisons. 2 were rigged, with the same model on both sides.

I picked the second clip on all 6 real comparisons, whichever model was in that slot. 6 of 6 on one position is a 1-in-64 coin flip. On its own that result goes in the bin, because it says I was answering by position and not by listening.

But I answered "no difference" on both rigged pairs. Someone reflexively saying "the second one" would have said it there too. So I was genuinely discriminating. I identified all 8 correctly as same or different, and my preference tracked the slot and not the model. The score across the 6 was 3-3. Detection was real and direction was meaningless.

**Round two.** The same test with the sides swapped. The same words and structure, the 2 models exchanged, the rigged pairs flipped as well, and the running order reshuffled.

| Round two result | Value |
|---|---|
| Rigged pairs answered correctly, across both rounds | 4 out of 4 |
| Real comparisons I could answer | 5 |
| Consistent across the swap | 2 |
| Reversed | 3 |
| Preference of the 2 consistent ones | opposite models |
| Tally across every answer in both rounds | 6-5 |

2 consistent out of 5 is what a coin flip produces.

## The cause

The condition I had written down was met. Over 1,000 extra clips and an hour and a half of extra audio produced a voice that was measurably different and not measurably better. The hours were not the cause.

What is left is the source recordings. The weaker voice's actor varies 5.3 dB in loudness from line to line, and the stronger one's varies 20. A model trained on a flatter performance reproduces a flatter performance, and no quantity of the same material changes that.

Every other theory I tested against that voice's gap has come back empty: retraining its expressive range, the timing floor, the randomness setting, compressing its loudness variation, corpus contamination, preprocessing, and now data volume. The loudness theory died against the best control I ran all week. The voice I rate at 95% deviates further from its source actor than the one I rate at 65%.

## The fix

I stopped. The model is not shipping. I deleted the 24.8 GB of training output and kept the logs. I do not have a fix for the flatter performance, and saying so is more useful than a ninth theory.

## What I got wrong, and cannot undo

Between the 2 rounds I fixed a real defect in my test harness. Round one's audio files began 10 milliseconds before the first word, and the Windows audio player can swallow the opening of a file. That would have damaged the first clip only and pushed me toward the second. Round two's files open with 300 milliseconds of silence.

I also read my own 6-of-6 pattern before running round two. Knowing about a bias is enough to change it. When position-picking fell from 6 of 6 to 3 of 6, I had 2 candidate causes and no way to separate them. The write-up says the drop cannot be claimed as evidence the fix worked. Next time the reversal round runs first and I read the direction afterwards.

## What I would still challenge

6 real comparisons per round is a small sample, and a real but slight preference could hide inside it. I would trust the same design run across 30 words in one sitting. I did not build it, because the measured gap was 1.12 times the models' own noise, and chasing a difference that small with a bigger test is how a null result becomes a false positive by persistence alone.

## What I took from it

I wrote down what would prove me wrong before I started, and when it happened I stopped instead of looking for more data.

---

*Project: Umbra. Rewritten for easier reading from the original in [umbra-sub-os-showcase / CASE-STUDIES.md](https://github.com/Ciaran11221/umbra-sub-os-showcase/blob/main/CASE-STUDIES.md#23-more-training-data-changed-the-voice-and-did-not-improve-it) (commit 5940844). Same facts and numbers.*
