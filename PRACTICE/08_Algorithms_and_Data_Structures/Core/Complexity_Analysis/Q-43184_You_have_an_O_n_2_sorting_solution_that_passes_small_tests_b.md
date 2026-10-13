---
type: "practice"
difficulty: "Intermediate"
---

# You have an O(n^2) sorting solution that passes small tests but times out on large inputs; how would you debug and fix it?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Sorting Linked Lists](<../../../../08_Algorithms_and_Data_Structures/Core/Complexity_Analysis/Sorting_Linked_Lists.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

What the question is really asking: You shipped a sort that is O(n²). It passes the tiny sample tests but times out (TLE) on large inputs. Describe a concrete, systematic debugging-and-fixing process — not "just use a faster sort," but how you'd confirm the diagnosis and then choose the right fix.

What the interviewer is probing (intermediate level): Do you reason about complexity from evidence (measure the growth), not vibes? Can you separate "wrong algorithm class" from "right algorithm, bad constant factors / accidental quadratic"? Do you know the standard fixes and their trade-offs, and can you verify the fix rather than assume it?

#### Assumptions
/ clarifications I'd state up front:

"Times out" = exceeds the time limit, so this is CPU/algorithmic, not a hang or deadlock.
I'll confirm the input size n that trips it and the limit (e.g., n = 10⁵, 1s limit → n² = 10¹⁰ ops → clearly TLE; anything above ~10⁸ simple ops per second is unsafe).
The data may or may not be already-sorted / adversarial (matters for quicksort pivot choice).

Plan: (1) reproduce and measure the scaling to confirm it's quadratic; (2) locate the quadratic — is the whole algorithm O(n²) or is there a hidden quadratic inside an otherwise-fine sort; (3) pick the correct fix (drop to O(n log n), or a linear-time non-comparison sort if applicable); (4) verify the fix restores linearithmic growth and passes.

Step 1 — Reproduce and quantify the growth (turn a hunch into evidence)

Before touching code, prove it's quadratic. Run the sort on increasing input sizes and time each:

n = 1k, 2k, 4k, 8k, 16k. If doubling n roughly quadruples the time (4×), that is the signature of O(n²). If it merely doubles, it's already linear-ish and the problem is elsewhere (I/O, GC).

This "double-the-input" test is the single most useful move: it converts a subjective "it's slow" into an empirical complexity measurement. It also tells me whether the timeout is purely the sort or something wrapping it.

Then compute the ceiling: if n = 10⁵ and the machine does ~10⁸ simple operations/sec, n² = 10¹⁰ ops ≈ 100 s — far over a 1–2 s limit. n log n = 10⁵ × 17 ≈ 1.7×10⁶ ops ≈ milliseconds. The gap is enormous, which confirms the fix must change the complexity class, not just tune constants.

Step 2 — Locate the quadratic (whole-algorithm vs. accidental)

Two very different root causes both present as O(n²):

(a) Wrong algorithm class. The sort itself is bubble/insertion/selection sort — inherently O(n²) on random data. Fix = replace with an O(n log n) sort.

(b) Accidental quadratic hiding inside otherwise-fine code — the sneaky case an interviewer loves:

Using list.remove() / del list[i] / arr.insert() inside a loop (each is O(n) → O(n²) total).
String concatenation in a loop, or building a list with + repeatedly.
Re-scanning the array to find min each pass (selection sort in disguise).
A comparator or key function that itself is O(n).
in membership tests against a list instead of a set inside the sort loop.
Quicksort that degrades to O(n²) on already-sorted / adversarial input because the pivot is always the first/last element.

I profile to find which. A quick cProfile (Python) / flamegraph / perf shows where time goes; if 95% of time is in one line doing list.remove, that's the accidental quadratic and the fix is a data-structure change, not a new sort.

Step 3 — Pick the correct fix

Now the decision:

