---
type: "practice"
difficulty: "Other"
---

# Longest Pairwise-Divisible Subset

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Dynamic Programming](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Dynamic_Programming.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given an array elements of distinct positive integers. Find the largest subset of these integers such that, for every pair of chosen values (a, b) in the subset, one divides the other — that is, either a % b == 0 or b % a == 0.

Return any one such subset of maximum possible size, in any order. If the input has a single element, that element by itself is a valid answer of size one.

If several different subsets tie for the maximum size, returning any of them is acceptable.

Input: an array elements of distinct positive integers.

Output: a list of integers forming a largest subset in which every pair is divisibility-related.

#### Cons
traints:

1 <= elements.length <= 1000.
1 <= elements[i] <= 2 * 10^9 (values can exceed the 32-bit signed range, so store them in a 64-bit integer type where necessary).
All integers in elements are unique.

Key property to exploit: if the values are sorted in increasing order, then a subset in which every adjacent (sorted) pair satisfies the divisibility relation automatically satisfies it for every pair, because divisibility is transitive (if x | y and y | z then x | z). This reduces the "all pairs" condition to a chain condition on sorted values.

#### Example
s & test cases
TEXT
Input: elements = [1, 2, 3] -> Output: [1, 2]
Why: {1,2} is pairwise divisible (1|2); {1,3} would also be valid. Max size is 2 since 2 and 3 do not divide each other.

Input: elements = [1, 2, 4, 8] -> Output: [1, 2, 4, 8]
Why: each value divides the next, so the whole array is a valid divisibility chain of size 4.

Input: elements = [7] -> Output: [7]
Why: a single element is trivially a valid subset of size 1.

Input: elements = [4, 8, 10, 240] -> Output: [4, 8, 240]
Why: 4|8 and 8|240, forming a chain of size 3; 10 divides none of them consecutively, and no size-4 chain exists.

Input: elements = [3, 5, 7, 11] -> Output: [3]
Why: all values are pairwise coprime, so no two share a divisibility relation; any single element is a maximal subset of size 1.

Input: elements = [2, 3, 6, 12, 24, 36] -> Output: [2, 6, 12, 24]
Why: 2|6, 6|12, 12|24 gives a chain of length 4; {3,6,12,24} is an equally valid answer.

#### Approach

Sort the values ascending. Because divisibility is transitive, a subset is fully pairwise-divisible iff, when its members are listed in sorted order, each element is divisible by its predecessor — so the problem becomes finding the longest such "divisibility chain," a variant of Longest Increasing Subsequence. Define size[i] as the length of the longest chain ending exactly at elements[i], and prev[i] as the index of the previous element in that best chain. For each i, scan all earlier j; if elements[i] % elements[j] == 0 and extending the chain at j beats the current best, set size[i] = size[j] + 1 and prev[i] = j. Track the global maximum index, then reconstruct the subset by following prev pointers backward and reversing. The pattern is DP over a sorted sequence with parent-pointer path reconstruction (LIS family).

Complexity

Time: O(n^2) — after an O(n log n) sort, the double loop compares every pair (i, j) once, doing O(1) work each, which dominates. Space: O(n) — two auxiliary arrays (size and prev) of length n, plus the output.

Python
PYTHON
from typing import List

class Solution:
    def largestDivisibleSubset(self, elements: List[int]) -> List[int]:
        n = len(elements)
        elements.sort()

        size = [1] * n      # longest chain ending at i
        prev = [-1] * n     # predecessor index in that chain
        best, best_idx = 0, 0

        for i in range(n):
            for j in range(i - 1, -1, -1):
                if elements[i] % elements[j] == 0 and size[j] + 1 > size[i]:
                    size[i] = size[j] + 1
                    prev[i] = j
            if size[i] > best:
                best, best_idx = size[i], i

        ans = []
        i = best_idx
        while i != -1:
            ans.append(elements[i])
            i = prev[i]
        ans.reverse()
        return ans
JavaScript
JAVASCRIPT
function largestDivisibleSubset(elements) {
    elements.sort((a, b) => a - b);

#### cons
t n = elements.length;

#### cons
t size = new Array(n).fill(1);   // longest chain ending at i

#### cons
t prev = new Array(n).fill(-1);  // predecessor index in that chain
    let best = 0, bestIdx = 0;

    for (let i = 0; i < n; i++) {
        for (let j = i - 1; j >= 0; j--) {
            if (elements[i] % elements[j] === 0 && size[j] + 1 > size[i]) {
                size[i] = size[j] + 1;
                prev[i] = j;
            }
        }
        if (size[i] > best) { best = size[i]; bestIdx = i; }
    }

#### cons
t ans = [];
    for (let i = bestIdx; i !== -1; i = prev[i]) ans.push(elements[i]);
    ans.reverse();
    return ans;
}
Java
JAVA
class Solution {
    public List<Long> largestDivisibleSubset(long[] elements) {
        final int n = elements.length;
        Arrays.sort(elements);

        int[] size = new int[n];   // longest chain ending at i
        int[] prev = new int[n];   // predecessor index in that chain
        Arrays.fill(size, 1);
        Arrays.fill(prev, -1);
        int best = 0, bestIdx = 0;

        for (int i = 0; i < n; ++i) {
            for (int j = i - 1; j >= 0; --j) {
                if (elements[i] % elements[j] == 0 && size[j] + 1 > size[i]) {
                    size[i] = size[j] + 1;
                    prev[i] = j;
                }
            }
            if (size[i] > best) { best = size[i]; bestIdx = i; }
        }

        LinkedList<Long> ans = new LinkedList<>();
        for (int i = bestIdx; i != -1; i = prev[i])
            ans.addFirst(elements[i]);
        return ans;
    }
}
C++
CPP
class Solution {
public:
    vector<long long> largestDivisibleSubset(vector<long long>& elements) {

#### cons
t int n = elements.size();
        sort(elements.begin(), elements.end());

        vector<int> size(n, 1);   // longest chain ending at i
        vector<int> prev(n, -1);  // predecessor index in that chain
        int best = 0, bestIdx = 0;

        for (int i = 0; i < n; ++i) {
            for (int j = i - 1; j >= 0; --j) {
                if (elements[i] % elements[j] == 0 && size[j] + 1 > size[i]) {
                    size[i] = size[j] + 1;
                    prev[i] = j;
                }
            }
            if (size[i] > best) { best = size[i]; bestIdx = i; }
        }

        vector<long long> ans;
        for (int i = bestIdx; i != -1; i = prev[i])
            ans.push_back(elements[i]);
        reverse(ans.begin(), ans.end());
        return ans;
    }
};
C
C
#include <stdlib.h>

static int cmpLong(const void* a, const void* b) {
    long x = *(const long*)a, y = *(const long*)b;
    return (x > y) - (x < y);
}

/* Returns a malloc'd array; *returnSize receives its length. */
long* largestDivisibleSubset(long* elements, int n, int* returnSize) {
    long* a = malloc(sizeof(long) * n);
    for (int i = 0; i < n; ++i) a[i] = elements[i];
    qsort(a, n, sizeof(long), cmpLong);

    int* size = malloc(sizeof(int) * n);   /* longest chain ending at i */
    int* prev = malloc(sizeof(int) * n);   /* predecessor index in that chain */
    int best = 0, bestIdx = 0;

    for (int i = 0; i < n; ++i) {
        size[i] = 1;
        prev[i] = -1;
        for (int j = i - 1; j >= 0; --j) {
            if (a[i] % a[j] == 0 && size[j] + 1 > size[i]) {
                size[i] = size[j] + 1;
                prev[i] = j;
            }
        }
        if (size[i] > best) { best = size[i]; bestIdx = i; }
    }

    long* res = malloc(sizeof(long) * best);
    int k = best;
    for (int i = bestIdx; i != -1; i = prev[i]) res[--k] = a[i];

    *returnSize = best;
    free(a); free(size); free(prev);
    return res;
}

#### Follow-up
questions
Return only the size of the largest subset without reconstructing it — can you save the prev array?
Count how many distinct largest subsets exist rather than returning one.
Explain why comparing only adjacent sorted elements is sufficient (transitivity proof) and where it would break if values were not distinct.
If the array had up to 10^5 elements, discuss why O(n^2) may be too slow and what factor/divisor-graph or hashing ideas could help on constrained value ranges.

