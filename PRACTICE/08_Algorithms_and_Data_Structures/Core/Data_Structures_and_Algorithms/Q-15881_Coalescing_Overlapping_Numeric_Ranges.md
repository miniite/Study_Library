---
type: "practice"
difficulty: "Other"
---

# Coalescing Overlapping Numeric Ranges

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Sorting](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Sorting.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given a list ranges, where each element ranges[i] = [start_i, end_i] describes a closed numeric range covering every value from start_i to end_i inclusive (with start_i <= end_i). Some of these ranges may overlap one another or merely touch at an endpoint.

Two ranges are considered to overlap if they share at least one point — and ranges that only touch (for example [2, 6] and [6, 8]) count as overlapping and must be merged. Combine every group of ranges that overlap directly or transitively into a single range, and return the resulting list of non-overlapping ranges that together cover exactly the same set of values as the input.

The returned ranges may be given in any order, though returning them sorted by start value is natural.

Input

ranges: a list of n ranges, each a two-element array [start_i, end_i] with start_i <= end_i.

Output

A list of merged, mutually non-overlapping ranges covering all the input ranges.

#### Cons
traints

1 <= n <= 10^4
Each ranges[i] has exactly 2 elements.
0 <= start_i <= end_i <= 10^4

#### Example
s & test cases
TEXT
Input: ranges = [[1,4],[2,5],[7,9],[12,15]] -> Output: [[1,5],[7,9],[12,15]]
Why: [1,4] and [2,5] overlap and merge into [1,5]; the other two are disjoint and stay as-is.

Input: ranges = [[2,6],[6,8]] -> Output: [[2,8]]
Why: the ranges touch at the point 6, which counts as overlapping, so they merge into [2,8].

Input: ranges = [[3,7]] -> Output: [[3,7]]
Why: a single range has nothing to merge with and is returned unchanged.

Input: ranges = [[1,10],[2,3],[4,5]] -> Output: [[1,10]]
Why: both [2,3] and [4,5] are fully contained inside [1,10]; the union is just [1,10].

Input: ranges = [[8,10],[1,3],[2,6],[15,18]] -> Output: [[1,6],[8,10],[15,18]]
Why: input is unsorted; after ordering by start, [1,3] and [2,6] merge to [1,6] while [8,10] and [15,18] remain separate.

Input: ranges = [[4,4],[4,4]] -> Output: [[4,4]]
Why: duplicate degenerate ranges (a single point) collapse into one range.

#### Approach

Sort-then-sweep. After sorting the ranges by their start value, any two ranges that belong to the same merged group become adjacent in the sorted order, so a single left-to-right pass suffices. Keep the most recently emitted range as the "current" range. For each next range, if its start is strictly greater than the current range's end, there is a gap and it begins a new output range; otherwise it overlaps (or touches) the current range and we extend the current end to the maximum of the two ends. Using < (not <=) for the gap test is what makes touching ranges like [2,6] and [6,8] merge. Sorting guarantees we never need to look backward, because once a range's start exceeds the current end, all later ranges have even larger starts and cannot reconnect to the closed group.

Complexity

Time: O(n log n) — dominated by sorting the ranges by start value; the single merge sweep afterward is O(n). Space: O(n) for the output list (O(log n) to O(n) auxiliary for the sort depending on the implementation), beyond which only O(1) extra state is used.

Python
PYTHON
from typing import List

def merge_ranges(ranges: List[List[int]]) -> List[List[int]]:
    ans: List[List[int]] = []
    for start, end in sorted(ranges):
        if not ans or ans[-1][1] < start:      # no overlap: start a new range
            ans.append([start, end])
        else:                                   # overlap: extend the current end
            ans[-1][1] = max(ans[-1][1], end)
    return ans
JavaScript
JAVASCRIPT
function mergeRanges(ranges) {
    ranges.sort((a, b) => a[0] - b[0]);

#### cons
t ans = [];
    for (const [start, end] of ranges) {

#### cons
t last = ans[ans.length - 1];
        if (ans.length === 0 || last[1] < start) {   // no overlap: new range
            ans.push([start, end]);
        } else {                                      // overlap: extend the end
            last[1] = Math.max(last[1], end);
        }
    }
    return ans;
}
Java
JAVA
import java.util.*;

class Solution {
    public int[][] mergeRanges(int[][] ranges) {
        Arrays.sort(ranges, (a, b) -> Integer.compare(a[0], b[0]));

        List<int[]> ans = new ArrayList<>();
        for (int[] r : ranges) {
            if (ans.isEmpty() || ans.get(ans.size() - 1)[1] < r[0])   // no overlap
                ans.add(new int[]{r[0], r[1]});
            else                                                       // overlap: extend
                ans.get(ans.size() - 1)[1] = Math.max(ans.get(ans.size() - 1)[1], r[1]);
        }
        return ans.toArray(new int[ans.size()][]);
    }
}
C++
CPP
#include <vector>
#include <algorithm>
using namespace std;

class Solution {
public:
    vector<vector<int>> mergeRanges(vector<vector<int>>& ranges) {
        sort(ranges.begin(), ranges.end(),
             [] (const vector<int>& a, const vector<int>& b) { return a[0] < b[0]; });

        vector<vector<int>> ans;
        for (const auto& r : ranges) {
            if (ans.empty() || ans.back()[1] < r[0])   // no overlap: new range
                ans.push_back(r);
            else                                        // overlap: extend end
                ans.back()[1] = max(ans.back()[1], r[1]);
        }
        return ans;
    }
};
C
C
#include <stdlib.h>

/* Compare two ranges by start value for qsort. */
static int cmpRange(const void* a, const void* b) {

#### cons
t int* x = *(const int**)a;

#### cons
t int* y = *(const int**)b;
    return x[0] - y[0];
}

int** mergeRanges(int** ranges, int rangesSize, int* rangesColSize,
                  int* returnSize, int** returnColumnSizes) {
    if (rangesSize == 0) {
        *returnSize = 0;
        *returnColumnSizes = NULL;
        return NULL;
    }
    qsort(ranges, rangesSize, sizeof(int*), cmpRange);

    int** ans = (int**)malloc(sizeof(int*) * rangesSize);
    *returnColumnSizes = (int*)malloc(sizeof(int) * rangesSize);
    int k = 0;

    for (int i = 0; i < rangesSize; ++i) {
        int s = ranges[i][0], e = ranges[i][1];
        if (k == 0 || ans[k - 1][1] < s) {          /* no overlap: start a new range */
            ans[k] = (int*)malloc(sizeof(int) * 2);
            ans[k][0] = s;
            ans[k][1] = e;
            (*returnColumnSizes)[k] = 2;
            ++k;
        } else if (e > ans[k - 1][1]) {              /* overlap: extend current end */
            ans[k - 1][1] = e;
        }
    }
    *returnSize = k;
    return ans;
}

#### Follow-up
questions
Given an already-merged, sorted list, insert one new range and re-merge in O(n) without re-sorting everything.
After merging, return the total number of distinct integer points covered by the ranges.
Find the maximum number of input ranges that overlap at any single point (peak coverage).
What changes if the ranges are treated as open intervals, so merely touching endpoints no longer counts as overlap?

