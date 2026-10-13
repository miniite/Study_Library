---
type: "practice"
difficulty: "Other"
---

# Traversable Pipe Path Across a Grid

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Union Find](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Union_Find.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given an m x n grid pipes. Each cell contains a single pipe segment, and each segment connects exactly two of its four sides. The integer stored in a cell tells you which two sides the segment joins:

1 connects the left side and the right side.
2 connects the top side and the bottom side.
3 connects the left side and the bottom side.
4 connects the right side and the bottom side.
5 connects the left side and the top side.
6 connects the right side and the top side.

You start inside the pipe of the top-left cell (0, 0) and want to travel to the pipe of the bottom-right cell (m - 1, n - 1). You may move between two side-adjacent cells only if their pipe openings line up: the current pipe must open toward the neighbor and the neighbor's pipe must open back toward the current cell. You cannot rotate, add, or remove any pipe segment.

Return true if a continuous route of connected pipes exists from (0, 0) to (m - 1, n - 1), and false otherwise.

Input
pipes: an m x n grid where every entry is an integer in [1, 6].
Output
A boolean: whether the top-left and bottom-right cells are joined by a connected pipe route.

#### Cons
traints
m == pipes.length
n == pipes[i].length
1 <= m, n <= 300
1 <= pipes[i][j] <= 6

#### Example
s & test cases
TEXT
Input: pipes = [[4,3],[2,5]] -> Output: true
Why: (0,0) opens right into (0,1), which opens down into (1,1) — a connected chain reaches the exit.

Input: pipes = [[4,1],[2,6]] -> Output: false
Why: from (0,0) both the right route (into (0,1)) and the down route (into (1,0)) dead-end, and the exit pipe (1,1) has no opening any reachable neighbor connects into.

Input: pipes = [[1,1,1]] -> Output: true
Why: a single row of left-right pipes forms one continuous horizontal channel from start to end.

Input: pipes = [[2,1,1]] -> Output: false
Why: the start pipe opens only top and bottom, but it is the only row, so it cannot connect rightward and the route never begins.

Input: pipes = [[3]] -> Output: true
Why: a single cell means the start and the exit are the same cell, so the path is trivially valid.

Input: pipes = [[2,4,3],[6,5,2]] -> Output: true
Why: the route winds down, right through the interior, and back down, chaining aligned openings all the way to the bottom-right cell.

#### Approach

Model each cell as a node in a disjoint-set (union-find) structure. Precompute, for each of the six pipe types, four boolean flags indicating whether it opens left, up, right, or down. Scanning the grid once, it suffices to test each cell against only its right and its down neighbor (the up and left connections are covered symmetrically when those neighbors are the current cell): union two adjacent cells whenever the current pipe opens toward the neighbor and the neighbor opens back. After the single pass, the answer is simply whether the top-left cell and the bottom-right cell share the same set root. Using union-find avoids the deep recursion a DFS would need on a 300x300 grid; an iterative BFS/DFS with matching connectivity rules is an equally valid alternative.

Complexity

Time: O(m * n * α(m * n)) ≈ O(m * n), since each cell inspects only its right and down neighbor and performs near-constant union-find operations (α is the inverse Ackermann function). Space: O(m * n) for the disjoint-set parent array.

Python
PYTHON
class Solution:
    def hasValidPath(self, pipes):
        m, n = len(pipes), len(pipes[0])
        # index 0 unused; entries indexed by pipe type 1..6
        open_left  = [0, 1, 0, 1, 0, 1, 0]
        open_up    = [0, 0, 1, 0, 0, 1, 1]
        open_right = [0, 1, 0, 0, 1, 0, 1]
        open_down  = [0, 0, 1, 1, 1, 0, 0]

        parent = list(range(m * n))

        def find(x):
            while parent[x] != x:
                parent[x] = parent[parent[x]]
                x = parent[x]
            return x

        for i in range(m):
            for j in range(n):
                t = pipes[i][j]
                if open_right[t] and j + 1 < n and open_left[pipes[i][j + 1]]:
                    parent[find(i * n + j)] = find(i * n + j + 1)
                if open_down[t] and i + 1 < m and open_up[pipes[i + 1][j]]:
                    parent[find(i * n + j)] = find((i + 1) * n + j)

        return find(0) == find(m * n - 1)
JavaScript
JAVASCRIPT
function hasValidPath(pipes) {

#### cons
t m = pipes.length, n = pipes[0].length;
    // index 0 unused; entries indexed by pipe type 1..6

#### cons
t openLeft  = [0, 1, 0, 1, 0, 1, 0];

#### cons
t openUp    = [0, 0, 1, 0, 0, 1, 1];

#### cons
t openRight = [0, 1, 0, 0, 1, 0, 1];

#### cons
t openDown  = [0, 0, 1, 1, 1, 0, 0];

#### cons
t parent = Array.from({ length: m * n }, (_, i) => i);

#### cons
t find = (x) => {
        while (parent[x] !== x) { parent[x] = parent[parent[x]]; x = parent[x]; }
        return x;
    };

    for (let i = 0; i < m; ++i) {
        for (let j = 0; j < n; ++j) {

#### cons
t t = pipes[i][j];
            if (openRight[t] && j + 1 < n && openLeft[pipes[i][j + 1]])
                parent[find(i * n + j)] = find(i * n + j + 1);
            if (openDown[t] && i + 1 < m && openUp[pipes[i + 1][j]])
                parent[find(i * n + j)] = find((i + 1) * n + j);
        }
    }
    return find(0) === find(m * n - 1);
}
Java
JAVA
class Solution {
    private int[] parent;

    private int find(int x) {
        while (parent[x] != x) { parent[x] = parent[parent[x]]; x = parent[x]; }
        return x;
    }

    public boolean hasValidPath(int[][] pipes) {
        int m = pipes.length, n = pipes[0].length;
        int[] openLeft  = {0, 1, 0, 1, 0, 1, 0};
        int[] openUp    = {0, 0, 1, 0, 0, 1, 1};
        int[] openRight = {0, 1, 0, 0, 1, 0, 1};
        int[] openDown  = {0, 0, 1, 1, 1, 0, 0};

        parent = new int[m * n];
        for (int i = 0; i < m * n; ++i) parent[i] = i;

        for (int i = 0; i < m; ++i) {
            for (int j = 0; j < n; ++j) {
                int t = pipes[i][j];
                if (openRight[t] == 1 && j + 1 < n && openLeft[pipes[i][j + 1]] == 1)
                    parent[find(i * n + j)] = find(i * n + j + 1);
                if (openDown[t] == 1 && i + 1 < m && openUp[pipes[i + 1][j]] == 1)
                    parent[find(i * n + j)] = find((i + 1) * n + j);
            }
        }
        return find(0) == find(m * n - 1);
    }
}
C++
CPP
#include <vector>
using namespace std;

class Solution {
    vector<int> parent;
    int find(int x) {
        while (parent[x] != x) { parent[x] = parent[parent[x]]; x = parent[x]; }
        return x;
    }
public:
    bool hasValidPath(vector<vector<int>>& pipes) {
        int m = pipes.size(), n = pipes[0].size();
        int openLeft[7]  = {0, 1, 0, 1, 0, 1, 0};
        int openUp[7]    = {0, 0, 1, 0, 0, 1, 1};
        int openRight[7] = {0, 1, 0, 0, 1, 0, 1};
        int openDown[7]  = {0, 0, 1, 1, 1, 0, 0};

        parent.resize(m * n);
        for (int i = 0; i < m * n; ++i) parent[i] = i;

        for (int i = 0; i < m; ++i) {
            for (int j = 0; j < n; ++j) {
                int t = pipes[i][j];
                if (openRight[t] && j + 1 < n && openLeft[pipes[i][j + 1]])
                    parent[find(i * n + j)] = find(i * n + j + 1);
                if (openDown[t] && i + 1 < m && openUp[pipes[i + 1][j]])
                    parent[find(i * n + j)] = find((i + 1) * n + j);
            }
        }
        return find(0) == find(m * n - 1);
    }
};
C
C
#include <stdbool.h>
#include <stdlib.h>

static int* parent;

static int find(int x) {
    while (parent[x] != x) {
        parent[x] = parent[parent[x]];   /* path halving */
        x = parent[x];
    }
    return x;
}

bool hasValidPath(int** pipes, int pipesSize, int* pipesColSize) {
    int m = pipesSize, n = pipesColSize[0];
    /* index 0 unused; entries indexed by pipe type 1..6 */
    int openLeft[7]  = {0, 1, 0, 1, 0, 1, 0};
    int openUp[7]    = {0, 0, 1, 0, 0, 1, 1};
    int openRight[7] = {0, 1, 0, 0, 1, 0, 1};
    int openDown[7]  = {0, 0, 1, 1, 1, 0, 0};

    parent = (int*)malloc(sizeof(int) * m * n);
    for (int i = 0; i < m * n; ++i) parent[i] = i;

    for (int i = 0; i < m; ++i) {
        for (int j = 0; j < n; ++j) {
            int t = pipes[i][j];
            if (openRight[t] && j + 1 < n && openLeft[pipes[i][j + 1]]) {
                int a = find(i * n + j), b = find(i * n + j + 1);
                if (a != b) parent[a] = b;
            }
            if (openDown[t] && i + 1 < m && openUp[pipes[i + 1][j]]) {
                int a = find(i * n + j), b = find((i + 1) * n + j);
                if (a != b) parent[a] = b;
            }
        }
    }

    bool ans = find(0) == find(m * n - 1);
    free(parent);
    return ans;
}

#### Follow-up
questions
Solve it with an iterative BFS/DFS instead of union-find and discuss why recursion depth matters on a 300x300 grid.
Now allow rotating any pipe by 90-degree turns freely — how does the reachability question change?
Return an actual sequence of cells forming the valid route, not just a boolean.
Report the number of independent connected pipe components in the whole grid.

