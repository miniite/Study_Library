---
type: "practice"
difficulty: "Other"
---

# Total Boundary Length of a Filled Region

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Matrix](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Matrix.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given a rectangular grid with rows rows and cols columns. Each cell holds either 1, meaning that cell is a filled block of solid ground, or 0, meaning it is empty space. Two filled cells are considered connected only when they share a full edge (up, down, left, or right) — diagonal touching does not count.

The whole grid is surrounded by empty space on all four outer sides, and every filled cell forms exactly one connected region (there are no separate second regions). The filled region also has no enclosed hole: any empty cell can be reached from the outside without passing through filled cells.

Think of every filled cell as a unit square with side length 1. The boundary of the region is the outline you would trace around the outside of all the filled squares. Return the total length of that boundary (its perimeter).

Input

grid: a 2D array of rows x cols integers, each 0 or 1.

Output

A single integer: the perimeter of the one filled region.

#### Cons
traints

rows == grid.length
cols == grid[0].length
1 <= rows, cols <= 100
Each grid[i][j] is either 0 or 1.
There is exactly one connected region of filled cells, and it has no enclosed empty pockets.

#### Example
s & test cases
TEXT
Input: grid = [[1]] -> Output: 4
Why: A lone filled square contributes all four of its sides.

Input: grid = [[1,1]] -> Output: 6
Why: Two horizontally adjacent squares expose 8 sides but share 1 edge, so 8 - 2 = 6.

Input: grid = [[1,1],[1,0]] -> Output: 8
Why: An L-shape of 3 cells has 12 total sides and 2 shared internal edges: 12 - 4 = 8.

Input: grid = [[1,1],[1,1]] -> Output: 8
Why: A solid 2x2 block is a square of side 2; 16 total sides minus 4 shared edges = 8.

Input: grid = [[0,1,0],[1,1,1],[0,1,0]] -> Output: 12
Why: A plus/cross of 5 cells has 20 total sides and 4 shared edges at the center: 20 - 8 = 12.

Input: grid = [[0,1,0,0],[1,1,1,0],[0,1,0,0],[1,1,0,0]] -> Output: 16
Why: 7 filled cells give 28 total sides; 6 shared internal edges remove 12, leaving 16.

#### Approach

Every filled cell would contribute 4 to the perimeter if it stood alone. When two filled cells sit next to each other they share exactly one edge, and that shared edge is interior, so it removes 2 from the total perimeter (one side from each cell). So the answer is 4 * (number of filled cells) - 2 * (number of shared edges between filled neighbours). To count each shared edge exactly once, scan the grid and, for each filled cell, only look at the neighbour above it and the neighbour to its left (a single-pass counting trick). This avoids any flood-fill or extra data structure. An equivalent way to see it: for each filled cell, add 1 for every one of its four sides whose neighbour is out of bounds or empty.

Complexity

Time: O(rows * cols) — each cell is visited once and does O(1) work. Space: O(1) — only two integer counters are kept, independent of grid size.

Python
PYTHON
from typing import List

def region_perimeter(grid: List[List[int]]) -> int:
    rows = len(grid)
    cols = len(grid[0]) if rows else 0
    cells = shared = 0
    for r in range(rows):
        for c in range(cols):
            if grid[r][c] == 1:
                cells += 1
                if r > 0 and grid[r - 1][c] == 1:   # edge above
                    shared += 1
                if c > 0 and grid[r][c - 1] == 1:   # edge to left
                    shared += 1
    return cells * 4 - shared * 2
JavaScript
JAVASCRIPT
function regionPerimeter(grid) {

#### cons
t rows = grid.length;

#### cons
t cols = rows ? grid[0].length : 0;
    let cells = 0, shared = 0;
    for (let r = 0; r < rows; r++) {
        for (let c = 0; c < cols; c++) {
            if (grid[r][c] === 1) {
                cells++;
                if (r > 0 && grid[r - 1][c] === 1) shared++; // edge above
                if (c > 0 && grid[r][c - 1] === 1) shared++; // edge to left
            }
        }
    }
    return cells * 4 - shared * 2;
}
Java
JAVA
class Solution {
    public int regionPerimeter(int[][] grid) {
        int rows = grid.length;
        int cols = grid[0].length;
        int cells = 0, shared = 0;
        for (int r = 0; r < rows; r++) {
            for (int c = 0; c < cols; c++) {
                if (grid[r][c] == 1) {
                    cells++;
                    if (r > 0 && grid[r - 1][c] == 1) shared++; // edge above
                    if (c > 0 && grid[r][c - 1] == 1) shared++; // edge to left
                }
            }
        }
        return cells * 4 - shared * 2;
    }
}
C++
CPP
#include <vector>
using namespace std;

int regionPerimeter(vector<vector<int>>& grid) {
    int rows = grid.size();
    int cols = grid.empty() ? 0 : grid[0].size();
    int cells = 0, shared = 0;
    for (int r = 0; r < rows; ++r) {
        for (int c = 0; c < cols; ++c) {
            if (grid[r][c] == 1) {
                ++cells;
                if (r > 0 && grid[r - 1][c] == 1) ++shared; // edge above
                if (c > 0 && grid[r][c - 1] == 1) ++shared; // edge to left
            }
        }
    }
    return cells * 4 - shared * 2;
}
C
C
#include <stddef.h>

/* grid is an array of `rows` pointers, each to a row of `cols` ints. */
int regionPerimeter(int** grid, int rows, int cols) {
    int cells = 0;   /* number of filled cells */
    int shared = 0;  /* number of shared edges between filled neighbours */
    for (int r = 0; r < rows; ++r) {
        for (int c = 0; c < cols; ++c) {
            if (grid[r][c] == 1) {
                ++cells;
                if (r > 0 && grid[r - 1][c] == 1) ++shared; /* edge above */
                if (c > 0 && grid[r][c - 1] == 1) ++shared; /* edge to left */
            }
        }
    }
    return cells * 4 - shared * 2;
}

#### Follow-up
questions
What if the grid can contain more than one separate region — return the combined perimeter of all of them?
What if the region is allowed to enclose empty holes (lakes) — does the counting formula still hold, and why?
Solve it with an explicit DFS or BFS flood fill and add 1 per side that faces water or the grid edge; compare against the counting approach.
The grid arrives one row at a time and cannot all be held in memory; compute the perimeter using only O(cols) extra space.

