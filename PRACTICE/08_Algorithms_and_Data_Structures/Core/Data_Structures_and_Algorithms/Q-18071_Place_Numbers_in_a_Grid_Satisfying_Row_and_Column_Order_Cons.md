---
type: "practice"
difficulty: "Other"
---

# Place Numbers in a Grid Satisfying Row and Column Order Constraints

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Topological Sort](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Topological_Sort.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given a positive integer n, together with two lists of ordering constraints:

rowPairs, where each element [x, y] means the number x must be placed in a row that is strictly above the row of number y.
colPairs, where each element [x, y] means the number x must be placed in a column that is strictly to the left of the column of number y.

All numbers in both lists lie between 1 and n inclusive.

Build an n x n grid that contains each of the numbers 1, 2, ..., n exactly once; every remaining cell must hold 0. The grid must satisfy every row constraint and every column constraint listed above.

Return any grid that satisfies all constraints. If no such grid exists, return an empty grid (a grid with no rows).

Input

n: the side length of the grid and the largest number to place.
rowPairs: a list of [x, y] pairs describing vertical (above/below) ordering.
colPairs: a list of [x, y] pairs describing horizontal (left/right) ordering.

Output

An n x n grid (list of n rows, each of n integers) meeting all constraints, or an empty grid if it is impossible.

#### Cons
traints

2 <= n <= 400
1 <= rowPairs.length, colPairs.length <= 10^4
Each pair has exactly two elements.
1 <= x, y <= n and x != y for every pair.

#### Example
s & test cases
TEXT
Input: n = 3, rowPairs = [[1,2],[3,2]], colPairs = [[2,1],[3,2]] -> Output: [[0,0,1],[3,0,0],[0,2,0]]
Why: A valid arrangement — 1 (row 0) is above 2 (row 2); 3 (row 1) is above 2 (row 2); 2 (col 1) is left of 1 (col 2); 3 (col 0) is left of 2 (col 1). Any grid meeting the same constraints is equally acceptable.

Input: n = 3, rowPairs = [[1,2],[2,3],[3,1]], colPairs = [[1,2]] -> Output: []
Why: The row constraints demand 1 above 2, 2 above 3, and 3 above 1 — a cycle. No consistent vertical ordering exists, so the result is the empty grid.

Input: n = 2, rowPairs = [[1,2]], colPairs = [[2,1]] -> Output: [[0,1],[2,0]]
Why: Minimum grid size. 1 must be above 2 and 2 must be left of 1; placing 1 at row 0/col 1 and 2 at row 1/col 0 satisfies both.

Input: n = 2, rowPairs = [], colPairs = [] -> Output: [[1,0],[0,2]]
Why: No constraints at all, so any placement works; the diagonal filling is one valid answer.

Input: n = 3, rowPairs = [[1,2]], colPairs = [[1,2],[2,3],[3,1]] -> Output: []
Why: The column constraints form a cycle (1 left of 2, 2 left of 3, 3 left of 1), so no valid horizontal ordering exists even though the row constraints are fine.

#### Approach

The row placement and the column placement are completely independent, and each is a pure ordering problem. Treat every number 1..n as a node. The row constraints [x, y] are directed edges "x before y" in a top-to-bottom ordering; the column constraints are the same for a left-to-right ordering. A valid vertical (or horizontal) arrangement of all numbers is exactly a topological order of that directed graph, and it exists if and only if the graph is a DAG. Run Kahn's algorithm (BFS on in-degree) twice: once on the row graph to get each number's row index, once on the column graph to get each number's column index. If either sort cannot emit all n nodes, a cycle exists and no grid is possible, so return an empty grid. Otherwise place value v at (rowIndex[v], colIndex[v]); because row and column positions come from independent total orders, every distinct number lands in a distinct cell, and all leftover cells stay 0. The core pattern is topological sorting with cycle detection, applied separately on two axes.

Complexity

Time: O(n^2 + R + C), where R and C are the numbers of row and column constraints — each topological sort is O(n + edges), and filling the n x n grid is O(n^2). Space: O(n^2) for the output grid plus O(n + edges) for adjacency lists, in-degree arrays, and queues.

Python
PYTHON
from typing import List
from collections import deque

def build_grid(n: int, row_pairs: List[List[int]], col_pairs: List[List[int]]) -> List[List[int]]:
    def topo(edges: List[List[int]]) -> List[int]:
        # Kahn's topological sort; returns an ordering of 1..n, or [] if a cycle exists.
        adj = [[] for _ in range(n + 1)]
        indeg = [0] * (n + 1)
        for a, b in edges:
            adj[a].append(b)
            indeg[b] += 1

        q = deque(i for i in range(1, n + 1) if indeg[i] == 0)
        order = []
        while q:
            u = q.popleft()
            order.append(u)
            for v in adj[u]:
                indeg[v] -= 1
                if indeg[v] == 0:
                    q.append(v)
        return order if len(order) == n else []

    row_order = topo(row_pairs)
    col_order = topo(col_pairs)
    if not row_order or not col_order:
        return []

    row_pos = [0] * (n + 1)
    col_pos = [0] * (n + 1)
    for i in range(n):
        row_pos[row_order[i]] = i
        col_pos[col_order[i]] = i

    grid = [[0] * n for _ in range(n)]
    for v in range(1, n + 1):
        grid[row_pos[v]][col_pos[v]] = v
    return grid
JavaScript
JAVASCRIPT
function buildGrid(n, rowPairs, colPairs) {
    // Kahn's topological sort; returns an ordering of 1..n, or [] if a cycle exists.
    function topo(edges) {

#### cons
t adj = Array.from({ length: n + 1 }, () => []);

#### cons
t indeg = new Array(n + 1).fill(0);
        for (const [a, b] of edges) { adj[a].push(b); indeg[b]++; }

#### cons
t queue = [];
        for (let i = 1; i <= n; i++) if (indeg[i] === 0) queue.push(i);

#### cons
t order = [];
        for (let h = 0; h < queue.length; h++) {

#### cons
t u = queue[h];
            order.push(u);
            for (const v of adj[u]) if (--indeg[v] === 0) queue.push(v);
        }
        return order.length === n ? order : [];
    }

#### cons
t rowOrder = topo(rowPairs);

#### cons
t colOrder = topo(colPairs);
    if (rowOrder.length === 0 || colOrder.length === 0) return [];

#### cons
t rowPos = new Array(n + 1).fill(0);

#### cons
t colPos = new Array(n + 1).fill(0);
    for (let i = 0; i < n; i++) { rowPos[rowOrder[i]] = i; colPos[colOrder[i]] = i; }

#### cons
t grid = Array.from({ length: n }, () => new Array(n).fill(0));
    for (let v = 1; v <= n; v++) grid[rowPos[v]][colPos[v]] = v;
    return grid;
}
Java
JAVA
import java.util.ArrayDeque;
import java.util.ArrayList;
import java.util.Deque;
import java.util.List;

class Solution {
    // Kahn's topological sort; returns an ordering of 1..n, or null if a cycle exists.
    private int[] topoSort(int n, int[][] edges) {
        List<List<Integer>> adj = new ArrayList<>();
        for (int i = 0; i <= n; i++) adj.add(new ArrayList<>());
        int[] indeg = new int[n + 1];
        for (int[] e : edges) { adj.get(e[0]).add(e[1]); indeg[e[1]]++; }

        Deque<Integer> q = new ArrayDeque<>();
        for (int i = 1; i <= n; i++) if (indeg[i] == 0) q.offer(i);
        int[] order = new int[n];
        int cnt = 0;
        while (!q.isEmpty()) {
            int u = q.poll();
            order[cnt++] = u;
            for (int v : adj.get(u)) if (--indeg[v] == 0) q.offer(v);
        }
        return cnt == n ? order : null;
    }

    public int[][] buildGrid(int n, int[][] rowPairs, int[][] colPairs) {
        int[] rowOrder = topoSort(n, rowPairs);
        int[] colOrder = topoSort(n, colPairs);
        if (rowOrder == null || colOrder == null) return new int[0][0];

        int[] rowPos = new int[n + 1], colPos = new int[n + 1];
        for (int i = 0; i < n; i++) { rowPos[rowOrder[i]] = i; colPos[colOrder[i]] = i; }

        int[][] grid = new int[n][n];
        for (int v = 1; v <= n; v++) grid[rowPos[v]][colPos[v]] = v;
        return grid;
    }
}
C++
CPP
#include <vector>
#include <queue>
using namespace std;

class Solution {
    // Kahn's topological sort; returns an ordering of 1..n, or {} if a cycle exists.
    vector<int> topoSort(int n, vector<vector<int>>& edges) {
        vector<vector<int>> adj(n + 1);
        vector<int> indeg(n + 1, 0);
        for (auto& e : edges) { adj[e[0]].push_back(e[1]); indeg[e[1]]++; }

        queue<int> q;
        for (int i = 1; i <= n; ++i) if (indeg[i] == 0) q.push(i);
        vector<int> order;
        while (!q.empty()) {
            int u = q.front(); q.pop();
            order.push_back(u);
            for (int v : adj[u]) if (--indeg[v] == 0) q.push(v);
        }
        return (int)order.size() == n ? order : vector<int>{};
    }

public:
    vector<vector<int>> buildGrid(int n, vector<vector<int>>& rowPairs,
                                  vector<vector<int>>& colPairs) {
        vector<int> rowOrder = topoSort(n, rowPairs);
        vector<int> colOrder = topoSort(n, colPairs);
        if (rowOrder.empty() || colOrder.empty()) return {};

        vector<int> rowPos(n + 1), colPos(n + 1);
        for (int i = 0; i < n; ++i) { rowPos[rowOrder[i]] = i; colPos[colOrder[i]] = i; }

        vector<vector<int>> grid(n, vector<int>(n, 0));
        for (int v = 1; v <= n; ++v) grid[rowPos[v]][colPos[v]] = v;
        return grid;
    }
};
C
C
#include <stdlib.h>

/* Kahn's topological sort of 1..n over the given directed edges.
   Writes a valid ordering into order[0..n-1]; returns 1 on success, 0 if a cycle exists. */
static int topoSort(int n, int** edges, int edgesSize, int* order) {
    int* indeg = (int*)calloc(n + 1, sizeof(int));
    int* head  = (int*)malloc((n + 1) * sizeof(int));
    for (int i = 0; i <= n; i++) head[i] = -1;
    int cap = edgesSize > 0 ? edgesSize : 1;
    int* to  = (int*)malloc(cap * sizeof(int));
    int* nxt = (int*)malloc(cap * sizeof(int));

    for (int e = 0; e < edgesSize; e++) {
        int a = edges[e][0], b = edges[e][1];
        to[e] = b; nxt[e] = head[a]; head[a] = e;
        indeg[b]++;
    }

    int* queue = (int*)malloc((n > 0 ? n : 1) * sizeof(int));
    int qh = 0, qt = 0, cnt = 0;
    for (int i = 1; i <= n; i++) if (indeg[i] == 0) queue[qt++] = i;
    while (qh < qt) {
        int u = queue[qh++];
        order[cnt++] = u;
        for (int e = head[u]; e != -1; e = nxt[e])
            if (--indeg[to[e]] == 0) queue[qt++] = to[e];
    }

    free(indeg); free(head); free(to); free(nxt); free(queue);
    return cnt == n;
}

int** buildGrid(int n, int** rowPairs, int rowPairsSize, int* rowPairsColSize,
                int** colPairs, int colPairsSize, int* colPairsColSize,
                int* returnSize, int** returnColumnSizes) {
    int* rowOrder = (int*)malloc(n * sizeof(int));
    int* colOrder = (int*)malloc(n * sizeof(int));

    if (!topoSort(n, rowPairs, rowPairsSize, rowOrder) ||
        !topoSort(n, colPairs, colPairsSize, colOrder)) {
        *returnSize = 0;
        *returnColumnSizes = NULL;
        free(rowOrder); free(colOrder);
        return NULL;
    }

    int* rowPos = (int*)malloc((n + 1) * sizeof(int));
    int* colPos = (int*)malloc((n + 1) * sizeof(int));
    for (int i = 0; i < n; i++) { rowPos[rowOrder[i]] = i; colPos[colOrder[i]] = i; }

    int** grid = (int**)malloc(n * sizeof(int*));
    *returnColumnSizes = (int*)malloc(n * sizeof(int));
    for (int i = 0; i < n; i++) {
        grid[i] = (int*)calloc(n, sizeof(int));
        (*returnColumnSizes)[i] = n;
    }
    for (int v = 1; v <= n; v++) grid[rowPos[v]][colPos[v]] = v;

    *returnSize = n;
    free(rowOrder); free(colOrder); free(rowPos); free(colPos);
    return grid;
}

#### Follow-up
questions
When no arrangement exists, how would you report the actual cycle of numbers that causes the conflict?
If multiple valid grids exist, how would you produce the lexicographically smallest row ordering (hint: a priority-queue variant of Kahn's algorithm)?
Could you implement the topological sort with DFS post-order instead of Kahn's BFS, and how would cycle detection differ?
What if a number were pinned to an exact row rather than only a relative order — how would that change the model?