General comparison sort → use O(n log n). Prefer the language's built-in (sorted() / Arrays.sort / std::sort) — Timsort/introsort are battle-tested, cache-friendly, and avoid quicksort's worst case (introsort falls back to heapsort; Timsort is stable and adaptive). This is the default, correct answer for "replace the O(n²) sort."
Quicksort that degraded on sorted input → randomize the pivot (or median-of-three) so worst case becomes astronomically unlikely, or switch to introsort/mergesort for a guaranteed O(n log n).
Keys are integers/ characters in a bounded range → counting sort / radix sort = O(n + k) or O(n·d), linear time, beating O(n log n). E.g., sorting 10⁷ 32-bit ints, ages 0–120, or fixed-length strings.
Accidental quadratic (case b) → fix the data structure: replace in-loop remove/insert with building a new list / using indices, replace list-membership with a set, precompute keys once (key= decorate-sort-undecorate), avoid repeated string + (use join). Sometimes the algorithm was fine; only the inner operation was quadratic.
Nearly-sorted data specifically → insertion sort is actually O(n) in that adaptive case, or Timsort exploits existing runs. Don't over-engineer if the real input is almost sorted.
n truly huge (doesn't fit in memory) → external merge sort.
Step 4 — Verify the fix (don't assume)
Re-run the double-the-input timing: time should now scale ~n log n (slightly more than doubling per doubling, not quadrupling). If it still quadruples, I fixed the wrong thing (probably a leftover accidental quadratic).
Correctness first: the output must still be a valid sort. Add property tests: output is sorted (all(a[i] <= a[i+1])), is a permutation of the input (same multiset), and matches the built-in on random and edge inputs (empty, one element, all-equal, reverse-sorted, already-sorted, with duplicates). Preserve stability if the original relied on it (Timsort/mergesort are stable; heapsort/plain quicksort are not).
Re-submit against the large test that was timing out; confirm it passes within the limit with headroom.
Summary — what a strong answer emphasizes
Measure before you fix: the doubling test proves quadratic growth and computes the ops ceiling.
Distinguish the two root causes: genuinely O(n²) algorithm vs. an accidental quadratic (in-loop remove/insert, list membership, string +, bad pivot). Profile to find which.
Fix to the right class: built-in O(n log n) for general data; counting/radix O(n+k) when keys are bounded — this is the standout answer that shows you can beat n log n; randomize the pivot for degraded quicksort.
Verify: re-measure growth, prove correctness/permutation/stability, and confirm the previously-failing large test now passes.

#### Common mistakes
jumping straight to "use quicksort" without checking that the input is what makes it quadratic (sorted input + first-element pivot = still O(n²)); replacing the sort but leaving an accidental quadratic in the surrounding loop so it still TLEs; forgetting stability and breaking a downstream tie-break; using O(n log n) when the keys are small integers and O(n) counting sort was available.

#### Follow-up
probes: "When is O(n²) acceptable?" (tiny n, or nearly-sorted with insertion sort). "How does introsort avoid quicksort's worst case?" (depth cap → heapsort). "Why might radix sort not win despite being O(n)?" (large constant, key length d, non-integer keys, cache/memory cost).

**Comparison:**

```
Root cause (both look like O(n^2))	Signature / how I detect it	Correct fix	New complexity
Genuinely quadratic sort (bubble/selection/insertion on random data)	doubling test → 4× per 2×	built-in sorted/Arrays.sort	O(n log n)
Quicksort degraded on sorted/adversarial input	fast on random, quadratic on sorted	randomize pivot / median-of-three / introsort	O(n log n) avg, guaranteed w/ introsort
Accidental: in-loop remove/insert/del	profiler shows one line dominating	use indices / rebuild list once	removes the O(n)-per-op
Accidental: x in list membership in loop	profiler → list __contains__	use a set	O(1) lookups
Accidental: string + in loop	quadratic build time	"".join(...)	linear build
Keys are bounded integers/chars	values in small range [0,k]	counting / radix sort	O(n + k) / O(n·d)
Nearly-sorted input	few inversions	insertion sort / Timsort (adaptive)	O(n) best case
```

**Code:**

```python
import time, random

# ---- Step 1: prove it's quadratic with the doubling test ----
def measure(sort_fn, sizes=(1000, 2000, 4000, 8000, 16000)):
    prev_t = None
    for n in sizes:
        data = [random.randint(0, 10**9) for _ in range(n)]
        t0 = time.perf_counter()
        sort_fn(data[:])                 # copy so each run is independent
        dt = time.perf_counter() - t0
        ratio = (dt / prev_t) if prev_t else float('nan')
        print(f"n={n:>6}  t={dt:8.4f}s  ratio_vs_prev={ratio:.2f}")
        prev_t = dt
    # ratio ~4.0 per doubling  => O(n^2);  ratio ~2.x => O(n log n)

# ---- The slow original: selection sort, O(n^2) ----
def slow_sort(a):
    for i in range(len(a)):
        m = i
        for j in range(i + 1, len(a)):     # inner scan -> quadratic
            if a[j] < a[m]:
                m = j
        a[i], a[m] = a[m], a[i]
    return a

# ---- Fix A: general data -> built-in O(n log n) (Timsort, stable) ----
def fast_sort(a):
    return sorted(a)

# ---- Fix B: keys are bounded ints -> counting sort O(n + k) ----
def counting_sort(a, k):                    # values in [0, k]
    count = [0] * (k + 1)
    for x in a:
        count[x] += 1
    out = []
    for v, c in enumerate(count):
        out.extend([v] * c)
    return out

# ---- Step 4: verify correctness (sorted + same multiset) ----
def is_valid_sort(original, result):
    from collections import Counter
    sorted_ok = all(result[i] <= result[i+1] for i in range(len(result)-1))
    perm_ok = Counter(original) == Counter(result)
    return sorted_ok and perm_ok
```

**Example input / output:**

```text
Doubling test on the ORIGINAL O(n^2) selection sort (illustrative timings):
  n=  1000  t=0.09s  ratio_vs_prev=nan
  n=  2000  t=0.36s  ratio_vs_prev=4.00   <- 2x input -> ~4x time  => O(n^2) CONFIRMED
  n=  4000  t=1.44s  ratio_vs_prev=4.00
  n=  8000  t=5.70s  ratio_vs_prev=3.96
  n= 16000  t=22.9s  ratio_vs_prev=4.02   <- would TLE at n=1e5

After Fix A (sorted / Timsort):
  n=  1000  t=0.0004s ratio=nan
  n=  2000  t=0.0009s ratio=2.25          <- ~2x, slightly more => O(n log n) CONFIRMED
  n= 16000  t=0.008s  ratio~2.2           <- comfortably within limit

Correctness check:
  original = [5, 2, 5, 1, 3];  result = sorted(original) = [1, 2, 3, 5, 5]
  is_valid_sort(original, result) -> True   (ordered AND same multiset, duplicates kept)

Counting sort (Fix B) for bounded keys, e.g. ages 0..120:
  counting_sort([3, 0, 2, 3, 1], k=3) -> [0, 1, 2, 3, 3]   (linear time)
```

**Flow:**

1. No, ~2x
2. Yes, ~4x = O n^2
3. inner compare loop
4. one line:remove/insert/in/str+
5. quicksort on sorted input
6. Yes
7. No
8. Yes
9. No
10. Sort TLEs on large input
11. Doubling test:2x input -> ~4x time?
12. Not the sort:check I/O, GC, wrapping loop
13. Profile: where is the time?
14. Keys bounded ints/chars?
15. Accidental quadraticfix data structure
16. Randomize pivot / introsort
17. Counting / Radix sort: O n+k
18. Built-in O n log n sort
19. Re-run doubling test +correctness
20. Now ~n log n and passes?
21. Ship

