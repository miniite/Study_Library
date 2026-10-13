---
type: "practice"
difficulty: "Intermediate"
---

# Given a list of numbers, find all pairs whose difference equals a given value k.

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Merge Sort and Counting Inversions](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Merge_Sort_and_Counting_Inversions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

What the question is really asking: Given a list of numbers and a target k, find all pairs (a, b) such that their difference equals k, i.e. a - b == k (equivalently |a - b| == k if unsigned). The interviewer is probing whether you can turn an O(n²) double loop into an O(n) hash-based (or O(n log n) sorted two-pointer) solution, and — critically — whether you handle the messy details correctly: duplicates, the k = 0 special case, and what "a pair" means (distinct values vs. distinct indices, ordered vs. unordered, and de-duplication of the output). Getting the algorithm right but botching k=0 or duplicate handling is the classic failure here.

Clarifying questions a strong candidate asks:

Is k the signed difference a - b = k, or the absolute difference |a - b| = k? (I'll solve a - b = k and note the symmetry.)
Are duplicates allowed in the input, and should the output contain distinct value-pairs or all index-pairs? I'll assume distinct value-pairs (most common phrasing).
Can k be 0 or negative? k = 0 means "find values that appear at least twice"; that needs special care.
Should each pair appear once (e.g., (5,3) but not also (3,5))?

Plan: Give the brute-force baseline, then the hash-set O(n) solution reframing the problem as "for each x, does x - k exist?", carefully handle k = 0 and duplicates, and offer the sorted two-pointer alternative for the O(1)-extra-space / sorted-output case. Then trade-offs and edge cases.

Step 1 — Brute force (baseline)

Nested loop over all ordered pairs (i, j), i != j; emit when arr[i] - arr[j] == k. This is O(n²) time, O(1) extra space. Correct but slow — state it, then improve.

Step 2 — Reframe for the hash approach (the key insight)

a - b == k ⟺ for a chosen larger element a, the partner is b = a - k. So the problem becomes membership queries: for each number x in the list, is x - k also present? If yes, (x, x - k) is a valid pair (difference exactly k). One pass to build a hash set of all values, a second pass (or the same pass) to test x - k. This is O(n) time, O(n) space.

Step 3 — Handle the two traps

Trap 1 — k = 0. Now b = a - 0 = a, so we're looking for values that occur at least twice. A plain set of distinct values would falsely report every element as its own partner. Fix: use a frequency map/Counter. For k == 0, emit x (as pair (x, x)) only when count[x] >= 2.

Trap 2 — duplicates and output de-duplication. If the input is [1, 5, 5, 9] with k = 4, the pair (5, 1) and (9, 5) should each appear once, not once per duplicate 5. Using a set of distinct values for the lookup and collecting results into a set of pairs (or iterating over distinct keys of the Counter) naturally de-duplicates. Decide up front whether the interviewer wants distinct value-pairs (usual) or every index combination.

Trap 3 — negative / signed k. a - b == k is directional. If they want |a - b| == k and k != 0, that's just the union of solving for +k and -k, but since (a,b) with a-b=k is the same unordered pair as (b,a) with b-a=-k, solving a - k in set for every distinct a already yields every unordered pair once. So I standardize on: iterate distinct values x, check x - k in the set (for k != 0), which gives each pair exactly once.

Step 4 — Two-pointer alternative (sorted input / low extra space)

Sort the array (O(n log n)), then use two pointers i and j. Advance them to make arr[j] - arr[i] approach k:

if arr[j] - arr[i] == k and i != j: record pair, advance both (skipping duplicates);
if the difference is < k: advance j (increase the gap);
if > k: advance i (decrease the gap).

This gives sorted, de-duplicated pairs in O(n log n) time and O(1) extra space (ignoring sort buffer). Keep i != j so a single element isn't paired with itself when k > 0. It's the better answer when the caller wants sorted output or forbids extra memory; the hash approach wins on raw time when the input is unsorted and memory is fine.

Edge cases checklist
Empty or single-element list → no pairs.
k < 0 → equivalent to solving +|k| on swapped roles; my "check x - k" formulation still works, but I'd normalize to k = abs(k) if the intent is absolute difference.
Duplicates with k = 0 → require count ≥ 2.
Very large values → hashing handles them; a bitmap/index trick does not (unlike the "1..n" duplicate problem).
Complexity summary

#### Approach
Time	Extra space	Output order	Best when
Brute force	O(n²)	O(1)	input order	never (baseline)
Hash set / Counter	O(n)	O(n)	unsorted	unsorted input, memory ok
Sort + two pointers	O(n log n)	O(1)*	sorted	need sorted / minimal space

*ignoring the sort's internal buffer.

What a strong answer emphasizes

Reframe a - b = k as "does x - k exist?", reach O(n) with a hash set, and explicitly call out the k = 0 (needs a frequency map, count ≥ 2) and duplicate/output-dedup handling — that's what separates a senior answer from a textbook one. Offer the two-pointer variant for sorted/low-memory needs. Common mistakes: pairing an element with itself when k = 0, emitting each pair twice, or using a distinct-value set (not a Counter) and missing the k = 0 case.

**Comparison:**

```
Approach	Time	Extra space	Output order	Handles k=0
Brute force (nested loop)	O(n²)	O(1)	input	yes (i≠j check)
Hash / Counter	O(n)	O(n)	unsorted	yes (count ≥ 2)
Sort + two pointers	O(n log n)	O(1)*	sorted	yes (i≠j, dup-skip)
```

**Code:**

```python
from collections import Counter
from typing import List, Tuple

# Approach A: hash / Counter — O(n) time, O(n) space.
# Returns DISTINCT unordered value-pairs (a, b) with a - b == k, k >= 0.
def pairs_with_diff(nums: List[int], k: int) -> List[Tuple[int, int]]:
    k = abs(k)                 # treat as absolute difference
    freq = Counter(nums)
    result = []
    if k == 0:
        # pairs (x, x): value must occur at least twice
        for x, c in freq.items():
            if c >= 2:
                result.append((x, x))
    else:
        for x in freq:         # iterate DISTINCT values -> auto de-dup
            if (x - k) in freq:
                result.append((x, x - k))   # x is the larger element
    return result

# Approach B: sort + two pointers — O(n log n) time, O(1) extra space.
def pairs_with_diff_sorted(nums: List[int], k: int) -> List[Tuple[int, int]]:
    k = abs(k)
    a = sorted(nums)
    res, i, j, n = [], 0, 1, len(nums)
    while j < n:
        diff = a[j] - a[i]
        if i != j and diff == k:
            res.append((a[j], a[i]))
            lo = a[i]
            # skip duplicates of both values to keep pairs distinct
            while i < n and a[i] == lo: i += 1
            while j < n and a[j] == lo + k: j += 1
        elif diff < k:
            j += 1
        else:                  # diff > k
            i += 1
            if i == j:
                j += 1
    return res

if __name__ == "__main__":
    print(pairs_with_diff([1, 5, 3, 4, 2], 2))     # (3,1),(5,3),(4,2) etc.
    print(pairs_with_diff([1, 1, 2, 3], 0))        # [(1, 1)]
```

**Example input / output:**

```text
Example 1  (k = 2)
  input : nums = [1, 5, 3, 4, 2], k = 2
  logic : for each distinct x, is (x - 2) present?
          5->3 yes, 4->2 yes, 3->1 yes, 2->0 no, 1->-1 no
  output: [(5,3), (4,2), (3,1)]   # each pair once

Example 2  (k = 0, duplicates)
  input : nums = [1, 1, 2, 3], k = 0
  logic : Counter = {1:2, 2:1, 3:1}; only value with count>=2 is 1
  output: [(1, 1)]

Example 3  (no pair)
  input : nums = [1, 2, 3], k = 10
  output: []
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. nums, k
6. k = abs(k)
7. k == 0 ?
8. Counter freq; emit x,x wherecount[x] >= 2
9. Build set/Counter of values
10. For each DISTINCT x
11. x - k in set ?
12. record pair x, x-k
13. skip
14. next x
15. return pairs

