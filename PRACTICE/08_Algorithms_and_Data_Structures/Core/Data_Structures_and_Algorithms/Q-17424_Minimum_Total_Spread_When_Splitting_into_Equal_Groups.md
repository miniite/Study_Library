---
type: "practice"
difficulty: "Other"
---

# Minimum Total Spread When Splitting into Equal Groups

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Dynamic Programming](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Dynamic_Programming.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given an integer array values and an integer groupCount. You must partition every element of values into exactly groupCount groups of equal size, subject to one rule: no group may contain two equal numbers.

For a single group, its spread is the difference between the largest and the smallest number in that group. Your task is to partition the array so that the sum of the spreads over all groupCount groups is as small as possible, and return that minimum sum. If no valid partition exists (because the "no duplicates within a group" rule cannot be satisfied), return -1.

A group is just a collection of elements with no particular order. Every element of values must belong to exactly one group, and all groups have the same number of elements, namely len(values) / groupCount.

Input: an array values and an integer groupCount.

Output: the minimum achievable sum of group spreads, or -1 if a valid partition is impossible.

#### Cons
traints:

1 <= groupCount <= len(values) <= 16
len(values) is divisible by groupCount
1 <= values[i] <= len(values)

Note: because the array length is at most 16, the group size len(values) / groupCount is small. A valid partition is impossible exactly when some value appears more times than there are groups (it could not be spread across distinct groups).

#### Example
s & test cases
TEXT
Input: values = [4,1,4,2], groupCount = 2 -> Output: 5
Why: the two 4s must be separated, forcing groups {4,1} and {4,2}; spreads are 3 and 2, summing to 5.

Input: values = [1,2,1,3], groupCount = 2 -> Output: 3
Why: separating the two 1s gives {1,2} and {1,3}, spreads 1 and 2, total 3.

Input: values = [3,1,5,2,5,3], groupCount = 3 -> Output: 5
Why: pairing equal numbers apart, {3,5},{3,5},{1,2} yields spreads 2+2+1 = 5, the best possible.

Input: values = [2,4,4,6,4,4], groupCount = 3 -> Output: -1
Why: the value 4 occurs four times but there are only three groups, so two 4s would have to share a group.

Input: values = [3,1,2], groupCount = 3 -> Output: 0
Why: group size is 1, every group holds a single element, so every spread is 0.

Input: values = [3,1,2], groupCount = 1 -> Output: 2
Why: one group {1,2,3} is all-distinct; its spread is 3 - 1 = 2.

#### Approach

This is a bitmask (state-compression) DP over which elements have already been placed. First precompute, for every subset mask of elements whose size equals the group size, its spread (max - min) — but only if all its values are distinct; otherwise the subset is not a legal group. Then let dp[mask] be the minimum total spread achievable by partitioning exactly the elements in mask into complete groups. Start from dp[0] = 0. To extend a state, always fix the lowest-indexed element not yet covered and force it into the next group; iterate over the precomputed legal groups that contain that element and are disjoint from mask, updating dp[mask | g] = min(dp[mask | g], dp[mask] + spread[g]). Fixing the lowest uncovered element makes every partition be built in exactly one canonical order, eliminating duplicate work. The answer is dp[fullMask], or -1 if it stays infinite. A quick frequency check (any value occurring more than groupCount times) short-circuits the impossible cases.

Complexity

Time: O(2^n * S) where n = len(values) and S is the number of valid equal-size groups (at most C(n, size)). Precomputing group spreads costs O(2^n * n); the DP visits each of the 2^n covered-element masks and, from each, tries only the groups that contain the lowest uncovered element. With n <= 16 this is comfortably fast. Space: O(2^n) for the dp and inc arrays over element-subset masks.

Python
PYTHON
from typing import List
from collections import Counter

class Solution:
    def minimumIncompatibility(self, values: List[int], groupCount: int) -> int:
        n = len(values)
        size = n // groupCount
        INF = float('inf')

        # a value repeated more than groupCount times cannot be split across groups
        if size > 1 and any(c > groupCount for c in Counter(values).values()):
            return -1

        total = 1 << n
        full = total - 1
        inc = [-1] * total
        groups = []
        for mask in range(total):
            if bin(mask).count('1') != size:
                continue
            seen, lo, hi, dup = 0, 17, 0, False
            for i in range(n):
                if mask & (1 << i):
                    v = values[i]
                    if seen & (1 << v):
                        dup = True
                        break
                    seen |= (1 << v)
                    lo = min(lo, v)
                    hi = max(hi, v)
            if not dup:
                inc[mask] = hi - lo
                groups.append(mask)

        dp = [INF] * total
        dp[0] = 0
        for mask in range(total):
            if dp[mask] == INF or mask == full:
                continue
            low = 0
            while mask & (1 << low):                 # lowest uncovered element
                low += 1
            for g in groups:
                if (g & mask) == 0 and (g >> low) & 1:
                    nm = mask | g
                    if dp[mask] + inc[g] < dp[nm]:
                        dp[nm] = dp[mask] + inc[g]
        return dp[full] if dp[full] != INF else -1
JavaScript
JAVASCRIPT
function minimumIncompatibility(values, groupCount) {

#### cons
t n = values.length;

#### cons
t size = Math.floor(n / groupCount);

#### cons
t INF = Infinity;

    // a value repeated more than groupCount times cannot be split across groups
    if (size > 1) {

#### cons
t freq = new Array(n + 1).fill(0);
        for (const v of values) {
            if (++freq[v] > groupCount) return -1;
        }
    }

#### cons
t total = 1 << n, full = total - 1;

#### cons
t inc = new Int32Array(total).fill(-1);

#### cons
t groups = [];
    for (let mask = 0; mask < total; mask++) {
        if (popcount(mask) !== size) continue;
        let seen = 0, lo = 17, hi = 0, dup = false;
        for (let i = 0; i < n; i++) {
            if (mask & (1 << i)) {

#### cons
t v = values[i];
                if (seen & (1 << v)) { dup = true; break; }
                seen |= (1 << v);
                if (v < lo) lo = v;
                if (v > hi) hi = v;
            }
        }
        if (!dup) { inc[mask] = hi - lo; groups.push(mask); }
    }

#### cons
t dp = new Array(total).fill(INF);
    dp[0] = 0;
    for (let mask = 0; mask < total; mask++) {
        if (dp[mask] === INF || mask === full) continue;
        let low = 0;
        while (mask & (1 << low)) low++;             // lowest uncovered element
        for (const g of groups) {
            if (!(g & mask) && ((g >> low) & 1)) {

#### cons
t nm = mask | g;
                if (dp[mask] + inc[g] < dp[nm]) dp[nm] = dp[mask] + inc[g];
            }
        }
    }
    return dp[full] === INF ? -1 : dp[full];

    function popcount(x) {
        let c = 0;
        while (x) { x &= x - 1; c++; }
        return c;
    }
}
Java
JAVA
import java.util.*;

class Solution {
    public int minimumIncompatibility(int[] values, int groupCount) {
        int n = values.length;
        int size = n / groupCount;
        final int INF = Integer.MAX_VALUE / 2;

        // a value repeated more than groupCount times cannot be split across groups
        if (size > 1) {
            int[] freq = new int[n + 1];
            for (int v : values)
                if (++freq[v] > groupCount) return -1;
        }

        int total = 1 << n, full = total - 1;
        int[] inc = new int[total];
        Arrays.fill(inc, -1);
        List<Integer> groups = new ArrayList<>();
        for (int mask = 0; mask < total; mask++) {
            if (Integer.bitCount(mask) != size) continue;
            int seen = 0, lo = 17, hi = 0; boolean dup = false;
            for (int i = 0; i < n; i++) {
                if ((mask & (1 << i)) != 0) {
                    int v = values[i];
                    if ((seen & (1 << v)) != 0) { dup = true; break; }
                    seen |= (1 << v);
                    lo = Math.min(lo, v); hi = Math.max(hi, v);
                }
            }
            if (!dup) { inc[mask] = hi - lo; groups.add(mask); }
        }

        int[] dp = new int[total];
        Arrays.fill(dp, INF);
        dp[0] = 0;
        for (int mask = 0; mask < total; mask++) {
            if (dp[mask] == INF || mask == full) continue;
            int low = 0;
            while ((mask & (1 << low)) != 0) low++;       // lowest uncovered element
            for (int g : groups) {
                if ((g & mask) == 0 && ((g >> low) & 1) != 0) {
                    int nm = mask | g;
                    dp[nm] = Math.min(dp[nm], dp[mask] + inc[g]);
                }
            }
        }
        return dp[full] == INF ? -1 : dp[full];
    }
}
C++
CPP
#include <vector>
#include <algorithm>
using namespace std;

class Solution {
public:
    int minimumIncompatibility(vector<int>& values, int groupCount) {
        int n = values.size();
        int size = n / groupCount;

#### cons
t int INF = 1e9;

        // a value repeated more than groupCount times cannot be split across groups
        if (size > 1) {
            vector<int> freq(n + 1, 0);
            for (int v : values)
                if (++freq[v] > groupCount) return -1;
        }

        int total = 1 << n, full = total - 1;
        vector<int> inc(total, -1);
        vector<int> groups;
        for (int mask = 0; mask < total; ++mask) {
            if (__builtin_popcount(mask) != size) continue;
            int seen = 0, lo = 17, hi = 0; bool dup = false;
            for (int i = 0; i < n; ++i) {
                if (mask & (1 << i)) {
                    int v = values[i];
                    if (seen & (1 << v)) { dup = true; break; }
                    seen |= (1 << v);
                    lo = min(lo, v); hi = max(hi, v);
                }
            }
            if (!dup) { inc[mask] = hi - lo; groups.push_back(mask); }
        }

        vector<int> dp(total, INF);
        dp[0] = 0;
        for (int mask = 0; mask < total; ++mask) {
            if (dp[mask] == INF || mask == full) continue;
            int low = 0;
            while (mask & (1 << low)) ++low;             // lowest uncovered element
            for (int g : groups) {
                if (!(g & mask) && ((g >> low) & 1)) {
                    int nm = mask | g;
                    dp[nm] = min(dp[nm], dp[mask] + inc[g]);
                }
            }
        }
        return dp[full] == INF ? -1 : dp[full];
    }
};
C
C
#include <stdlib.h>
#include <limits.h>

static int popcnt(int x) { int c = 0; while (x) { x &= x - 1; c++; } return c; }

int minimumIncompatibility(int* values, int valuesSize, int groupCount) {
    int n = valuesSize;
    int size = n / groupCount;

#### cons
t int INF = INT_MAX / 2;

    /* a value repeated more than groupCount times cannot be split across groups */
    if (size > 1) {
        int freq[17] = {0};
        for (int i = 0; i < n; i++)
            if (++freq[values[i]] > groupCount) return -1;
    }

    int total = 1 << n, full = total - 1;
    int* inc    = (int*) malloc(sizeof(int) * total);
    int* groups = (int*) malloc(sizeof(int) * total);
    int gCount = 0;
    for (int mask = 0; mask < total; mask++) {
        inc[mask] = -1;
        if (popcnt(mask) != size) continue;
        int seen = 0, lo = 17, hi = 0, dup = 0;
        for (int i = 0; i < n; i++) {
            if (mask & (1 << i)) {
                int v = values[i];
                if (seen & (1 << v)) { dup = 1; break; }
                seen |= (1 << v);
                if (v < lo) lo = v;
                if (v > hi) hi = v;
            }
        }
        if (!dup) { inc[mask] = hi - lo; groups[gCount++] = mask; }
    }

    int* dp = (int*) malloc(sizeof(int) * total);
    for (int i = 0; i < total; i++) dp[i] = INF;
    dp[0] = 0;
    for (int mask = 0; mask < total; mask++) {
        if (dp[mask] == INF || mask == full) continue;
        int low = 0;
        while (mask & (1 << low)) low++;              /* lowest uncovered element */
        for (int gi = 0; gi < gCount; gi++) {
            int g = groups[gi];
            if (!(g & mask) && ((g >> low) & 1)) {
                int nm = mask | g;
                if (dp[mask] + inc[g] < dp[nm]) dp[nm] = dp[mask] + inc[g];
            }
        }
    }
    int ans = dp[full] == INF ? -1 : dp[full];
    free(inc); free(groups); free(dp);
    return ans;
}

#### Follow-up
questions
What changes if the groups are allowed to have unequal sizes?
Instead of minimizing the sum of spreads, minimize the maximum single-group spread (a minimax variant).
Beyond n = 16 the 2^n DP is infeasible — what heuristic or approximation (e.g., greedy pairing after sorting) would you fall back on, and when is it exact?
Return the actual assignment of elements to groups that achieves the minimum, not just the value.

