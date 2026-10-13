---
type: "practice"
difficulty: "Other"
---

# Fewest Repeaters to Cover a Cable Segment

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Greedy](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Greedy.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

A straight communication cable lies on a number line, starting at coordinate 0 and ending at coordinate L, so its total length is L. Along the cable there are exactly L + 1 signal repeaters installed at the integer positions 0, 1, 2, ..., L.

You are given the integer L and an integer array cover of length L + 1. For each repeater i (0-indexed), cover[i] is its coverage radius: if repeater i is switched on, it energizes the closed segment [i - cover[i], i + cover[i]] of the cable. A repeater with radius 0 energizes only the single point at its own position.

Switching on more repeaters costs more power, so you want to turn on as few as possible while still keeping the entire cable from 0 to L energized (every real point in [0, L] must be covered by at least one switched-on repeater; coverage of points outside [0, L] is irrelevant).

Return the minimum number of repeaters that must be switched on to cover the whole cable [0, L]. If it is impossible to cover the whole cable even with every repeater on, return -1.

#### Cons
traints

1 <= L <= 10^4
cover.length == L + 1
0 <= cover[i] <= 100
Coverage segments are closed intervals; two segments that merely touch at a shared endpoint (e.g. one ends at 3 and the next begins at 3) together cover everything between and including those points, with no gap.

#### Example
s & test cases
TEXT
Input: L = 6, cover = [2,0,0,3,0,0,1] -> Output: 1
Why: Repeater 3 alone energizes [3-3, 3+3] = [0,6], which is the whole cable, so one repeater suffices.

Input: L = 4, cover = [1,0,0,0,1] -> Output: -1
Why: The reachable segments are [0,1], [1,1], [2,2], [3,3], [3,4]. After covering up to point 1 the next available segment starts at 2, leaving the open gap (1,2) uncovered; no combination closes it.

Input: L = 1, cover = [1,0] -> Output: 1
Why: Repeater 0 covers [0,1] (its radius reaches point 1, capped at L), covering the entire tiny cable with a single unit.

Input: L = 5, cover = [3,1,1,2,0,2] -> Output: 2
Why: Repeater 0 covers [0,3] and repeater 3 covers [1,5]; together they cover [0,5]. No single repeater spans all of [0,5], so two is optimal.

Input: L = 3, cover = [0,0,0,0] -> Output: -1
Why: Every repeater has radius 0 and energizes only its own integer point, so the open intervals between consecutive integers are never covered; the cable cannot be fully energized.

#### Approach

Turn every repeater into an interval: repeater i covers [i - cover[i], i + cover[i]], clamped to [0, L]. The task "cover [0, L] with the fewest intervals" is exactly the minimum-interval-cover problem, which is solved greedily like Jump Game II. Bucket the intervals by their left endpoint into maxReach[left] = max right reachable from that start. Then sweep positions 0..L-1 tracking farthest (the best right endpoint reachable using any interval starting at or before the current point) and end (the boundary of the current chosen interval). Each time the sweep reaches end, we are forced to commit one more repeater and jump end to farthest. If at any point farthest <= i, there is an uncovered gap immediately after point i and the answer is -1. This greedy "extend as far as possible, then commit" strategy is optimal for contiguous interval covering.

Complexity

Time: O(L). One pass builds the maxReach array and one greedy pass sweeps the cable; each is linear in L. Space: O(L) for the maxReach array (the coverage radius bound of 100 does not change the asymptotics).

Python
PYTHON
from typing import List

def min_repeaters(L: int, cover: List[int]) -> int:
    # max_reach[p] = farthest right coordinate coverable by any repeater
    # whose clamped left endpoint is p.
    max_reach = [0] * (L + 1)
    for i, r in enumerate(cover):
        left = max(0, i - r)
        right = min(L, i + r)
        max_reach[left] = max(max_reach[left], right)

    taps = end = farthest = 0
    for i in range(L):
        farthest = max(farthest, max_reach[i])
        if farthest <= i:            # cannot cover point just past i
            return -1
        if i == end:                 # must open a new repeater here
            taps += 1
            end = farthest
    return taps
JavaScript
JAVASCRIPT
function minRepeaters(L, cover) {
    // maxReach[p] = farthest right coord coverable by a repeater with left endpoint p

#### cons
t maxReach = new Array(L + 1).fill(0);
    for (let i = 0; i <= L; i++) {

#### cons
t left = Math.max(0, i - cover[i]);

#### cons
t right = Math.min(L, i + cover[i]);
        maxReach[left] = Math.max(maxReach[left], right);
    }

    let taps = 0, end = 0, farthest = 0;
    for (let i = 0; i < L; i++) {
        farthest = Math.max(farthest, maxReach[i]);
        if (farthest <= i) return -1;              // stuck at point i
        if (i === end) { taps++; end = farthest; }
    }
    return taps;
}
Java
JAVA
class Solution {
    public int minRepeaters(int L, int[] cover) {
        // maxReach[p] = farthest right coord coverable by a repeater with left endpoint p
        int[] maxReach = new int[L + 1];
        for (int i = 0; i <= L; i++) {
            int left  = Math.max(0, i - cover[i]);
            int right = Math.min(L, i + cover[i]);
            maxReach[left] = Math.max(maxReach[left], right);
        }

        int taps = 0, end = 0, farthest = 0;
        for (int i = 0; i < L; i++) {
            farthest = Math.max(farthest, maxReach[i]);
            if (farthest <= i) return -1;          // cannot advance past point i
            if (i == end) { taps++; end = farthest; }
        }
        return taps;
    }
}
C++
CPP
#include <vector>
#include <algorithm>
using namespace std;

class Solution {
public:
    int minRepeaters(int L, vector<int>& cover) {
        // maxReach[p] = farthest right coord coverable by a repeater with left endpoint p
        vector<int> maxReach(L + 1, 0);
        for (int i = 0; i <= L; i++) {
            int left  = max(0, i - cover[i]);
            int right = min(L, i + cover[i]);
            maxReach[left] = max(maxReach[left], right);
        }

        int taps = 0, end = 0, farthest = 0;
        for (int i = 0; i < L; i++) {
            farthest = max(farthest, maxReach[i]);
            if (farthest <= i) return -1;          // stuck at point i
            if (i == end) { taps++; end = farthest; }
        }
        return taps;
    }
};
C
C
#include <stdio.h>
#include <stdlib.h>

int minRepeaters(int L, int* cover, int coverSize) {
    /* maxReach[p] = farthest right coordinate coverable by any repeater
       whose (clamped) left endpoint is p. */
    int* maxReach = (int*)calloc(L + 1, sizeof(int));
    for (int i = 0; i <= L; i++) {
        int left = i - cover[i] < 0 ? 0 : i - cover[i];
        int right = i + cover[i] > L ? L : i + cover[i];
        if (right > maxReach[left]) maxReach[left] = right;
    }

    int taps = 0, end = 0, farthest = 0;
    for (int i = 0; i < L; i++) {
        if (maxReach[i] > farthest) farthest = maxReach[i];
        if (farthest <= i) { free(maxReach); return -1; } /* cannot pass point i */
        if (i == end) { taps++; end = farthest; }
    }
    free(maxReach);
    return taps;
}

#### Follow-up
questions
What if the covering intervals are given directly as arbitrary real-valued [start, end] pairs instead of derived from integer radii? (Sort by start, apply the same greedy.)
Return the actual set of repeaters to switch on, not just the count.
Minimize total power consumption (the sum of the chosen radii) instead of the number of repeaters, and discuss why plain greedy no longer works.
Solve it with dynamic programming (dp[x] = fewest repeaters to cover [0, x]) and compare its O(L * maxRadius) cost to the O(L) greedy.

