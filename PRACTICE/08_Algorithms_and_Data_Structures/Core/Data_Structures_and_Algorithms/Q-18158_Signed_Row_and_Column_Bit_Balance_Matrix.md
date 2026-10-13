---
type: "practice"
difficulty: "Other"
---

# Signed Row and Column Bit Balance Matrix

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Matrix](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Matrix.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given a binary matrix board with r rows and c columns. Every entry board[i][j] is either 0 or 1.

Build and return a new integer matrix score of the same dimensions, where each cell combines four counts taken from the row and column that the cell belongs to:

Let onesInRow[i] be the number of 1s in row i.
Let onesInCol[j] be the number of 1s in column j.
Let zerosInRow[i] be the number of 0s in row i.
Let zerosInCol[j] be the number of 0s in column j.

Then define:

CODE
score[i][j] = onesInRow[i] + onesInCol[j] - zerosInRow[i] - zerosInCol[j]

Return the full score matrix.

Input: a matrix board of 0/1 values. Output: an r x c integer matrix score computed by the formula above (values may be negative).

#### Cons
traints:

r == board.length
c == board[i].length
1 <= r, c <= 10^5
1 <= r * c <= 10^5
Each board[i][j] is either 0 or 1.

#### Example
s & test cases
TEXT
Input: board = [[1,0,0],[0,1,0]] -> Output: [[-1,-1,-3],[-1,-1,-3]]
Why: Each row has 1 one and 2 zeros; columns 0 and 1 have one 1, column 2 is all zeros, so the last column is the most negative — a normal mixed case with negative results.

Input: board = [[1,1],[1,1]] -> Output: [[4,4],[4,4]]
Why: Every row/column is all ones (2 ones, 0 zeros each), giving the maximal 2+2-0-0 = 4 everywhere.

Input: board = [[0]] -> Output: [[-2]]
Why: Minimal 1x1 case with a single zero: 0 ones and 1 zero in its row and column give 0+0-1-1 = -2.

Input: board = [[1]] -> Output: [[2]]
Why: Minimal 1x1 case with a single one: 1+1-0-0 = 2 (edge / smallest positive result).

Input: board = [[1,1,0],[0,0,1],[1,0,1]] -> Output: [[2,0,2],[0,-2,0],[2,0,2]]
Why: A 3x3 mix that exercises unequal row/column counts and produces a symmetric spread of positive, zero, and negative cells.

#### Approach

Notice that within a fixed row, the number of zeros is c - onesInRow, and within a fixed column the number of zeros is r - onesInCol. Substituting into onesRow + onesCol - zerosRow - zerosCol collapses the formula to 2*onesInRow[i] + 2*onesInCol[j] - r - c, which depends only on two precomputed vectors. So the pattern is a classic counting / prefix-aggregation trick: precompute the ones per row and per column in a single sweep, then compute every cell in O(1). No per-cell recount is needed, avoiding an O((r*c)^2) brute force.

Complexity

Time: O(r * c) — one pass to tally per-row and per-column ones, one pass to fill the result. Space: O(r + c) auxiliary for the two count arrays, plus the O(r * c) output matrix that must be produced.

Python
PYTHON
from typing import List

class Solution:
    def differenceMatrix(self, board: List[List[int]]) -> List[List[int]]:
        r, c = len(board), len(board[0])
        ones_row = [0] * r
        ones_col = [0] * c

        for i in range(r):
            row = board[i]
            for j in range(c):
                ones_row[i] += row[j]
                ones_col[j] += row[j]

        # zeros = total - ones, so score = 2*onesRow + 2*onesCol - r - c
        return [[2 * ones_row[i] + 2 * ones_col[j] - r - c
                 for j in range(c)]
                for i in range(r)]
JavaScript
JAVASCRIPT
function differenceMatrix(board) {

#### cons
t r = board.length, c = board[0].length;

#### cons
t onesRow = new Array(r).fill(0);

#### cons
t onesCol = new Array(c).fill(0);

    for (let i = 0; i < r; i++) {
        for (let j = 0; j < c; j++) {
            onesRow[i] += board[i][j];
            onesCol[j] += board[i][j];
        }
    }

#### cons
t score = Array.from({ length: r }, () => new Array(c).fill(0));
    for (let i = 0; i < r; i++) {
        for (let j = 0; j < c; j++) {
            score[i][j] = 2 * onesRow[i] + 2 * onesCol[j] - r - c;
        }
    }
    return score;
}
Java
JAVA
class Solution {
    public int[][] differenceMatrix(int[][] board) {
        int r = board.length, c = board[0].length;
        int[] onesRow = new int[r];
        int[] onesCol = new int[c];

        for (int i = 0; i < r; i++)
            for (int j = 0; j < c; j++) {
                onesRow[i] += board[i][j];
                onesCol[j] += board[i][j];
            }

        int[][] score = new int[r][c];
        for (int i = 0; i < r; i++)
            for (int j = 0; j < c; j++)
                score[i][j] = 2 * onesRow[i] + 2 * onesCol[j] - r - c;

        return score;
    }
}
C++
CPP
#include <vector>
using namespace std;

class Solution {
public:
    vector<vector<int>> differenceMatrix(vector<vector<int>>& board) {
        int r = board.size(), c = board[0].size();
        vector<int> onesRow(r, 0), onesCol(c, 0);

        for (int i = 0; i < r; i++)
            for (int j = 0; j < c; j++) {
                onesRow[i] += board[i][j];
                onesCol[j] += board[i][j];
            }

        vector<vector<int>> score(r, vector<int>(c));
        for (int i = 0; i < r; i++)
            for (int j = 0; j < c; j++)
                score[i][j] = 2 * onesRow[i] + 2 * onesCol[j] - r - c;

        return score;
    }
};
C
C
#include <stdlib.h>

/* Returns an r x c matrix; caller frees it and *returnColumnSizes. */
int** differenceMatrix(int** board, int r, int* colSizes,
                       int* returnSize, int** returnColumnSizes) {
    int c = colSizes[0];
    int* onesRow = (int*)calloc(r, sizeof(int));
    int* onesCol = (int*)calloc(c, sizeof(int));

    for (int i = 0; i < r; i++)
        for (int j = 0; j < c; j++) {
            onesRow[i] += board[i][j];
            onesCol[j] += board[i][j];
        }

    int** score = (int**)malloc(r * sizeof(int*));
    *returnColumnSizes = (int*)malloc(r * sizeof(int));
    for (int i = 0; i < r; i++) {
        score[i] = (int*)malloc(c * sizeof(int));
        (*returnColumnSizes)[i] = c;
        for (int j = 0; j < c; j++)
            /* zeros = total - ones, so the formula simplifies to this */
            score[i][j] = 2 * onesRow[i] + 2 * onesCol[j] - r - c;
    }

    *returnSize = r;
    free(onesRow);
    free(onesCol);
    return score;
}

#### Follow-up
questions
Can you compute the answer using only O(r + c) extra memory beyond the output? (Yes — the two count arrays.)
What changes if entries could be arbitrary integers rather than just 0/1, and you must count values above/below a threshold instead?
How would you stream the result row by row if the full matrix cannot fit in memory at once?
Prove that zerosInRow[i] = c - onesInRow[i] and use it to simplify the formula.

