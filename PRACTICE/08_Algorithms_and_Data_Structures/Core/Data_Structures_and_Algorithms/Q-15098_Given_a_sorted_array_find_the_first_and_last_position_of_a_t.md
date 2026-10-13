---
type: "practice"
difficulty: "Intermediate"
---

# Given a sorted array, find the first and last position of a target element.

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Interval Merging](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Interval_Merging.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

What is being asked: Given a sorted array nums and a target, return [first, last] — the indices of the first and last occurrence of target. If the target is absent, return [-1, -1]. This is the classic "search for a range" problem.

What the interviewer is probing: Whether you recognize that "sorted array + find position" screams binary search, and whether you can adapt vanilla binary search to find boundaries (leftmost / rightmost occurrence) rather than just "any" match. The naive linear scan is the trap — it's easy, but it throws away the sorted property and runs in O(n). A strong candidate reaches for O(log n) and, crucially, gets the boundary edge cases right (duplicates, empty array, target smaller/larger than everything).

Clarifying questions worth stating:

Is the array sorted ascending? (Assume yes.)
Are there duplicates? (Yes — that's the whole point; otherwise first == last.)
0-indexed output? (Yes.)
What to return if not found? ([-1, -1].)
Integers, could be negative? (Doesn't affect the algorithm.)

Plan: Run binary search twice. Once biased to keep moving left when we hit the target (to pin the first index), once biased to keep moving right (to pin the last index). Each search is O(log n), so the total is O(log n).

From first principles to the solution

Plain binary search answers "does target exist and where is some occurrence?" When there are duplicates, that "some" index is arbitrary — the algorithm stops at whichever equal element it lands on first. To get a boundary, we change what we do when nums[mid] == target: instead of returning immediately, we record the candidate and keep searching on one side.

Finding the first (leftmost) index. When nums[mid] == target, the answer could be mid — but there might be an earlier equal element to the left. So we save mid as a candidate and move hi = mid - 1 to look further left. When nums[mid] < target we move right (lo = mid + 1); when nums[mid] > target we move left (hi = mid - 1). We loop while lo <= hi. The last candidate we recorded before the window collapses is the leftmost occurrence.

Finding the last (rightmost) index. Symmetric: when nums[mid] == target, record mid and move lo = mid + 1 to look further right.

Why this is correct: each time we find target, we never discard the current mid as a possible answer (we stored it), we only shrink the window toward the side that might hold a better (further-left or further-right) answer. Because the array is sorted, all equal elements are contiguous, so this monotone shrinking is guaranteed to converge on the true boundary.

Alternative framing — lower_bound / upper_bound. A cleaner mental model many engineers prefer: compute lower = first index with nums[i] >= target and upper = first index with nums[i] > target. Then the range is [lower, upper - 1], and target exists iff lower < n && nums[lower] == target. This is the C++ STL model (std::lower_bound, std::upper_bound) and generalizes to counting duplicates: count = upper - lower. I show both styles in the code — the "record candidate" version is the most beginner-safe.

Complexity
Time: O(log n) — two independent binary searches, each halving the search space.
Space: O(1) — a handful of index variables, no recursion needed (iterative).
Edge cases to nail (these are where candidates fail)
Empty array → [-1, -1].
Target smaller than all / larger than all → [-1, -1].
Single element equal to target → [0, 0].
Whole array equals target → [0, n-1].
Overflow of mid: in languages like Java/C++, write mid = lo + (hi - lo) / 2 instead of (lo + hi) / 2 to avoid integer overflow on huge indices.
What a strong answer emphasizes
Immediately connects "sorted" → "binary search" and states the O(log n) target before coding.
Explains the single behavioral change (don't return on equality; record and keep going) that turns plain binary search into a boundary search — that insight is the whole point.
Handles the not-found case cleanly (candidate stays -1).
Mentions the lower_bound/upper_bound reformulation and that upper - lower gives the count of the target — a nice extension the interviewer may probe.

#### Common mistakes
returning on the first == (gives an arbitrary index, not a boundary); off-by-one in the [lower, upper-1] framing; using (lo+hi)/2 and overflowing; forgetting that when the candidate is never set you must return -1; doing a linear scan after finding one occurrence to expand both ways — that degrades to O(n) when the array is all duplicates.

#### Follow-up
probes you should be ready for: "Count occurrences of target" → last - first + 1 or upper - lower. "Find insertion position if absent" → that's exactly lower_bound. "What if the array is rotated?" → different problem (search in rotated sorted array), needs a pivot-aware binary search.

**Comparison:**

```
Approach	Time	Space	Notes
Linear scan (find one, expand both sides)	O(n)	O(1)	Ignores sortedness; O(n) worst case (all duplicates)
Two binary searches (record-candidate)	O(log n)	O(1)	Recommended; one behavioral tweak on ==
lower_bound + upper_bound	O(log n)	O(1)	Cleanest; upper - lower = occurrence count
```

**Code:**

```

// Style 1: "record candidate, keep searching one side" — most readable. class Solution { public int[] searchRange(int[] nums, int target) { int first = findBound(nums, target, true); if (first == -1) return new int[]{-1, -1}; // target absent int last = findBound(nums, target, false); return new int[]{first, last}; }

CODE
Copy
// leftmost=true  -> first occurrence; leftmost=false -> last occurrence
private int findBound(int[] nums, int target, boolean leftmost) {
    int lo = 0, hi = nums.length - 1, ans = -1;
    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2;      // overflow-safe
        if (nums[mid] == target) {
            ans = mid;                     // candidate — do NOT return yet
            if (leftmost) hi = mid - 1;    // pull window left
            else          lo = mid + 1;    // pull window right
        } else if (nums[mid] < target) {
            lo = mid + 1;
        } else {
            hi = mid - 1;
        }
    }
    return ans;
}

}

CODE
Copy

```java
// Style 2: lower_bound / upper_bound. Range = [lower, upper - 1].
// Bonus: (upper - lower) == number of occurrences of target.
class SolutionBounds {
    public int[] searchRange(int[] nums, int target) {
        int lower = lowerBound(nums, target);        // first i with nums[i] >= target
        if (lower == nums.length || nums[lower] != target)
            return new int[]{-1, -1};
        int upper = lowerBound(nums, target + 1);    // first i with nums[i] > target
        return new int[]{lower, upper - 1};
    }
    private int lowerBound(int[] nums, int key) {
        int lo = 0, hi = nums.length;                // note: hi = n, half-open
        while (lo < hi) {
            int mid = lo + (hi - lo) / 2;
            if (nums[mid] < key) lo = mid + 1;
            else                 hi = mid;
        }
        return lo;
    }
}
```

**Example input / output:**

```text
Example 1
  nums = [5, 7, 7, 8, 8, 8, 10], target = 8
  -> [3, 5]

  findBound(leftmost=true):
    lo=0 hi=6 mid=3 nums[3]=8 == target -> ans=3, hi=2
    lo=0 hi=2 mid=1 nums[1]=7 < 8       -> lo=2
    lo=2 hi=2 mid=2 nums[2]=7 < 8       -> lo=3
    lo=3 hi=2 -> stop. first = 3

  findBound(leftmost=false):
    lo=0 hi=6 mid=3 nums[3]=8 == target -> ans=3, lo=4
    lo=4 hi=6 mid=5 nums[5]=8 == target -> ans=5, lo=6
    lo=6 hi=6 mid=6 nums[6]=10 > 8      -> hi=5
    lo=6 hi=5 -> stop. last = 5
  => [3, 5]

Example 2 (absent)
  nums = [5, 7, 7, 8, 8, 8, 10], target = 6  -> [-1, -1]

Example 3 (empty)
  nums = [], target = 0  -> [-1, -1]

Example 4 (all equal)
  nums = [2, 2, 2, 2], target = 2  -> [0, 3]
```

**Flow:**

1. no
2. yes
3. ==
4. yes
5. no
6. < target
7. > target
8. lo=0, hi=n-1, ans=-1
9. lo <= hi ?
10. return ans
11. mid = lo + hi-lo/2
12. nums mid vs target
13. ans = mid
14. leftmost?
15. hi = mid-1
16. lo = mid+1
17. lo = mid+1
18. hi = mid-1

