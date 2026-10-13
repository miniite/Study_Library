---
type: "practice"
difficulty: "Deep"
---

# What is the complexity of finding the k-th largest element using a heap versus sorting the whole array?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Top-k Complexity](<../../../../08_Algorithms_and_Data_Structures/Core/Complexity_Analysis/Top-k_Complexity.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

The question asks me to compare two ways of finding the k-th largest element of an array of size n: (1) a heap-based method versus (2) fully sorting the array. The interviewer wants the exact time and space complexities of each, why they differ, and the judgment to pick the right one for a given situation. This is a classic "do you actually understand what work each algorithm does?" probe — and a good candidate also brings up the third option, Quickselect, because "k-th largest" is the textbook order-statistic problem.

Clarifying assumptions:

k-th largest means the element that would sit at index k-1 if the array were sorted descending (k=1 → the maximum). Distinct vs. duplicate handling is usually "by position," not "distinct value" — I'll note both.
The array is in memory (this is not a stream — if it were, we'd keep a size-k min-heap; see the streaming variant note).
We want one query for the k-th largest, not repeated queries.

Plan: derive each method's cost from what it physically does, tabulate, then give the decision rule.

Method 1 — Sort the whole array

Sort the array (ascending or descending), then index the k-th largest directly: arr[n-k] if sorted ascending, or arr[k-1] if sorted descending.

Time: O(n log n) — that's the cost of a comparison sort (mergesort/heapsort always; quicksort on average).
Space: O(1)–O(n) depending on the sort. In-place heapsort is O(1) extra; typical library sorts (Timsort in Python/Java Arrays.sort for objects) use O(n); quicksort uses O(log n) stack.

Why this is more than we need: sorting produces a total order over all n elements. We only asked for one element's rank. We paid O(n log n) to fully order n−1 elements we don't care about. The upside: after sorting, answering the k-th largest for any k is O(1), so if you'll issue many such queries, the one-time sort amortizes well.

Method 2 — Heap

There are two standard heap approaches; be explicit about which:

2a. Min-heap of size k (the usual "heap" answer, great when k << n): Keep a min-heap holding the k largest elements seen so far. Push the first k, then for each remaining element, if it's larger than the heap root (the current k-th largest), replace the root. After processing all n, the root is the k-th largest.

Building the initial heap of k: O(k) (heapify) or O(k log k) (k pushes).
Each of the remaining n−k elements: one O(log k) compare-and-maybe-replace.
Time: O(n log k). Space: O(k). Because log k ≤ log n, this is ≤ O(n log n), and when k is small (e.g., k=5, n=10⁶), log k is essentially a small constant, so it behaves like O(n). This variant is also the one you use for a stream, since it never needs all n at once.

2b. Max-heap, extract k times (good when k is small, or you want the top-k in order): Build a max-heap of all n elements in O(n), then pop k times; the k-th pop is the k-th largest.

Time: O(n + k log n). Space: O(1) extra if built in place on the array, else O(n). When k is small this is roughly O(n); when k ≈ n it degrades toward O(n log n) (that's just heapsort).
Method 3 — Quickselect (the asymptotically best in-memory single query)

Worth naming because it dominates both for a single query. Quickselect partitions the array around a pivot (like quicksort) but recurses into only the side containing the k-th position.

Time: O(n) average, O(n²) worst (worst case avoidable with median-of-medians → deterministic O(n), or randomized pivots to make worst case astronomically unlikely).
Space: O(1)–O(log n) (in-place; recursion stack). It reorders/destroys the input.
The comparison, distilled

For a single k-th-largest query on an in-memory array:

Sorting: O(n log n) — simplest to write, but does the most unnecessary work; wins only if you also need the array sorted or will run many k-th queries.
Heap (min-heap of k): O(n log k), O(k) space — the sweet spot when k << n, and the only option for streams. Strictly beats sorting whenever k < n.
Quickselect: O(n) average — asymptotically the fastest single-query in-memory method; the "correct" interview answer when the whole array is available and you don't need it sorted afterward.

So "heap vs sorting": the heap is asymptotically better than or equal to sorting (O(n log k) ≤ O(n log n)) and uses far less space (O(k) vs up to O(n)) when k is small — and it works on streams. Sorting only pulls ahead in practice when k ≈ n (then both are O(n log n) and a tuned library sort has great constants) or when you'll answer many order-statistic queries and want O(1) lookups after a one-time sort.

Summary — what a strong answer emphasizes

State the three: sort = O(n log n)/up to O(n) space; min-heap-of-k = O(n log k)/O(k) space (and streamable); Quickselect = O(n) avg/O(1)–O(log n) space, but O(n²) worst. Explain the "why": sorting orders everything; the heap only tracks k; Quickselect only recurses toward the target rank. Give the decision rule: k << n and/or streaming → min-heap of k; whole array in memory, one query, no need to keep it sorted → Quickselect; need repeated k-th queries or a sorted array anyway → sort once. Common mistakes: quoting O(n log n) for the heap (it's O(n log k) for the size-k variant), forgetting Quickselect's worst case, or ignoring that the heap wins on space and streams, not just time.

**Comparison:**

```
Method	Time	Extra space	Modifies input?	Streamable?	Best when
Sort whole array	O(n log n)	O(1)–O(n)	yes (or copy)	no	Many k-th queries; already need sorted data; k≈n
Min-heap of size k	O(n log k)	O(k)	no	yes	k << n; single/continuous query; stream
Max-heap, pop k times	O(n + k log n)	O(1)–O(n)	yes	no	Need top-k in order, k small
Quickselect	O(n) avg / O(n²) worst	O(1)–O(log n)	yes	no	Whole array in memory, one query, best avg time

Note: O(n log k) ≤ O(n log n) always (k ≤ n); the heap ties sorting only when k ≈ n.
```

**Code:**

```python
import heapq

# Method 1: sort — O(n log n) time, O(1)-O(n) space
def kth_largest_sort(arr, k):
    return sorted(arr, reverse=True)[k - 1]

# Method 2a: min-heap of size k — O(n log k) time, O(k) space (also streamable)
def kth_largest_heap(arr, k):
    heap = arr[:k]
    heapq.heapify(heap)                 # O(k)
    for x in arr[k:]:                   # each op O(log k)
        if x > heap[0]:
            heapq.heapreplace(heap, x)
    return heap[0]                      # root = k-th largest

# Method 3: Quickselect — O(n) average, O(1)-O(log n) space
import random
def kth_largest_quickselect(arr, k):
    target = len(arr) - k              # index in ascending order
    lo, hi = 0, len(arr) - 1
    while lo < hi:
        p = partition(arr, lo, hi, random.randint(lo, hi))
        if   p == target: return arr[p]
        elif p <  target: lo = p + 1
        else:             hi = p - 1
    return arr[lo]

def partition(arr, lo, hi, pivot_idx):
    arr[pivot_idx], arr[hi] = arr[hi], arr[pivot_idx]
    pivot, store = arr[hi], lo
    for i in range(lo, hi):
        if arr[i] < pivot:
            arr[i], arr[store] = arr[store], arr[i]; store += 1
    arr[store], arr[hi] = arr[hi], arr[store]
    return store
```

**Example input / output:**

```text
arr = [3, 2, 1, 5, 6, 4], k = 2   (2nd largest -> 5)

kth_largest_sort:  sorted desc = [6,5,4,3,2,1]; index k-1=1 -> 5  ✓

kth_largest_heap (min-heap of size k=2):
  heap = [3,2] -> heapify -> [2,3]
  x=1: 1>root(2)? no
  x=5: 5>root(2)? yes -> replace -> [3,5]
  x=6: 6>root(3)? yes -> replace -> [5,6]
  x=4: 4>root(5)? no
  root = 5  ✓

kth_largest_quickselect: target = 6-2 = 4; partitions until arr[4]=5 -> 5  ✓
```

**Flow:**

1. yes
2. no
3. yes
4. no
5. Need k-th largest
6. Stream / must keep memoryO(k)?
7. Min-heap of size k: O(n log k),O(k)
8. Many k-th queries or needarray sorted anyway?
9. Sort once: O(n log n), thenO(1) per query
10. Quickselect: O(n) avg, in place

