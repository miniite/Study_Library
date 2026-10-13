---
type: "practice"
difficulty: "Intermediate"
---

# If n can be up to 10^5, why is an O(n^2) solution likely to time out and what complexity should you aim for?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Big-O vs Constant Factors](<../../../../08_Algorithms_and_Data_Structures/Core/Complexity_Analysis/Big-O_vs_Constant_Factors.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

What the question is really asking: With n ≤ 10^5, quantify why an O(n^2) algorithm blows the time limit, and state what complexity target actually passes. This is the direct, practical companion to "guess complexity from constraints."

What the interviewer is probing for: (1) Can I turn a Big-O into a concrete operation count and translate that into wall-clock seconds? (2) Do I know the target (O(n log n) / O(n)) and, ideally, how to hit it? A shaky candidate says "n² is slow"; a strong one says "n² is 10^{10} ops ≈ 10–100 s against a 1–2 s limit, so I need n log n, roughly 1.7·10^6 ops, using sort + two pointers / hashing / binary search."

#### Assumptions
~10^8 simple ops/sec for a compiled language on a typical judge (I mentally scale to ~10^7 for Python/JS), and a 1–2 second time limit. Constant factors move this by 2–10×, so the conclusion is about orders of magnitude, not a single cycle count.

Plan: compute n^2 at n = 10^5 → convert to seconds → compare to the limit → show n log n and n counts for contrast → give the concrete techniques that turn a quadratic into n log n/n, using a canonical example.

Why O(n²) times out — the arithmetic

At n = 10^5:

CODE
n^2 = (10^5)^2 = 10^{10} operations

Divide by the budget: 10^{10} / 10^8 = 100. So an O(n^2) algorithm does roughly 10 billion operations, taking on the order of ~10–100 seconds in C++ (and minutes in Python). Against a 1–2 s limit, that's 50–100× over budget — a guaranteed TLE, even before accounting for cache misses or per-operation overhead that only make it worse.

Contrast the two viable targets at the same n:

CODE
O(n log n) = 10^5 * log2(10^5) ≈ 10^5 * 17 ≈ 1.7·10^6 operations  (~0.02 s)
O(n)       = 10^5                = 1·10^5   operations             (~0.001 s)

So O(n log n) uses under 2% of the budget and O(n) under 0.1%. The gap between n^2 and n log n here is a factor of ~6000× — this is not a "tune the constant" difference, it's a different algorithm class. That's the whole point: at n = 10^5 the constraint is telling me the intended solution is O(n log n) or O(n), occasionally O(n √n) (≈ 3·10^7, still fine) or O(n log^2 n).

A useful rule of thumb: O(n^2) stays safe only up to about n ≈ 5000 (2.5·10^7 ops). Every additional factor of 10 in n multiplies quadratic work by 100, so from 5000 to 10^5 (20×) the work grows ~400×, pushing it far past the limit.

What to aim for — and how to get there

Target: O(n log n), ideally O(n). The recurring toolkit that collapses a naive O(n^2) double loop:

Sorting + two pointers / sliding window — replaces "for each pair" with one linear sweep after an O(n log n) sort. (pair-sum, closest pair on a line, interval merging.)
Hashing / hash map — trade the inner search loop for O(1) lookups. (two-sum, counting, dedup → O(n) average.)
Prefix sums / difference arrays — answer range aggregates in O(1) after O(n) preprocessing instead of re-summing.
Binary search (on the array or on the answer) — turns an O(n) inner search into O(log n), giving O(n log n).
Balanced BST / heap / ordered set / segment tree / BIT — maintain a running structure for O(log n) per element.
Monotonic stack/queue — next-greater-element, max-in-window in amortized O(n).

Canonical example — "does any pair sum to target?" The brute force is the classic O(n^2):

PYTHON
for i in range(n):
    for j in range(i+1, n):
        if a[i] + a[j] == target: return True

At n = 10^5 that's ~5·10^9 pair checks → TLE. Two rewrites fix it:

Hash set, O(n): one pass, for each x check if target - x was seen.
Sort + two pointers, O(n log n): sort once, converge lo/hi pointers in a linear sweep.

Both drop from 10^{10} down to ~10^6, i.e., from ~10 s to ~10 ms.

