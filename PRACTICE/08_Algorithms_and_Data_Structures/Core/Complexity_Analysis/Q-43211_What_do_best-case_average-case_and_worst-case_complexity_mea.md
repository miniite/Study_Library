---
type: "practice"
difficulty: "Basic"
---

# What do best-case, average-case, and worst-case complexity mean?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Stack-based Complexity](<../../../../08_Algorithms_and_Data_Structures/Core/Complexity_Analysis/Stack-based_Complexity.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

What is really being asked: Define best-case, average-case, and worst-case time complexity, and — crucially — explain what each one means, when it happens, and why we care about each. Even though it's a "basic concept" question, a shallow "best is fast, worst is slow" answer won't land. The interviewer wants to see that you understand these describe the same algorithm run on different inputs of the same size, that they map to the Big-O / Big-Ω / Big-Θ notations, and that you know which one to actually engineer around (usually worst case). A concrete example (linear search, quicksort) proves you get it.

#### Assumptions
/ clarifying questions: I'll assume we're talking about time complexity as a function of input size n, though the same idea applies to space. The key framing question I'd state: "these three cases all fix the input size n and vary the input arrangement — they are not about small vs large n." That distinction is the heart of the answer.

Plan: Define all three precisely, tie each to a notation, illustrate with one running example everyone knows (searching), add a second example where the cases genuinely differ (quicksort), explain which to use for engineering decisions, and clear up the classic average-case pitfalls.

The core idea

For a fixed input size n, an algorithm can take different amounts of time depending on the exact input. Best/average/worst case describe the extremes and the typical of that spread:

Best case = the minimum time the algorithm can take over all inputs of size n — the luckiest input.
Worst case = the maximum time over all inputs of size n — the most adversarial input.
Average case = the expected time, averaged over all inputs of size n (weighted by how likely each input is — usually assuming a uniform/random distribution).

They are three measurements of the same algorithm, differing only in which input of size n you feed it. This is the single most important point: best/worst is not "small n vs big n." It's "easy arrangement vs hard arrangement, at the same n."

Mapping to notation

These cases are often (loosely) paired with asymptotic notations:

Worst case → Big-O (O) — an upper bound; "never slower than this."
Best case → Big-Omega (Ω) — a lower bound; "never faster than this."
Average / tight case → Big-Theta (Θ) — when upper and lower bounds match, a tight bound.

A subtlety worth knowing: O, Ω, Θ are just bounds and can technically be applied to any case (e.g., "worst-case time is Θ(n)"). But in everyday interview usage, "the complexity of X is O(...)" almost always means worst-case, because that's the guarantee we can rely on.

Running example — linear search (the clearest one)

Search an unsorted array of n items for a target:

Best case — O(1): the target is the first element → 1 comparison, regardless of n. Lucky input.
Average case — O(n): if the target is equally likely at any position, you scan ~n/2 elements → still linear.
Worst case — O(n): the target is the last element, or not present → all n comparisons. Adversarial input.

Same algorithm, same n; the arrangement of the data (where the target sits) decides which case you hit.

A sharper example — quicksort (where the cases really diverge)

Quicksort makes the three cases dramatically different, which is why it's the classic teaching example:

Best/average case — O(n log n): when each pivot splits the array into roughly equal halves (which happens for random data), the recursion is log n deep with O(n) work per level.
Worst case — O(n²): when the pivot is always the smallest or largest element (e.g., already-sorted input with a naive first-element pivot), each partition removes just one element → n levels of O(n) work.

Here the average case (O(n log n)) is what you get in practice and is far better than the worst (O(n²)), which is why quicksort is used despite its ugly worst case — and why real implementations randomize the pivot to make the worst case astronomically unlikely.

Which one should you actually use?
Worst case is the default for engineering guarantees. It's a promise: "no matter what input, it won't be slower than this." Latency SLAs, real-time systems, and anything adversary-facing (where a malicious user could craft the worst input — e.g., a hash-collision DoS) must be reasoned about via worst case.
Average case matters when inputs are naturally random and you care about typical throughput (e.g., hash-table lookups are O(1) average but O(n) worst; quicksort is chosen for its O(n log n) average).
Best case is mostly of theoretical interest — it tells you the floor, but you rarely design around getting lucky. It's useful for spotting early-exit optimizations (e.g., a sort that's O(n) best case on already-sorted data, like insertion sort).

