---
type: "practice"
difficulty: "Other"
---

# Maximum Combined Reward with a Fixed Split

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Greedy](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Greedy.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

A sorting line has n numbered bins, indexed from 0 to n - 1. Each bin must be emptied by exactly one of two automated pickers, picker A or picker B.

For every bin i:

If picker A empties it, the line earns gainA[i] points.
If picker B empties it, the line earns gainB[i] points.

You are given two positive-integer arrays gainA and gainB of equal length n, and a non-negative integer m. Picker A must empty exactly m of the bins, and picker B empties the remaining n - m bins.

Return the maximum total number of points the line can earn when the bins are split this way.

#### Cons
traints:

1 <= n == gainA.length == gainB.length <= 10^5
1 <= gainA[i], gainB[i] <= 1000
0 <= m <= n

#### Example
s & test cases
TEXT
Input: gainA = [2,3,5,6], gainB = [7,7,2,1], m = 2 -> Output: 25
Why: A empties bins 2 and 3 (5 + 6), B empties bins 0 and 1 (7 + 7); 11 + 14 = 25 is optimal.

Input: gainA = [3,3], gainB = [3,3], m = 2 -> Output: 6
Why: A must empty both bins; every choice yields the same total, 3 + 3 = 6 (all values equal edge case).

Input: gainA = [9,2,2], gainB = [1,8,8], m = 0 -> Output: 17
Why: A empties nothing, so all bins go to B: 1 + 8 + 8 = 17 (m = 0 boundary).

Input: gainA = [9,2,2], gainB = [1,8,8], m = 3 -> Output: 13
Why: A must empty every bin: 9 + 2 + 2 = 13 (m = n boundary).

Input: gainA = [10], gainB = [4], m = 1 -> Output: 10
Why: single bin assigned to A because m = 1 forces it (minimal input).

#### Approach

Exchange-argument greedy. First assume picker B handles every bin, accumulating sum(gainB) as a baseline. Reassigning bin i to picker A changes the total by exactly gainA[i] - gainB[i]. We must perform exactly m such reassignments and want the total change as large as possible, so we sort the differences in descending order and add the top m of them to the baseline. Because each bin's contribution is independent of the others, greedily picking the m largest deltas is optimal.

Complexity

Time: O(n log n) — dominated by sorting the n differences (the two linear passes are O(n)). Space: O(n) for the array of differences.

Python
PYTHON
def max_combined_reward(gainA, gainB, m):
    n = len(gainA)
    total = sum(gainB)                 # start with everything on picker B
    diffs = [gainA[i] - gainB[i] for i in range(n)]  # gain of moving bin i to A
    diffs.sort(reverse=True)           # largest gains first
    for i in range(m):                 # take the m most valuable switches
        total += diffs[i]
    return total
JavaScript
JAVASCRIPT
function maxCombinedReward(gainA, gainB, m) {

#### cons
t n = gainA.length;
    let total = 0;

#### cons
t diffs = new Array(n);
    for (let i = 0; i < n; ++i) {
        total += gainB[i];                 // begin with all bins on picker B
        diffs[i] = gainA[i] - gainB[i];    // gain from moving bin i to picker A
    }
    diffs.sort((a, b) => b - a);           // descending
    for (let i = 0; i < m; ++i) total += diffs[i]; // best m switches
    return total;
}
Java
JAVA
import java.util.*;

class Solution {
    public long maxCombinedReward(int[] gainA, int[] gainB, int m) {
        int n = gainA.length;
        long total = 0;
        Integer[] diffs = new Integer[n];
        for (int i = 0; i < n; ++i) {
            total += gainB[i];               // begin with every bin on picker B
            diffs[i] = gainA[i] - gainB[i];  // benefit of switching bin i to A
        }
        Arrays.sort(diffs, Collections.reverseOrder()); // descending
        for (int i = 0; i < m; ++i) total += diffs[i];  // best m switches
        return total;
    }
}
C++
CPP
#include <vector>
#include <algorithm>
using namespace std;

class Solution {
public:
    long long maxCombinedReward(vector<int>& gainA, vector<int>& gainB, int m) {
        int n = gainA.size();
        long long total = 0;
        vector<int> diffs(n);
        for (int i = 0; i < n; ++i) {
            total += gainB[i];               // start with all bins on picker B
            diffs[i] = gainA[i] - gainB[i];  // gain from moving bin i to picker A
        }
        sort(diffs.rbegin(), diffs.rend());  // largest gains first
        for (int i = 0; i < m; ++i) total += diffs[i];
        return total;
    }
};
C
C
#include <stdlib.h>

static int cmpDesc(const void* a, const void* b) {
    int x = *(const int*)a, y = *(const int*)b;
    return (x < y) - (x > y);   /* descending order */
}

long long maxCombinedReward(int* gainA, int* gainB, int n, int m) {
    long long total = 0;
    int* diffs = (int*)malloc(sizeof(int) * n);
    for (int i = 0; i < n; ++i) {
        total += gainB[i];              /* start with everything on picker B */
        diffs[i] = gainA[i] - gainB[i]; /* net change if this bin moves to A */
    }
    qsort(diffs, n, sizeof(int), cmpDesc);
    for (int i = 0; i < m; ++i) total += diffs[i]; /* take the m best switches */
    free(diffs);
    return total;
}

#### Follow-up
questions
What changes if picker A may empty AT MOST m bins instead of exactly m?
Can you achieve O(n) expected time using quickselect / a size-bounded heap instead of a full sort?
Extend to three or more pickers, each with a fixed quota of bins.
Instead of maximizing the sum, maximize the minimum of the two pickers' individual totals.

