---
type: "practice"
difficulty: "Intermediate"
---

# Given an array, find the contiguous subarray with the largest sum (Kadane's algorithm).

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Maximum Subarray](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Maximum_Subarray.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage
— what's really being asked

The interviewer wants the maximum-sum contiguous subarray problem, and they specifically named Kadane's algorithm, so they are probing whether I can (a) recognize this as a classic dynamic-programming problem, (b) derive the O(n) one-pass solution rather than the naive O(n²)/O(n³) brute force, and (c) handle the tricky edge cases — especially all-negative arrays and the empty-subarray convention. "Contiguous" is the key word: elements must be adjacent (this is a subarray, not a subsequence), so I cannot skip elements.

Clarifying questions I'd ask:

Must the subarray be non-empty? This decides the all-negative case. The standard interview convention is non-empty — for [-3, -1, -4] the answer is -1. If empty subarrays are allowed the answer would be 0. I'll assume non-empty.
Do we also need the indices (start/end) of the subarray, or just the sum? I'll give both.
Integer overflow? For very large arrays with large values I'd use 64-bit sums; in most interviews int is fine.

Plan: start from brute force to motivate the insight, derive the DP recurrence, present Kadane's O(n)/O(1) solution, extend it to return indices, dry-run it, and discuss edge cases and the divide-and-conquer alternative.

From first principles

Brute force. Try every subarray. There are O(n²) subarrays; summing each naively is O(n) giving O(n³). Keeping a running sum as the right endpoint moves reduces this to O(n²). That works but is too slow for large inputs and is not what "Kadane's" implies.

The key insight (DP). Define best_ending_here(i) = the maximum sum of a subarray that ends exactly at index i. Every contiguous subarray ends at some index, so the global answer is max over all i of best_ending_here(i).

Now the recurrence. A subarray ending at i is either:

just the single element nums[i], or
the best subarray ending at i-1, extended by nums[i].

We pick whichever is larger:

CODE
best_ending_here(i) = max( nums[i],  best_ending_here(i-1) + nums[i] )

The intuition: if the best sum we could carry into position i (best_ending_here(i-1)) is negative, it only drags us down, so we're better off starting fresh at nums[i]. If it's positive, we keep accumulating. This "reset when the running sum goes negative" behavior is exactly Kadane's algorithm.

Because best_ending_here(i) only depends on best_ending_here(i-1), we don't need an array — a single rolling variable suffices. That gives O(n) time, O(1) space.

Algorithm (sum only):

CODE
current = nums[0]
best    = nums[0]
for i from 1 to n-1:
    current = max(nums[i], current + nums[i])
    best    = max(best, current)
return best

Initializing both to nums[0] (not 0) is what makes the all-negative case correct — best can end up negative, which is the right answer under the non-empty convention. A common bug is initializing best = 0, which wrongly returns 0 for an all-negative array.

Returning the indices

To report which subarray, track a tentative start. Whenever we "start fresh" (i.e., current + nums[i] < nums[i]), we move the tentative start to i. When current beats the global best, we lock in the current window as the answer.

See code_blocks for the full indexed version and input_output for a dry run.

Complexity and correctness
Time: O(n) — one pass.
Space: O(1) — two scalars (plus a couple for indices).
Correctness: by induction, current always equals best_ending_here(i) and best equals the max over the prefix ending at or before i; after the loop best is the global maximum. Because we consider a subarray ending at every index, and each such subarray is optimally either "extend" or "restart," no candidate is missed.
Edge cases
Case	Input	Answer	Why
Single element	[5]	5	Trivial
All negative	[-3,-1,-4]	-1	Non-empty convention; init to nums[0]
All positive	[1,2,3]	6	Whole array
Mixed	[-2,1,-3,4,-1,2,1,-5,4]	6 (subarray [4,-1,2,1])	Classic
Zeros	[0,-1,0]	0	Zeros neither help nor hurt
Alternatives and follow-ups
Divide & conquer: split at the middle; the answer is the max of (best in left half, best in right half, best subarray crossing the midpoint). The crossing sum is computed in O(n), giving T(n) = 2T(n/2) + O(n) = O(n log n). Elegant, and it's the natural extension to a segment tree if the interviewer asks for range-max-subarray queries with updates. But Kadane's O(n) beats it for the plain problem.

#### Follow-ups
I'd be ready for: maximum-product subarray (track both min and max because negatives flip sign), maximum sum with at most one deletion, circular-array max subarray (answer = max of normal Kadane vs. totalSum − minSubarray), and 2-D maximum submatrix (fix a pair of rows, compress columns, run Kadane — O(rows²·cols)).
What a strong answer emphasizes

Name the DP recurrence explicitly, explain the "reset on negative prefix" intuition, initialize to the first element (not 0) and justify it with the all-negative case, state O(n)/O(1), and offer to return indices and mention the D&C variant. The single most common mistake this question screens for is the best = 0 initialization bug.

**Comparison:**

```
Approach	Time	Space	Notes
Brute force (all subarrays, naive sum)	O(n³)	O(1)	Too slow
Brute force (running sum)	O(n²)	O(1)	Simple but slow
Kadane (DP)	O(n)	O(1)	Best for plain problem
Divide & conquer	O(n log n)	O(log n)	Generalizes to segment tree
```

**Code:**

```

def max_subarray(nums): """Kadane's algorithm. Returns (max_sum, start_idx, end_idx). Assumes non-empty array; subarray must be non-empty.""" if not nums: raise ValueError("array must be non-empty")

CODE
Copy
best = current = nums[0]
best_start = best_end = 0
temp_start = 0

for i in range(1, len(nums)):
    # extend the previous subarray, or start a new one at i
    if current + nums[i] < nums[i]:
        current = nums[i]
        temp_start = i
    else:
        current = current + nums[i]

    if current > best:
        best = current
        best_start = temp_start
        best_end = i

return best, best_start, best_end
CODE
Copy

```java
// Sum-only version, Java
public int maxSubArray(int[] nums) {
    int best = nums[0], current = nums[0];
    for (int i = 1; i < nums.length; i++) {
        current = Math.max(nums[i], current + nums[i]);
        best = Math.max(best, current);
    }
    return best;
}
```

**Example input / output:**

```text
Input:  nums = [-2, 1, -3, 4, -1, 2, 1, -5, 4]
Output: max_sum = 6, subarray = [4, -1, 2, 1] (indices 3..6)

Dry run (current = best_ending_here, best = global max):

| i | nums[i] | current = max(nums[i], current+nums[i]) | best | note |
|---|---------|------------------------------------------|------|------|
| 0 | -2 |  -2 (init)                     | -2 | start |
| 1 |  1 | max(1, -2+1=-1) = 1            |  1 | restart at 1 |
| 2 | -3 | max(-3, 1-3=-2) = -2          |  1 | extend |
| 3 |  4 | max(4, -2+4=2) = 4            |  4 | restart at 3 |
| 4 | -1 | max(-1, 4-1=3) = 3           |  4 | extend |
| 5 |  2 | max(2, 3+2=5) = 5            |  5 | extend |
| 6 |  1 | max(1, 5+1=6) = 6           |  6 | extend -> new best |
| 7 | -5 | max(-5, 6-5=1) = 1          |  6 | extend |
| 8 |  4 | max(4, 1+4=5) = 5          |  6 | extend |

Final best = 6, subarray indices 3..6 -> [4, -1, 2, 1].

All-negative check:
Input:  [-3, -1, -4]
Output: -1  (subarray [-1], index 1)  -- correct because we init to nums[0], not 0.
```

**Flow:**

1. no
2. yes
3. yes
4. no
5. yes
6. no
7. Start: best = current = nums0
8. More elements?
9. Return best
10. Read nums i
11. current + nums_i < nums_i ?
12. current = nums_i; restartwindow at i
13. current = current + nums_i;extend window
14. current > best ?
15. best = current; save windowbounds

