---
type: "practice"
difficulty: "Other"
---

# Non-Attacking Queen Arrangements — All Configurations

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Backtracking](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Backtracking.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given a single integer k describing a square grid with k rows and k columns.

A queen is a piece that controls every cell sharing its row, its column, or either of its two diagonals. Your task is to place exactly k queens on the grid so that no two queens control one another — that is, no two queens may share a row, a column, or a diagonal.

Return every distinct arrangement that satisfies this rule. The arrangements may be returned in any order.

Represent each arrangement as a list of k strings, each of length k. In every string, the character 'Q' marks a cell that holds a queen and the character '.' marks an empty cell. Exactly one 'Q' must appear in each row and each column of a valid arrangement.

If no valid arrangement exists, return an empty list.

Input

An integer k, the side length of the square grid.

Output

A list of arrangements. Each arrangement is a list of k strings of length k built from 'Q' and '.' as described above.

#### Cons
traints

1 <= k <= 9
The number of returned arrangements can be 0 (for example k = 2 or k = 3) or grow quickly (for example k = 8 yields 92 arrangements).

#### Example
s & test cases
TEXT
Input: k = 1 -> Output: [["Q"]]
Why: A single cell trivially holds one queen with nothing to attack.

Input: k = 4 -> Output: [[".Q..","...Q","Q...","..Q."],["..Q.","Q...","...Q",".Q.."]]
Why: The smallest interesting case; there are exactly two mirror-image arrangements.

Input: k = 2 -> Output: []
Why: Any two cells on a 2x2 grid share a row, column, or diagonal, so no valid placement exists.

Input: k = 3 -> Output: []
Why: Every attempt to place 3 mutually non-attacking queens on a 3x3 grid fails; the result is empty.

Input: k = 5 -> Output: (10 distinct arrangements)
Why: A normal medium case where multiple valid boards exist; the function must enumerate all 10.

Input: k = 8 -> Output: (92 distinct arrangements)
Why: The classic full-board size; confirms the search enumerates the complete solution set without duplicates.

#### Approach

Backtracking, one row at a time. Because every valid arrangement has exactly one queen per row, we descend row by row and only decide which column to use. A column c at row row is legal only if that column is free and both diagonals through the cell are free. The two diagonal families are indexed in O(1): the "down-right" diagonal is constant along row + c, and the "down-left" diagonal is constant along row - c, shifted by k - 1 to keep the index non-negative. Three boolean arrays (columns, diag1, diag2) let each placement test and update run in O(1). We mark the three markers, recurse, and unmark on the way back, recording a board snapshot whenever all k rows are filled.

Complexity

Time: O(k!) in the worst case — row row tries at most k - row viable columns, and the constant-time conflict checks prune the rest; building each of the S valid boards costs O(k^2), so total work is O(k! + Sk^2). Space: O(k) for the recursion stack and the three marker arrays (O(k) each), excluding the O(Sk^2) needed to store the returned boards.

Python
PYTHON
from typing import List

class Solution:
    def solveNQueens(self, k: int) -> List[List[str]]:
        ans: List[List[str]] = []
        cols = [False] * k
        diag1 = [False] * (2 * k - 1)          # row + col
        diag2 = [False] * (2 * k - 1)          # row - col + (k - 1)

        def dfs(row: int, board: List[str]) -> None:
            if row == k:
                ans.append(board)
                return
            for c in range(k):
                if cols[c] or diag1[row + c] or diag2[row - c + k - 1]:
                    continue
                cols[c] = diag1[row + c] = diag2[row - c + k - 1] = True
                dfs(row + 1, board + ['.' * c + 'Q' + '.' * (k - c - 1)])
                cols[c] = diag1[row + c] = diag2[row - c + k - 1] = False

        dfs(0, [])
        return ans
JavaScript
JAVASCRIPT
function solveNQueens(k) {

#### cons
t ans = [];

#### cons
t cols = new Array(k).fill(false);

#### cons
t diag1 = new Array(2 * k - 1).fill(false); // row + col

#### cons
t diag2 = new Array(2 * k - 1).fill(false); // row - col + (k - 1)

  function dfs(row, board) {
    if (row === k) {
      ans.push(board.slice());
      return;
    }
    for (let c = 0; c < k; c++) {
      if (cols[c] || diag1[row + c] || diag2[row - c + k - 1]) continue;
      cols[c] = diag1[row + c] = diag2[row - c + k - 1] = true;
      board.push('.'.repeat(c) + 'Q' + '.'.repeat(k - c - 1));
      dfs(row + 1, board);
      board.pop();
      cols[c] = diag1[row + c] = diag2[row - c + k - 1] = false;
    }
  }

  dfs(0, []);
  return ans;
}
Java
JAVA
import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;

class Solution {
    public List<List<String>> solveNQueens(int k) {
        List<List<String>> ans = new ArrayList<>();
        char[][] board = new char[k][k];
        for (char[] row : board) Arrays.fill(row, '.');
        dfs(k, 0, new boolean[k], new boolean[2 * k - 1], new boolean[2 * k - 1], board, ans);
        return ans;
    }

    private void dfs(int n, int row, boolean[] cols, boolean[] d1, boolean[] d2,
                     char[][] board, List<List<String>> ans) {
        if (row == n) {
            List<String> snapshot = new ArrayList<>();
            for (char[] r : board) snapshot.add(new String(r));
            ans.add(snapshot);
            return;
        }
        for (int c = 0; c < n; c++) {
            if (cols[c] || d1[row + c] || d2[row - c + n - 1]) continue;
            cols[c] = d1[row + c] = d2[row - c + n - 1] = true;
            board[row][c] = 'Q';
            dfs(n, row + 1, cols, d1, d2, board, ans);
            board[row][c] = '.';
            cols[c] = d1[row + c] = d2[row - c + n - 1] = false;
        }
    }
}
C++
CPP
#include <string>
#include <vector>
using namespace std;

class Solution {
public:
    vector<vector<string>> solveNQueens(int k) {
        vector<vector<string>> ans;
        vector<bool> cols(k, false);
        vector<bool> diag1(2 * k - 1, false);  // r + c
        vector<bool> diag2(2 * k - 1, false);  // r - c + (k - 1)
        vector<string> board(k, string(k, '.'));
        dfs(k, 0, cols, diag1, diag2, board, ans);
        return ans;
    }

private:
    void dfs(int n, int row, vector<bool>& cols, vector<bool>& d1,
             vector<bool>& d2, vector<string>& board,
             vector<vector<string>>& ans) {
        if (row == n) {
            ans.push_back(board);
            return;
        }
        for (int c = 0; c < n; ++c) {
            if (cols[c] || d1[row + c] || d2[row - c + n - 1]) continue;
            cols[c] = d1[row + c] = d2[row - c + n - 1] = true;
            board[row][c] = 'Q';
            dfs(n, row + 1, cols, d1, d2, board, ans);
            board[row][c] = '.';
            cols[c] = d1[row + c] = d2[row - c + n - 1] = false;
        }
    }
};
C
C
#include <stdlib.h>
#include <string.h>

static void dfs(int n, int row, int* pos, char* cols, char* d1, char* d2,
                char*** out, int* colSizes, int* count) {
    if (row == n) {
        char** board = (char**)malloc(sizeof(char*) * n);
        for (int i = 0; i < n; i++) {
            board[i] = (char*)malloc(n + 1);
            memset(board[i], '.', n);
            board[i][pos[i]] = 'Q';
            board[i][n] = '\0';
        }
        out[*count] = board;
        colSizes[*count] = n;
        (*count)++;
        return;
    }
    for (int c = 0; c < n; c++) {
        if (cols[c] || d1[row + c] || d2[row - c + n - 1]) continue;
        cols[c] = d1[row + c] = d2[row - c + n - 1] = 1;
        pos[row] = c;
        dfs(n, row + 1, pos, cols, d1, d2, out, colSizes, count);
        cols[c] = d1[row + c] = d2[row - c + n - 1] = 0;
    }
}

/* Returns a list of boards; each board is an array of n strings.
   *returnSize = number of boards; (*returnColumnSizes)[i] = rows in board i. */
char*** solveNQueens(int k, int* returnSize, int** returnColumnSizes) {
    int cap = 1024;  /* k <= 9 => at most 352 arrangements */
    char*** out = (char***)malloc(sizeof(char**) * cap);
    *returnColumnSizes = (int*)malloc(sizeof(int) * cap);
    int* pos = (int*)malloc(sizeof(int) * k);
    char* cols = (char*)calloc(k, 1);
    char* d1 = (char*)calloc(2 * k - 1, 1);
    char* d2 = (char*)calloc(2 * k - 1, 1);
    int count = 0;
    dfs(k, 0, pos, cols, d1, d2, out, *returnColumnSizes, &count);
    free(pos); free(cols); free(d1); free(d2);
    *returnSize = count;
    return out;
}

#### Follow-up
questions
Return only the number of valid arrangements instead of every board — can you avoid materializing the boards at all?
Return just one valid arrangement as early as possible and stop.
Replace the three boolean arrays with bitmasks (integers) for the columns and both diagonals — how does that change the constant factor?
Exploit the board's mirror symmetry to search only half the first-row columns and reconstruct the rest.

