---
type: "practice"
difficulty: "Other"
---

# Ground Cells That Cannot Reach the Map Edge

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Depth-First Search](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Depth-First_Search.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given a rectangular map represented by an rows x cols grid of integers. Each cell holds either 0, meaning water, or 1, meaning solid ground.

A traveller stands on a ground cell. In a single step the traveller may either:

move to a horizontally or vertically adjacent cell (up, down, left, right) that is also ground, or
if currently standing on a ground cell that lies on the outer border of the map (the first/last row or the first/last column), walk off the edge of the map entirely.

A ground cell is called landlocked if, starting from that cell, there is no sequence of steps (of any length) that lets the traveller walk off the edge of the map. In other words, the connected block of ground containing that cell never touches the border.

Return the total number of landlocked ground cells in the grid.

Diagonal movement is not allowed — connectivity is 4-directional only.

#### Cons
traints

rows == grid.length
cols == grid[i].length
1 <= rows, cols <= 500
Every grid[i][j] is either 0 or 1.

#### Example
s & test cases
TEXT
Input: grid = [[0,0,0],[0,1,0],[0,0,0]] -> Output: 1
Why: the single ground cell at the centre is surrounded by water on all four sides and never touches the border, so it is landlocked.

Input: grid = [[0,1,0],[0,1,0],[0,0,0]] -> Output: 0
Why: the top ground cell sits on the border (row 0), and the cell below is connected to it, so the whole block can walk off the edge — nothing is landlocked.

Input: grid = [[0,0,0,0],[0,1,1,0],[0,1,0,1],[0,0,0,0]] -> Output: 3
Why: the block {(1,1),(1,2),(2,1)} is fully enclosed by water (3 landlocked cells); the lone cell (2,3) lies on the last column, touches the border, and escapes.

Input: grid = [[1,1],[1,1]] -> Output: 0
Why: in a 2x2 map every cell lies on the border, so every ground cell can immediately walk off the edge.

Input: grid = [[0,0],[0,0]] -> Output: 0
Why: there is no ground at all, so there are zero landlocked cells (minimum case).

#### Approach

Any ground cell that can reach the border is safe; everything asked for is the complement. Instead of testing each interior cell separately, invert the problem: start a flood fill from every ground cell that already sits on the border and mark all ground connected to it (4-directionally). These are exactly the cells that can walk off the edge. After the fill, any ground cell that was never marked belongs to a block sealed off by water, so count those. This is the classic "escape from the boundary" / reverse-flood-fill pattern; an explicit stack (iterative DFS) is used instead of recursion so a 500x500 all-ground grid cannot overflow the call stack. BFS or Union-Find over border cells would work equally well.

Complexity

Time: O(rows * cols) — every cell is pushed and popped at most once during the flood fill and scanned a constant number of times. Space: O(rows * cols) for the seen marker grid and the explicit stack (worst case an all-ground map fills the stack).

Python
PYTHON
from typing import List

def count_landlocked(grid: List[List[int]]) -> int:
    if not grid or not grid[0]:
        return 0
    rows, cols = len(grid), len(grid[0])
    seen = [[False] * cols for _ in range(rows)]
    stack = []

    # Seed the stack with every ground cell on the border.
    for r in range(rows):
        for c in range(cols):
            on_border = r == 0 or r == rows - 1 or c == 0 or c == cols - 1
            if on_border and grid[r][c] == 1 and not seen[r][c]:
                seen[r][c] = True
                stack.append((r, c))

    # Flood-fill everything reachable from the border.
    while stack:
        r, c = stack.pop()
        for dr, dc in ((1, 0), (-1, 0), (0, 1), (0, -1)):
            nr, nc = r + dr, c + dc
            if 0 <= nr < rows and 0 <= nc < cols and \
                    grid[nr][nc] == 1 and not seen[nr][nc]:
                seen[nr][nc] = True
                stack.append((nr, nc))

    # Untouched ground is landlocked.
    return sum(
        1
        for r in range(rows)
        for c in range(cols)
        if grid[r][c] == 1 and not seen[r][c]
    )
JavaScript
JAVASCRIPT
function countLandlocked(grid) {
    if (grid.length === 0 || grid[0].length === 0) return 0;

#### cons
t rows = grid.length, cols = grid[0].length;

#### cons
t seen = Array.from({ length: rows }, () => new Array(cols).fill(false));

#### cons
t stack = [];

    // Seed with all border ground cells.
    for (let r = 0; r < rows; r++)
        for (let c = 0; c < cols; c++) {

#### cons
t onBorder = r === 0 || r === rows - 1 || c === 0 || c === cols - 1;
            if (onBorder && grid[r][c] === 1 && !seen[r][c]) {
                seen[r][c] = true;
                stack.push([r, c]);
            }
        }

#### cons
t dirs = [[1, 0], [-1, 0], [0, 1], [0, -1]];

    while (stack.length) {

#### cons
t [r, c] = stack.pop();
        for (const [dr, dc] of dirs) {

#### cons
t nr = r + dr, nc = c + dc;
            if (nr >= 0 && nr < rows && nc >= 0 && nc < cols &&
                grid[nr][nc] === 1 && !seen[nr][nc]) {
                seen[nr][nc] = true;
                stack.push([nr, nc]);
            }
        }
    }

    let total = 0;
    for (let r = 0; r < rows; r++)
        for (let c = 0; c < cols; c++)
            if (grid[r][c] === 1 && !seen[r][c]) total++;
    return total;
}
Java
JAVA
import java.util.ArrayDeque;
import java.util.Deque;

class Solution {
    public int countLandlocked(int[][] grid) {
        if (grid.length == 0 || grid[0].length == 0) return 0;
        int rows = grid.length, cols = grid[0].length;

        boolean[][] seen = new boolean[rows][cols];
        Deque<int[]> stack = new ArrayDeque<>();

        // Seed with all border ground cells.
        for (int r = 0; r < rows; r++)
            for (int c = 0; c < cols; c++)
                if ((r == 0 || r == rows - 1 || c == 0 || c == cols - 1)
                        && grid[r][c] == 1 && !seen[r][c]) {
                    seen[r][c] = true;
                    stack.push(new int[]{r, c});
                }

        int[] dr = {1, -1, 0, 0}, dc = {0, 0, 1, -1};

        while (!stack.isEmpty()) {
            int[] cell = stack.pop();
            int r = cell[0], c = cell[1];
            for (int k = 0; k < 4; k++) {
                int nr = r + dr[k], nc = c + dc[k];
                if (nr >= 0 && nr < rows && nc >= 0 && nc < cols
                        && grid[nr][nc] == 1 && !seen[nr][nc]) {
                    seen[nr][nc] = true;
                    stack.push(new int[]{nr, nc});
                }
            }
        }

        int total = 0;
        for (int r = 0; r < rows; r++)
            for (int c = 0; c < cols; c++)
                if (grid[r][c] == 1 && !seen[r][c])
                    total++;
        return total;
    }
}
C++
CPP
#include <vector>
#include <stack>
#include <utility>
using namespace std;

int countLandlocked(vector<vector<int>>& grid) {
    if (grid.empty() || grid[0].empty()) return 0;
    int rows = grid.size(), cols = grid[0].size();

    vector<vector<char>> seen(rows, vector<char>(cols, 0));
    stack<pair<int,int>> st;

    // Seed with all border ground cells.
    for (int r = 0; r < rows; r++)
        for (int c = 0; c < cols; c++)
            if ((r == 0 || r == rows - 1 || c == 0 || c == cols - 1) &&
                grid[r][c] == 1 && !seen[r][c]) {
                seen[r][c] = 1;
                st.push({r, c});
            }

    int dr[4] = {1, -1, 0, 0}, dc[4] = {0, 0, 1, -1};

    while (!st.empty()) {
        auto [r, c] = st.top(); st.pop();
        for (int k = 0; k < 4; k++) {
            int nr = r + dr[k], nc = c + dc[k];
            if (nr >= 0 && nr < rows && nc >= 0 && nc < cols &&
                grid[nr][nc] == 1 && !seen[nr][nc]) {
                seen[nr][nc] = 1;
                st.push({nr, nc});
            }
        }
    }

    int total = 0;
    for (int r = 0; r < rows; r++)
        for (int c = 0; c < cols; c++)
            if (grid[r][c] == 1 && !seen[r][c])
                total++;
    return total;
}
C
C
#include <stdlib.h>

int countLandlocked(int** grid, int rows, int cols) {
    if (rows == 0 || cols == 0) return 0;

    char* seen = (char*)calloc((size_t)rows * cols, sizeof(char));
    int* stack = (int*)malloc(sizeof(int) * (size_t)rows * cols);
    int top = 0;

    /* Seed the stack with every ground cell on the border. */
    for (int r = 0; r < rows; r++)
        for (int c = 0; c < cols; c++)
            if ((r == 0 || r == rows - 1 || c == 0 || c == cols - 1) &&
                grid[r][c] == 1 && !seen[r * cols + c]) {
                seen[r * cols + c] = 1;
                stack[top++] = r * cols + c;
            }

    int dr[4] = {1, -1, 0, 0};
    int dc[4] = {0, 0, 1, -1};

    /* Flood-fill all ground reachable from the border. */
    while (top > 0) {
        int cell = stack[--top];
        int r = cell / cols, c = cell % cols;
        for (int k = 0; k < 4; k++) {
            int nr = r + dr[k], nc = c + dc[k];
            if (nr >= 0 && nr < rows && nc >= 0 && nc < cols &&
                grid[nr][nc] == 1 && !seen[nr * cols + nc]) {
                seen[nr * cols + nc] = 1;
                stack[top++] = nr * cols + nc;
            }
        }
    }

    /* Ground cells never touched by the flood are landlocked. */
    int total = 0;
    for (int r = 0; r < rows; r++)
        for (int c = 0; c < cols; c++)
            if (grid[r][c] == 1 && !seen[r * cols + c])
                total++;

    free(seen);
    free(stack);
    return total;
}

#### Follow-up
questions
Switch the flood fill to an explicit BFS queue and argue why recursion is risky at the 500x500 limit.
Solve it with a Union-Find structure: union all border ground cells to a virtual "ocean" node, then count ground not connected to it.
Instead of the count of landlocked cells, return the size of the largest fully enclosed block, or the number of distinct enclosed blocks.
Change connectivity to 8-directional (include diagonals) and adjust the neighbour set.

