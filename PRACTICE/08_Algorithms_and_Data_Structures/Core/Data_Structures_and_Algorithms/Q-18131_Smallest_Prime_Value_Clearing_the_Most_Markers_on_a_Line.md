---
type: "practice"
difficulty: "Other"
---

# Smallest Prime Value Clearing the Most Markers on a Line

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Hash Table](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Hash_Table.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given a 0-indexed array points of positive integers, each representing the location of a marker on a number line, and a positive integer gap.

A sweeper can be primed with one of the marker values points[i]. Once primed with points[i], it clears every marker whose value can be written as points[i] + k * gap, where k is any non-negative integer (so it clears points[i] itself and every equally-spaced value above it that appears in the array). You want to clear the maximum possible number of markers.

Return the minimum priming value points[i] that achieves this maximum clearance.

Key observation for the statement: two markers can be cleared by the same priming value only if they leave the same remainder when divided by gap. Within one such remainder group, priming with the smallest member of the group clears the entire group (every other member is that smallest value plus a non-negative multiple of gap). So the best clearance for a group equals its size, and the ideal priming value for that group is its smallest member.

Input

points: a list of positive integers (marker locations).
gap: a positive integer spacing.

Output

The smallest marker value you can prime the sweeper with so that the number of markers cleared is maximized.

#### Cons
traints

1 <= points.length <= 10^5
1 <= points[i] <= 10^9
1 <= gap <= 10^9

#### Example
s & test cases
TEXT
- Input: `points = [3,7,8,1,1,5], gap = 2` -> Output: `1`
  Why: markers 3,7,1,1,5 all leave remainder 1 (a group of size 5); priming with the group's smallest value 1 clears all five, the maximum possible.
- Input: `points = [1,3,5,2,4,6], gap = 2` -> Output: `1`
  Why: odds {1,3,5} and evens {2,4,6} are both groups of size 3 (a tie); the odds' smallest member is 1 and the evens' is 2, so the smallest priming value among the best groups is 1.
- Input: `points = [6,2,5], gap = 100` -> Output: `2`
  Why: each value has a distinct remainder, so every group has size 1; among all size-1 groups the smallest member is 2.
- Input: `points = [9], gap = 4` -> Output: `9`
  Why: a single marker forms the only group; its smallest (and only) member is 9 (minimum-size input edge case).
- Input: `points = [10,20,30,40], gap = 5` -> Output: `10`
  Why: all four values share remainder 0 (group of size 4); priming with the smallest, 10, clears every marker.
- Input: `points = [4,4,4], gap = 3` -> Output: `4`
  Why: duplicates all share the same remainder and value; the group of size 3 has smallest member 4.

#### Approach

Any priming value only ever clears markers congruent to it modulo gap, because clearing points[i] + k*gap preserves the remainder points[i] % gap. So partition the markers by their remainder modulo gap: markers in the same remainder class form an arithmetic progression, and priming with the smallest value in a class clears the whole class (every other member is that minimum plus a non-negative multiple of gap). Therefore the maximum clearance for a class equals its size, and its ideal priming value is its minimum member. Build a hash map from remainder to (group size, group minimum) in one pass, then scan the groups keeping the one with the largest size, breaking ties by the smallest group-minimum, and return that minimum. This is a grouping/counting-by-remainder (modular bucketing) pattern.

Complexity

Time: O(n) — a single pass over the n markers to build the remainder groups, then a pass over the at-most-n distinct remainders to pick the best. Space: O(n) — the hash map stores one entry (count and running minimum) per distinct remainder, which is at most n. Justification: every marker is hashed by points[i] % gap exactly once and each group lookup/update is O(1) amortized.

Python
PYTHON
from typing import List

class Solution:
    def minPrimingValue(self, points: List[int], gap: int) -> int:
        count = {}     # remainder -> group size
        min_val = {}   # remainder -> smallest value in the group
        for v in points:
            r = v % gap
            count[r] = count.get(r, 0) + 1
            min_val[r] = v if r not in min_val else min(min_val[r], v)

        best_cnt = -1
        best_min = 0
        for r, c in count.items():
            m = min_val[r]
            if c > best_cnt or (c == best_cnt and m < best_min):
                best_cnt = c
                best_min = m
        return best_min
JavaScript
JAVASCRIPT
function minPrimingValue(points, gap) {

#### cons
t count = new Map();   // remainder -> group size

#### cons
t minVal = new Map();  // remainder -> smallest value in the group
    for (const v of points) {

#### cons
t r = v % gap;
        count.set(r, (count.get(r) || 0) + 1);
        minVal.set(r, minVal.has(r) ? Math.min(minVal.get(r), v) : v);
    }
    let bestCnt = -1;
    let bestMin = 0;
    for (const [r, c] of count) {

#### cons
t m = minVal.get(r);
        if (c > bestCnt || (c === bestCnt && m < bestMin)) {
            bestCnt = c;
            bestMin = m;
        }
    }
    return bestMin;
}
Java
JAVA
import java.util.HashMap;
import java.util.Map;

class Solution {
  public long minPrimingValue(long[] points, long gap) {
    Map<Long, int[]> count = new HashMap<>();          // remainder -> {count}
    Map<Long, Long> minVal = new HashMap<>();           // remainder -> smallest value
    for (long v : points) {
      long r = v % gap;
      count.computeIfAbsent(r, k -> new int[1])[0]++;
      minVal.merge(r, v, Math::min);
    }
    int bestCnt = -1;
    long bestMin = 0;
    for (Map.Entry<Long, int[]> e : count.entrySet()) {
      int c = e.getValue()[0];
      long m = minVal.get(e.getKey());
      if (c > bestCnt || (c == bestCnt && m < bestMin)) {
        bestCnt = c;
        bestMin = m;
      }
    }
    return bestMin;
  }
}
C++
CPP
#include <unordered_map>
#include <vector>
using namespace std;

class Solution {
 public:
  long long minPrimingValue(vector<long long>& points, long long gap) {
    // remainder -> {count, smallest value in that remainder group}
    unordered_map<long long, pair<int, long long>> groups;
    for (long long v : points) {
      long long r = v % gap;
      auto it = groups.find(r);
      if (it == groups.end()) groups[r] = {1, v};
      else { it->second.first++; it->second.second = min(it->second.second, v); }
    }
    int bestCnt = -1;
    long long bestMin = 0;
    for (auto& [r, g] : groups) {
      if (g.first > bestCnt || (g.first == bestCnt && g.second < bestMin)) {
        bestCnt = g.first;
        bestMin = g.second;
      }
    }
    return bestMin;
  }
};
C
C
#include <stdlib.h>

/* Open-addressing hash map keyed by remainder; tracks group count and group minimum. */
long minPrimingValue(long* points, int n, long gap, int* dummy) {
    (void)dummy;
    int cap = 1;
    while (cap < n * 2) cap <<= 1;
    long* keys = (long*)malloc(sizeof(long) * cap);
    int*  cnt  = (int*) malloc(sizeof(int)  * cap);
    long* mn   = (long*)malloc(sizeof(long) * cap);
    char* used = (char*)calloc(cap, 1);

    for (int i = 0; i < n; ++i) {
        long r = points[i] % gap;
        int idx = (int)((r * 1000000007L) & (cap - 1));
        while (used[idx] && keys[idx] != r) idx = (idx + 1) & (cap - 1);
        if (!used[idx]) {
            used[idx] = 1; keys[idx] = r; cnt[idx] = 1; mn[idx] = points[i];
        } else {
            cnt[idx]++;
            if (points[i] < mn[idx]) mn[idx] = points[i];
        }
    }

    int bestCnt = -1;
    long bestMin = 0;
    for (int i = 0; i < cap; ++i) {
        if (used[i] && (cnt[i] > bestCnt || (cnt[i] == bestCnt && mn[i] < bestMin))) {
            bestCnt = cnt[i];
            bestMin = mn[i];
        }
    }
    free(keys); free(cnt); free(mn); free(used);
    return bestMin;
}

#### Follow-up
questions
Why is it always optimal to prime with the smallest member of a remainder group rather than any larger member, given the non-negative multiplier constraint?
What if the sweeper could clear markers both above and below (any integer multiplier k, positive or negative) — does the answer or the grouping change?
If you also had to report how many markers get cleared by the returned value, how would you extend the solution?
How would the approach change if gap could be 0, or if marker values could be negative?

