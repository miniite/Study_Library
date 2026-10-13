---
type: "practice"
difficulty: "Other"
---

# In-Place 90-Degree Clockwise Rotation of a Square Grid

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Matrix](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Matrix.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given a square grid of integers grid with n rows and n columns, representing a pixel image. Rotate the whole grid by 90 degrees clockwise.

You must perform the rotation in place: modify the given grid directly and do not allocate a second n x n grid to hold the result. Only O(1) extra memory beyond the grid itself is allowed.

A clockwise quarter turn sends the value originally at row r, column c to row c, column n - 1 - r. Concretely, the first column (read top to bottom) becomes the first row (read left to right), the second column becomes the second row, and so on.

Input

grid: an n x n list of lists of integers.

Output

Nothing is returned; the function mutates grid so that it holds the rotated image.

#### Cons
traints

n == grid.length == grid[i].length
1 <= n <= 20
-1000 <= grid[i][j] <= 1000

#### Example
s & test cases
TEXT
Input: grid = [[1,2,3],[4,5,6],[7,8,9]] -> Output: [[7,4,1],[8,5,2],[9,6,3]]
Why: the first column (1,4,7) becomes the top row read left-to-right (7,4,1) — the canonical odd-sized case.

Input: grid = [[1,2],[3,4]] -> Output: [[3,1],[4,2]]
Why: smallest non-trivial case; column (1,3) rotates to top row (3,1).

Input: grid = [[5]] -> Output: [[5]]
Why: a 1x1 grid is unchanged by rotation (minimum-size edge case).

Input: grid = [[-1,-2],[-3,-4]] -> Output: [[-3,-1],[-4,-2]]
Why: exercises negative values; rotation logic is value-agnostic.

Input: grid = [[5,1,9,11],[2,4,8,10],[13,3,6,7],[15,14,12,16]] -> Output: [[15,13,2,5],[14,3,4,1],[12,6,8,9],[16,7,10,11]]
Why: even-sized 4x4 case covering all four concentric layers.

#### Approach

Decompose a clockwise 90-degree rotation into two in-place operations: (1) reverse the order of the rows (a vertical flip), then (2) transpose the grid across its main diagonal by swapping grid[i][j] with grid[j][i] for every j > i. Composing a vertical flip with a transpose is algebraically equivalent to a clockwise quarter turn, and both steps mutate the grid directly, so no extra 2D buffer is needed. (An equivalent alternative is a four-way cyclic swap of the four corners of each concentric layer.)

Complexity

Time: O(n^2) — each of the n^2 cells is touched a constant number of times across the row reversal and the transpose. Space: O(1) auxiliary — the row-pointer reversal and the diagonal swaps both work in place, with no second grid allocated.

Python
PYTHON
from typing import List

class Solution:
    def rotate(self, grid: List[List[int]]) -> None:
        grid.reverse()  # flip top-to-bottom
        n = len(grid)
        for i in range(n):  # transpose in place
            for j in range(i + 1, n):
                grid[i][j], grid[j][i] = grid[j][i], grid[i][j]
JavaScript
JAVASCRIPT
function rotate(grid) {
  grid.reverse(); // flip top-to-bottom

#### cons
t n = grid.length;
  for (let i = 0; i < n; i++) {
    // transpose in place
    for (let j = i + 1; j < n; j++) {

#### cons
t t = grid[i][j];
      grid[i][j] = grid[j][i];
      grid[j][i] = t;
    }
  }
}
Java
JAVA
class Solution {
    public void rotate(int[][] grid) {
        int n = grid.length;
        // Step 1: reverse the row order (flip top-to-bottom).
        for (int i = 0, j = n - 1; i < j; ++i, --j) {
            int[] tmp = grid[i];
            grid[i] = grid[j];
            grid[j] = tmp;
        }
        // Step 2: transpose across the main diagonal.
        for (int i = 0; i < n; ++i)
            for (int j = i + 1; j < n; ++j) {
                int t = grid[i][j];
                grid[i][j] = grid[j][i];
                grid[j][i] = t;
            }
    }
}
C++
CPP
#include <algorithm>
#include <vector>
using namespace std;

class Solution {
 public:
  void rotate(vector<vector<int>>& grid) {
    reverse(grid.begin(), grid.end());  // flip top-to-bottom

#### cons
t int n = grid.size();
    for (int i = 0; i < n; ++i)         // transpose in place
      for (int j = i + 1; j < n; ++j)
        swap(grid[i][j], grid[j][i]);
  }
};
C
C
#include <stdio.h>

void rotate(int** grid, int gridSize, int* gridColSize) {
    (void)gridColSize;
    int n = gridSize;
    /* Step 1: reverse the order of the rows (flip top-to-bottom). */
    for (int i = 0, j = n - 1; i < j; ++i, --j) {
        int* tmp = grid[i];
        grid[i] = grid[j];
        grid[j] = tmp;
    }
    /* Step 2: transpose across the main diagonal. */
    for (int i = 0; i < n; ++i)
        for (int j = i + 1; j < n; ++j) {
            int t = grid[i][j];
            grid[i][j] = grid[j][i];
            grid[j][i] = t;
        }
}

int main(void) {
    int r0[] = {1, 2, 3}, r1[] = {4, 5, 6}, r2[] = {7, 8, 9};
    int* grid[] = {r0, r1, r2};
    int colSize[] = {3, 3, 3};
    rotate(grid, 3, colSize);
    for (int i = 0; i < 3; ++i) {
        for (int j = 0; j < 3; ++j) printf("%d ", grid[i][j]);
        printf("\n");
    }
    return 0;
}

#### Follow-up
questions
Rotate 90 degrees counter-clockwise instead (reverse the columns, or transpose then reverse each row).
Rotate the grid by 180 degrees in place.
Handle a non-square m x n grid — why does in-place rotation no longer work, and what buffer is required?
Generalize to rotate by an arbitrary multiple of 90 degrees given as a parameter.

