---
type: "practice"
difficulty: "Other"
---

# Maximum Units Collected on a Storage Floor

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Backtracking](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Backtracking.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

A rectangular storage floor is divided into rows x cols square tiles. Each tile holds a non-negative number of stored units; a value of 0 means the tile is empty. You control a retrieval bot that walks across the floor to gather units, subject to these rules:

Whenever the bot occupies a tile, it immediately picks up all units on that tile.
From its current tile, the bot may step to an orthogonally adjacent tile (up, down, left, or right).
The bot may not enter the same tile more than once during a run.
The bot may not step onto an empty tile (a tile whose value is 0).
The bot may begin its run on any non-empty tile and end it on any non-empty tile.

Return the maximum total number of units the bot can collect in a single run.

Input: a 2D integer array grid where grid[r][c] is the number of units on the tile at row r, column c. Output: a single integer — the maximum collectable total.

#### Cons
traints:

rows == grid.length
cols == grid[r].length
1 <= rows, cols <= 15
0 <= grid[r][c] <= 100
There are at most 25 non-empty tiles in the whole grid.

#### Example
s & test cases
TEXT
Input: grid = [[0,3,0],[4,7,2],[0,6,0]] -> Output: 17
Why: The only non-empty neighbours all touch the center 7; best run is 6 -> 7 -> 4 = 17
(a cross where you enter through one arm, cross the middle, and exit another arm).

Input: grid = [[2,0,5,3,4]] -> Output: 12
Why: The lone 2 is isolated by an empty tile; the connected strip 5-3-4 yields 5+3+4 = 12.

Input: grid = [[1,2,3],[4,5,6]] -> Output: 21
Why: Every tile is non-empty and the 2x3 block admits a snake path visiting all six tiles
(1 -> 4 -> 5 -> 2 -> 3 -> 6), so the whole floor is collected.

Input: grid = [[8]] -> Output: 8
Why: Minimum-size grid with a single non-empty tile; start and end there.

Input: grid = [[0,0],[0,0]] -> Output: 0
Why: No tile has any units, so nothing can be collected (the bot may not step on empties).

#### Approach

This is an exhaustive-path search over a grid, solved with backtracking (depth-first search). Because the bot may start and end anywhere, we try starting a DFS from every non-empty tile. Within a DFS, we "collect" the current tile by temporarily setting its value to 0 (which also serves as the visited-marker, since empties are illegal to enter), recurse into the four orthogonal neighbours taking the best single branch, then restore the tile's value on the way out so other paths can still use it. The answer is the maximum over all starting tiles. The core pattern is backtracking with in-place state marking/undoing on a grid.

Complexity

Time: O(rows * cols + k * 3^k), where k is the number of non-empty tiles (at most 25). The grid scan launches a DFS from each of up to k starting tiles; after the first step each DFS call branches into at most 3 unvisited directions, giving 3^k paths in the worst case. With k <= 25 this is fully tractable. Space: O(k) for the recursion stack (path depth is bounded by the number of non-empty tiles); no extra visited array is needed because a tile is zeroed in place and restored on backtrack.

Python
PYTHON
def max_units(grid):
    if not grid or not grid[0]:
        return 0
    rows, cols = len(grid), len(grid[0])

    def dfs(r, c):
        if r < 0 or r >= rows or c < 0 or c >= cols or grid[r][c] == 0:
            return 0
        val = grid[r][c]
        grid[r][c] = 0                         # mark visited
        gained = max(dfs(r + 1, c), dfs(r - 1, c),
                     dfs(r, c + 1), dfs(r, c - 1))
        grid[r][c] = val                       # restore
        return val + gained

    best = 0
    for r in range(rows):
        for c in range(cols):
            if grid[r][c] != 0:
                best = max(best, dfs(r, c))
    return best
JavaScript
JAVASCRIPT
function maxUnits(grid) {
    if (grid.length === 0 || grid[0].length === 0) return 0;

#### cons
t rows = grid.length, cols = grid[0].length;

    function dfs(r, c) {
        if (r < 0 || r >= rows || c < 0 || c >= cols || grid[r][c] === 0)
            return 0;

#### cons
t val = grid[r][c];
        grid[r][c] = 0;                         // mark visited

#### cons
t gained = Math.max(dfs(r + 1, c), dfs(r - 1, c),
                                dfs(r, c + 1), dfs(r, c - 1));
        grid[r][c] = val;                       // restore
        return val + gained;
    }

    let best = 0;
    for (let r = 0; r < rows; r++)
        for (let c = 0; c < cols; c++)
            if (grid[r][c] !== 0)
                best = Math.max(best, dfs(r, c));
    return best;
}
Java
JAVA
class Solution {
    private int rows, cols;

    public int maxUnits(int[][] grid) {
        if (grid.length == 0 || grid[0].length == 0) return 0;
        rows = grid.length;
        cols = grid[0].length;
        int best = 0;
        for (int r = 0; r < rows; r++)
            for (int c = 0; c < cols; c++)
                if (grid[r][c] != 0)
                    best = Math.max(best, dfs(grid, r, c));
        return best;
    }

    private int dfs(int[][] grid, int r, int c) {
        if (r < 0 || r >= rows || c < 0 || c >= cols || grid[r][c] == 0)
            return 0;
        int val = grid[r][c];
        grid[r][c] = 0;                       // mark visited
        int gained = Math.max(
            Math.max(dfs(grid, r + 1, c), dfs(grid, r - 1, c)),
            Math.max(dfs(grid, r, c + 1), dfs(grid, r, c - 1)));
        grid[r][c] = val;                     // restore
        return val + gained;
    }
}
C++
CPP
#include <vector>
#include <algorithm>
using namespace std;

class Solution {
public:
    int rows, cols;

    int dfs(vector<vector<int>>& grid, int r, int c) {
        if (r < 0 || r >= rows || c < 0 || c >= cols || grid[r][c] == 0)
            return 0;
        int val = grid[r][c];
        grid[r][c] = 0;                      // mark visited
        int gained = max({ dfs(grid, r + 1, c), dfs(grid, r - 1, c),
                           dfs(grid, r, c + 1), dfs(grid, r, c - 1) });
        grid[r][c] = val;                    // restore
        return val + gained;
    }

    int maxUnits(vector<vector<int>>& grid) {
        if (grid.empty() || grid[0].empty()) return 0;
        rows = grid.size();
        cols = grid[0].size();
        int best = 0;
        for (int r = 0; r < rows; r++)
            for (int c = 0; c < cols; c++)
                if (grid[r][c] != 0)
                    best = max(best, dfs(grid, r, c));
        return best;
    }
};
C
C
#include <stddef.h>

static int g_rows, g_cols;

static int dfs(int** grid, int r, int c) {
    if (r < 0 || r >= g_rows || c < 0 || c >= g_cols || grid[r][c] == 0)
        return 0;
    int val = grid[r][c];
    grid[r][c] = 0;               /* mark visited */
    int a = dfs(grid, r + 1, c);
    int b = dfs(grid, r - 1, c);
    int d = dfs(grid, r, c + 1);
    int e = dfs(grid, r, c - 1);
    int gained = a;
    if (b > gained) gained = b;
    if (d > gained) gained = d;
    if (e > gained) gained = e;
    grid[r][c] = val;             /* restore for other paths */
    return val + gained;
}

int maxUnits(int** grid, int gridSize, int* gridColSize) {
    if (gridSize == 0 || gridColSize[0] == 0) return 0;
    g_rows = gridSize;
    g_cols = gridColSize[0];
    int best = 0;
    for (int r = 0; r < g_rows; r++) {
        for (int c = 0; c < g_cols; c++) {
            if (grid[r][c] != 0) {
                int got = dfs(grid, r, c);
                if (got > best) best = got;
            }
        }
    }
    return best;
}

#### Follow-up
questions
Instead of the total, return the actual sequence of tiles on the best path.
What changes if the bot may also move diagonally (8-directional adjacency)?
Suppose a tile may be entered at most twice; how does the state/marking change?
If the number of non-empty tiles could be in the hundreds, exact search becomes infeasible — what heuristic or approximation would you use?

