---
type: "practice"
difficulty: "Other"
---

# Fewest Removals to Separate All Occupied Cells in a Grid

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Graph](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Graph.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given a rectangular grid of cells with m rows and n columns. Every cell holds one of two values: 1 (the cell is occupied) or 0 (the cell is empty). The grid is 0-indexed.

In a single operation you may pick any occupied cell and clear it (change its value from 1 to 0). You may never turn an empty cell into an occupied one.

The grid is called separated when no two occupied cells are directly adjacent — that is, there is no pair of occupied cells that share an edge horizontally or vertically. (Diagonal contact does not count as adjacency.)

Return the minimum number of operations needed to make the grid separated.

Input

grid: a 2-D array of m rows by n columns, each entry 0 or 1.

Output

A single integer: the least number of occupied cells you must clear so that no two remaining occupied cells touch edge-to-edge.

#### Cons
traints

m == grid.length
n == grid[i].length
1 <= m, n <= 300
Every grid[i][j] is either 0 or 1.

<<FIELD:question_note>> (This is the minimum-vertex-cover of the adjacency graph of occupied cells; because that graph is bipartite by cell parity, the answer equals the maximum bipartite matching.)

#### Example
s & test cases
TEXT
Input: grid = [[1,1,0],[0,1,1],[1,1,1]] -> Output: 3
Why: Clearing the cells at (0,1), (1,2), and (2,1) leaves no two occupied cells edge-adjacent; three is the fewest possible.

Input: grid = [[0,0,0],[0,0,0]] -> Output: 0
Why: There are no occupied cells at all, so the grid is already separated — an all-empty (or occupied-free) edge case.

Input: grid = [[0,1],[1,0]] -> Output: 0
Why: The two occupied cells touch only diagonally, which does not count as adjacency, so nothing must be cleared.

Input: grid = [[1,1],[1,1]] -> Output: 2
Why: The four occupied cells form a 4-cycle; a minimum vertex cover of a 4-cycle has size 2 (e.g. clear (0,0) and (1,1)).

Input: grid = [[1,1,1,1]] -> Output: 2
Why: A single row of four occupied cells is a path of 4 nodes; its minimum vertex cover is 2 (clear the 2nd and... clear indices 1 and 2 covers all three edges).

Input: grid = [[1]] -> Output: 0
Why: A lone occupied cell has no neighbor, so it is already separated — the minimum grid edge case.

#### Approach

Think of each occupied cell as a graph vertex and put an edge between two occupied cells whenever they are 4-directionally adjacent. Clearing a cell removes a vertex and all its edges; making the grid "separated" means destroying every edge, so we want the smallest set of vertices touching all edges — a minimum vertex cover. Coloring cells by the parity of row + col two-colors the grid like a checkerboard, and any adjacency joins an even cell to an odd cell, so the graph is bipartite. By Koenig's theorem, in a bipartite graph the minimum vertex cover equals the maximum matching. We compute that matching with Kuhn's augmenting-path algorithm: iterate over even-parity occupied cells, and for each try to find an augmenting path to an unmatched (or re-matchable) adjacent odd-parity occupied cell. The number of matched pairs is the answer. Pattern: reduction to bipartite matching / min vertex cover.

Complexity

Time: O(V * E) in the worst case, where V is the number of occupied cells on the even-parity side and E is the number of adjacent occupied-cell pairs (at most ~2mn edges). Each of the V starting cells may scan the edge set once during augmentation. Space: O(m*n) for the match, visited, and recursion structures. The core insight (min vertex cover = max matching on a bipartite graph) is what makes a polynomial solution possible at all.

Python
PYTHON
import sys

def min_operations(grid):
    """Fewest occupied cells to clear so none remain edge-adjacent.

    Occupied cells split by (row+col) parity into a bipartite graph whose edges
    are 4-directional adjacencies. The answer is the minimum vertex cover, which
    by Koenig's theorem equals the maximum bipartite matching. Found with Kuhn's
    augmenting-path algorithm.
    """
    if not grid or not grid[0]:
        return 0
    m, n = len(grid), len(grid[0])
    match_row = [-1] * (m * n)   # right-cell id -> matched left-cell id, or -1
    seen = [0] * (m * n)         # visited marker per augmentation, by token
    dirs = ((1, 0), (-1, 0), (0, 1), (0, -1))
    token = 0

    def augment(idx):
        r, c = divmod(idx, n)
        for dr, dc in dirs:
            nr, nc = r + dr, c + dc
            if 0 <= nr < m and 0 <= nc < n and grid[nr][nc] == 1:
                rid = nr * n + nc          # odd-parity "right" cell
                if seen[rid] != token:
                    seen[rid] = token
                    if match_row[rid] == -1 or augment(match_row[rid]):
                        match_row[rid] = idx
                        return True
        return False

    sys.setrecursionlimit(1 << 20)
    matching = 0
    for r in range(m):
        for c in range(n):
            if grid[r][c] == 1 and (r + c) % 2 == 0:   # even parity = left side
                token += 1
                if augment(r * n + c):
                    matching += 1
    return matching

if __name__ == "__main__":
    print(min_operations([[1, 1, 0], [0, 1, 1], [1, 1, 1]]))  # 3
    print(min_operations([[1, 1], [1, 1]]))                   # 2
    print(min_operations([[0, 1], [1, 0]]))                   # 0
    print(min_operations([[1, 1, 1, 1]]))                     # 2
JavaScript
JAVASCRIPT
function minOperations(grid) {

#### cons
t M = grid.length, N = grid[0].length;

#### cons
t matchRow = new Int32Array(M * N).fill(-1); // right id -> left id, or -1

#### cons
t seen = new Int32Array(M * N);              // visited marker by token

#### cons
t DR = [1, -1, 0, 0], DC = [0, 0, 1, -1];
    let token = 0;

    function augment(idx) {

#### cons
t r = (idx / N) | 0, c = idx % N;
        for (let d = 0; d < 4; d++) {

#### cons
t nr = r + DR[d], nc = c + DC[d];
            if (nr < 0 || nr >= M || nc < 0 || nc >= N) continue;
            if (grid[nr][nc] !== 1) continue;         // neighbor must be occupied

#### cons
t rid = nr * N + nc;                   // odd-parity "right" cell
            if (seen[rid] !== token) {
                seen[rid] = token;
                if (matchRow[rid] === -1 || augment(matchRow[rid])) {
                    matchRow[rid] = idx;
                    return true;
                }
            }
        }
        return false;
    }

    let matching = 0;
    for (let r = 0; r < M; r++)
        for (let c = 0; c < N; c++)
            if (grid[r][c] === 1 && ((r + c) & 1) === 0) { // even parity = left side
                token++;
                if (augment(r * N + c)) matching++;
            }
    return matching;   // max matching == min vertex cover (Koenig)
}

// Demonstration

#### cons
ole.log(minOperations([[1,1,0],[0,1,1],[1,1,1]])); // 3

#### cons
ole.log(minOperations([[1,1],[1,1]]));             // 2

#### cons
ole.log(minOperations([[0,1],[1,0]]));             // 0

#### cons
ole.log(minOperations([[1,1,1,1]]));               // 2
Java
JAVA
import java.util.*;

public class Solution {
    private int M, N, token;
    private int[] matchRow, seen;
    private int[][] g;
    private static final int[] DR = {1, -1, 0, 0};
    private static final int[] DC = {0, 0, 1, -1};

    private boolean augment(int idx) {
        int r = idx / N, c = idx % N;
        for (int d = 0; d < 4; d++) {
            int nr = r + DR[d], nc = c + DC[d];
            if (nr < 0 || nr >= M || nc < 0 || nc >= N) continue;
            if (g[nr][nc] != 1) continue;          // neighbor must be occupied
            int rid = nr * N + nc;                  // odd-parity "right" cell
            if (seen[rid] != token) {
                seen[rid] = token;
                if (matchRow[rid] == -1 || augment(matchRow[rid])) {
                    matchRow[rid] = idx;
                    return true;
                }
            }
        }
        return false;
    }

    public int minOperations(int[][] grid) {
        M = grid.length;
        N = grid[0].length;
        g = grid;
        matchRow = new int[M * N];
        seen = new int[M * N];
        Arrays.fill(matchRow, -1);
        token = 0;
        int matching = 0;
        for (int r = 0; r < M; r++)
            for (int c = 0; c < N; c++)
                if (grid[r][c] == 1 && ((r + c) & 1) == 0) {  // even parity = left side
                    token++;
                    if (augment(r * N + c)) matching++;
                }
        return matching;   // max matching == min vertex cover (Koenig)
    }

    public static void main(String[] args) {

#### Solution
s = new Solution();
        System.out.println(s.minOperations(new int[][]{{1,1,0},{0,1,1},{1,1,1}})); // 3
        System.out.println(s.minOperations(new int[][]{{1,1},{1,1}}));             // 2
        System.out.println(s.minOperations(new int[][]{{0,1},{1,0}}));             // 0
        System.out.println(s.minOperations(new int[][]{{1,1,1,1}}));              // 2
    }
}
C++
CPP
#include <bits/stdc++.h>
using namespace std;

class Solution {
    int M, N, token;
    vector<int> matchRow, seen;
    vector<vector<int>>* g;

#### cons
t int DR[4] = {1, -1, 0, 0};

#### cons
t int DC[4] = {0, 0, 1, -1};

    bool augment(int idx) {
        int r = idx / N, c = idx % N;
        for (int d = 0; d < 4; d++) {
            int nr = r + DR[d], nc = c + DC[d];
            if (nr < 0 || nr >= M || nc < 0 || nc >= N) continue;
            if ((*g)[nr][nc] != 1) continue;      // neighbor must be occupied
            int rid = nr * N + nc;                 // odd-parity "right" cell
            if (seen[rid] != token) {
                seen[rid] = token;
                if (matchRow[rid] == -1 || augment(matchRow[rid])) {
                    matchRow[rid] = idx;
                    return true;
                }
            }
        }
        return false;
    }

public:
    int minOperations(vector<vector<int>>& grid) {
        M = grid.size();
        N = grid[0].size();
        g = &grid;
        matchRow.assign(M * N, -1);
        seen.assign(M * N, 0);
        token = 0;
        int matching = 0;
        for (int r = 0; r < M; r++)
            for (int c = 0; c < N; c++)
                if (grid[r][c] == 1 && ((r + c) & 1) == 0) {  // even parity = left side
                    token++;
                    if (augment(r * N + c)) matching++;
                }
        return matching;   // max matching == min vertex cover (Koenig)
    }
};

int main() {

#### Solution
s;
    vector<vector<int>> g1 = {{1,1,0},{0,1,1},{1,1,1}};
    vector<vector<int>> g2 = {{1,1},{1,1}};
    vector<vector<int>> g3 = {{0,1},{1,0}};
    cout << s.minOperations(g1) << "\n";  // 3
    cout << s.minOperations(g2) << "\n";  // 2
    cout << s.minOperations(g3) << "\n";  // 0
    return 0;
}
C
C
#include <stdio.h>
#include <stdlib.h>

static int M, N, gtoken;
static int *matchRow;   /* right-cell id -> matched left-cell id, or -1 */
static int *seen;       /* visited marker per augmentation, by token    */
static int **G;
static const int DR[4] = {1, -1, 0, 0};
static const int DC[4] = {0, 0, 1, -1};

/* Try to find an augmenting path starting from an even-parity occupied cell. */
static int augment(int idx) {
    int r = idx / N, c = idx % N, d;
    for (d = 0; d < 4; d++) {
        int nr = r + DR[d], nc = c + DC[d];
        if (nr < 0 || nr >= M || nc < 0 || nc >= N) continue;
        if (G[nr][nc] != 1) continue;               /* neighbor must be occupied */
        int rid = nr * N + nc;                        /* an odd-parity "right" cell */
        if (seen[rid] != gtoken) {
            seen[rid] = gtoken;
            if (matchRow[rid] == -1 || augment(matchRow[rid])) {
                matchRow[rid] = idx;
                return 1;
            }
        }
    }
    return 0;
}

int minOperations(int** grid, int gridSize, int* gridColSize) {
    int r, c, total, matching = 0;
    M = gridSize;
    N = gridColSize[0];
    G = grid;
    total = M * N;
    matchRow = (int*)malloc(sizeof(int) * total);
    seen     = (int*)malloc(sizeof(int) * total);
    for (r = 0; r < total; r++) { matchRow[r] = -1; seen[r] = 0; }
    gtoken = 0;
    for (r = 0; r < M; r++)
        for (c = 0; c < N; c++)
            if (grid[r][c] == 1 && ((r + c) & 1) == 0) {  /* even-parity = left side */
                gtoken++;
                if (augment(r * N + c)) matching++;
            }
    free(matchRow);
    free(seen);
    return matching;   /* max matching == min vertex cover (Koenig) */
}

int main(void) {
    int r0[] = {1,1,0}, r1[] = {0,1,1}, r2[] = {1,1,1};
    int* g[] = {r0, r1, r2};
    int cols[] = {3, 3, 3};
    printf("%d\n", minOperations(g, 3, cols));  /* 3 */

    int a0[] = {1,1}, a1[] = {1,1};
    int* g2[] = {a0, a1};
    int cols2[] = {2, 2};
    printf("%d\n", minOperations(g2, 2, cols2)); /* 2 */
    return 0;
}

#### Follow-up
questions
What changes if diagonal contact also counts as adjacency? (The graph is no longer bipartite by parity, so Koenig's theorem no longer applies and minimum vertex cover becomes NP-hard in general.)
Instead of just the count, return the actual set of cells to clear (recover a minimum vertex cover from the matching via the alternating-path / Koenig construction).
Some occupied cells are "fixed" and can never be cleared — how do you enforce that a cover avoids them, and when is it even feasible?
The grid can be up to millions of cells but very sparse; switch Kuhn's O(VE) matching to Hopcroft-Karp's O(Esqrt(V)) and store only occupied cells.

