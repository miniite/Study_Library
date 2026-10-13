---
type: "practice"
difficulty: "Basic"
---

# What is the difference between O(n) and O(log n), and give an algorithm example of each?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Sorting Algorithms](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Sorting_Algorithms.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

What the question is really asking: Explain the difference between two Big-O growth rates — O(n) (linear) and O(log n) (logarithmic) — in terms of how the amount of work scales with input size, and name a concrete algorithm that exhibits each. The interviewer wants intuition ("what does log n mean"), not just definitions.

What the interviewer is probing for: Do you understand Big-O as a description of growth as n → ∞? Can you explain why log n is dramatically better than n (each step eliminates a fraction of the input vs. touching every element)? Can you tie each to a real algorithm — the canonical pairing is linear search = O(n) and binary search = O(log n). A strong answer also states the precondition for the log-n algorithm (sorted input) and shows the numbers that make log n feel almost free.

#### Assumptions
/ clarifications I'd state: Big-O describes the worst-case (upper bound) growth of operations as a function of input size n, ignoring constants and lower-order terms. log n here is base-2 (base only changes the constant factor, which Big-O ignores).

From first principles to the answer

1. What Big-O measures. Big-O tells you how the number of basic operations grows as the input n grows, not the exact runtime in seconds. We drop constants and slower terms because we care about the shape of growth for large n. So O(3n + 7) = O(n).

2. O(n) — linear. The work grows in direct proportion to the input. Double n, and you do roughly double the work. Typically this means you touch every element once. If n goes from 1,000 to 1,000,000, you do about 1,000× more work.

3. O(log n) — logarithmic. log₂ n answers "how many times can I halve n before I reach 1?" The work grows very slowly: each step discards a fixed fraction (usually half) of the remaining input, so you never need to look at most of it. Doubling n adds just one more step. That's the key mental model: linear touches everything; logarithmic keeps throwing half away.

4. Why the gap is enormous. The numbers make it visceral:

n	O(n) work	O(log₂ n) work
16	16	4
1,000	1,000	~10
1,000,000	1,000,000	~20
1,000,000,000	1,000,000,000	~30

A billion-element search is ~30 steps with log n vs. a billion with linear. That's why O(log n) is considered nearly as good as constant time in practice.

5. The canonical examples.

O(n) — Linear search. To find a value in an unsorted array, you must potentially check every element; the worst case (value at the end or absent) is n comparisons → O(n). No way around it when there's no structure to exploit.
O(log n) — Binary search. To find a value in a sorted array, look at the middle element. If it matches, done. If the target is smaller, the answer can only be in the left half; if larger, the right half — so you discard half the array every comparison. Starting from n, after one step you have n/2, then n/4, … reaching 1 after log₂ n steps → O(log n). The precondition — the array must be sorted — is what lets you rule out half.

6. Other classic O(log n) examples. Search/insert/delete in a balanced binary search tree (AVL/Red-Black) is O(log n) because the tree height is log n and each step goes down one level. Binary-heap push/pop is O(log n) for the same reason (bubble up/down the height).

Nuances a strong candidate adds
The sorted precondition has a cost. Binary search is O(log n) only if the data is already sorted. Sorting first is O(n log n). So for a single lookup on unsorted data, linear search (O(n)) can be cheaper than sort-then-binary-search (O(n log n)); binary search wins when you do many lookups and amortize the one-time sort.
Base of the log doesn't matter in Big-O. log₂ n, log₁₀ n, and ln n differ only by a constant multiplier, which Big-O ignores. That's why we usually just write O(log n).
Growth rate ranking: O(1) < O(log n) < O(n) < O(n log n) < O(n²). O(log n) sits just above constant time and far below linear.
Worked example (binary search dry-run)

Sorted array [2, 5, 8, 12, 16, 23, 38, 56, 72, 91] (n = 10), search for 23:

lo=0, hi=9, mid=4 → arr[4]=16; 23 > 16 → search right, lo=5
lo=5, hi=9, mid=7 → arr[7]=56; 23 < 56 → search left, hi=6
lo=5, hi=6, mid=5 → arr[5]=23 → found in 3 comparisons

log₂ 10 ≈ 3.3, matching the ~3 steps. Linear search would have taken 6 comparisons (23 is the 6th element).

Summary — what a strong answer emphasizes

O(n) means work grows linearly — proportional to input size, because you touch each element (e.g., linear search over an unsorted array). O(log n) means work grows logarithmically — each step eliminates a fixed fraction (typically half) of what's left, so doubling n adds only one step (e.g., binary search on a sorted array, or search in a balanced BST). The practical takeaway: log n is dramatically more scalable — a billion elements is ~30 steps vs. a billion. A strong answer explains why (halving vs. scanning), names the algorithms, and states the log-n precondition (sorted / structured data).

#### Common mistakes
/ follow-ups: claiming binary search is O(log n) without noting the array must be sorted; confusing O(log n) with O(n log n); saying "log n is faster" without explaining the halving intuition. Likely follow-up: "If the array is unsorted and you search once, which is better?" → linear O(n), because sorting to enable binary search costs O(n log n).

**Comparison:**

```
	O(n) — linear	O(log n) — logarithmic
How work scales	Proportional to n (touch each element)	Add 1 step each time n doubles
Core mechanic	Scan everything	Discard half every step
Example algorithm	Linear search (unsorted array)	Binary search (sorted array), balanced BST lookup
Precondition	None	Data must be sorted / structured
Steps for n = 1,000,000	1,000,000	~20
n	O(n)	O(log₂ n)
16	16	4
1,000	1,000	~10
1,000,000	1,000,000	~20
```

**Code:**

```java
// O(n) — Linear search: may inspect every element (unsorted OK).
int linearSearch(int[] a, int target) {
    for (int i = 0; i < a.length; i++)     // up to n iterations
        if (a[i] == target) return i;
    return -1;
}

// O(log n) — Binary search: REQUIRES a sorted array; halves range each step.
int binarySearch(int[] a, int target) {   // a must be sorted ascending
    int lo = 0, hi = a.length - 1;
    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2;      // avoids overflow
        if (a[mid] == target) return mid;
        if (a[mid] < target)  lo = mid + 1; // discard left half
        else                  hi = mid - 1; // discard right half
    }
    return -1;                              // ~log2(n) iterations
}
```

**Example input / output:**

```text
arr = [2, 5, 8, 12, 16, 23, 38, 56, 72, 91]   (sorted, n=10), target = 23

Binary search (O(log n)):
  lo=0 hi=9 mid=4 -> arr[4]=16, 23>16 -> lo=5
  lo=5 hi=9 mid=7 -> arr[7]=56, 23<56 -> hi=6
  lo=5 hi=6 mid=5 -> arr[5]=23 -> FOUND at index 5  (3 comparisons ~= log2(10))

Linear search (O(n)) for same target:
  scans 2,5,8,12,16,23 -> FOUND at index 5  (6 comparisons)
```

**Flow:**

1. no
2. yes
3. yes
4. no
5. yes
6. no
7. Binary search: range lo..hi
8. lo <= hi?
9. Not found
10. mid = midpoint
11. arr[mid] == target?
12. Found
13. arr[mid] < target?
14. lo = mid+1discard LEFT half
15. hi = mid-1discard RIGHT half

