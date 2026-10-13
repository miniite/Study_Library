---
type: "practice"
difficulty: "Intermediate"
---

# What are the best, worst, and average case complexities of quicksort?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Sorting Linked Lists](<../../../../08_Algorithms_and_Data_Structures/Core/Complexity_Analysis/Sorting_Linked_Lists.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

The question asks me to state and justify the best, worst, and average case time complexities of quicksort. On the surface it's a one-line factual recall (O(n log n), O(n²), O(n log n)), but at an intermediate level the interviewer is really probing whether I understand why each case arises, what input or pivot choice triggers it, and the space cost that people usually forget. A strong answer connects the complexity to the mechanics of partitioning and the recursion tree depth, and mentions space and stability.

Clarifying assumptions: classic Lomuto/Hoare partition quicksort, comparison-based, sorting n elements. Unless stated otherwise, "quicksort" means the naive version where the pivot is chosen deterministically (e.g., first or last element). I'll note how randomization changes the story.

Plan: (1) recall how quicksort works so the complexity is derivable, not memorized; (2) derive each case from the recurrence / recursion-tree depth; (3) give the space complexity; (4) summarize in a table and note the practical fixes.

How quicksort works (so the complexity is derivable)

Quicksort is divide-and-conquer:

Pick a pivot from the array.
Partition: rearrange so all elements < pivot are left of it and all > pivot are right of it. The pivot lands in its final sorted position. Partitioning one subarray of size m costs Θ(m) comparisons — every element is compared once against the pivot.
Recurse on the left and right partitions.

So the total work is: Θ(n) of partition work per "level" of recursion, multiplied by the number of levels (the depth of the recursion tree). The whole complexity question reduces to: how deep is the recursion tree, and how much total partitioning happens at each depth?

Best case — Θ(n log n)

The best case happens when every partition splits the array into two (near-)equal halves. The pivot lands right in the middle each time.

Level 0: one problem of size n → Θ(n) partition work.
Level 1: two problems of size n/2 → total Θ(n).
Level 2: four problems of size n/4 → total Θ(n).
... and so on.

Because we halve the size each level, the tree has log₂ n levels, and each level does Θ(n) total work. So total = Θ(n) × log n = Θ(n log n).

Formally the recurrence is T(n) = 2T(n/2) + Θ(n), which by the Master Theorem gives Θ(n log n).

Worst case — O(n²)

The worst case happens when every partition is maximally unbalanced — one side gets n-1 elements and the other gets 0 (the pivot is the smallest or largest element every time).

The classic trigger: a first- or last-element pivot on already-sorted (or reverse-sorted) input. Each partition peels off exactly one element, so the recursion tree becomes a chain of depth n instead of log n.

Level 0: partition size n → n work.
Level 1: size n-1 → n-1 work.
... down to size 1.

Total work = n + (n-1) + (n-2) + ... + 1 = n(n+1)/2 = Θ(n²).

The recurrence here is T(n) = T(n-1) + Θ(n), which solves to Θ(n²).

This is quicksort's famous weakness: on the exact input you'd naively test with (a sorted array), a naive first-element pivot degrades to O(n²).

Average case — Θ(n log n)

For a random permutation of the input, the pivot lands "somewhere in the middle" on average, so most splits are reasonably balanced. Even a lopsided-but-constant split like 1:9 (10%/90%) still gives a tree of depth Θ(log n) (log base 10/9), so it's still Θ(n log n).

The rigorous average-case analysis counts the expected number of comparisons over all n! input orderings. Element i and element j (in sorted order) are compared iff one of them is picked as pivot before any element between them; the probability of that is 2/(j - i + 1). Summing over all pairs gives an expected comparison count of 2n·Hₙ ≈ 1.39 n log₂ n, i.e. Θ(n log n). So on average quicksort is n log n with a small constant — which, combined with excellent cache behavior and in-place operation, is why it's often faster in practice than merge sort despite the same asymptotic average.

Space complexity (the commonly forgotten part)

Quicksort partitions in place, so it uses only O(1) auxiliary storage for the partition step itself. The real space cost is the recursion call stack:

Best/average case: stack depth O(log n).
Worst case: stack depth O(n) (the chain).

A production implementation avoids the O(n) stack blow-up by recursing into the smaller partition and looping on the larger one (tail-call elimination), which caps stack depth at O(log n) even in the worst case.

How to avoid the worst case (what the interviewer wants to hear)

