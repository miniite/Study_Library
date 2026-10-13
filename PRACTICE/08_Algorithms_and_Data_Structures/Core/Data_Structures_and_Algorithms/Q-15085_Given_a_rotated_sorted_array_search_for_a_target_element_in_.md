---
type: "practice"
difficulty: "Deep"
---

# Given a rotated sorted array, search for a target element in O(log n).

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Interval Merging](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Interval_Merging.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

What is being asked: You are given a sorted array that has been rotated at some unknown pivot (e.g., [0,1,2,4,5,6,7] rotated becomes [4,5,6,7,0,1,2]). Find the index of a target value in O(log n) time. The O(log n) constraint rules out a linear scan and forces a modified binary search.

What the interviewer is probing: Do you realize that rotation breaks the "array is globally sorted" precondition that plain binary search relies on, but that at least one half of any midpoint split is still sorted? Can you exploit that to decide which half to keep, and — crucially — get all the boundary comparisons (<= vs <) exactly right? This is a classic where the idea is easy but the implementation is bug-prone, so they watch your boundary reasoning closely.

Clarifying questions:

Are elements distinct? (I'll solve distinct first — clean O(log n) — then note that duplicates degrade the worst case to O(n).)
Return the index or a boolean found/not-found? (I'll return the index, -1 if absent.)
Could the array be empty or rotated by 0 (i.e., not rotated)? (Yes — my code handles both; a non-rotated array is just the fully-sorted case, which the same logic covers.)
Why plain binary search doesn't directly work

Plain binary search needs the whole array sorted so it can compare the target to arr[mid] and discard half. After rotation the array is only piecewise sorted — it has one "drop" point (the pivot). So arr[mid] < target no longer tells you globally which side to go.

Key insight: one half is always sorted

Take any split at mid. The pivot (the single drop) can lie in only one of the two halves [lo..mid] or [mid..hi]. Therefore the other half is fully sorted. We can detect which half is sorted with a single comparison, and within that sorted half we can decide precisely whether the target lies inside it (because a sorted range has known min/max = its endpoints).

Concretely, at each step compute mid. If arr[mid] == target, done. Otherwise:

If arr[lo] <= arr[mid] → the left half [lo..mid] is sorted.
If arr[lo] <= target < arr[mid], the target must be in the left half → search left (hi = mid - 1).
Else search right (lo = mid + 1).
Else → the right half [mid..hi] is sorted.
If arr[mid] < target <= arr[hi], the target must be in the right half → search right (lo = mid + 1).
Else search left (hi = mid - 1).

Each iteration halves the search space, so it's O(log n) time, O(1) space (iterative). Every decision is exact because we always test membership against a sorted range whose bounds we know.

The boundary details that make or break it
The arr[lo] <= arr[mid] uses <=, not <. When lo == mid (a 2-element window), we must treat the single-element left side as "sorted" so we don't misclassify. Using < there causes off-by-one bugs.
Inside the sorted left half, the membership test is arr[lo] <= target && target < arr[mid] — inclusive on lo (the target could equal the left endpoint) and exclusive on mid (we already checked arr[mid] != target). Mirror logic on the right: arr[mid] < target && target <= arr[hi].
The loop condition is while (lo <= hi) with mid = lo + (hi - lo) / 2 (the lo + (hi-lo)/2 form avoids integer overflow on large indices, which a senior interviewer notices).
Correctness argument

Invariant: if the target exists, it is always within the current [lo, hi] window. At each step we identify a fully sorted half and test — using that half's exact endpoints — whether the target could be inside it. If yes, we keep that half; if no, the target (if present) must be in the other half, so we keep that. We never discard a window that could contain the target, and the window strictly shrinks each step, so the loop terminates and returns the correct index or -1.

Complexity and the duplicates caveat
Distinct elements: O(log n) time, O(1) space (iterative). A recursive version is O(log n) stack depth.
With duplicates: the check arr[lo] <= arr[mid] can become ambiguous when arr[lo] == arr[mid] == arr[hi] (e.g., [1,1,1,0,1]) — you can't tell which half is sorted. The standard fix is: when arr[lo] == arr[mid] == arr[hi], shrink both ends by one (lo++, hi--) and retry. This makes the worst case O(n) (imagine all-equal-except-one), because you may only peel one element at a time. This is an important nuance to state — many candidates claim O(log n) even with duplicates, which is wrong.
Alternative (two-step) approach

You could instead find the pivot index first via binary search (the smallest element), then binary-search the correct sorted segment. That's also O(log n) but two passes and more code. The single-pass "which half is sorted" method above is cleaner and generally preferred in interviews, so I'd lead with it and mention the pivot approach as an alternative.

#### Common mistakes
/ follow-up probes
Using < instead of <= in the sorted-half test → fails on 2-element windows and when the array isn't actually rotated.
Comparing target only against arr[mid] (treating it like plain binary search) → wrong on the unsorted half.
Forgetting the duplicates degradation, or trying to keep O(log n) with duplicates.

#### Follow-ups
an interviewer may add: "find the rotation count / minimum element" (same which-half idea), "what if it's rotated multiple times" (rotating a sorted array any number of times is still a single-pivot rotation, so nothing changes), and "return whether it exists with duplicates" (LeetCode "Search in Rotated Sorted Array II").
Summary — what a strong answer emphasizes

Lead with the insight: rotation leaves at least one half of any split fully sorted. Detect the sorted half with arr[lo] <= arr[mid], then use that half's known endpoints to decide, with exact <=/< boundaries, which half to keep. State O(log n)/O(1) for distinct elements, and proactively flag that duplicates degrade the worst case to O(n) and require the lo++, hi-- shrink. Use the overflow-safe midpoint. Nailing the boundary comparisons and the duplicates caveat is what separates a strong answer from a buggy one.

**Comparison:**

```
Case	Which half sorted (test)	Decide keep-left when	Else
arr[lo] <= arr[mid]	Left [lo..mid]	arr[lo] <= target < arr[mid]	go right
arr[lo] > arr[mid]	Right [mid..hi]	(else of) arr[mid] < target <= arr[hi]	keep right / go left
Element property	Time	Space	Note
Distinct	O(log n)	O(1)	Clean binary search
With duplicates	O(n) worst	O(1)	arr[lo]==arr[mid]==arr[hi] → shrink both ends
```

**Code:**

```

// Distinct elements: O(log n) time, O(1) space public int search(int[] arr, int target) { int lo = 0, hi = arr.length - 1; while (lo <= hi) { int mid = lo + (hi - lo) / 2; // overflow-safe if (arr[mid] == target) return mid;

CODE
Copy
    if (arr[lo] <= arr[mid]) {              // left half [lo..mid] is sorted
        if (arr[lo] <= target && target < arr[mid]) {
            hi = mid - 1;                   // target in sorted left half
        } else {
            lo = mid + 1;                   // otherwise go right
        }
    } else {                                // right half [mid..hi] is sorted
        if (arr[mid] < target && target <= arr[hi]) {
            lo = mid + 1;                   // target in sorted right half
        } else {
            hi = mid - 1;                   // otherwise go left
        }
    }
}
return -1;                                   // not found

}

CODE
Copy

```python
# Python, distinct elements
def search(arr, target):
    lo, hi = 0, len(arr) - 1
    while lo <= hi:
        mid = (lo + hi) // 2
        if arr[mid] == target:
            return mid
        if arr[lo] <= arr[mid]:                 # left half sorted
            if arr[lo] <= target < arr[mid]:
                hi = mid - 1
            else:
                lo = mid + 1
        else:                                   # right half sorted
            if arr[mid] < target <= arr[hi]:
                lo = mid + 1
            else:
                hi = mid - 1
    return -1
PYTHON
Copy
# Variant with DUPLICATES: worst case O(n). Returns True/False.
def search_with_dups(arr, target):
    lo, hi = 0, len(arr) - 1
    while lo <= hi:
        mid = (lo + hi) // 2
        if arr[mid] == target:
            return True
        if arr[lo] == arr[mid] == arr[hi]:      # ambiguous -> peel ends
            lo += 1
            hi -= 1
        elif arr[lo] <= arr[mid]:               # left sorted
            if arr[lo] <= target < arr[mid]:
                hi = mid - 1
            else:
                lo = mid + 1
        else:                                   # right sorted
            if arr[mid] < target <= arr[hi]:
                lo = mid + 1
            else:
                hi = mid - 1
    return False
```

**Example input / output:**

```

Example: arr = [4,5,6,7,0,1,2], target = 0 → expected index 4.

Dry run:

CODE
Copy
lo=0 hi=6  mid=3  arr[3]=7 != 0
           arr[0]=4 <= arr[3]=7  -> LEFT half [0..3] sorted
           is 4 <= 0 < 7 ? No    -> go right, lo=4
lo=4 hi=6  mid=5  arr[5]=1 != 0
           arr[4]=0 <= arr[5]=1  -> LEFT half [4..5] sorted
           is 0 <= 0 < 1 ? Yes   -> go left, hi=4
lo=4 hi=4  mid=4  arr[4]=0 == 0  -> return 4  ✓

More cases:

arr	target	output
[4,5,6,7,0,1,2]	0	4
[4,5,6,7,0,1,2]	3	-1
[4,5,6,7,0,1,2]	4	0
[5,1,3]	3	2
[1]	0	-1
[1,2,3,4,5] (not rotated)	4	3
```

**Flow:**

1. No
2. Yes
3. Yes
4. No
5. Yes
6. Yes
7. No
8. No
9. Yes
10. No
11. lo=0, hi=n-1
12. lo <= hi?
13. return -1
14. mid = lo + hi-lo/2
15. arr[mid] == target?
16. return mid
17. arr[lo] <= arr[mid]? left sorted
18. arr[lo] <= target < arr[mid]?
19. hi = mid-1
20. lo = mid+1
21. arr[mid] < target <= arr[hi]?
22. lo = mid+1
23. hi = mid-1

