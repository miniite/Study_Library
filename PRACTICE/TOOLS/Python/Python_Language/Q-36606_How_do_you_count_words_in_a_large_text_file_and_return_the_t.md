---
type: "practice"
difficulty: "Intermediate"
---

# How do you count words in a large text file and return the top N most common? Walk through your approach.

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Membership and Collection Complexity](<../../../../TOOLS/Python/Python_Language/Membership_and_Collection_Complexity.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

The task: read a large text file, count how often each word appears, and return the top N most frequent words. The interviewer is probing several things at once: (1) do I stream the file instead of loading it into memory (the word "large" is the signal), (2) do I know the idiomatic Python tools — collections.Counter and its most_common(n) — instead of hand-rolling, (3) can I reason about time/space complexity, and (4) do I handle the messy realities of text: case, punctuation, encoding, and defining what a "word" is.

Clarifying questions I'd ask: Is a "word" case-insensitive? Do we strip punctuation? Include numbers? How is N vs. the vocabulary size (are there ties at the boundary)? Does the file fit in memory once tokenized, or is the vocabulary itself huge (billions of distinct words → need external/streaming aggregation)? For a normal interview I'll assume: case-insensitive, alphanumeric tokens, and that the distinct-word count fits in RAM even though the raw file doesn't.

Plan: stream line by line → tokenize/normalize each line → feed a Counter → call most_common(N). Then discuss complexity and the scale-out variant.

Step 1 — Stream, don't slurp

open(path).read() pulls the entire file into memory; for a multi-GB log that's fatal. Iterating over a file object yields one line at a time and only holds one line plus the counter in memory:

PYTHON
with open(path, encoding="utf-8", errors="ignore") as f:
    for line in f:
        ...

The counter's memory is proportional to the number of distinct words (vocabulary), not the file size — and natural-language vocabulary grows sub-linearly (Heaps' law), so even a huge file usually has a manageable vocabulary of tens of thousands to a few million entries.

Step 2 — Normalize and tokenize

I need a consistent definition of a word. Splitting on whitespace alone leaves punctuation attached ("dog." != "dog") and mixes case ("The" != "the"). A robust choice is a regex that extracts alphanumeric runs, lowercased:

PYTHON
import re
WORD_RE = re.compile(r"[a-z0-9]+")
tokens = WORD_RE.findall(line.lower())

re.findall on already-lowercased text pulls out clean tokens and drops punctuation. If Unicode letters matter (accents, non-English), use re.findall(r"\w+", line.lower(), re.UNICODE) — but \w also keeps underscores, so I'd state the tradeoff. The exact tokenizer is a product decision; the interviewer wants to see that I made the decision explicitly rather than ignoring it.

Step 3 — Count with Counter

collections.Counter is a dict subclass built for exactly this. counter.update(iterable) increments counts for every element — I can hand it the whole token list per line:

PYTHON
from collections import Counter
counter = Counter()
...
counter.update(WORD_RE.findall(line.lower()))

update is implemented in C and is faster than a manual for w in tokens: counter[w] += 1 loop, though both are O(total tokens).

Step 4 — Top N

counter.most_common(N) returns the N highest-count (word, count) pairs, already sorted descending. Crucially, it does not fully sort the whole vocabulary — under the hood it uses heapq.nlargest, which is O(V log N) for vocabulary size V, far cheaper than O(V log V) when N is small (e.g., top 10 of a million words).

PYTHON
return counter.most_common(N)
Complexity
Time: O(T) to tokenize and count (T = total tokens) + O(V log N) to extract the top N (V = distinct words). Since T dominates, effectively O(T).
Space: O(V) for the counter — proportional to vocabulary, not file size. This is why streaming works even for files far bigger than RAM, as long as the vocabulary fits.
Edge cases and correctness
Encoding: always pass encoding="utf-8"; decide on errors="ignore"/"replace" for dirty data.
Ties at the boundary: most_common(N) breaks ties by insertion order (first-seen wins) in modern CPython — not alphabetical. If a stable/defined tie-break is required, sort explicitly: sorted(counter.items(), key=lambda kv: (-kv[1], kv[0]))[:N].
Empty file / N > vocabulary: returns fewer than N items gracefully — no crash.
Stop words: in real NLP you'd filter the/a/of etc.; mention it as a product choice.
Scaling beyond one machine