The O(n²) worst case is a pivot-choice problem, not an inherent one:

Randomized pivot (or random shuffle first): makes worst case exponentially unlikely; expected time Θ(n log n) for any input.
Median-of-three pivot: kills the sorted-input worst case cheaply.
Introsort (used by C++ std::sort): runs quicksort but switches to heapsort once recursion depth exceeds ~2 log n, guaranteeing O(n log n) worst case while keeping quicksort's average-case speed.
Summary — what a strong answer emphasizes
Best: Θ(n log n) (balanced splits). Worst: O(n²) (maximally unbalanced, e.g. sorted input + naive pivot). Average: Θ(n log n).
The whole story is recursion depth × Θ(n) per level: depth log n gives n log n, depth n gives n².
Space is O(log n) stack (avg) / O(n) (worst), O(1) extra otherwise; in-place, not stable.
The worst case is fixable via randomized/median-of-three pivots or introsort — real libraries never actually hit O(n²).

#### Common mistakes
saying the average is O(n²) (it's not — worst ≠ average); forgetting quicksort's space is the call stack; claiming it's stable (it isn't); not connecting the complexity to pivot choice / partition balance.

**Comparison:**

```
Case	Condition that triggers it	Recurrence	Time	Aux space (stack)
Best	Every partition splits ~50/50	T(n)=2T(n/2)+Θ(n)	Θ(n log n)	O(log n)
Average	Random input / random pivot	expected ≈1.39 n log₂ n	Θ(n log n)	O(log n)
Worst	Maximally unbalanced split (sorted input + naive pivot)	T(n)=T(n-1)+Θ(n)	Θ(n²)	O(n) (or O(log n) with smaller-first recursion)
Property	Quicksort
In place	Yes (O(1) extra beyond stack)
Stable	No
Worst-case guarantee	O(n²) naive; O(n log n) with introsort
```

**Code:**

```
In-place quicksort (Lomuto partition) — the version whose worst case is O(n^2)

def quicksort(a, lo=0, hi=None): if hi is None: hi = len(a) - 1 if lo >= hi: return p = partition(a, lo, hi) # Theta(hi - lo) comparisons at this level quicksort(a, lo, p - 1) quicksort(a, p + 1, hi)

def partition(a, lo, hi): pivot = a[hi] # last-element pivot -> O(n^2) on sorted input i = lo - 1 for j in range(lo, hi): if a[j] < pivot: i += 1 a[i], a[j] = a[j], a[i] a[i + 1], a[hi] = a[hi], a[i + 1] return i + 1

CODE
Copy

```python
import random
# Randomized pivot: expected Theta(n log n) for ANY input, avoids the sorted-input worst case
def partition_rand(a, lo, hi):
    r = random.randint(lo, hi)
    a[r], a[hi] = a[hi], a[r]   # swap random pivot to the end, then Lomuto as above
    return partition(a, lo, hi)
```

**Example input / output:**

```

Worst-case trace (last-element pivot on sorted input) — every partition peels off ONE element:

CODE
Copy
Input: [1, 2, 3, 4, 5]   (already sorted, pivot = last element)

partition [1,2,3,4,5], pivot=5 -> pivot ends at index 4; recurse on [1,2,3,4]   (size 4)
partition [1,2,3,4],   pivot=4 -> pivot ends at index 3; recurse on [1,2,3]     (size 3)
partition [1,2,3],     pivot=3 -> pivot ends at index 2; recurse on [1,2]       (size 2)
partition [1,2],       pivot=2 -> pivot ends at index 1; recurse on [1]         (size 1)

Comparisons: 4 + 3 + 2 + 1 = 10 = n(n-1)/2  ->  Theta(n^2)

Best-case trace (balanced splits):

CODE
Copy
Input size n=8, each pivot lands in the middle:
  size 8  -> two size-3/size-4 halves       (level work ~ 8)
  size 4  -> two size-1/size-2 halves        (level work ~ 8 total across nodes)
  ...
  depth = log2(8) = 3 levels, each ~n work  ->  8 * 3 = Theta(n log n)
```

**Flow:**

1. yes
2. no
3. balanced - depth log n - nlog n
4. unbalanced n-1/0 - depth n- n^2
5. quicksort(array)
6. size <= 1?
7. return (sorted)
8. pick pivot
9. partition: Theta(size)comparisons
10. recurse LEFT (< pivot)
11. recurse RIGHT (> pivot)
12. BEST/AVG
13. WORST

