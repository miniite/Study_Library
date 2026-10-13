---
type: "practice"
difficulty: "Other"
---

# Minimum Belt Speed to Meet a Delivery Deadline

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Binary Search](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Binary_Search.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

A parcel must pass through n sorting hubs that are connected, in a fixed order, by conveyor legs. To move the parcel from one hub to the next you ride a single conveyor leg; the i-th leg is dist[i] meters long. Every conveyor in the facility runs at the same integer speed v (meters per hour), and you get to choose that one speed.

There is a scheduling quirk: each conveyor leg can only begin at a whole-hour mark. So after a leg finishes, if the current time is not already an integer number of hours, the parcel must wait until the next whole hour before the following leg starts. The very last leg has no such rounding — the parcel is considered delivered the instant it finishes the last leg.

Concretely, at speed v:

Legs 0 through n-2 each take ceil(dist[i] / v) hours of wall-clock time (the ride plus the wait until the next whole hour).
The final leg n-1 takes exactly dist[n-1] / v hours (no wait afterward).

You are given a floating-point deadline budget (in hours). Return the minimum positive integer speed v such that the total wall-clock time is at most budget. If no positive integer speed can meet the deadline, return -1.

The data is arranged so that whenever an answer exists it does not exceed 10^7, and budget has at most two digits after the decimal point.

Input
dist: an integer array of length n, where dist[i] is the length of the i-th conveyor leg.
budget: a floating-point number, the deadline in hours.
Output
The smallest positive integer speed that delivers on time, or -1 if impossible.

#### Cons
traints
n == dist.length
1 <= n <= 10^5
1 <= dist[i] <= 10^5
1 <= budget <= 10^9
budget has at most two digits after the decimal point.
If an answer exists it is guaranteed to be at most 10^7.

#### Example
s & test cases
TEXT
- Input: `dist = [3, 6, 4], budget = 5` -> Output: `3`
  Why: At v=3 the times are ceil(3/3)=1, ceil(6/3)=2, and 4/3=1.333 for the last leg = 4.333 <= 5. At v=2 the total is 2+3+2 = 7 > 5, so 3 is the smallest that works. Normal binary-search case.
- Input: `dist = [3, 6, 4], budget = 13` -> Output: `1`
  Why: Even the slowest speed v=1 gives 3+6+4 = 13 exactly on time; equality counts, so the minimum is 1. Exact-deadline boundary.
- Input: `dist = [3, 6, 4], budget = 1.5` -> Output: `-1`
  Why: n=3, so the first two legs alone consume at least ceil>=1 each = 2 hours regardless of speed; 1.5 < 2 is unreachable. Impossible case (budget <= n-1).
- Input: `dist = [10], budget = 2.5` -> Output: `4`
  Why: A single leg has no waiting, so we need 10/v <= 2.5, i.e. v >= 4. At v=3, 10/3=3.33 > 2.5. Single-leg / no-rounding edge case.
- Input: `dist = [1, 1, 100], budget = 2.2` -> Output: `500`
  Why: The two short legs each round up to 1 hour = 2 hours of waiting, leaving 0.2 hours for the last leg: 100/v <= 0.2 -> v >= 500. Rounding of tiny legs dominates the answer.

#### Approach

This is binary search on the answer. The total travel time is a non-increasing function of the chosen speed v: faster belts can only shorten each leg, so if some speed meets the deadline, every larger speed does too. That monotonicity means the set of feasible speeds is a suffix [v*, 10^7], and we binary-search for its smallest element v*. The feasibility predicate at speed v computes sum(ceil(dist[i]/v)) for the first n-1 legs (ride time plus the wait to the next whole hour) plus dist[n-1]/v for the last leg, and asks whether that is <= budget. To sidestep floating-point error we clear denominators: multiply through by v and by 100 (since the deadline has at most two decimals), reducing the check to the exact integer inequality (waited*v + dist[n-1]) * 100 <= round(budget*100) * v. If even the maximum speed 10^7 is infeasible, no answer exists and we return -1.

Complexity

Time: O(n log(V)) where V = 10^7 is the speed search bound — each of the ~24 binary-search steps sums the ceilings over all n legs in O(n). Space: O(1) extra beyond the input. The feasibility test is monotone in v (higher speed never increases total time), which is exactly what licenses the binary search.

Python
PYTHON
from math import ceil
from typing import List

class Solution:
    def minBeltSpeed(self, dist: List[int], budget: float) -> int:
        budget100 = round(budget * 100)   # two-decimal deadline as an integer
        n = len(dist)

        def feasible(v: int) -> bool:
            # whole-hour cost of every leg except the last
            waited = sum((dist[i] + v - 1) // v for i in range(n - 1))
            # (waited*v + last) * 100 <= budget100 * v   (exact, no floats)
            return (waited * v + dist[-1]) * 100 <= budget100 * v

        lo, hi = 1, 10_000_000
        if not feasible(hi):
            return -1
        while lo < hi:
            mid = (lo + hi) // 2
            if feasible(mid):
                hi = mid
            else:
                lo = mid + 1
        return lo
JavaScript
JAVASCRIPT
function minBeltSpeed(dist, budget) {

#### cons
t budget100 = Math.round(budget * 100); // deadline scaled to an integer

#### cons
t n = dist.length;

    // Exact integer check: (waited*v + last)*100 <= budget100*v

#### cons
t feasible = (v) => {
        let waited = 0;
        for (let i = 0; i < n - 1; i++) waited += Math.ceil(dist[i] / v);
        return (waited * v + dist[n - 1]) * 100 <= budget100 * v;
    };

    let lo = 1, hi = 10000000;
    if (!feasible(hi)) return -1;
    while (lo < hi) {

#### cons
t mid = Math.floor((lo + hi) / 2);
        if (feasible(mid)) hi = mid;
        else lo = mid + 1;
    }
    return lo;
}
Java
JAVA
class Solution {
    // Exact integer feasibility: (waited*v + last)*100 <= budget100*v
    private boolean feasible(int[] dist, long v, long budget100) {
        long waited = 0;
        int n = dist.length;
        for (int i = 0; i < n - 1; i++)
            waited += (dist[i] + v - 1) / v;            // ceil(dist[i]/v)
        long lhs = (waited * v + dist[n - 1]) * 100L;
        return lhs <= budget100 * v;
    }

    public int minBeltSpeed(int[] dist, double budget) {
        long budget100 = Math.round(budget * 100.0);
        long lo = 1, hi = 10000000;
        if (!feasible(dist, hi, budget100)) return -1;
        while (lo < hi) {
            long mid = lo + (hi - lo) / 2;
            if (feasible(dist, mid, budget100)) hi = mid;
            else lo = mid + 1;
        }
        return (int) lo;
    }
}
C++
CPP
#include <vector>
#include <cmath>
using namespace std;

class Solution {
    // Exact integer check: (waited*v + dist.back())*100 <= budget100*v
    static bool feasible(const vector<int>& dist, long long v, long long budget100) {
        long long waited = 0;
        int n = dist.size();
        for (int i = 0; i < n - 1; i++)
            waited += (dist[i] + v - 1) / v;            // ceil(dist[i]/v)
        long long lhs = (waited * v + dist[n - 1]) * 100LL;
        return lhs <= budget100 * v;
    }
public:
    int minBeltSpeed(const vector<int>& dist, double budget) {
        long long budget100 = llround(budget * 100.0);
        long long lo = 1, hi = 10000000;
        if (!feasible(dist, hi, budget100)) return -1;
        while (lo < hi) {
            long long mid = lo + (hi - lo) / 2;
            if (feasible(dist, mid, budget100)) hi = mid;
            else lo = mid + 1;
        }
        return (int)lo;
    }
};
C
C
#include <math.h>

/* Exact integer feasibility test: is total wall-clock time <= budget at speed v?
   budget100 = round(budget * 100) turns the two-decimal deadline into an integer.
   Condition:  waited * v + dist[n-1]  <=  budget * v
   scaled by 100:  (waited * v + dist[n-1]) * 100  <=  budget100 * v          */
static int feasible(const int *dist, int n, long long v, long long budget100) {
    long long waited = 0;                 /* whole-hour cost of legs 0..n-2 */
    for (int i = 0; i < n - 1; i++)
        waited += (dist[i] + v - 1) / v;  /* ceil(dist[i]/v) */
    long long lhs = (waited * v + dist[n - 1]) * 100LL;
    return lhs <= budget100 * v;
}

int minBeltSpeed(const int *dist, int n, double budget) {
    long long budget100 = (long long)llround(budget * 100.0);
    long long lo = 1, hi = 10000000; /* 1e7 upper bound per constraints */
    if (!feasible(dist, n, hi, budget100)) return -1;
    while (lo < hi) {
        long long mid = lo + (hi - lo) / 2;
        if (feasible(dist, n, mid, budget100)) hi = mid;
        else lo = mid + 1;
    }
    return (int)lo;
}

#### Follow-up
questions
What changes if the speed is allowed to be any real number rather than an integer? (Binary-search on a real interval to a tolerance, or solve the piecewise expression directly.)
Suppose there were no whole-hour waiting between legs — how does the feasibility formula simplify, and does the answer become easier to compute in closed form?
If each leg had its own maximum allowed speed cap, how would you incorporate per-leg caps into the feasibility check?
The deadline can be as large as 10^9 with two decimals; explain precisely why the scaled integer comparison avoids the floating-point pitfalls of comparing sums of ceil(dist[i]/v) against budget directly.

