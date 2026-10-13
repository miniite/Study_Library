---
type: "practice"
difficulty: "Other"
---

# Earliest Day to Assemble the Required Number of Bundles

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Binary Search](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Binary_Search.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given an integer array ready of length n, along with two integers groups and span.

Think of n slots arranged in a line, numbered from left to right. Slot i becomes usable on day ready[i]: on any day d with d >= ready[i], slot i is considered ready; before that day it is not.

You want to assemble groups bundles. To form one bundle you must take span consecutive slots that are all ready on the chosen day. Each slot can belong to at most one bundle (once used in a bundle it cannot be reused).

Return the minimum day d on which you can assemble at least groups bundles from the line of slots. If it is impossible to ever assemble that many bundles, return -1.

Input: an array ready, an integer groups, and an integer span. Output: a single integer — the earliest feasible day, or -1 if impossible.

#### Cons
traints:

ready.length == n
1 <= n <= 10^5
1 <= ready[i] <= 10^9
1 <= groups <= 10^6
1 <= span <= n

#### Example
s & test cases
TEXT
Input: ready = [2,20,4,20,3], groups = 3, span = 1 -> Output: 4
Why: Each bundle needs just 1 ready slot. By day 3 only slots with ready value <= 3 exist (the 2 and the 3) giving 2 bundles; on day 4 the slot with value 4 also becomes ready, yielding 3 single-slot bundles.

Input: ready = [2,20,4,20,3], groups = 3, span = 2 -> Output: -1
Why: Three bundles of 2 consecutive slots each need 6 slots, but only 5 exist, so it can never be done.

Input: ready = [8,8,8,8,15,8,8], groups = 2, span = 3 -> Output: 15
Why: On day 8 the ready slots split into runs of length 4 and 2 (the value-15 slot breaks the line), giving only one bundle of 3. Only on day 15, when all 7 slots are ready in one run, can two bundles of 3 consecutive slots be formed.

Input: ready = [500], groups = 1, span = 1 -> Output: 500
Why: The single slot becomes ready on day 500 and forms exactly one 1-slot bundle.

Input: ready = [1,2,3], groups = 4, span = 1 -> Output: -1
Why: Four bundles each need at least 1 distinct slot, but only 3 slots exist, so it is impossible.

#### Approach

The answer is monotonic in the day: if you can assemble the required bundles by day d, you can also do it by any later day (more slots are ready, never fewer). That monotonic yes/no boundary is exactly what binary search on the answer exploits. Search the day value between the earliest and latest ready times; for a candidate day, run a linear greedy scan that counts consecutive ready slots and, every time the run length reaches span, commits a bundle and resets the run (a bundle greedily formed as early as possible is never worse). If the scan yields at least groups bundles the day is feasible, so move the upper bound down; otherwise raise the lower bound. A quick impossibility check up front — groups * span > n means there aren't enough slots at all — returns -1 (computed with 64-bit width to avoid overflow). Pattern: binary search on the answer + greedy linear feasibility check.

Complexity

Time: O(n log(maxReady)) — each feasibility check scans the array in O(n), and binary search over the day range [min(ready), max(ready)] runs O(log(maxReady - minReady)) checks. Space: O(1) extra beyond the input.

Python
PYTHON
def earliest_bundle_day(ready, groups, span):
    n = len(ready)
    if groups * span > n:
        return -1

    def feasible(day):
        bundles = run = 0
        for r in ready:
            if r <= day:
                run += 1
                if run == span:
                    bundles += 1
                    run = 0
            else:
                run = 0
        return bundles >= groups

    lo, hi = min(ready), max(ready)
    while lo < hi:
        mid = (lo + hi) // 2
        if feasible(mid):
            hi = mid
        else:
            lo = mid + 1
    return lo
JavaScript
JAVASCRIPT
function earliestBundleDay(ready, groups, span) {

#### cons
t n = ready.length;
    if (groups * span > n) return -1;

    let lo = ready[0], hi = ready[0];
    for (const r of ready) {
        if (r < lo) lo = r;
        if (r > hi) hi = r;
    }

#### cons
t feasible = (day) => {
        let bundles = 0, run = 0;
        for (const r of ready) {
            if (r <= day) {
                if (++run === span) { bundles++; run = 0; }
            } else {
                run = 0;
            }
        }
        return bundles >= groups;
    };

    while (lo < hi) {

#### cons
t mid = lo + Math.floor((hi - lo) / 2);
        if (feasible(mid)) hi = mid;
        else lo = mid + 1;
    }
    return lo;
}
Java
JAVA
class Solution {
    public int earliestBundleDay(int[] ready, int groups, int span) {
        int n = ready.length;
        if ((long) groups * span > n) return -1;

        int lo = Integer.MAX_VALUE, hi = Integer.MIN_VALUE;
        for (int r : ready) {
            lo = Math.min(lo, r);
            hi = Math.max(hi, r);
        }

        while (lo < hi) {
            int mid = lo + (hi - lo) / 2;
            if (feasible(ready, mid, groups, span)) hi = mid;
            else lo = mid + 1;
        }
        return lo;
    }

    private boolean feasible(int[] ready, int day, int groups, int span) {
        long bundles = 0;
        int run = 0;
        for (int r : ready) {
            if (r <= day) {
                if (++run == span) { bundles++; run = 0; }
            } else {
                run = 0;
            }
        }
        return bundles >= groups;
    }
}
C++
CPP
#include <vector>
#include <algorithm>
using namespace std;

class Solution {
public:
    int earliestBundleDay(vector<int>& ready, int groups, int span) {
        int n = (int)ready.size();
        if ((long long)groups * span > n) return -1;

        int lo = *min_element(ready.begin(), ready.end());
        int hi = *max_element(ready.begin(), ready.end());

        auto feasible = [&] (int day) {
            long long bundles = 0;
            int run = 0;
            for (int r : ready) {
                if (r <= day) {
                    if (++run == span) { bundles++; run = 0; }
                } else {
                    run = 0;
                }
            }
            return bundles >= (long long)groups;
        };

        while (lo < hi) {
            int mid = lo + (hi - lo) / 2;
            if (feasible(mid)) hi = mid;
            else lo = mid + 1;
        }
        return lo;
    }
};
C
C
#include <stdbool.h>

static bool feasible(int *ready, int n, long day, long groups, int span) {
    long bundles = 0;
    int run = 0;
    for (int i = 0; i < n; i++) {
        if (ready[i] <= day) {
            if (++run == span) {
                bundles++;
                run = 0;
            }
        } else {
            run = 0;
        }
    }
    return bundles >= groups;
}

int earliestBundleDay(int *ready, int n, int groups, int span) {
    if ((long)groups * span > n) return -1;

    long lo = ready[0], hi = ready[0];
    for (int i = 1; i < n; i++) {
        if (ready[i] < lo) lo = ready[i];
        if (ready[i] > hi) hi = ready[i];
    }

    while (lo < hi) {
        long mid = lo + (hi - lo) / 2;
        if (feasible(ready, n, mid, groups, span)) hi = mid;
        else lo = mid + 1;
    }
    return (int)lo;
}

#### Follow-up
questions
Why is binary search valid here? (Feasibility is monotonic in the day, so there is a single yes/no threshold.)
Prove the greedy feasibility check is correct: forming a bundle as soon as span consecutive ready slots appear never blocks a later achievable bundle.
What if bundles could use non-adjacent slots (any span ready slots)? Then feasibility reduces to counting ready slots and dividing by span, still binary-searchable.
How would you handle astronomically large ready[i] values or a very wide range without changing the asymptotics? (Binary search bounds stay tied to the actual min/max present values.)

