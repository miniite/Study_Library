---
type: "practice"
difficulty: "Other"
---

# Split Index With Smallest Average Gap

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Prefix Sum](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Prefix_Sum.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given a 0-indexed integer array values of length n.

For an index i (where 0 <= i < n), split the array into two parts:

a left part consisting of the first i + 1 elements, and
a right part consisting of the remaining n - i - 1 elements.

The average gap at index i is the absolute difference between the floor of the left part's average and the floor of the right part's average. Both averages use integer (floor) division:

left average = floor( (sum of the first i + 1 elements) / (i + 1) )
right average = floor( (sum of the last n - i - 1 elements) / (n - i - 1) )

If a part contains 0 elements, its average is defined to be 0 (this happens for the right part when i == n - 1).

Return the index with the minimum average gap. If several indices tie for the minimum, return the smallest such index.

Input

values: a 0-indexed array of non-negative integers.

Output

An integer index in [0, n - 1] that minimizes the average gap (smallest index on ties).

#### Cons
traints

1 <= n <= 10^5
0 <= values[i] <= 10^5

(Note: the total sum can exceed the range of a 32-bit integer, so use 64-bit accumulators.)

#### Example
s & test cases
TEXT
Input: values = [1,4,2,8,4,2] -> Output: 3
Why: gaps are |1-4|=3, |2-4|=2, |2-4|=2, |3-3|=0, |3-2|=1, |3-0|=3; index 3 gives 0.

Input: values = [5] -> Output: 0
Why: the only index; left avg 5, right part empty -> 0, gap 5, but it is still the answer.

Input: values = [3,3] -> Output: 0
Why: index 0 -> |3-3|=0, index 1 -> |3-0|=3; the minimum is at index 0.

Input: values = [0,0,0] -> Output: 0
Why: every gap is 0; on a tie return the smallest index — the all-equal / tie edge case.

Input: values = [100000,100000] -> Output: 0
Why: index 0 -> |100000-100000|=0; confirms 64-bit sums are needed (2e5 total) and floor division is exact.

#### Approach

This is a prefix-sum sweep. Precompute the total sum once. Then scan indices left to right, maintaining a running prefix sum of the left part; the right part's sum is simply total - prefix. For each index the left average is prefix / (i + 1) and the right average is (total - prefix) / (n - i - 1) using integer (floor) division, treating an empty right part as average 0. Track the minimum absolute difference seen so far, updating the best index only on a strictly smaller gap so that ties naturally keep the smallest index. The key insight is that a running prefix sum turns each of the n split evaluations into O(1), avoiding recomputation of sums.

Complexity

Time: O(n) — one pass to compute the total sum and a second pass that maintains a running prefix sum and evaluates each split index in O(1). Space: O(1) auxiliary — only running sums and the best index are stored. Values fit in 64-bit accumulators (max total 10^5 * 10^5 = 10^10).

Python
PYTHON
from typing import List

class Solution:
    def splitIndexSmallestAverageGap(self, values: List[int]) -> int:
        n = len(values)
        total = sum(values)
        prefix = 0
        best = -1
        best_idx = 0
        for i in range(n):
            prefix += values[i]
            left_avg = prefix // (i + 1)
            right_count = n - i - 1
            right_avg = 0 if right_count == 0 else (total - prefix) // right_count
            gap = abs(left_avg - right_avg)
            if best < 0 or gap < best:
                best = gap
                best_idx = i
        return best_idx
JavaScript
JAVASCRIPT
function splitIndexSmallestAverageGap(values) {

#### cons
t n = values.length;
    let total = 0;
    for (const v of values) total += v;

    let prefix = 0;
    let best = -1;
    let bestIdx = 0;
    for (let i = 0; i < n; i++) {
        prefix += values[i];

#### cons
t leftAvg = Math.floor(prefix / (i + 1));

#### cons
t rightCount = n - i - 1;

#### cons
t rightAvg = rightCount === 0 ? 0 : Math.floor((total - prefix) / rightCount);

#### cons
t gap = Math.abs(leftAvg - rightAvg);
        if (best < 0 || gap < best) {
            best = gap;
            bestIdx = i;
        }
    }
    return bestIdx;
}
Java
JAVA
class Solution {
    public int splitIndexSmallestAverageGap(int[] values) {
        int n = values.length;
        long total = 0;
        for (int v : values) total += v;

        long prefix = 0, best = -1;
        int bestIdx = 0;
        for (int i = 0; i < n; i++) {
            prefix += values[i];
            long leftAvg = prefix / (i + 1);
            long rightCount = n - i - 1;
            long rightAvg = rightCount == 0 ? 0 : (total - prefix) / rightCount;
            long gap = Math.abs(leftAvg - rightAvg);
            if (best < 0 || gap < best) {
                best = gap;
                bestIdx = i;
            }
        }
        return bestIdx;
    }
}
C++
CPP
#include <vector>
#include <cstdlib>
using namespace std;

class Solution {
public:
    int splitIndexSmallestAverageGap(vector<int>& values) {
        int n = (int)values.size();
        long long total = 0;
        for (int v : values) total += v;

        long long prefix = 0, best = -1;
        int bestIdx = 0;
        for (int i = 0; i < n; i++) {
            prefix += values[i];
            long long leftAvg = prefix / (long long)(i + 1);
            long long rightCount = n - i - 1;
            long long rightAvg = rightCount == 0 ? 0 : (total - prefix) / rightCount;
            long long gap = llabs(leftAvg - rightAvg);
            if (best < 0 || gap < best) {
                best = gap;
                bestIdx = i;
            }
        }
        return bestIdx;
    }
};
C
C
int splitIndexSmallestAverageGap(int* values, int valuesSize) {
    long long total = 0;
    for (int i = 0; i < valuesSize; i++) total += values[i];

    long long prefix = 0;
    long long best = -1;
    int bestIdx = 0;
    for (int i = 0; i < valuesSize; i++) {
        prefix += values[i];
        long long leftAvg = prefix / (long long)(i + 1);
        long long rightCount = (long long)(valuesSize - i - 1);
        long long rightAvg = rightCount == 0 ? 0 : (total - prefix) / rightCount;
        long long gap = leftAvg - rightAvg;
        if (gap < 0) gap = -gap;
        if (best < 0 || gap < best) {
            best = gap;
            bestIdx = i;
        }
    }
    return bestIdx;
}

#### Follow-up
questions
What overflow guarantees do you need, and how do you choose the accumulator type from the constraints?
How would the answer change if averages were rounded to the nearest integer instead of floored?
Can you return all indices that achieve the minimum gap, not just the smallest one?
How would you support streaming updates where a value changes and you must re-answer quickly?

