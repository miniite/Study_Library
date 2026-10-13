---
type: "practice"
difficulty: "Intermediate"
---

# Given a 2D grid of land and water, count the number of islands.

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Grid Islands](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Grid_Islands.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

What's really being asked: Given a 2D grid where each cell is either land ('1') or water ('0'), count the number of distinct islands. An island is a maximal group of land cells connected horizontally or vertically (4-directional adjacency), surrounded by water or the grid edge.

What the interviewer is probing: Do you recognize this as a connected-components on an implicit graph problem? Can you write a clean grid traversal (DFS/BFS/Union-Find), handle boundaries, avoid revisiting cells (infinite loops), and reason about time/space complexity? It's a bread-and-butter graph question, so cleanliness and correctness under edge cases matter more than cleverness.

Clarifying questions I'd ask:

Is connectivity 4-directional (up/down/left/right) or 8-directional (also diagonals)? Standard is 4-directional; I'll code that and note the one-line change for 8-dir.
Can I mutate the input grid (mark visited in place), or must I keep it immutable? In-place marking saves the O(m·n) visited array. I'll show both.
What are the value types — chars '1'/'0' (LeetCode style) or ints? I'll assume chars; trivially adaptable.
Grid size / memory limits — matters for choosing DFS recursion vs. BFS/iterative to avoid stack overflow on huge grids.

Plan: Model the grid as a graph where each land cell is a node connected to its 4 land neighbors. The number of islands = number of connected components. I'll scan every cell; when I hit an unvisited land cell, that's a new island — I increment the counter and flood-fill (DFS or BFS) all connected land, marking it visited so I never count it again. I'll then cover Union-Find as an alternative and discuss complexity/edge cases.

Core algorithm (flood fill)
CODE
count = 0
for each cell (r, c) in grid:
    if grid[r][c] == '1' (land, unvisited):
        count += 1
        flood_fill(r, c)   # sink the whole island so it's counted once
return count

The flood_fill explores the entire connected land mass starting at (r,c) and marks each visited cell (e.g., set it to '0' in place, or set a visited[r][c]=true). Because we sink an island the first time we touch it, the outer loop only ever starts a flood fill once per island — hence the count equals the number of islands.

DFS variant: recurse into the 4 neighbors that are in-bounds and land. Simple, but recursion depth can reach O(m·n) on a grid that is entirely land → risk of stack overflow for very large grids.

BFS variant: use a queue; pop a cell, enqueue its unvisited land neighbors, mark visited on enqueue (not on dequeue) to prevent the same cell being queued twice. Iterative → no recursion-depth risk. Preferred for very large grids.

Complexity
Time: O(m · n). Every cell is examined a constant number of times: once by the outer scan and, if it's land, once when flood-filled. Marking visited guarantees no cell is processed twice.
Space:
DFS recursion: O(m · n) worst case (all land → deep stack).
BFS queue: O(min(m, n)) in the best framing but O(m · n) worst case for the queue.
Extra visited array: O(m · n); eliminated if we mutate the grid in place.
Alternative: Union-Find (Disjoint Set Union)

Model each land cell as its own set. For every land cell, union it with its land neighbor to the right and below (only two directions needed to cover all adjacencies without double work). The answer is the number of distinct roots among land cells. This shines when the grid is dynamic (a natural follow-up: "Number of Islands II" — cells turn from water to land one at a time and you must report the island count after each addition). With path compression + union by rank, each op is near-O(α(n)) (inverse Ackermann, effectively constant). For a static grid, DFS/BFS is simpler and equally O(m·n), so I'd default to flood fill unless the dynamic variant is asked.

Edge cases to call out
Empty grid (null or 0 rows/cols) → return 0.
All water → 0. All land → 1 (one giant island).
Single cell → 1 if land, 0 if water.
Diagonal-only touching cells are separate islands under 4-dir connectivity (e.g., a checkerboard of land = many single-cell islands).
Boundary handling: the bounds check 0 <= r < m && 0 <= c < n must come before indexing to avoid out-of-bounds.
Mutating the input: if the caller needs the grid afterward, restore it or use a separate visited set.
What a strong answer emphasizes
Framing it as connected components on an implicit graph — that's the "aha."
A clean flood fill with correct visited-marking (mark on visit/enqueue, not later) so islands aren't double-counted and there are no infinite loops.
Rigorous bounds checks and edge cases (empty, all-water, all-land, diagonals-don't-connect).
O(m·n) time; being honest about DFS recursion depth vs. iterative BFS for large inputs.
Knowing Union-Find as the right tool if the grid becomes dynamic (Number of Islands II) or if you need the parallelizable/streaming variant.

#### Follow-up
probes I'd be ready for: count island sizes / the largest island (return max region area); Number of Islands II (dynamic additions → Union-Find with a running count that decrements on each successful union); max area of island; and number of distinct island shapes (canonicalize each shape by normalized coordinates). All are variations on this same flood-fill/DSU skeleton.

Summary: Scan the grid; each time you find unvisited land, increment the count and flood-fill (DFS or BFS) to sink that entire island so it's counted exactly once. O(m·n) time, O(m·n) worst-case space (or O(1) extra if you mutate in place). Use Union-Find when the grid is dynamic.

**Comparison:**

```
Approach	When to use	Time	Extra space	Notes
DFS flood fill (in place)	Static grid, simplest	O(m·n)	O(m·n) stack worst case	Recursion depth risk on all-land grids
BFS flood fill (queue)	Static grid, very large	O(m·n)	O(m·n) queue worst case	Iterative; mark on enqueue
Union-Find (DSU)	Dynamic grid (Islands II), streaming adds	O(m·n·α)	O(m·n)	Near-constant per op; decrement count on union

(α = inverse Ackermann ≈ constant. 8-directional connectivity = add the 4 diagonal offsets.)
```

**Code:**

```

// DFS flood fill, in-place marking (no extra visited array) class Solution { public int numIslands(char[][] grid) { if (grid == null || grid.length == 0) return 0; int m = grid.length, n = grid[0].length, count = 0; for (int r = 0; r < m; r++) { for (int c = 0; c < n; c++) { if (grid[r][c] == '1') { // new, unvisited island count++; dfs(grid, r, c); // sink it } } } return count; } private void dfs(char[][] grid, int r, int c) { if (r < 0 || c < 0 || r >= grid.length || c >= grid[0].length || grid[r][c] != '1') return; // bounds + water/visited check grid[r][c] = '0'; // mark visited (sink) dfs(grid, r + 1, c); dfs(grid, r - 1, c); dfs(grid, r, c + 1); dfs(grid, r, c - 1); } }

CODE
Copy

```java
// BFS flood fill (iterative — safe for very large grids), marks visited on enqueue
class SolutionBFS {
    public int numIslands(char[][] grid) {
        if (grid == null || grid.length == 0) return 0;
        int m = grid.length, n = grid[0].length, count = 0;
        int[][] dirs = {{1,0},{-1,0},{0,1},{0,-1}};
        java.util.Deque<int[]> q = new java.util.ArrayDeque<>();
        for (int r = 0; r < m; r++) {
            for (int c = 0; c < n; c++) {
                if (grid[r][c] != '1') continue;
                count++;
                grid[r][c] = '0';                 // mark on enqueue
                q.add(new int[]{r, c});
                while (!q.isEmpty()) {
                    int[] cell = q.poll();
                    for (int[] d : dirs) {
                        int nr = cell[0] + d[0], nc = cell[1] + d[1];
                        if (nr >= 0 && nc >= 0 && nr < m && nc < n && grid[nr][nc] == '1') {
                            grid[nr][nc] = '0';   // mark BEFORE enqueue -> no duplicates
                            q.add(new int[]{nr, nc});
                        }
                    }
                }
            }
        }
        return count;
    }
}
PYTHON
Copy
# Python DFS (concise)
class Solution:
    def numIslands(self, grid):
        if not grid:
            return 0
        m, n, count = len(grid), len(grid[0]), 0
        def dfs(r, c):
            if r < 0 or c < 0 or r >= m or c >= n or grid[r][c] != '1':
                return
            grid[r][c] = '0'
            dfs(r+1, c); dfs(r-1, c); dfs(r, c+1); dfs(r, c-1)
        for r in range(m):
            for c in range(n):
                if grid[r][c] == '1':
                    count += 1
                    dfs(r, c)
        return count
```

**Example input / output:**

```

Example 1:

CODE
Copy
Input grid:
1 1 0 0 0
1 1 0 0 0
0 0 1 0 0
0 0 0 1 1
Output: 3

Trace: scan hits (0,0)=1 -> count=1, flood-fill sinks the top-left 2x2 block. Next new land at (2,2) -> count=2, sinks the single cell. Next new land at (3,3) -> count=3, sinks the two-cell strip (3,3)-(3,4). Total = 3.

Example 2 (one big island):

CODE
Copy
Input:
1 1 1
1 1 1
1 1 1
Output: 1

Example 3 (diagonals do NOT connect under 4-dir):

CODE
Copy
Input:
1 0 1
0 1 0
1 0 1
Output: 5   (five isolated single-cell islands)

Edge cases:

CODE
Copy
Input: []            -> Output: 0
Input: [[0,0],[0,0]] -> Output: 0   (all water)
Input: [[1]]         -> Output: 1
```

**Flow:**

1. no
2. yes
3. all cells done
4. Start: count = 0
5. Scan cells row by row
6. cell == land '1'?
7. count += 1
8. Flood fill (DFS/BFS):sink all connected land,mark visited
9. return count

