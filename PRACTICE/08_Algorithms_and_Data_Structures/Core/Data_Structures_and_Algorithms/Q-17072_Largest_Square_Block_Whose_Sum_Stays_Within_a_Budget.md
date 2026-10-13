---
type: "practice"
difficulty: "Other"
---

# Largest Square Block Whose Sum Stays Within a Budget

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Prefix Sum](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Prefix_Sum.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given a grid of non-negative integers with rows rows and cols columns, plus an integer limit.

#### Cons
ider every axis-aligned square sub-block of the grid (a square is a contiguous region whose height equals its width). The weight of a sub-block is the sum of all the integers it covers.

Return the largest possible side length of a square sub-block whose weight is at most limit. If no square — not even a single cell — has weight at most limit, return 0.

Input

grid: a 2D array of size rows x cols of non-negative integers.
limit: a non-negative integer (the maximum allowed weight).

Output

A single integer: the maximum side length k such that some k x k square inside the grid has a sum less than or equal to limit, or 0 if none exists.

#### Cons
traints

1 <= rows, cols <= 300
0 <= grid[i][j] <= 10^4
0 <= limit <= 10^5

#### Example
s & test cases
TEXT
Input: grid = [[2,1,2],[1,3,1],[2,1,2]], limit = 8 -> Output: 2
Why: The 2x2 top-left block sums to 2+1+1+3 = 7 <= 8, but the full 3x3 block sums to 15 > 8, so the best side length is 2.

Input: grid = [[5,5],[5,5]], limit = 4 -> Output: 0
Why: Even a single cell weighs 5, which already exceeds the limit of 4, so no valid square exists.

Input: grid = [[0]], limit = 0 -> Output: 1
Why: The lone cell has weight 0 <= 0, so a 1x1 square qualifies. Edge case: minimal grid and a zero limit.

Input: grid = [[1,2],[3,4]], limit = 10 -> Output: 2
Why: The whole 2x2 block sums to exactly 10, which is <= 10, so the maximum side length is 2 (boundary equality).

Input: grid = [[4,4,4],[4,4,4]], limit = 15 -> Output: 1
Why: Any 2x2 block sums to 16 > 15, but a single cell (4) fits, so the answer is 1.

Input: grid = [[1,1,1,1],[1,1,1,1],[1,1,1,1],[1,1,1,1]], limit = 9 -> Output: 3
Why: A 3x3 block of ones sums to 9 <= 9, while a 4x4 block sums to 16 > 9. Tests a larger square and equality at the boundary.

#### Approach

Build a 2D prefix-sum (integral image) so the sum of any square can be read in O(1) via inclusion-exclusion: pre[i][j] - pre[i-k][j] - pre[i][j-k] + pre[i-k][j-k]. The key optimization is monotonicity: once a square of side best has been found, no smaller side is worth checking, so for each new bottom-right corner we only attempt side best + 1 (and keep growing while it still fits). Because best never decreases and is capped by the smaller dimension, the total work beyond the prefix table is linear. Pattern: 2D prefix sums + a monotone "expand the current answer" sweep (a linear alternative to binary-searching the side length).

Complexity

Time: O(rows * cols). Building the 2D prefix-sum table is O(rows * cols). In the scan, each cell performs one O(1) prefix-sum probe for the current candidate side; the side length best only ever increases and is bounded by min(rows, cols), so the extra "grow" iterations sum to at most min(rows, cols) across the entire scan — amortized O(1) per cell. Space: O(rows * cols) for the prefix-sum table.

Python
PYTHON
def max_square_side(grid, limit):
    rows, cols = len(grid), len(grid[0])
    pre = [[0] * (cols + 1) for _ in range(rows + 1)]

    for i in range(1, rows + 1):
        for j in range(1, cols + 1):
            pre[i][j] = (grid[i - 1][j - 1] + pre[i - 1][j]
                         + pre[i][j - 1] - pre[i - 1][j - 1])

    best = 0
    for i in range(1, rows + 1):
        for j in range(1, cols + 1):
            k = best + 1                          # only ever try to beat the current best
            while i >= k and j >= k:
                s = (pre[i][j] - pre[i - k][j]
                     - pre[i][j - k] + pre[i - k][j - k])
                if s <= limit:
                    best = k
                    k = best + 1                  # success: try to grow further
                else:
                    break
    return best
JavaScript
JAVASCRIPT
function maxSquareSide(grid, limit) {

#### cons
t rows = grid.length, cols = grid[0].length;

#### cons
t pre = Array.from({ length: rows + 1 },
                           () => new Array(cols + 1).fill(0));

    for (let i = 1; i <= rows; i++)
        for (let j = 1; j <= cols; j++)
            pre[i][j] = grid[i - 1][j - 1] + pre[i - 1][j]
                        + pre[i][j - 1] - pre[i - 1][j - 1];

    let best = 0;
    for (let i = 1; i <= rows; i++) {
        for (let j = 1; j <= cols; j++) {
            let k = best + 1;                     // only try to beat the current best
            while (i >= k && j >= k) {

#### cons
t sum = pre[i][j] - pre[i - k][j]
                            - pre[i][j - k] + pre[i - k][j - k];
                if (sum <= limit) {
                    best = k;
                    k = best + 1;                 // success: try to grow further
                } else {
                    break;
                }
            }
        }
    }
    return best;
}
Java
JAVA
class Solution {
    public int maxSquareSide(int[][] grid, int limit) {
        int rows = grid.length, cols = grid[0].length;
        long[][] pre = new long[rows + 1][cols + 1];

        for (int i = 1; i <= rows; i++)
            for (int j = 1; j <= cols; j++)
                pre[i][j] = grid[i - 1][j - 1] + pre[i - 1][j]
                            + pre[i][j - 1] - pre[i - 1][j - 1];

        int best = 0;
        for (int i = 1; i <= rows; i++) {
            for (int j = 1; j <= cols; j++) {
                int k = best + 1;                 // only try to beat the current best
                while (i >= k && j >= k) {
                    long sum = pre[i][j] - pre[i - k][j]
                               - pre[i][j - k] + pre[i - k][j - k];
                    if (sum <= limit) {
                        best = k;
                        k = best + 1;             // success: try to grow further
                    } else {
                        break;
                    }
                }
            }
        }
        return best;
    }
}
C++
CPP
#include <vector>
#include <algorithm>
using namespace std;

class Solution {
public:
    int maxSquareSide(vector<vector<int>>& grid, int limit) {
        int rows = grid.size(), cols = grid[0].size();
        vector<vector<long long>> pre(rows + 1, vector<long long>(cols + 1, 0));

        for (int i = 1; i <= rows; ++i)
            for (int j = 1; j <= cols; ++j)
                pre[i][j] = grid[i - 1][j - 1] + pre[i - 1][j]
                            + pre[i][j - 1] - pre[i - 1][j - 1];

        int best = 0;
        for (int i = 1; i <= rows; ++i) {
            for (int j = 1; j <= cols; ++j) {
                int k = best + 1;                 // only try to beat the current best
                while (i >= k && j >= k) {
                    long long sum = pre[i][j] - pre[i - k][j]
                                    - pre[i][j - k] + pre[i - k][j - k];
                    if (sum <= limit) {
                        best = k;
                        k = best + 1;             // success: try to grow further
                    } else {
                        break;
                    }
                }
            }
        }
        return best;
    }
};
C
C
#include <stdlib.h>

int maxSquareSide(int** grid, int rows, int cols, int limit) {
    /* pre[i][j] = sum of the sub-rectangle from (0,0) to (i-1,j-1) inclusive */
    long** pre = (long**)malloc((rows + 1) * sizeof(long*));
    for (int i = 0; i <= rows; i++)
        pre[i] = (long*)calloc(cols + 1, sizeof(long));

    for (int i = 1; i <= rows; i++)
        for (int j = 1; j <= cols; j++)
            pre[i][j] = grid[i - 1][j - 1] + pre[i - 1][j]
                        + pre[i][j - 1] - pre[i - 1][j - 1];

    int best = 0;
    for (int i = 1; i <= rows; i++) {
        for (int j = 1; j <= cols; j++) {
            int k = best + 1;               /* only ever try to beat the current best */
            while (i >= k && j >= k) {
                long sum = pre[i][j] - pre[i - k][j]
                           - pre[i][j - k] + pre[i - k][j - k];
                if (sum <= (long)limit) {
                    best = k;
                    k = best + 1;           /* success: try to grow further */
                } else {
                    break;
                }
            }
        }
    }

    for (int i = 0; i <= rows; i++) free(pre[i]);
    free(pre);
    return best;
}

#### Follow-up
questions
What if you needed the largest rectangle (not just a square) with sum at most the limit? How does the search space change?
What if the grid could contain negative values? Prefix sums still work, but the monotone "grow the answer" trick breaks — how would you adapt?
Given many queries with different limits on the same fixed grid, how do you answer each quickly? (Precompute the prefix table once, then binary-search the side per query.)
Return the actual top-left coordinates of one maximum-side square, not just its side length.

