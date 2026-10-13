---
type: "practice"
difficulty: "Basic"
---

# Order these from fastest to slowest growing: O(1), O(log n), O(n), O(n log n), O(n^2), O(2^n), O(n!).

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Big-O vs Constant Factors](<../../../../08_Algorithms_and_Data_Structures/Core/Complexity_Analysis/Big-O_vs_Constant_Factors.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

The question asks me to order seven common Big-O growth rates from fastest to slowest growing: O(1), O(log n), O(n), O(n log n), O(n²), O(2ⁿ), O(n!). This is a "basic concept" question, so the interviewer wants to confirm two things:

Do you know the canonical ordering cold (and can you not confuse "fast to run" with "fast to grow")?
Can you attach a real algorithm to each class so the ordering isn't just memorized symbols?

One important clarifying point about wording: "fastest to slowest growing" means ordering by how quickly the running time increases as n gets large. O(1) grows the slowest (it doesn't grow at all), and O(n!) grows the fastest. This is the opposite of "which algorithm is fastest to finish" — the slowest-growing class is the best-performing one. I'll answer for "growth rate" and call out both readings so there's no ambiguity.

The correct ordering

Slowest-growing (best) → fastest-growing (worst):

O(1) < O(log n) < O(n) < O(n log n) < O(n²) < O(2ⁿ) < O(n!)

So, ordered by growth rate, fastest-growing first (as literally asked):

O(n!) > O(2ⁿ) > O(n²) > O(n log n) > O(n) > O(log n) > O(1)

A quick mnemonic for the "good → bad" order: "constant, logarithmic, linear, linearithmic, quadratic, exponential, factorial."

Why this order — from first principles

Big-O describes how the number of operations scales as input size n grows. Let me walk each rung and prove the ordering by seeing what each does as n goes from 10 to 1,000,000.

O(1) — constant. Work is independent of n. Array index lookup arr[i], pushing to a stack, a hash-map get. Whether n is 10 or a billion, it's the same fixed number of steps. Grows the slowest — it doesn't grow at all.

O(log n) — logarithmic. Each step halves the remaining problem, so doubling n adds only one more step. Binary search, or descending a balanced BST. For n = 1,000,000, log₂ n ≈ 20. Barely grows.

O(n) — linear. Work is proportional to n. One pass over an array, a linear scan, finding a max. Double n → double the work.

O(n log n) — linearithmic. n linear work, each carrying a log n factor. The complexity of efficient comparison sorts (merge sort, heap sort, average quicksort) and many divide-and-conquer algorithms. Slightly worse than linear but still very scalable — this is the practical ceiling for "fast" large-scale algorithms.

O(n²) — quadratic. Work grows with the square of n — typically nested loops over the same data. Bubble/insertion/selection sort, comparing all pairs, a naive 2-sum. Double n → 4× the work. Fine for n in the thousands, painful beyond ~10⁴–10⁵.

O(2ⁿ) — exponential. Work doubles with each +1 to n — the size of a binary recursion tree or the count of all subsets. Naive recursive Fibonacci, subset/power-set enumeration, brute-force 0/1 knapsack. Only usable for tiny n (roughly ≤ 25–30).

O(n!) — factorial. Work equals the number of orderings of n items. Generating all permutations, brute-force Travelling Salesman. Grows the fastest of all — dies around n ≈ 11–13.

Seeing it with numbers

Nothing sells the ordering like a table. For n = 1,000,000:

O(1) → 1 operation
O(log n) → ~20
O(n) → 1,000,000
O(n log n) → ~20,000,000
O(n²) → 1,000,000,000,000 (10¹²) — already ~2.8 hours at 10⁸ ops/s
O(2ⁿ) and O(n!) at n = 1,000,000 are astronomically beyond the number of atoms in the universe — which is exactly why they're only ever used for very small n.

Notice the enormous jumps: log n and n are close, n and n log n are close, but n² explodes, and the last two leave the realm of feasibility entirely. That "explosion at the top" is the ordering.

Where the boundary matters in practice

The line between n log n and n² is the one you feel most at work: choosing merge sort over bubble sort, or a hash-set lookup (O(1)) inside one loop (total O(n)) instead of a nested scan (O(n²)). The line between polynomial (n² and below) and exponential/factorial (2ⁿ, n!) is the line between "solvable at scale" and "needs approximation, DP, or pruning."

Summary — what a strong answer emphasizes
The order is: O(1) < O(log n) < O(n) < O(n log n) < O(n²) < O(2ⁿ) < O(n!) (slowest-growing/best → fastest-growing/worst).
"Fastest to slowest growing" = worst to best performing, so it reverses: n! → 2ⁿ → n² → n log n → n → log n → 1.
Anchor each class to a concrete algorithm (hash lookup, binary search, linear scan, merge sort, nested loops, subsets, permutations).
Note the practical cliffs: n log n is the scalable ceiling; n² breaks in the ~10⁵ range; 2ⁿ/n! are only for tiny n.

#### Common mistakes
/ follow-ups: reversing the wording (calling O(1) "fastest growing"); forgetting n log n sits strictly between n and n²; thinking n! and 2ⁿ are similar (they're not — n! dominates for n ≥ 4); and mixing up growth rate with actual wall-clock speed at small n (constants matter when n is small, but Big-O is about the trend as n → ∞).

**Comparison:**

```
Rank (growth)	Complexity	Name	Example algorithm	Double n → work ×
1 (fastest grow)	O(n!)	Factorial	All permutations, brute TSP	(2n)!/n! blow-up
2	O(2ⁿ)	Exponential	Subsets, naive Fibonacci	squares (×2ⁿ)
3	O(n²)	Quadratic	Nested loops, bubble sort	×4
4	O(n log n)	Linearithmic	Merge/heap/quick sort	slightly >2×
5	O(n)	Linear	Single scan, find max	×2
6	O(log n)	Logarithmic	Binary search, balanced BST	+1 step
7 (slowest grow)	O(1)	Constant	Array index, hash get	unchanged
```

**Code:**

```python
# One concrete algorithm per complexity class (fastest-growing at top)

def all_permutations(a):          # O(n!)  -- factorial
    if len(a) <= 1: return [a]
    return [[a[i]] + p for i in range(len(a))
            for p in all_permutations(a[:i] + a[i+1:])]

def subsets(a):                   # O(2^n) -- exponential
    out = [[]]
    for x in a:
        out += [s + [x] for s in out]
    return out

def has_duplicate_pair(a):        # O(n^2) -- quadratic (nested loops)
    for i in range(len(a)):
        for j in range(i + 1, len(a)):
            if a[i] == a[j]: return True
    return False

def merge_sort(a):                # O(n log n) -- linearithmic
    if len(a) <= 1: return a
    m = len(a) // 2
    l, r = merge_sort(a[:m]), merge_sort(a[m:])
    res, i, j = [], 0, 0
    while i < len(l) and j < len(r):
        if l[i] <= r[j]: res.append(l[i]); i += 1
        else:            res.append(r[j]); j += 1
    return res + l[i:] + r[j:]

def find_max(a):                  # O(n) -- linear
    m = a[0]
    for x in a: m = max(m, x)
    return m

def binary_search(a, target):     # O(log n) -- logarithmic (a is sorted)
    lo, hi = 0, len(a) - 1
    while lo <= hi:
        mid = (lo + hi) // 2
        if a[mid] == target: return mid
        if a[mid] < target: lo = mid + 1
        else:               hi = mid - 1
    return -1

def first_element(a):             # O(1) -- constant
    return a[0]
```

**Example input / output:**

```text
Ordering (fastest-growing -> slowest-growing, as asked):
   O(n!) > O(2^n) > O(n^2) > O(n log n) > O(n) > O(log n) > O(1)

Ordering (best -> worst performance, common textbook form):
   O(1) < O(log n) < O(n) < O(n log n) < O(n^2) < O(2^n) < O(n!)

Operation counts as n grows:
        n = 10        n = 1,000            n = 1,000,000
O(1)       1              1                     1
O(log n)   ~3            ~10                   ~20
O(n)       10           1,000               1,000,000
O(n log n) ~33         ~10,000            ~20,000,000
O(n^2)     100        1,000,000       1,000,000,000,000 (1e12)
O(2^n)     1,024      ~1.07e301          (way beyond feasible)
O(n!)      3,628,800   (astronomical)     (astronomical)
```

**Flow:**

1. Growth rate axis (left = grows slowest/best, right = grows fastest/worst):
2. O(1) --- O(log n) --- O(n) --- O(n log n) --- O(n^2) ==== O(2^n) ==== O(n!)
3. best                                          |          exponential  factorial
4. (flat)   (barely)  (linear) (a bit more)   quadratic  <-- explosion zone -->
5. ^ practical cliff ~ n=10^5
6. '---' polynomial / tractable ---'   '==== intractable, tiny n only ===='

