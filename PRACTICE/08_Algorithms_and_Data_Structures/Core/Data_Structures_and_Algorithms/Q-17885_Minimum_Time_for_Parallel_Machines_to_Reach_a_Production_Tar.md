---
type: "practice"
difficulty: "Other"
---

# Minimum Time for Parallel Machines to Reach a Production Target

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Binary Search](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Binary_Search.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

A workshop has several identical-purpose machines running in parallel. You are given an integer array dur, where dur[i] is the number of seconds machine i takes to produce one finished item.

Each machine works continuously and independently: as soon as it finishes an item it immediately starts the next one, and the machines never interfere with each other. So after T seconds, machine i has finished exactly floor(T / dur[i]) items.

You are also given an integer target, the total number of finished items the workshop must produce combined across all machines. Return the minimum number of seconds T such that the machines together produce at least target items.

Input

dur: an array of positive integers, dur[i] = seconds per item for machine i.
target: a positive integer, the required total item count.

Output

A single integer T: the smallest time in seconds at which the combined output is at least target.

Note: T can be as large as roughly min(dur) * target, which may exceed the range of a 32-bit integer, so use a 64-bit type for the time and the running item count.

#### Cons
traints

1 <= dur.length <= 10^5
1 <= dur[i] <= 10^7
1 <= target <= 10^7

#### Example
s & test cases
TEXT
- Input: `dur = [3,3,3], target = 5` -> Output: `6`
  Why: at `T = 5` each machine has finished `5/3 = 1` item (total 3 < 5); at `T = 6` each has `6/3 = 2` (total 6 >= 5), and no smaller time reaches 5.
- Input: `dur = [5], target = 1` -> Output: `5`
  Why: a single machine finishes its first item exactly at second 5 (single-machine edge case).
- Input: `dur = [1,1,1], target = 3` -> Output: `1`
  Why: at `T = 1` all three machines finish one item each, hitting the target immediately (fastest possible case).
- Input: `dur = [2,3], target = 10` -> Output: `12`
  Why: at `T = 11` output is `11/2 + 11/3 = 5 + 3 = 8`; at `T = 12` it is `6 + 4 = 10 >= 10`.
- Input: `dur = [7], target = 3` -> Output: `21`
  Why: one machine needs `3 * 7 = 21` seconds to finish 3 items.
- Input: `dur = [10000000], target = 1` -> Output: `10000000`
  Why: maximum per-item time with a single machine and one item stresses the 64-bit upper bound.

#### Approach

Search on the answer. Let f(T) be the combined number of items finished by time T: f(T) = sum(T / dur[i]) using integer division, since machine i completes floor(T / dur[i]) items. f is monotonic non-decreasing in T (more time never reduces output), so the set of times satisfying f(T) >= target is a suffix [answer, infinity). That monotonicity is exactly what lets us binary search for the smallest such T. Bounds: the answer is at least 1 and at most min(dur) * target — even if only the single fastest machine ran, it would reach target items by then. On each candidate mid we compute f(mid) in O(n) (breaking early once the target is met, which also prevents the running sum from overflowing), then move hi = mid when it is enough or lo = mid + 1 when it is not. Use 64-bit arithmetic because min(dur) * target and intermediate times can exceed 32-bit range. This is the standard binary-search-on-the-answer pattern applied to a monotone feasibility predicate.

Complexity

Time: O(n log(minDur * target)) — the answer lies in [1, minDur * target], and binary search halves that range on each step, spending O(n) to count combined output per candidate time. Space: O(1) — only a few scalar accumulators beyond the input. Justification: total output as a function of time is monotonic non-decreasing, so a single binary search over the time axis suffices instead of scanning every second.

Python
PYTHON
from typing import List

class Solution:
    def minTimeToTarget(self, dur: List[int], target: int) -> int:
        lo, hi = 1, min(dur) * target       # upper bound: fastest machine alone
        while lo < hi:
            mid = (lo + hi) // 2
            items = 0
            for d in dur:
                items += mid // d
                if items >= target:
                    break
            if items >= target:
                hi = mid
            else:
                lo = mid + 1
        return lo
JavaScript
JAVASCRIPT
function minTimeToTarget(dur, target) {
    let minDur = dur[0];
    for (const d of dur) if (d < minDur) minDur = d;
    let lo = 1, hi = minDur * target;       // safe within JS integer range for given bounds
    while (lo < hi) {

#### cons
t mid = Math.floor((lo + hi) / 2);
        let items = 0;
        for (const d of dur) {
            items += Math.floor(mid / d);
            if (items >= target) break;
        }
        if (items >= target) hi = mid;
        else lo = mid + 1;
    }
    return lo;
}
Java
JAVA
class Solution {
  public long minTimeToTarget(int[] dur, long target) {
    int minDur = dur[0];
    for (int d : dur) minDur = Math.min(minDur, d);
    long lo = 1, hi = (long) minDur * target;
    while (lo < hi) {
      long mid = lo + (hi - lo) / 2;
      long items = 0;
      for (int d : dur) {
        items += mid / d;
        if (items >= target) break;
      }
      if (items >= target) hi = mid;
      else lo = mid + 1;
    }
    return lo;
  }
}
C++
CPP
class Solution {
 public:
  long long minTimeToTarget(vector<int>& dur, long long target) {
    long long minDur = *min_element(dur.begin(), dur.end());
    long long lo = 1, hi = minDur * target;
    while (lo < hi) {
      long long mid = lo + (hi - lo) / 2;
      long long items = 0;
      for (int d : dur) {
        items += mid / d;
        if (items >= target) break;
      }
      if (items >= target) hi = mid;
      else lo = mid + 1;
    }
    return lo;
  }
};
C
C
long long minTimeToTarget(const int* dur, int n, long long target) {
    int minDur = dur[0];
    for (int i = 1; i < n; ++i) if (dur[i] < minDur) minDur = dur[i];
    long long lo = 1, hi = (long long)minDur * target;   /* fastest machine alone bounds the answer */
    while (lo < hi) {
        long long mid = lo + (hi - lo) / 2;
        long long items = 0;
        for (int i = 0; i < n; ++i) {
            items += mid / dur[i];
            if (items >= target) break;                  /* early exit + overflow guard */
        }
        if (items >= target) hi = mid;
        else lo = mid + 1;
    }
    return lo;
}

#### Follow-up
questions
Prove the monotonicity of f(T) that makes binary search valid, and argue why min(dur) * target is a correct upper bound.
What if each machine also had a fixed warmup delay w[i] before finishing its first item — how does f(T) change while keeping the same search framework?
After finding the minimum time, report exactly how many items each machine finished at that instant.
Suppose each machine has a maximum number of items it may produce (a per-machine cap); adapt the counting function accordingly.

