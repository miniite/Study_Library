---
type: "practice"
difficulty: "Other"
---

# Immutable Array Range-Sum Queries

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Prefix Sum](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Prefix_Sum.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given a fixed integer array values that never changes after it is created. You must answer many queries of the form: what is the sum of the elements between two indices, inclusive of both endpoints?

Design a data structure that supports:

build(values): initialize the structure from the integer array values.
rangeSum(lo, hi): return values[lo] + values[lo + 1] + ... + values[hi], where 0 <= lo <= hi < n.

Because the array is immutable and there may be a very large number of queries, each rangeSum call should run in constant time after an initial preprocessing pass.

Input: an integer array values of length n, followed by a sequence of (lo, hi) index pairs.

Output: for each (lo, hi) pair, a single integer equal to the inclusive sum of elements in that index range.

#### Cons
traints:

1 <= n <= 10^4
-10^5 <= values[i] <= 10^5
0 <= lo <= hi < n
At most 10^4 calls will be made to rangeSum.

Note: the sum of a range can be as large in magnitude as n * 10^5 = 10^9, so pick a return type that comfortably holds that value.

#### Example
s & test cases
TEXT
Input: values = [4, -1, 2, 7, -3, 5], rangeSum(0, 2) -> Output: 5
Why: 4 + (-1) + 2 = 5, a normal interior range starting at index 0.

Input: values = [4, -1, 2, 7, -3, 5], rangeSum(2, 5) -> Output: 11
Why: 2 + 7 + (-3) + 5 = 11, a range that ends at the last index.

Input: values = [4, -1, 2, 7, -3, 5], rangeSum(0, 5) -> Output: 14
Why: the whole array sums to 14; covers the full span.

Input: values = [4, -1, 2, 7, -3, 5], rangeSum(3, 3) -> Output: 7
Why: single-element range returns that element (values[3] = 7).

Input: values = [10], rangeSum(0, 0) -> Output: 10
Why: minimum-size array with the only valid query.

Input: values = [-5, -2, -8], rangeSum(0, 2) -> Output: -15
Why: all-negative array; the running total stays negative.

#### Approach

Precompute a prefix-sum array where prefix[i] holds the sum of the first i elements (prefix[0] = 0). The sum of any inclusive range [lo, hi] is then prefix[hi + 1] - prefix[lo], because the second term cancels everything before index lo. Building prefix takes one linear pass; every query afterward is a constant-time difference. This is the classic prefix-sum / cumulative-sum precomputation pattern, ideal when the data is immutable and read (queried) far more often than written.

Complexity

Time: O(n) once to build the prefix-sum array, then O(1) per rangeSum query since each answer is a single subtraction. Space: O(n) for the auxiliary prefix array (one extra entry so the empty-prefix boundary is handled without special cases).

Python
PYTHON
from itertools import accumulate
from typing import List

class RangeSum:
    def __init__(self, values: List[int]):
        # prefix[i] = sum of the first i elements
        self.prefix = [0] + list(accumulate(values))

    def range_sum(self, lo: int, hi: int) -> int:
        # inclusive sum over [lo, hi] in O(1)
        return self.prefix[hi + 1] - self.prefix[lo]
JavaScript
JAVASCRIPT
class RangeSum {

#### cons
tructor(values) {
        // prefix[i] = sum of the first i elements
        this.prefix = new Array(values.length + 1).fill(0);
        for (let i = 0; i < values.length; i++) {
            this.prefix[i + 1] = this.prefix[i] + values[i];
        }
    }

    // inclusive sum over [lo, hi] in O(1)
    rangeSum(lo, hi) {
        return this.prefix[hi + 1] - this.prefix[lo];
    }
}
Java
JAVA
class RangeSum {
    private final long[] prefix;   // prefix[i] = sum of first i elements

    public RangeSum(int[] values) {
        prefix = new long[values.length + 1];
        for (int i = 0; i < values.length; ++i)
            prefix[i + 1] = prefix[i] + values[i];
    }

    // inclusive sum over [lo, hi] in O(1)
    public long rangeSum(int lo, int hi) {
        return prefix[hi + 1] - prefix[lo];
    }
}
C++
CPP
#include <vector>
using namespace std;

class RangeSum {
public:
    explicit RangeSum(const vector<int>& values) : prefix(values.size() + 1, 0) {
        for (size_t i = 0; i < values.size(); ++i)
            prefix[i + 1] = prefix[i] + values[i];
    }

    // inclusive sum over [lo, hi] in O(1)
    long rangeSum(int lo, int hi) const {
        return prefix[hi + 1] - prefix[lo];
    }

private:
    vector<long> prefix;   // prefix[i] = sum of first i elements
};
C
C
#include <stdlib.h>

typedef struct {
    long *prefix;   /* prefix[i] = sum of first i elements */
    int   n;
} RangeSum;

/* build the structure from the immutable array */
RangeSum* rangeSumCreate(int* values, int n) {
    RangeSum* rs = (RangeSum*)malloc(sizeof(RangeSum));
    rs->n = n;
    rs->prefix = (long*)malloc((n + 1) * sizeof(long));
    rs->prefix[0] = 0;
    for (int i = 0; i < n; ++i)
        rs->prefix[i + 1] = rs->prefix[i] + values[i];
    return rs;
}

/* inclusive sum over [lo, hi] in O(1) */
long rangeSumQuery(RangeSum* rs, int lo, int hi) {
    return rs->prefix[hi + 1] - rs->prefix[lo];
}

void rangeSumFree(RangeSum* rs) {
    free(rs->prefix);
    free(rs);
}

#### Follow-up
questions
Make the array mutable: support point updates to values[i] while keeping fast range sums (use a Fenwick/Binary Indexed Tree or a segment tree, each O(log n) per operation).
Extend the design to a 2D matrix and answer rectangle sums in O(1).
The array is enormous and cannot fit in memory at once — how would you support range sums over a stream or blocks on disk?
Return the sum modulo a large prime, or answer range-minimum/range-max queries instead of range-sum.

