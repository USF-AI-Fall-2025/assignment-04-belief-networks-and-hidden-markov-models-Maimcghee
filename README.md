# Belief-Networks-Hidden-Markov-Models
**Fall 2025 — CS 362/562**

## Running Instructions

1. Make sure `aspell.txt` is in the same folder as `spell_check_v2.ipynb`.
2. Restart the kernel and run all cells top to bottom.
3. This builds the probability tables, runs the automated tests, and prints the evaluation table across six thresholds.
4. Correct text: `fix_text("your sentence here")`
5. Evaluate a specific threshold: `evaluate(threshold_value)`

## Normalization / Token-Handling Decisions

- All words are lowercased before training and decoding, since capitalization isn't a spelling error this model needs to represent.
- Correct/typed pairs that are identical after lowercasing (i.e. Presbyterian/presbyterian) are kept as valid training examples — they teach the emission table that typing a letter correctly is the common case.
- Pairs where the correct and typed words have different lengths are dropped before training, since this model only handles substitution errors. This drops the majority of typos in `aspell.txt`, since most real typos are insertions or deletions, not substitutions.
- In `fix_text`, punctuation and unsupported characters are stripped from each word before decoding. If nothing supported remains, the original token is returned unchanged rather than crashing.

## Reflection

### 1. Overall performance

The full HMM barely outperformed the baseline on exact word accuracy. Both scored around 2.3% at threshold=0, since the test set contains only one word the model corrected exactly right: *liquify* / *liquuf* → *liquefy*. At higher thresholds, the HMM's exact word accuracy matched the baseline exactly at 2.3%, because it stopped attempting any corrections that would count as successful — while character accuracy improved slightly (76.7% → 78.4%) as the threshold rose, since the model made fewer risky wrong guesses. The successful correction rate was 2.4% at threshold=0 (1 of 42 misspelled test words) and 0% at every higher threshold tested, since that one success's improvement score (~0.85–1.18) didn't clear thresholds of 1 or above.

| Threshold | Exact | Success |
|---|---|---|
| t=0 | 2.3% | 2.4% |
| t=1 | 0.0% | 0.0% |
| t=2 | 0.0% | 0.0% |
| t=3 | 2.3% | 0.0% |
| t=5 | 2.3% | 0.0% |

### 2. Failure case: "a" → "e"

The one-letter word 'a' was changed to 'e' at threshold=0. This happened because, for a single-letter word, the score depends heavily on `start_probs` and `end_probs` for that one letter, and `end_probs["e"] = 0.22` was roughly 10x higher than `end_probs["a"] = 0.02` in the training data, since 'e' is a much more common word-ending letter overall. That advantage was enough to outweigh the fact that `em_probs["a"]["a"]` is itself a strong, common pattern. The model doesn't have a concept of "is this a real, complete word," so a short word with few real training examples of its own is especially vulnerable to this kind of mistake.

### 3. Failure case: "hte" (intended "the")

When I tested with "hte" (intended "the") it was never corrected, at any threshold. Looking at the trained probabilities: `em_probs["t"]["h"]` (intended t, typed as h) was only about 0.008, very close to the Laplace smoothing floor, meaning the training set had little to no evidence of t being mistyped as h. Meanwhile `em_probs["h"]["h"]` (no typo) was about 0.34, a much stronger signal. With no competing evidence for t at that position, Viterbi's own best-guess path was "hte" itself — the threshold was never even the limiting factor here, since no better correction was ever proposed.

### 4. Threshold selection

At threshold=0, the model accepted any positive improvement, which let through both its one real success (liquify → liquefy) and several false corrections (like a → e). Raising the threshold to 3 blocked all observed false corrections (false correction rate dropped from 100% to 0%) and character accuracy improved slightly, but it also blocked the one real success, dropping successful correction rate to 0%. I selected threshold=3 as a reasonable choice.

### 5. Real-world limitations

Real typos would include far more insertions and deletions than this dataset's substitution-only typos; this model cannot attempt those at all. So real-world accuracy would likely look considerably worse than these numbers, unless paired with a more general alignment-based model that can handle typed and intended words of different lengths.

## AI Assistance

I used Claude AI to help fill some gaps. I tried not to let it give me the answers and instead rewatched parts of lecture videos to fill in details that were missing. For the most part I asked it to remind me of the structure of how to do certain things, and for the Viterbi decoding I asked it to give me the steps, which gave me a bit more detail than the spec did. I also asked it to remind me about smoothing and used its response to work out the equation. For everything else, I tried to implement it on my own, testing and debugging as needed.

Transcript of the conversation: [https://claude.ai/share/7d5774a1-ebd3-42bf-8b8f-9139b354ed30](https://claude.ai/share/7d5774a1-ebd3-42bf-8b8f-9139b354ed30)