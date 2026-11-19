# 23. More training data changed the voice and did not improve it

**The problem.** I have two synthetic voices, built the same way from the same
kind of source recordings. One of them I would ship — I rate it about 95%. The
other sits around 65%: understandable, clearly the right character, but with a
roboty edge on individual words that I kept hearing and could not tune out.

The obvious difference between them was volume. The good one was trained on
8.54 hours of speech, the poor one on 6.92. So the theory wrote itself: more
hours, better voice.

**What I wrote down before starting.** Before spending an hour of GPU time on
it, I wrote the condition that would make me abandon the idea — the result that
would prove me wrong, rather than the result I was hoping for:

> If the model trained on more data does not win a blind listening test, then
> the hours were a coincidence and not the cause — and the answer is to stop,
> not to go looking for even more data.

That sentence is the only reason this case study has an ending.

**Where the extra hours came from.** An earlier cleanup pass had removed
echoey recordings, and it worked at the wrong granularity: it judged groups of
files together, so if enough of a group was reverberant the whole group went.
That threw away 37% of this voice's material against 21% of the other's, and a
lot of what went was fine.

Re-testing the discarded files one at a time, against the same quality bar,
recovered 1,052 usable clips — 6.92 hours to 8.41.

**The trap inside that, which cost me a rebuild.** My first attempt applied the
quality bar to every file individually, including the ones already in use. It
*rejected 4,804 files that were already passing* and made the training set
smaller instead of larger.

The cause was measurable and slightly embarrassing: the quality measure
correlates with how long the clip is (+0.51). One-to-two second clips pass it
17% of the time; five-second clips pass it 100%. Applied per file, an "is this
echoey" test is substantially an "is this long" test. The original cleanup had
only ever applied it to long lines — and the word "long" was doing load-bearing
work that I read straight past.

The rebuild now only ever *adds*: it tests the discarded pile and never
re-judges anything already accepted, with that guarantee asserted in the code
so it fails loudly rather than silently shrinking.

**Two failed runs before one finished.** The first died out of memory partway
through, with a game running on the same GPU. The second died with nothing else
running at all, which killed the easy explanation. The real cause was that the
recovered clips were 3.3 times more likely to run over eight seconds, and
memory use scales with the longest clip in a batch. Capping the additions at
the original set's own length limit fixed it.

**Before asking my ear: was there anything to hear?** The synthesiser is
non-deterministic — render the same word twice from the *same* model and you
get two different files. So "the new model sounds different" means nothing
unless the difference between the two models is bigger than a model's
disagreement with itself.

I measured both. Five renders per model per word, eight words, compared in a
way that ignores length differences. The gap between the two models came out
1.12 times the gap within a single model — against a threshold of 1.10 that I
wrote into the script before running it.

A real difference, and a small one. That mattered later: it told me in advance
that "I cannot tell them apart" would be a legitimate answer rather than a
failure of attention.

**The first listening test was unusable, and the reason was structural.** I
built it as full sentences and could not use it:

> the tempo of the words is wrong and thats fine for now but its making it hard
> to tell subtle differences. Can we use individual words side by side?

That voice has uneven pacing at sentence length. So two renderings of the same
sentence differ in their *pauses* before they differ in anything about the
voice itself — and pause structure is the loudest thing in the file. It was
drowning out the exact property I was trying to judge.

Rebuilding it as one word per file did not reduce that problem. It removed it.
There is no sentence, so there is no pacing to listen past.

**Two comparisons that did nothing, and saved the whole result.** Eight words,
each played as A-B-A-B, with me not knowing which side was which. Six were real
comparisons. **Two were rigged: the same model on both sides.**

Round one came back with a clean answer and an ugly pattern. I picked the
second clip on all six real comparisons — every one, regardless of which model
was in that slot. Six of six on one position is a 1-in-64 coin flip. On its own
that result goes in the bin, because it says I was answering by position rather
than by listening.

**Except I answered "no difference" on both rigged pairs.** Someone reflexively
saying "the second one" would have said it there too. The rigged pairs proved I
was genuinely discriminating — and once that is established the rest of the
sheet reads clearly: I identified all eight correctly as same-or-different, and
my *preference* tracked the slot instead of the model. The score across the six
was 3–3.

Detection real. Direction meaningless.

**Round two was the same test with the sides swapped.** Same words, same
structure, the two models exchanged, the rigged pairs flipped as well, the
running order reshuffled.

Both rigged pairs came back correct again — four out of four across the two
rounds. Of the five real comparisons I could answer, two were consistent across
the swap and three reversed, and the two consistent ones favoured *opposite*
models. Across every answer in both rounds the tally was 6–5.

Two consistent out of five is exactly what a coin flip produces.

**So the condition I had written down was met, and I stopped.** Over a thousand
extra clips and an hour and a half of extra audio produced a voice that was
measurably different and not measurably better. The model is not shipping. I
deleted the 24.8 GB of training output and kept the logs.

**What I got wrong, and cannot undo.** Between the two rounds I fixed a real
defect in my own test harness: round one's audio files began ten milliseconds
before the first word, and the Windows audio player can swallow the opening of
a file — which would have damaged the first clip only, pushing me toward the
second. Round two's files open with 300 milliseconds of silence.

But I also read my own six-of-six pattern *before* running round two. Knowing
about a bias is enough to change it. So when position-picking fell from six of
six to three of six, I have two candidate causes and no way to separate them.
The honest note in the write-up is that the drop cannot be claimed as evidence
the fix worked. Next time the reversal round runs first and I read the
direction afterwards.

**What this leaves.** Every theory I have tested against that voice's remaining
gap has now come back empty — retraining its expressive range, the timing
floor, the randomness setting, compressing its loudness variation, corpus
contamination, preprocessing, and now data volume. The loudness one died
against the best control I ran all week: the voice I rate at 95% deviates
*further* from its source actor than the one I rate at 65%.

What is left is the source recordings themselves. The weaker voice's actor
varies 5.3 dB in loudness from line to line; the stronger one's varies 20. A
model trained on a flatter performance reproduces a flatter performance, and no
quantity of that same material changes it. I do not have a fix for that, and
saying so is more useful than a ninth theory.

**What I would still challenge.** Six real comparisons per round is a small
sample, and a genuine but slight preference could hide inside it. What I would
trust more is the same design run across thirty words in one sitting. I did not
build that, because the measured gap between the models was 1.12 times their
own noise — and chasing a difference that small with a bigger test is how a
null result gets converted into a false positive by persistence alone.

---

*Project: Umbra. Copied 19 Nov 2026 from [umbra-sub-os-showcase / CASE-STUDIES.md](https://github.com/Ciaran11221/umbra-sub-os-showcase/blob/main/CASE-STUDIES.md#23-more-training-data-changed-the-voice-and-did-not-improve-it) at commit 5940844. The showcase copy is kept as well; if they differ, the showcase is newer.*