#### Common mistakes
& follow-ups
Confusing case with input size. Best/worst do NOT mean small/large n. All three are functions of n; they differ by input arrangement at the same n.
Assuming worst case is rare/irrelevant. For adversarial inputs it may be the only case that matters (security). Hash tables are the classic example: O(1) average, but crafted collisions force O(n).
Thinking average case is just "(best + worst) / 2". No — it's a probability-weighted expectation over the input distribution, and it depends on the assumed distribution. Change the distribution and the average changes.
Mixing up notation and case. Big-O describes an upper bound; it is not synonymous with "worst case," even though they're often paired. You can have an O(n) bound on the average case, for instance.

#### Follow-up
probes: "Give an algorithm whose best and worst case differ" (quicksort, insertion sort). "Why is hash-map lookup O(1) average but O(n) worst?" (collisions). "What's amortized complexity, and how is it different?" (average cost per operation over a sequence, guaranteed even in the worst sequence — e.g., dynamic-array append is O(1) amortized).
Summary

Best, average, and worst case describe how an algorithm's running time varies across different inputs of the same size n: best = the fastest (luckiest) input (lower bound, Ω), worst = the slowest (adversarial) input (upper bound, O), average = the expected time over the input distribution (often Θ). Linear search makes it concrete — O(1) best, O(n) average and worst; quicksort shows the cases can diverge sharply — O(n log n) average vs O(n²) worst. In practice we engineer to the worst case for hard guarantees, use the average when inputs are naturally random, and treat best as mostly informational. The key insight to convey: these are the same algorithm on different arrangements, not different sizes.

**Comparison:**

```
Case	Meaning	Which input triggers it	Bound / notation
Best	Minimum time over all inputs of size n	The luckiest arrangement	Lower bound → Ω
Average	Expected time over the input distribution	A "typical"/random input	Tight bound → Θ (often)
Worst	Maximum time over all inputs of size n	The most adversarial arrangement	Upper bound → O
Algorithm	Best	Average	Worst
Linear search	O(1)	O(n)	O(n)
Binary search (sorted)	O(1)	O(log n)	O(log n)
Quicksort	O(n log n)	O(n log n)	O(n²)
Insertion sort	O(n)	O(n²)	O(n²)
Hash-map lookup	O(1)	O(1)	O(n) (collisions)
```

**Code:**

```python
# One algorithm, three cases — driven purely by the INPUT, not by n.
def linear_search(a, target):
    for i in range(len(a)):     # each iteration = 1 comparison
        if a[i] == target:
            return i
    return -1

n = 1000
arr = list(range(n))            # [0, 1, 2, ..., 999]

# BEST case  -> target is first  -> 1 comparison        -> O(1)
linear_search(arr, 0)

# WORST case -> target is absent -> 1000 comparisons    -> O(n)
linear_search(arr, -1)

# AVERAGE    -> target random    -> ~n/2 comparisons     -> O(n)
#   averaged over all positions, expected work is (n+1)/2
```

**Example input / output:**

```

Same linear_search, same size n = 5, three different inputs:

CODE
Copy
arr = [10, 20, 30, 40, 50]     (n = 5 in all three)

BEST:    target = 10  -> checks index 0            -> 1 comparison   (O(1))
AVERAGE: target = 30  -> checks indices 0,1,2      -> 3 comparisons  (~n/2)
WORST:   target = 99  -> checks all 0..4, absent   -> 5 comparisons  (O(n))

Quicksort — cases genuinely diverge (n = 8):

CODE
Copy
Input [4,2,7,1,5,3,8,6] (random-ish) -> balanced splits -> ~ n log n  work
Input [1,2,3,4,5,6,7,8] (sorted, naive first-pivot) -> splits of size (n-1)
        each time -> n levels -> ~ n^2 work  (WORST case)
```

**Flow:**

1. Fix input SIZE = n
2. Run the SAME algorithm onmany inputs of size n
3. Best case = fastest run (mintime) => lower bound(Omega)
4. Average case = expected timeover input distribution =>tight (Theta)
5. Worst case = slowest run (maxtime) => upper bound (Big-O)
6. Engineer to THIS forguarantees / SLAs / adversarialinputs