Caveats a strong candidate adds
The 10^8/sec figure is a calibrated estimate, not a law. Simple integer ops on cache-friendly arrays can hit 10^9/s; heavy STL/map/modular work can drop to 10^7/s. So O(n^2) at n = 10^5 is robustly TLE (100× margin), but n √n or n log^2 n cases need judgment, not a table lookup.
Watch the fine print: if there are t test cases, the real bound is usually "sum of n ≤ 10^5"; and a_i up to 10^9 means no 10^9-sized array (use a map/compression) even though n is small.
Sometimes O(n^2) is genuinely intended — but only when the constraint is small (n ≤ 2000–5000). At n = 10^5 it never is.
Memory can bind before time: an O(n^2) table at n = 10^5 is 10^{10} cells — impossible in any normal memory limit — so quadratic space is out too.
Summary — what a strong answer emphasizes
O(n^2) at n = 10^5 = 10^{10} ops ≈ 10–100 s vs a 1–2 s limit → 50–100× over budget → TLE.
Aim for O(n log n) (~1.7·10^6 ops, ~0.02 s) or O(n) (~10^5 ops); O(n √n) is a fine fallback.
Get there with sort+two-pointers, hashing, prefix sums, binary search, heaps/BIT/segment trees, monotonic stacks.
The reasoning is order-of-magnitude and language-adjusted; also check sum-of-n, value range, and memory.

#### Common mistakes
quoting "n² is slow" without the number; forgetting Python is ~10× slower; missing the sum-of-n line; introducing O(n^2) space to fix O(n^2) time.

#### Follow-up
probes: "Name the data structure that realizes your O(n log n) here." / "At what n would you accept O(n^2)?" / "What if the values, not the count, are up to 10^9?"

**Comparison:**

```
Complexity	Ops at n=10^5	~Wall clock (compiled, 1e8/s)	Verdict vs 1–2 s
O(n²)	1.0·10^10	~10–100 s	TLE (50–100× over)
O(n√n)	~3.2·10^7	~0.3 s	OK
O(n log n)	~1.7·10^6	~0.02 s	OK
O(n)	1.0·10^5	~0.001 s	OK
Naive O(n²) pattern	O(n log n)/O(n) replacement
for each pair (double loop)	sort + two pointers / sliding window
inner linear search	hash map/set (O(1) lookup)
recompute range sums	prefix sums / difference array
inner "find position"	binary search on array or answer
running min/max/order stats	heap / ordered set / segment tree / BIT
next-greater / window max	monotonic stack / deque
```

**Code:**

```python
# Same problem, three complexities. At n=1e5 only the last two pass.
def pair_sum_bruteforce(a, target):     # O(n^2)  -> ~5e9 checks -> TLE at n=1e5
    n = len(a)
    for i in range(n):
        for j in range(i + 1, n):
            if a[i] + a[j] == target:
                return True
    return False

def pair_sum_hashset(a, target):        # O(n)    -> ~1e5 ops   -> OK
    seen = set()
    for x in a:
        if target - x in seen:
            return True
        seen.add(x)
    return False

def pair_sum_two_pointers(a, target):   # O(n log n) sort + O(n) sweep -> OK
    a = sorted(a)
    lo, hi = 0, len(a) - 1
    while lo < hi:
        s = a[lo] + a[hi]
        if s == target: return True
        if s < target:  lo += 1
        else:           hi -= 1
    return False
```

**Example input / output:**

```

n = 10^5, time limit ~1-2 s, budget ~1e8 ops/sec:

O(n^2) : (1e5)^2 = 1.0e10 ops -> ~10-100 s -> TLE (50-100x over) O(n log n) : 1e5 * 17 = 1.7e6 ops -> ~0.02 s -> OK O(n) : 1e5 = 1.0e5 ops -> ~0.001 s -> OK

Speedup from O(n^2) to O(n log n) at n=1e5: 1.0e10 / 1.7e6 ~= 6000x

CODE
Copy

Concrete run of the example (`a = [2, 7, 11, 15]`, `target = 9`):

pair_sum_bruteforce -> True (2 + 7) pair_sum_hashset -> True (sees 2, then 7 needs 9-7=2 in set -> hit) pair_sum_two_pointers-> True (sorted [2,7,11,15]; lo=2,hi=15=17>9 -> hi--; 2+11=13>9 -> hi--; 2+7=9 -> True)
```

**Flow:**

1. No, ~50-100x over
2. n = 10^5, O(n^2) candidate
3. ops = n^2 = 1e10
4. seconds = 1e10 / 1e8 = ~100 s
5. <= 1-2 s limit?
6. TLE: reject O(n^2)
7. Target O(n log n) (~1.7e6) orO(n) (~1e5)
8. Pick technique
9. sort + two pointers
10. hash map / set
11. prefix sums
12. binary search
13. heap / segtree / BIT

