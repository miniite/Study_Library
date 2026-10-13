---
type: "practice"
difficulty: "Intermediate"
---

# Given activities with start and end times, select the maximum number of non-overlapping activities.

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Sorting Algorithms](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Sorting_Algorithms.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

This is the classic Activity Selection Problem: given n activities each with a start time s[i] and finish time f[i], select the maximum number of mutually non-overlapping activities (a single resource, e.g., one conference room, can host one activity at a time). Level is "intermediate experience," so the interviewer expects: the correct greedy strategy, why it is correct (not just that it works), the algorithm, complexity, edge cases, and ideally the proof sketch and the distinction from harder weighted variants. I'll clarify the standard assumptions: two activities overlap if their intervals intersect; back-to-back is allowed (activity ending at time t and another starting at t do not conflict, i.e., intervals are half-open [s, f)). If the interviewer wants closed intervals, we just change the comparison from >= to >.

Reframing the problem

Maximize the size of a subset of intervals such that no two overlap. This is unweighted interval scheduling — every activity is worth the same (count them), so we only care about how many we fit, not which. That equal-weight fact is exactly what makes a greedy approach optimal.

The key insight: greedy by earliest finish time

Always pick the activity that finishes earliest among those still compatible with what you've chosen. Sort activities by finish time ascending; iterate; take an activity if its start is ≥ the finish time of the last activity taken.

Why earliest finish, intuitively: the activity that ends soonest leaves the maximum remaining time free for the rest. Any activity that finishes later than the earliest-finishing one either conflicts with more future activities or, at best, ties — it can never let us fit more. So committing to the earliest finisher is never a mistake.

Common wrong greedies (good to mention to show depth):

Earliest start time — fails: one very long activity that starts first can block many short ones. Counterexample: [0,10], [1,2], [3,4] → picks [0,10] (1 activity) instead of the two short ones.
Shortest duration — fails: a short activity in the middle can knock out two activities that don't overlap each other. Counterexample: [0,5],[4,6],[6,11] with a short [4,6] blocks the optimal [0,5]+[6,11].
Fewest conflicts — more complex and also not always optimal here.

Only earliest finish time is provably optimal for maximizing count.

Algorithm
Sort activities by finish time ascending — O(n log n).
Select the first activity; set lastFinish = f[0].
For each subsequent activity, if start >= lastFinish, select it and update lastFinish = its finish.
Count/collect the selected activities.

Steps 2–4 are a single O(n) pass, so total time is O(n log n) dominated by the sort, and O(1) extra space (O(n) if returning the chosen list or if we must preserve the input order).

Proof of correctness (exchange / greedy-stays-ahead)

Let greedy pick activities g1, g2, …, gk (by increasing finish). Suppose an optimal solution picks o1, o2, …, om also sorted by finish, with m > k (assume greedy is not optimal, for contradiction).

Greedy stays ahead: finish(g1) ≤ finish(o1) because greedy takes the globally earliest finisher. By induction, finish(gi) ≤ finish(oi) for all i up to k: since finish(g_{i-1}) ≤ finish(o_{i-1}) ≤ start(o_i), activity o_i is compatible with greedy's prefix, so greedy had o_i (or something finishing no later) available and picked the earliest finisher, giving finish(g_i) ≤ finish(o_i).

Now if m > k, consider o_{k+1}: start(o_{k+1}) ≥ finish(o_k) ≥ finish(g_k), so o_{k+1} is compatible with greedy's g_k and greedy would not have stopped at k — it would have selected o_{k+1} (or another compatible activity). Contradiction. Hence k = m: greedy is optimal.

Edge cases
Empty input → 0 activities.
Single activity → 1.
All identical intervals → 1 (they all mutually overlap).
Back-to-back (s == previous f) → allowed with half-open intervals; use >=. If the problem treats endpoints as conflicting, use >.
Duplicate finish times → sort is stable enough; any order among equal-finish activities yields the same optimal count.
Variants (shows senior awareness)
Weighted interval scheduling (each activity has a value; maximize total value) — greedy no longer works; needs DP with binary search, O(n log n): sort by finish, dp[i] = max(dp[i-1], value[i] + dp[p(i)]) where p(i) is the latest non-conflicting earlier activity.
Multiple rooms / max overlap (minimum resources to schedule all activities) — sort start and end times, sweep, track concurrent count; that's a different objective.
Interval partitioning — assign all activities to the fewest rooms (equals the max number of overlapping intervals at any point).
Summary — what a strong answer emphasizes

The maximum non-overlapping activity count is solved by greedy on earliest finish time: sort by finish O(n log n), then one linear pass taking any activity whose start ≥ the last chosen finish. State why earliest-finish is optimal (it maximizes remaining free time; proven by a greedy-stays-ahead / exchange argument), rule out the tempting wrong greedies (earliest start, shortest duration) with counterexamples, handle the half-open vs closed-interval convention, and note that adding weights breaks greedy and pushes you to DP. Total: O(n log n) time, O(1) extra space.

**Comparison:**

```
Greedy criterion	Optimal for max count?	Failing counterexample
Earliest finish time	YES (provably)	—
Earliest start time	No	[0,10],[1,2],[3,4] → 1 vs optimal 2
Shortest duration	No	[0,5],[4,6],[6,11] → short [4,6] blocks 2
Fewest conflicts	No (and slower)	interval where the low-conflict pick splits two chains
Property	Value
Time complexity	O(n log n) (sort) + O(n) (scan) = O(n log n)
Extra space	O(1) (O(n) to return chosen list)
Objective	maximize COUNT (equal weights)
Weighted version	needs DP + binary search, O(n log n)
```

**Code:**

```

def max_non_overlapping_activities(intervals): """ intervals: list of (start, finish). Returns (count, chosen) maximizing the number of non-overlapping activities. Half-open convention: activity ending at t and another starting at t do NOT conflict. Time: O(n log n) | Extra space: O(1) beyond the output list. """ if not intervals: return 0, []

CODE
Copy
# 1. sort by finish time ascending
activities = sorted(intervals, key=lambda a: a[1])

count = 0
last_finish = float('-inf')
chosen = []

# 2. single greedy pass
for start, finish in activities:
    if start >= last_finish:      # compatible with everything chosen so far
        chosen.append((start, finish))
        last_finish = finish
        count += 1

return count, chosen

if name == "main": acts = [(1, 3), (2, 5), (4, 7), (1, 8), (5, 9), (8, 10), (9, 11), (11, 14), (13, 16)] print(max_non_overlapping_activities(acts))

CODE
Copy

```java
// Java equivalent
import java.util.*;

class ActivitySelection {
    static int maxActivities(int[][] a) {          // a[i] = {start, finish}
        Arrays.sort(a, Comparator.comparingInt(x -> x[1])); // sort by finish
        int count = 0, lastFinish = Integer.MIN_VALUE;
        for (int[] act : a) {
            if (act[0] >= lastFinish) {            // start >= last finish
                count++;
                lastFinish = act[1];
            }
        }
        return count;
    }
}
```

**Example input / output:**

```

Worked example (start, finish) pairs:

CODE
Copy
Input: [(1,3),(2,5),(4,7),(1,8),(5,9),(8,10),(9,11),(11,14),(13,16)]

After sort by finish:
(1,3),(2,5),(4,7),(1,8),(5,9),(8,10),(9,11),(11,14),(13,16)

Greedy pass (last_finish starts at -inf):
  (1,3)  : 1  >= -inf  -> TAKE, last=3        chosen=[(1,3)]
  (2,5)  : 2  >= 3 ?  no  -> skip
  (4,7)  : 4  >= 3 ?  yes -> TAKE, last=7      chosen=[(1,3),(4,7)]
  (1,8)  : 1  >= 7 ?  no  -> skip
  (5,9)  : 5  >= 7 ?  no  -> skip
  (8,10) : 8  >= 7 ?  yes -> TAKE, last=10     chosen=[...,(8,10)]
  (9,11) : 9  >= 10?  no  -> skip
  (11,14): 11 >= 10?  yes -> TAKE, last=14     chosen=[...,(11,14)]
  (13,16): 13 >= 14?  no  -> skip

Output: count = 4, chosen = [(1,3),(4,7),(8,10),(11,14)]

Counterexample proving "earliest START" is wrong:

CODE
Copy
Input: [(0,10),(1,2),(3,4)]
earliest-start greedy -> takes (0,10) only  => 1
earliest-finish greedy -> (1,2),(3,4)       => 2  (OPTIMAL)
```

**Flow:**

1. no
2. yes
3. yes
4. no
5. Read n activities: start, finish
6. Sort by finish time ascending
7. lastFinish = -inf, count = 0
8. More activities?
9. Return count / chosen
10. Take next activity by finishorder
11. start >= lastFinish ?
12. select it; lastFinish = finish;count++

