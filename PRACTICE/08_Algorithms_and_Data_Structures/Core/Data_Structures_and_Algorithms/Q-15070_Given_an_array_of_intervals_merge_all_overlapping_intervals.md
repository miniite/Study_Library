---
type: "practice"
difficulty: "Intermediate"
---

# Given an array of intervals, merge all overlapping intervals.

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Interval Merging](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Interval_Merging.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

What is being asked: Given a list of intervals like [[1,3],[2,6],[8,10],[15,18]], merge every group of intervals that overlap into a single interval, and return the merged set. This is a classic array/greedy problem.

What the interviewer is probing (intermediate level): Do you spot that sorting by start turns an O(n²) pairwise-comparison problem into a single O(n log n) linear sweep? Can you state the exact overlap condition, handle the boundary case (do intervals that merely touch count as overlapping?), and implement it cleanly with correct edge handling (empty input, single interval, fully-nested intervals)?

Clarifying questions I'd ask:

Are intervals [start, end] inclusive? Do adjacent/touching intervals like [1,4] and [4,5] count as overlapping? (Convention matters: usually yes, touching merges into [1,5].) I'll assume touching merges.
Are inputs already sorted? (Assume not — I'll sort.)
Can I mutate/return a new list? (I'll build a new result list.)
Any guarantees like start <= end per interval? (Assume valid intervals.)

Plan: Sort by start → sweep once, keeping the "current merged" interval → for each next interval, either extend the current (overlap) or push it and start a new current (no overlap) → return. Then dry-run, state complexity, cover edge cases.

Reasoning from first principles

Naive idea: compare every pair of intervals and union overlapping ones — but overlaps chain (A overlaps B, B overlaps C, but maybe A doesn't reach C), so you'd repeatedly re-scan. That's O(n²) or worse and fiddly.

Key insight: If I sort intervals by their start time, then any interval that could overlap a given interval must come immediately after it in sorted order. So I can walk left to right maintaining a single "current" merged interval [curStart, curEnd]:

For the next interval [s, e]:
Overlap / touch when s <= curEnd (its start falls inside or exactly at the end of the current merged block). Then extend: curEnd = max(curEnd, e). Using max is essential because the next interval might be fully nested (e.g., current [1,10], next [2,5] — end must stay 10, not shrink to 5).
No overlap when s > curEnd. The current block is finalized; push it to the result and start a fresh current = [s, e].

Because the array is sorted by start, curStart is always the smallest start of the block, so I never need to min the start — I only ever grow the end.

Why s <= curEnd and not s < curEnd? With <=, touching intervals ([1,4] and [4,6]) merge into [1,6]. If the problem defines touching as not overlapping, use strict <. I'd confirm this with the interviewer; the common LeetCode-style convention is <= (touching merges).

Complexity
Time: O(n log n) — dominated by the sort. The sweep itself is a single O(n) pass.
Space: O(n) for the output (O(log n)–O(n) auxiliary for the sort depending on implementation). If sorting in place and reusing the array, extra space can be O(1) beyond the sort.

This is optimal: any correct algorithm must at least distinguish orderings, and the sort-based bound O(n log n) is the standard lower-bound-matching solution for the general (unsorted, unbounded-value) case. (If interval endpoints were small bounded integers, a bucket/line-sweep on a difference array could hit O(n + range).)

Dry run

Input: [[1,3],[2,6],[8,10],[15,18]] (already sorted by start).

current = [1,3].
next [2,6]: 2 <= 3 → overlap → curEnd = max(3,6) = 6 → current [1,6].
next [8,10]: 8 > 6 → no overlap → push [1,6]; current = [8,10].
next [15,18]: 15 > 10 → push [8,10]; current = [15,18].
End of list → push [15,18].
Result: [[1,6],[8,10],[15,18]]. ✔

Nested case: [[1,4],[2,3]] → sort → current [1,4], next [2,3]: 2<=4, curEnd=max(4,3)=4 → stays [1,4]. ✔ (This is exactly why max matters.)

Touching case: [[1,4],[4,5]] → 4<=4 → merge → [1,5]. ✔

Edge cases to mention
Empty input → return [].
Single interval → return it as-is.
Unsorted input → sorting handles it; e.g. [[5,6],[1,2],[1,4]] → sorted [[1,2],[1,4],[5,6]] → [[1,4],[5,6]].
Equal starts → sort keeps them adjacent; max on end merges them correctly.
Duplicate intervals → merge into one.
Summary / what a strong answer emphasizes

The whole trick is sort by start, then one linear sweep maintaining a running merged interval; overlap iff the next start <= current end, and always extend with curEnd = max(curEnd, e) to survive nested intervals. Total O(n log n) time, O(n) output space. A strong candidate explicitly clarifies the touching convention (<= vs <), uses max (not blind assignment) for the end, and enumerates edge cases (empty, single, nested, unsorted). Common mistakes: forgetting to sort, using curEnd = e instead of max(...) (breaks on nested intervals), off-by-one on the touch boundary, and mutating the input list while iterating. Follow-up probes: "insert one new interval into an already-sorted merged list in O(n)" (Insert Interval), "return the total covered length," or "find the minimum number of intervals to remove to make the rest non-overlapping" (greedy by end time).

**Comparison:**

```
Approach	Time	Space	Notes
Pairwise compare + repeat	O(n²)	O(n)	Chained overlaps force re-scans; avoid
Sort by start + linear sweep	O(n log n)	O(n)	Standard optimal for general case
Line sweep on bounded int range	O(n + range)	O(range)	Only if endpoints are small integers
```

**Code:**

```

from typing import List

def merge_intervals(intervals: List[List[int]]) -> List[List[int]]: if not intervals: return [] # 1. Sort by start time -> overlapping intervals become adjacent. intervals.sort(key=lambda iv: iv[0]) merged = [list(intervals[0])] # copy so we never mutate input # 2. Single linear sweep. for s, e in intervals[1:]: last = merged[-1] if s <= last[1]: # overlap or touch -> extend last[1] = max(last[1], e) # max() is critical for nested intervals else: # disjoint -> start a new block merged.append([s, e]) return merged

CODE
Copy
```java
// Java equivalent
import java.util.*;
class Solution {
    public int[][] merge(int[][] intervals) {
        if (intervals.length == 0) return new int[0][];
        Arrays.sort(intervals, (a, b) -> Integer.compare(a[0], b[0]));
        List<int[]> out = new ArrayList<>();
        int[] cur = intervals[0].clone();
        for (int i = 1; i < intervals.length; i++) {
            if (intervals[i][0] <= cur[1]) {                 // overlap/touch
                cur[1] = Math.max(cur[1], intervals[i][1]);  // extend end
            } else {
                out.add(cur);
                cur = intervals[i].clone();
            }
        }
        out.add(cur);
        return out.toArray(new int[0][]);
    }
}
```

**Example input / output:**

```text
Input : [[1,3],[2,6],[8,10],[15,18]]
Output: [[1,6],[8,10],[15,18]]        # [1,3] & [2,6] overlap -> [1,6]

Input : [[1,4],[4,5]]
Output: [[1,5]]                        # touching intervals merge (<=)

Input : [[1,4],[2,3]]
Output: [[1,4]]                        # [2,3] fully nested inside [1,4]

Input : [[5,6],[1,2],[1,4]]           # unsorted
Output: [[1,4],[5,6]]                 # sort first -> [1,2]&[1,4] merge

Input : []
Output: []
```

**Flow:**

1. no
2. yes
3. yes
4. no
5. Sort intervals by start
6. current = first interval
7. More intervals?
8. Push current; return result
9. Take next s,e
10. s <= current.end ?
11. current.end = max current.end,e
12. Push current; current = s,e