If the file is enormous and the vocabulary itself won't fit in RAM (e.g., counting URLs or user IDs, billions distinct), a single Counter breaks. Then:

Chunk + merge: process the file in shards, produce partial Counters, and merge with + / Counter.update — still needs the merged vocab to fit somewhere. Partial counters can be spilled to disk.
MapReduce / Spark: classic word-count map (emit (word, 1)) → reduce (sum) → top-N per partition then global merge. This is the textbook distributed word count.
Approximate top-K: if exact counts aren't required, a Count-Min Sketch + a bounded min-heap gives top-K in sublinear memory (the "heavy hitters" problem). Good to name for a senior signal.
What a strong answer emphasizes

Stream the file (don't .read() it), normalize deliberately (case + punctuation), use Counter().update() + most_common(N) (and know most_common uses a heap, giving O(V log N)), and articulate that memory scales with vocabulary, not file size — plus the distributed/approximate fallback when even the vocabulary is too large.

#### Common mistakes
/ probes
Loading the whole file with .read().
Splitting on spaces only, leaving punctuation/case bugs.
Fully sorting the vocabulary instead of using most_common.
Not knowing the tie-break behavior of most_common.

#### Follow-up
probe: "counts don't fit in memory?" → shard/merge, MapReduce, or Count-Min Sketch.

**Comparison:**

```
Concern	Choice	Why
Read strategy	Iterate for line in f	O(1) memory in file size; avoids .read() OOM
Tokenize	re.findall(r"[a-z0-9]+", line.lower())	Handles case + strips punctuation
Count	Counter().update(tokens)	C-level, O(total tokens)
Top N	most_common(N)	Heap-based, O(V log N), not O(V log V)
Time	O(T) total	T = total tokens dominates
Space	O(V)	V = distinct words (vocabulary), not file size
Vocab too big for RAM	Shard+merge / MapReduce / Count-Min Sketch	Distributed or approximate top-K
```

**Code:**

```python
import re
from collections import Counter

WORD_RE = re.compile(r"[a-z0-9]+")  # alphanumeric tokens, punctuation dropped

def top_n_words(path: str, n: int, encoding: str = "utf-8"):
    """Stream a large file and return the n most common words as (word, count)."""
    counter = Counter()
    with open(path, encoding=encoding, errors="ignore") as f:
        for line in f:                       # stream one line at a time (low memory)
            counter.update(WORD_RE.findall(line.lower()))  # normalize + count
    return counter.most_common(n)            # heapq.nlargest under the hood: O(V log n)

# Deterministic tie-break variant (count desc, then word asc):
def top_n_stable(path: str, n: int, encoding: str = "utf-8"):
    counter = Counter()
    with open(path, encoding=encoding, errors="ignore") as f:
        for line in f:
            counter.update(WORD_RE.findall(line.lower()))
    return sorted(counter.items(), key=lambda kv: (-kv[1], kv[0]))[:n]
```

**Example input / output:**

```

Input:

PYTHON
Copy
top_n_words("sample.txt", 3)
# tokens after lowercase + regex:
# ['the','quick','brown','fox','the','lazy','dog','the','quick','fox']

Output:

PYTHON
Copy
[('the', 3), ('quick', 2), ('fox', 2)]
# counts: the=3, quick=2, fox=2, brown=1, lazy=1, dog=1
# most_common(3) -> top 3 by count (quick/fox tie broken by first-seen: quick before fox)
```

**Data:**

```

Example file sample.txt:

CODE
Copy
The quick brown fox.
The lazy dog; the QUICK fox!
```

**Flow:**

1. EOF
2. No
3. Open file with encoding
4. Iterate line by line - streaming
5. line.lower + regex findall ->tokens
6. Counter.update tokens
7. counter.most_common N
8. Return top-N word,count pairs
9. Vocabulary fits in RAM?
10. Shard+merge / MapReduce /Count-Min Sketch

