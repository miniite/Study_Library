---
type: "practice"
difficulty: "Other"
---

# Kth Smallest Entry of a Row-by-Column Product Grid

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Binary Search](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Binary_Search.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Imagine a grid with rows rows and cols columns whose rows and columns are both numbered starting from 1. The entry sitting in row i and column j holds the product i * j. So the top-left entry is always 1, the entry in the last row and last column is rows * cols, and in general every entry equals its row index multiplied by its column index.

You are never actually asked to build this grid (it can be enormous). Instead, imagine all rows * cols entries collected into one list and sorted in non-decreasing order. Given three integers rows, cols, and k, return the value of the k-th smallest entry in that sorted list. Duplicate values are counted separately — for example if the value 6 appears in three different cells, it occupies three consecutive positions in the sorted order.

Input

rows — the number of rows in the product grid.
cols — the number of columns in the product grid.
k — a 1-based rank; you want the k-th smallest entry.

Output

A single integer: the value of the k-th smallest entry.

#### Cons
traints

1 <= rows, cols <= 3 * 10^4
1 <= k <= rows * cols
The product rows * cols fits in a 32-bit signed integer.

#### Example
s & test cases
TEXT
Input: rows = 4, cols = 5, k = 8 -> Output: 4
Why: sorted entries are 1,2,2,3,3,4,4,4,5,... so the 8th smallest is 4 (the value 4 fills three cells).

Input: rows = 2, cols = 3, k = 6 -> Output: 6
Why: entries sorted are 1,2,2,3,4,6; the last (6th) equals the biggest entry rows*cols = 6.

Input: rows = 3, cols = 3, k = 9 -> Output: 9
Why: k equals rows*cols, so the answer is the single largest entry 3*3 = 9.

Input: rows = 6, cols = 4, k = 1 -> Output: 1
Why: the smallest entry of any product grid is always the top-left cell 1*1 = 1.

Input: rows = 1, cols = 7, k = 5 -> Output: 5
Why: a single row is just 1,2,3,4,5,6,7, so the k-th smallest is simply k when k <= cols.

#### Approach

Binary search on the answer value, not on an index. The candidate answer lies somewhere in [1, rows*cols]. For any candidate value x, the number of grid entries that are <= x is easy to count without building the grid: in row i the entries are i, 2i, 3i, ..., so exactly min(x / i, cols) of them are <= x. Summing that over all rows gives the count in O(rows). This count is monotonically non-decreasing in x, so we binary-search for the smallest x whose count is at least k. That smallest x is guaranteed to actually appear in the grid (the count jumps by at least one exactly at real grid values), which is why the boundary value is the k-th smallest entry.

Complexity

Time: O(rows * log(rows * cols)) — binary search over the value range [1, rowscols] runs O(log(rowscols)) iterations, and each iteration counts entries not exceeding a candidate value in O(rows) by scanning one row at a time. Space: O(1) — only a handful of integer counters are used; the grid itself is never materialized.

Python
PYTHON
class Solution:
    def findKthNumber(self, rows: int, cols: int, k: int) -> int:
        def count_no_greater(target: int) -> int:
            # Row i contributes min(target // i, cols) entries that are <= target.
            total = 0
            for i in range(1, rows + 1):
                total += min(target // i, cols)
            return total

        lo, hi = 1, rows * cols
        while lo < hi:
            mid = (lo + hi) // 2
            if count_no_greater(mid) >= k:
                hi = mid
            else:
                lo = mid + 1
        return lo
JavaScript
JAVASCRIPT
function findKthNumber(rows, cols, k) {

#### cons
t countNoGreater = (target) => {
        // Each row i contributes min(floor(target / i), cols) entries <= target.
        let count = 0;
        for (let i = 1; i <= rows; i++) {
            count += Math.min(Math.floor(target / i), cols);
        }
        return count;
    };

    let lo = 1;
    let hi = rows * cols;
    while (lo < hi) {

#### cons
t mid = lo + Math.floor((hi - lo) / 2);
        if (countNoGreater(mid) >= k) {
            hi = mid;
        } else {
            lo = mid + 1;
        }
    }
    return lo;
}
Java
JAVA
class Solution {
    public int findKthNumber(int rows, int cols, int k) {
        int lo = 1;
        int hi = rows * cols;
        while (lo < hi) {
            int mid = lo + (hi - lo) / 2;
            if (countNoGreater(rows, cols, mid) >= k)
                hi = mid;
            else
                lo = mid + 1;
        }
        return lo;
    }

    // Number of grid entries <= target; row i supplies min(target / i, cols).
    private int countNoGreater(int rows, int cols, int target) {
        int count = 0;
        for (int i = 1; i <= rows; i++)
            count += Math.min(target / i, cols);
        return count;
    }
}
C++
CPP
#include <algorithm>
using namespace std;

class Solution {
public:
    int findKthNumber(int rows, int cols, int k) {
        int lo = 1, hi = rows * cols;
        while (lo < hi) {
            int mid = lo + (hi - lo) / 2;
            if (countNoGreater(rows, cols, mid) >= k)
                hi = mid;
            else
                lo = mid + 1;
        }
        return lo;
    }

private:
    // Entries <= target: each row i has min(target / i, cols) of them.
    int countNoGreater(int rows, int cols, int target) {
        int count = 0;
        for (int i = 1; i <= rows; ++i)
            count += min(target / i, cols);
        return count;
    }
};
C
C
#include <stdio.h>

/* Count how many grid entries are <= target: row i contributes min(target / i, cols). */
static int countNoGreater(int rows, int cols, int target) {
    int count = 0;
    for (int i = 1; i <= rows; ++i) {
        int inRow = target / i;
        count += (inRow < cols) ? inRow : cols;
    }
    return count;
}

int findKthNumber(int rows, int cols, int k) {
    int lo = 1;
    int hi = rows * cols;
    while (lo < hi) {
        int mid = lo + (hi - lo) / 2;
        if (countNoGreater(rows, cols, mid) >= k)
            hi = mid;
        else
            lo = mid + 1;
    }
    return lo;
}

#### Follow-up
questions
How do you know the value returned by the binary search is actually present in the grid, rather than a gap between real entries?
Generalize to a grid where entry (i, j) = a[i] * b[j] for arbitrary sorted positive arrays a and b — does the counting step still work in O(rows)?
Return the k-th largest entry instead of the k-th smallest, ideally reusing the same counting routine.
Given a target value v, report its rank (how many entries are strictly smaller) in the sorted order.

