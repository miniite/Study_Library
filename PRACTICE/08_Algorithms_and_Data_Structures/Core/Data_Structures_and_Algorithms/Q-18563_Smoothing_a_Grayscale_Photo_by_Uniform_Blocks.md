---
type: "practice"
difficulty: "Other"
---

# Smoothing a Grayscale Photo by Uniform Blocks

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Matrix](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Matrix.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given an m x n grid pixels representing a grayscale photograph, where pixels[i][j] is the brightness of a pixel, an integer in the range [0, 255]. You are also given a non-negative integer tol (a tolerance).

Two pixels are adjacent if they share an edge (up, down, left, or right — not diagonal).

A block is a 3 x 3 sub-grid in which the absolute difference in brightness between every pair of adjacent pixels (inside that sub-grid) is at most tol. Every one of the 9 pixels in a valid block is said to belong to that block. A single pixel may belong to several blocks (because 3 x 3 windows overlap).

You must produce an m x n grid result defined as follows:

Compute, for each valid block, the average brightness of its 9 pixels, rounded down (floor) to the nearest integer.
For a pixel pixels[i][j] that belongs to at least one block, result[i][j] is the average of those per-block rounded-down averages, again rounded down to the nearest integer. Important: the averaging uses the already-rounded-down block averages, not the exact fractional block averages.
If pixels[i][j] belongs to no valid block, then result[i][j] is simply equal to pixels[i][j] (unchanged).

Return the grid result.

#### Cons
traints:

3 <= m, n <= 500
0 <= pixels[i][j] <= 255
0 <= tol <= 255

#### Example
s & test cases
TEXT
Input: pixels = [[8,11,12],[13,14,15],[16,17,18]], tol = 5 -> Output: [[13,13,13],[13,13,13],[13,13,13]]
Why: The single 3x3 window is valid (max adjacent diff is 5). Its sum is 124, floor(124/9) = 13, and every pixel belongs only to this one block. Shows floor of a non-integer block average.

Input: pixels = [[20,22,24,26],[23,25,27,29],[26,28,30,32]], tol = 3 -> Output: [[25,26,26,27],[25,26,26,27],[25,26,26,27]]
Why: Two overlapping horizontal blocks. Left block (cols 0-2) averages 225/9 = 25; right block (cols 1-3) averages 243/9 = 27. Column 0 gets 25, column 3 gets 27, and the shared columns 1-2 get floor((25+27)/2) = 26. Demonstrates overlap and averaging of averages.

Input: pixels = [[12,18,24],[16,22,28],[20,26,32],[24,30,36]], tol = 6 -> Output: [[22,22,22],[24,24,24],[24,24,24],[26,26,26]]
Why: Two overlapping vertical blocks. Top block (rows 0-2) averages 198/9 = 22; bottom block (rows 1-3) averages 234/9 = 26. Row 0 -> 22, row 3 -> 26, shared rows 1-2 -> floor((22+26)/2) = 24.

Input: pixels = [[3,5,9],[10,12,14],[15,17,19]], tol = 1 -> Output: [[3,5,9],[10,12,14],[15,17,19]]
Why: The only 3x3 window has adjacent vertical differences like |3-10| = 7 > 1, so it is invalid. No pixel belongs to any block, so the result equals the input. Edge case: no valid block.

Input: pixels = [[7,7,7],[7,7,7],[7,7,7]], tol = 0 -> Output: [[7,7,7],[7,7,7],[7,7,7]]
Why: All adjacent differences are 0 <= tol, so the single block is valid; its average is 63/9 = 7. Edge case: tol = 0 with a perfectly uniform region.

#### Approach

Directly simulate the definition. Slide a 3x3 window over every possible top-left corner (r, c) with r+2 < m and c+2 < n. For each window, check validity by comparing each cell only with its right and bottom neighbor inside the window (that covers every adjacent pair exactly once): if any such absolute difference exceeds tol, the block is invalid. For a valid block, sum its 9 cells, take the floor average sum // 9, and add that floored average into an accumulator acc[i][j] for each of the 9 cells while incrementing a count cnt[i][j]. Because the specification averages the already-rounded block averages, you must add the floored block average (not the raw sum) into the accumulator. Finally, for each cell: if it was covered by at least one block, output floor(acc / cnt); otherwise leave it as the original pixel. Pattern: fixed-size sliding-window / grid simulation with per-cell aggregation.

Complexity

Time: O(mn) — there are O(mn) possible top-left corners for a 3x3 window, and validating a window plus adding its floored average to the 9 cells is constant O(9) = O(1) work each. Space: O(m*n) for the accumulator and count grids.

Python
PYTHON
from typing import List

def resultGrid(pixels: List[List[int]], tol: int) -> List[List[int]]:
    m, n = len(pixels), len(pixels[0])
    acc = [[0] * n for _ in range(m)]
    cnt = [[0] * n for _ in range(m)]

    def valid(r: int, c: int) -> bool:
        for dr in range(3):
            for dc in range(3):
                if dc < 2 and abs(pixels[r+dr][c+dc] - pixels[r+dr][c+dc+1]) > tol:
                    return False
                if dr < 2 and abs(pixels[r+dr][c+dc] - pixels[r+dr+1][c+dc]) > tol:
                    return False
        return True

    for r in range(m - 2):
        for c in range(n - 2):
            if not valid(r, c):
                continue
            total = 0
            for dr in range(3):
                for dc in range(3):
                    total += pixels[r+dr][c+dc]
            avg = total // 9
            for dr in range(3):
                for dc in range(3):
                    acc[r+dr][c+dc] += avg
                    cnt[r+dr][c+dc] += 1

    result = [[0] * n for _ in range(m)]
    for i in range(m):
        for j in range(n):
            result[i][j] = acc[i][j] // cnt[i][j] if cnt[i][j] > 0 else pixels[i][j]
    return result
JavaScript
JAVASCRIPT
function resultGrid(pixels, tol) {

#### cons
t m = pixels.length, n = pixels[0].length;

#### cons
t acc = Array.from({ length: m }, () => new Array(n).fill(0));

#### cons
t cnt = Array.from({ length: m }, () => new Array(n).fill(0));

#### cons
t valid = (r, c) => {
        for (let dr = 0; dr < 3; dr++)
            for (let dc = 0; dc < 3; dc++) {
                if (dc < 2 && Math.abs(pixels[r+dr][c+dc] - pixels[r+dr][c+dc+1]) > tol) return false;
                if (dr < 2 && Math.abs(pixels[r+dr][c+dc] - pixels[r+dr+1][c+dc]) > tol) return false;
            }
        return true;
    };

    for (let r = 0; r + 2 < m; r++)
        for (let c = 0; c + 2 < n; c++) {
            if (!valid(r, c)) continue;
            let total = 0;
            for (let dr = 0; dr < 3; dr++)
                for (let dc = 0; dc < 3; dc++)
                    total += pixels[r+dr][c+dc];

#### cons
t avg = Math.floor(total / 9);
            for (let dr = 0; dr < 3; dr++)
                for (let dc = 0; dc < 3; dc++) {
                    acc[r+dr][c+dc] += avg;
                    cnt[r+dr][c+dc]++;
                }
        }

#### cons
t result = Array.from({ length: m }, () => new Array(n).fill(0));
    for (let i = 0; i < m; i++)
        for (let j = 0; j < n; j++)
            result[i][j] = cnt[i][j] > 0 ? Math.floor(acc[i][j] / cnt[i][j]) : pixels[i][j];
    return result;
}
Java
JAVA
class Solution {
    public int[][] resultGrid(int[][] pixels, int tol) {
        int m = pixels.length, n = pixels[0].length;
        long[][] acc = new long[m][n];
        int[][] cnt = new int[m][n];

        for (int r = 0; r + 2 < m; r++)
            for (int c = 0; c + 2 < n; c++) {
                if (!valid(pixels, r, c, tol)) continue;
                int total = 0;
                for (int dr = 0; dr < 3; dr++)
                    for (int dc = 0; dc < 3; dc++)
                        total += pixels[r+dr][c+dc];
                int avg = total / 9;
                for (int dr = 0; dr < 3; dr++)
                    for (int dc = 0; dc < 3; dc++) {
                        acc[r+dr][c+dc] += avg;
                        cnt[r+dr][c+dc]++;
                    }
            }

        int[][] result = new int[m][n];
        for (int i = 0; i < m; i++)
            for (int j = 0; j < n; j++)
                result[i][j] = cnt[i][j] > 0 ? (int)(acc[i][j] / cnt[i][j]) : pixels[i][j];
        return result;
    }

    private boolean valid(int[][] p, int r, int c, int tol) {
        for (int dr = 0; dr < 3; dr++)
            for (int dc = 0; dc < 3; dc++) {
                if (dc < 2 && Math.abs(p[r+dr][c+dc] - p[r+dr][c+dc+1]) > tol) return false;
                if (dr < 2 && Math.abs(p[r+dr][c+dc] - p[r+dr+1][c+dc]) > tol) return false;
            }
        return true;
    }
}
C++
CPP
#include <vector>
#include <cstdlib>
using namespace std;

class Solution {
public:
    vector<vector<int>> resultGrid(vector<vector<int>>& pixels, int tol) {
        int m = pixels.size(), n = pixels[0].size();
        vector<vector<long long>> acc(m, vector<long long>(n, 0));
        vector<vector<int>> cnt(m, vector<int>(n, 0));

        auto valid = [&] (int r, int c) {
            for (int dr = 0; dr < 3; dr++)
                for (int dc = 0; dc < 3; dc++) {
                    if (dc < 2 && abs(pixels[r+dr][c+dc] - pixels[r+dr][c+dc+1]) > tol) return false;
                    if (dr < 2 && abs(pixels[r+dr][c+dc] - pixels[r+dr+1][c+dc]) > tol) return false;
                }
            return true;
        };

        for (int r = 0; r + 2 < m; r++)
            for (int c = 0; c + 2 < n; c++) {
                if (!valid(r, c)) continue;
                int total = 0;
                for (int dr = 0; dr < 3; dr++)
                    for (int dc = 0; dc < 3; dc++)
                        total += pixels[r+dr][c+dc];
                int avg = total / 9;
                for (int dr = 0; dr < 3; dr++)
                    for (int dc = 0; dc < 3; dc++) {
                        acc[r+dr][c+dc] += avg;
                        cnt[r+dr][c+dc]++;
                    }
            }

        vector<vector<int>> result(m, vector<int>(n, 0));
        for (int i = 0; i < m; i++)
            for (int j = 0; j < n; j++)
                result[i][j] = cnt[i][j] > 0 ? (int)(acc[i][j] / cnt[i][j]) : pixels[i][j];
        return result;
    }
};
C
C
#include <stdlib.h>

static int validBlock(int** pixels, int r, int c, int tol) {
    for (int dr = 0; dr < 3; dr++)
        for (int dc = 0; dc < 3; dc++) {
            if (dc < 2 && abs(pixels[r+dr][c+dc] - pixels[r+dr][c+dc+1]) > tol) return 0;
            if (dr < 2 && abs(pixels[r+dr][c+dc] - pixels[r+dr+1][c+dc]) > tol) return 0;
        }
    return 1;
}

/* Returns an m x n grid; caller frees. *returnSize = m and
   (*returnColumnSizes)[i] = n for each row. */
int** resultGrid(int** pixels, int m, int* pixelsColSize, int tol,
                 int* returnSize, int** returnColumnSizes) {
    int n = pixelsColSize[0];
    long long** acc = (long long**)malloc(sizeof(long long*) * m);
    int** cnt = (int**)malloc(sizeof(int*) * m);
    int** result = (int**)malloc(sizeof(int*) * m);
    *returnColumnSizes = (int*)malloc(sizeof(int) * m);
    for (int i = 0; i < m; i++) {
        acc[i] = (long long*)calloc(n, sizeof(long long));
        cnt[i] = (int*)calloc(n, sizeof(int));
        result[i] = (int*)malloc(sizeof(int) * n);
        (*returnColumnSizes)[i] = n;
    }
    for (int r = 0; r + 2 < m; r++)
        for (int c = 0; c + 2 < n; c++) {
            if (!validBlock(pixels, r, c, tol)) continue;
            int total = 0;
            for (int dr = 0; dr < 3; dr++)
                for (int dc = 0; dc < 3; dc++)
                    total += pixels[r+dr][c+dc];
            int avg = total / 9;
            for (int dr = 0; dr < 3; dr++)
                for (int dc = 0; dc < 3; dc++) {
                    acc[r+dr][c+dc] += avg;
                    cnt[r+dr][c+dc]++;
                }
        }
    for (int i = 0; i < m; i++)
        for (int j = 0; j < n; j++)
            result[i][j] = cnt[i][j] > 0 ? (int)(acc[i][j] / cnt[i][j]) : pixels[i][j];
    for (int i = 0; i < m; i++) { free(acc[i]); free(cnt[i]); }
    free(acc); free(cnt);
    *returnSize = m;
    return result;
}

#### Follow-up
questions
If the block size grew from 3x3 to an arbitrary k x k, how would you keep block-sum computation O(1) using a 2D prefix-sum so the whole algorithm stays O(m*n)?
What if adjacency also included the four diagonal neighbors when testing a block's validity?
How would you guard against integer overflow if brightness values and grid sizes were much larger?
Can you avoid the O(m*n) auxiliary count grid, or compute the result in a single streaming pass?

