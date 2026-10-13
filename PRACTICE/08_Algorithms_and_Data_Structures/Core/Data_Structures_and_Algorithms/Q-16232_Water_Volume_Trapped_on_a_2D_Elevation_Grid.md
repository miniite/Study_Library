---
type: "practice"
difficulty: "Other"
---

# Water Volume Trapped on a 2D Elevation Grid

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Heap (Priority Queue)](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Heap_(Priority_Queue).md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given a rectangular grid terrain of size m x n, where terrain[r][c] is a non-negative integer giving the height of the ground at cell (r, c). Imagine it starts raining uniformly over the whole grid until the terrain is saturated. Water settles on top of any cell whose height is lower than the walls of ground surrounding it. Water that reaches the outer edge of the grid simply spills off and is lost — the boundary cells cannot hold any water.

The water level over an interior cell is limited by the lowest wall it must climb over to escape to the border. In other words, water flows freely between orthogonally adjacent cells (up, down, left, right); a puddle can only rise as high as the lowest point along the cheapest path out of the grid.

Return the total volume of water trapped over the whole grid after the rain, where the water sitting on a single cell contributes waterLevel(r, c) - terrain[r][c] (which is 0 if the cell is not submerged).

Input:

terrain: a list of m lists, each of length n, of non-negative integers.

Output:

A single integer: the total trapped water volume.

#### Cons
traints:

m == terrain.length, n == terrain[r].length
1 <= m, n <= 200
0 <= terrain[r][c] <= 20000
Any grid with fewer than 3 rows or fewer than 3 columns has no interior and traps 0 water.

#### Example
s & test cases
TEXT
- Input: terrain = [[6,6,6,6],[6,2,3,6],[6,6,6,6]] -> Output: 7
  Why: an enclosed pond with walls of height 6; the two interior cells (2 and 3) fill up to level 6, giving (6-2)+(6-3)=7. Basic enclosed case.
- Input: terrain = [[4,4,4],[4,0,4],[4,4,4]] -> Output: 4
  Why: a single deep pit surrounded by walls of height 4; water rises to 4, volume 4-0=4. Minimum interior (1 cell).
- Input: terrain = [[5,5,5,5],[5,1,4,1],[5,5,5,5]] -> Output: 3
  Why: there is a leak at boundary cell (1,3)=1; the interior cell (1,1)=1 can only fill up to 4 because its cheapest escape path passes over the height-4 cell before spilling out, so it traps 4-1=3, and cell (1,2)=4 traps nothing. Tests the bottleneck logic.
- Input: terrain = [[1,2],[3,4]] -> Output: 0
  Why: a 2x2 grid has no interior cell, so no water can be trapped. Edge case: dimension below 3.
- Input: terrain = [[3,3,3],[3,3,3]] -> Output: 0
  Why: a flat 2-row grid; no interior and everything is level, nothing trapped. Edge case: flat and thin.

#### Approach

This is a classic "flood from the outside in" problem solved with a boundary min-heap (Dijkstra-like best-first BFS). Water can only rise as high as the lowest wall along its escape route to the border, so we process cells in increasing order of the effective wall height that surrounds the unexplored region. Seed a min-heap with all boundary cells (they hold no water). Repeatedly pop the lowest cell h; this is the current lowest point of the "rim" enclosing everything not yet visited. For each unvisited orthogonal neighbor, the water level there is at least h: if the neighbor's ground is below h, it traps h - ground units of water and is re-inserted at level h (acting as a wall of that height for cells deeper inside); otherwise it is inserted at its own ground height. Marking cells seen when first pushed prevents reprocessing. The pattern is best-first search / Dijkstra on a grid, using the heap to always expand the globally lowest surrounding boundary.

Complexity

Time: O(mnlog(mn)) — every cell is pushed to and popped from the heap exactly once, and each heap operation costs O(log(mn)). Space: O(mn) for the seen grid and the heap, which can hold up to O(mn) cells.

Python
PYTHON
import heapq
from typing import List

def trap_water(terrain: List[List[int]]) -> int:
    m = len(terrain)
    if m < 3:
        return 0
    n = len(terrain[0])
    if n < 3:
        return 0

    seen = [[False] * n for _ in range(m)]
    heap = []  # entries: (height, row, col)

    for i in range(m):
        for j in range(n):
            if i in (0, m - 1) or j in (0, n - 1):
                heapq.heappush(heap, (terrain[i][j], i, j))
                seen[i][j] = True

    ans = 0
    dirs = ((1, 0), (-1, 0), (0, 1), (0, -1))

    while heap:
        h, r, c = heapq.heappop(heap)
        for dr, dc in dirs:
            x, y = r + dr, c + dc
            if 0 <= x < m and 0 <= y < n and not seen[x][y]:
                seen[x][y] = True
                if terrain[x][y] < h:
                    ans += h - terrain[x][y]           # trapped water on this cell
                    heapq.heappush(heap, (h, x, y))
                else:
                    heapq.heappush(heap, (terrain[x][y], x, y))

    return ans
JavaScript
JAVASCRIPT
function trapWater(terrain) {

#### cons
t m = terrain.length;
    if (m < 3) return 0;

#### cons
t n = terrain[0].length;
    if (n < 3) return 0;

#### cons
t seen = Array.from({ length: m }, () => new Array(n).fill(false));

    // Binary min-heap of [height, row, col].

#### cons
t heap = [];

#### cons
t swap = (i, j) => { const t = heap[i]; heap[i] = heap[j]; heap[j] = t; };

#### cons
t push = (v) => {
        heap.push(v);
        let i = heap.length - 1;
        while (i > 0) {

#### cons
t p = (i - 1) >> 1;
            if (heap[p][0] <= heap[i][0]) break;
            swap(p, i);
            i = p;
        }
    };

#### cons
t pop = () => {

#### cons
t top = heap[0];

#### cons
t last = heap.pop();
        if (heap.length) {
            heap[0] = last;
            let i = 0;

#### cons
t sz = heap.length;
            while (true) {

#### cons
t l = 2 * i + 1, r = 2 * i + 2;
                let s = i;
                if (l < sz && heap[l][0] < heap[s][0]) s = l;
                if (r < sz && heap[r][0] < heap[s][0]) s = r;
                if (s === i) break;
                swap(s, i);
                i = s;
            }
        }
        return top;
    };

    for (let i = 0; i < m; i++)
        for (let j = 0; j < n; j++)
            if (i === 0 || i === m - 1 || j === 0 || j === n - 1) {
                push([terrain[i][j], i, j]);
                seen[i][j] = true;
            }

    let ans = 0;

#### cons
t dirs = [[1, 0], [-1, 0], [0, 1], [0, -1]];

    while (heap.length) {

#### cons
t [h, r, c] = pop();
        for (const [dr, dc] of dirs) {

#### cons
t x = r + dr, y = c + dc;
            if (x < 0 || x >= m || y < 0 || y >= n || seen[x][y]) continue;
            seen[x][y] = true;
            if (terrain[x][y] < h) {
                ans += h - terrain[x][y];       // trapped water on this cell
                push([h, x, y]);
            } else {
                push([terrain[x][y], x, y]);
            }
        }
    }
    return ans;
}
Java
JAVA
import java.util.*;

class Solution {
    public int trapWater(int[][] terrain) {
        int m = terrain.length;
        if (m < 3) return 0;
        int n = terrain[0].length;
        if (n < 3) return 0;

        // Each entry is {height, row, col}, ordered by height ascending.
        PriorityQueue<int[]> pq = new PriorityQueue<>((a, b) -> a[0] - b[0]);
        boolean[][] seen = new boolean[m][n];

        for (int i = 0; i < m; i++)
            for (int j = 0; j < n; j++)
                if (i == 0 || i == m - 1 || j == 0 || j == n - 1) {
                    pq.offer(new int[]{terrain[i][j], i, j});
                    seen[i][j] = true;
                }

        long ans = 0;
        int[] dr = {1, -1, 0, 0}, dc = {0, 0, 1, -1};

        while (!pq.isEmpty()) {
            int[] cur = pq.poll();
            int h = cur[0], r = cur[1], c = cur[2];
            for (int k = 0; k < 4; k++) {
                int x = r + dr[k], y = c + dc[k];
                if (x < 0 || x >= m || y < 0 || y >= n || seen[x][y]) continue;
                seen[x][y] = true;
                if (terrain[x][y] < h) {
                    ans += h - terrain[x][y];   // water fills this cell up to h
                    pq.offer(new int[]{h, x, y});
                } else {
                    pq.offer(new int[]{terrain[x][y], x, y});
                }
            }
        }
        return (int) ans;
    }
}
C++
CPP
#include <vector>
#include <queue>
#include <tuple>
using namespace std;

class Solution {
public:
    int trapWater(vector<vector<int>>& terrain) {
        int m = terrain.size();
        if (m < 3) return 0;
        int n = terrain[0].size();
        if (n < 3) return 0;

        using T = tuple<int, int, int>;  // {height, row, col}
        priority_queue<T, vector<T>, greater<T>> pq;
        vector<vector<bool>> seen(m, vector<bool>(n, false));

        for (int i = 0; i < m; ++i)
            for (int j = 0; j < n; ++j)
                if (i == 0 || i == m - 1 || j == 0 || j == n - 1) {
                    pq.emplace(terrain[i][j], i, j);
                    seen[i][j] = true;
                }

        long ans = 0;
        int dr[4] = {1, -1, 0, 0}, dc[4] = {0, 0, 1, -1};

        while (!pq.empty()) {
            auto [h, r, c] = pq.top();
            pq.pop();
            for (int k = 0; k < 4; ++k) {
                int x = r + dr[k], y = c + dc[k];
                if (x < 0 || x >= m || y < 0 || y >= n || seen[x][y]) continue;
                seen[x][y] = true;
                if (terrain[x][y] < h) {
                    ans += h - terrain[x][y];   // trapped water on this cell
                    pq.emplace(h, x, y);
                } else {
                    pq.emplace(terrain[x][y], x, y);
                }
            }
        }
        return (int)ans;
    }
};
C
C
#include <stdlib.h>

typedef struct { int h, r, c; } Cell;

static void siftUp(Cell *heap, int i) {
    while (i > 0) {
        int p = (i - 1) / 2;
        if (heap[p].h <= heap[i].h) break;
        Cell t = heap[p]; heap[p] = heap[i]; heap[i] = t;
        i = p;
    }
}

static void siftDown(Cell *heap, int size, int i) {
    while (1) {
        int l = 2 * i + 1, r = 2 * i + 2, s = i;
        if (l < size && heap[l].h < heap[s].h) s = l;
        if (r < size && heap[r].h < heap[s].h) s = r;
        if (s == i) break;
        Cell t = heap[s]; heap[s] = heap[i]; heap[i] = t;
        i = s;
    }
}

/* terrain is an m x n grid; returns total trapped water volume. */
int trapWater(int** terrain, int m, int n) {
    if (m < 3 || n < 3) return 0;
    Cell *heap = (Cell *)malloc(sizeof(Cell) * (size_t)m * n);
    char *seen = (char *)calloc((size_t)m * n, 1);
    int size = 0;

    /* Seed the min-heap with every boundary cell. */
    for (int i = 0; i < m; ++i)
        for (int j = 0; j < n; ++j)
            if (i == 0 || i == m - 1 || j == 0 || j == n - 1) {
                heap[size].h = terrain[i][j];
                heap[size].r = i;
                heap[size].c = j;
                siftUp(heap, size);
                ++size;
                seen[i * n + j] = 1;
            }

    long ans = 0;
    int dr[4] = {1, -1, 0, 0};
    int dc[4] = {0, 0, 1, -1};

    while (size > 0) {
        Cell top = heap[0];
        heap[0] = heap[--size];
        siftDown(heap, size, 0);
        for (int k = 0; k < 4; ++k) {
            int x = top.r + dr[k], y = top.c + dc[k];
            if (x < 0 || x >= m || y < 0 || y >= n || seen[x * n + y]) continue;
            seen[x * n + y] = 1;
            int g = terrain[x][y];
            if (g < top.h) {
                ans += top.h - g;         /* water fills this cell up to top.h */
                heap[size].h = top.h;
            } else {
                heap[size].h = g;
            }
            heap[size].r = x;
            heap[size].c = y;
            siftUp(heap, size);
            ++size;
        }
    }

    free(heap);
    free(seen);
    return (int)ans;
}

#### Follow-up
questions
How does the solution simplify to the 1D version (a single row of bars) using two pointers in O(n)?
Instead of the total volume, return the final water level of every cell, or the set of submerged cells.
What changes if water can also flow diagonally between cells?
How would you handle a grid too large to fit the whole seen matrix or heap in memory?

