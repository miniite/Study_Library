---
type: "practice"
difficulty: "Other"
---

# Rolling a Marble to the Pocket With the Shortest, Smallest Path

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Graph](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Graph.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

A frictionless marble sits on a rectangular grid. Each cell is either open (0) or a wall (1). Somewhere on the grid there is a single pocket cell.

The marble moves by rolling in one of four directions — up (u), down (d), left (l), or right (r). Because there is no friction, once the marble starts rolling in a direction it does not stop until either:

the next cell in that direction is a wall, or the grid boundary (assume every border cell is effectively a wall), in which case it comes to rest against that wall; or
it rolls onto the pocket, in which case it drops in and stops there immediately (even in the middle of a longer roll).

When the marble is at rest it may pick a new direction and roll again. Given the grid, the marble's starting cell start = [startRow, startCol], and the pocket cell pocket = [pocketRow, pocketCol], return a string of move instructions that drops the marble into the pocket while traveling the shortest possible distance.

The distance of a route is the number of open cells the marble passes through, counting from the cell after the start up to and including the pocket. Among all routes achieving the minimum distance, return the one whose instruction string is lexicographically smallest (ordering the letters as 'd' < 'l' < 'r' < 'u' by normal character order). If the marble can never reach the pocket, return the string "impossible".

Input

grid: an m x n matrix of 0/1 values.
start: the marble's starting coordinates [startRow, startCol] (an open cell).
pocket: the pocket's coordinates [pocketRow, pocketCol] (an open cell, different from start).

Output

The lexicographically smallest instruction string among all shortest routes, or "impossible" if none exists.

#### Cons
traints

m == grid.length, n == grid[i].length
1 <= m, n <= 100
grid[i][j] is 0 or 1.
start.length == pocket.length == 2
Both start and pocket lie on open cells and are not the same cell.
The grid contains at least two open cells.

#### Example
s & test cases
TEXT
Input: grid = [[0,0,0,0,0],[1,1,0,0,1],[0,0,0,0,0],[0,1,0,0,1],[0,1,0,0,0]], start = [4,3], pocket = [0,1] -> Output: "lul"
Why: two routes reach the pocket in the minimum distance 6 — "lul" and "ul"; "lul" is lexicographically smaller because 'l' < 'u'.

Input: grid = [[0,0,0,0,0],[1,1,0,0,1],[0,0,0,0,0],[0,1,0,0,1],[0,1,0,0,0]], start = [4,3], pocket = [3,0] -> Output: "impossible"
Why: every rolling path either overshoots or is blocked, so the marble can never come to rest in that pocket.

Input: grid = [[0,0,0,0,0,0,0],[0,0,1,0,0,1,0],[0,0,0,0,1,0,0],[0,0,0,0,0,0,1]], start = [0,4], pocket = [3,5] -> Output: "dldr"
Why: the unique shortest drop rolls down, left, down, then right into the pocket.

Input: grid = [[0,0]], start = [0,0], pocket = [0,1] -> Output: "r"
Why: a single roll to the right lands directly on the pocket at distance 1 — the minimal case.

Input: grid = [[0,1,0]], start = [0,0], pocket = [0,2] -> Output: "impossible"
Why: a wall in the middle stops the marble before the pocket, and no other direction reaches it.

#### Approach

Treat each cell where the marble can come to rest (against a wall) plus the pocket as graph nodes; an edge is one full roll in a direction, whose weight is the number of open cells crossed. Run Dijkstra from the start, but make the priority a pair (distance, instruction-string) so that among equal-distance routes the lexicographically smallest path is expanded first — the first time the pocket is popped, its path is both shortest and smallest. Rolling stops early when it lands on the pocket (the marble drops in mid-roll). A key detail that keeps the greedy tie-break correct: you can never reach the same rest cell at the same distance via one path and a strict prefix of it (a roll always advances at least one cell), so keeping the single best (distance, path) per cell never discards the true optimum. The underlying pattern is weighted shortest path (Dijkstra) with a lexicographic secondary key.

Complexity

Time: O(m * n * log(m * n)) — each rest cell is settled once and each pop performs O(1) rolls of total length O(m + n); the priority queue holds O(m * n) states, so heap operations dominate at a log factor. Space: O(m * n) for the distance grid and the queued states (path strings add up to O(m * n) total in the worst case).

Python
PYTHON
import heapq
from typing import List

class Solution:
    def shortestDropPath(self, grid: List[List[int]], start: List[int], pocket: List[int]) -> str:
        m, n = len(grid), len(grid[0])
        dirs = [(1, 0, 'd'), (0, -1, 'l'), (0, 1, 'r'), (-1, 0, 'u')]
        dist = [[float('inf')] * n for _ in range(m)]
        dist[start[0]][start[1]] = 0
        # heap ordered by (distance, path); ties broken by lexicographic path
        pq = [(0, "", start[0], start[1])]

        while pq:
            cd, path, x, y = heapq.heappop(pq)
            if [x, y] == pocket:
                return path
            if cd > dist[x][y]:
                continue
            for dx, dy, ch in dirs:
                r, c, d = x, y, 0
                while 0 <= r + dx < m and 0 <= c + dy < n and grid[r + dx][c + dy] == 0:
                    r += dx
                    c += dy
                    d += 1
                    if [r, c] == pocket:  # dropped in
                        break
                if d == 0:
                    continue
                nd = cd + d
                if nd < dist[r][c]:
                    dist[r][c] = nd
                    heapq.heappush(pq, (nd, path + ch, r, c))
        return "impossible"
JavaScript
JAVASCRIPT
function shortestDropPath(grid, start, pocket) {

#### cons
t m = grid.length, n = grid[0].length;

#### cons
t dirs = [[1, 0, "d"], [0, -1, "l"], [0, 1, "r"], [-1, 0, "u"]];

#### cons
t dist = Array.from({ length: m }, () => Array(n).fill(Infinity));
    dist[start[0]][start[1]] = 0;
    // small array used as a priority queue, ordered by (distance, path)

#### cons
t pq = [[0, "", start[0], start[1]]];

    while (pq.length) {
        pq.sort((a, b) => a[0] - b[0] || a[1].localeCompare(b[1]));

#### cons
t [cd, path, x, y] = pq.shift();
        if (x === pocket[0] && y === pocket[1]) return path;
        if (cd > dist[x][y]) continue;
        for (const [dx, dy, ch] of dirs) {
            let r = x, c = y, d = 0;
            while (r + dx >= 0 && r + dx < m && c + dy >= 0 && c + dy < n && grid[r + dx][c + dy] === 0) {
                r += dx; c += dy; d++;
                if (r === pocket[0] && c === pocket[1]) break; // dropped in
            }
            if (d === 0) continue;

#### cons
t nd = cd + d;
            if (nd < dist[r][c]) {
                dist[r][c] = nd;
                pq.push([nd, path + ch, r, c]);
            }
        }
    }
    return "impossible";
}
Java
JAVA
import java.util.*;

class Solution {
    public String shortestDropPath(int[][] grid, int[] start, int[] pocket) {
        int m = grid.length, n = grid[0].length;
        int[] dR = {1, 0, 0, -1}, dC = {0, -1, 1, 0};
        char[] chr = {'d', 'l', 'r', 'u'};

        int[][] dist = new int[m][n];
        for (int[] row : dist) Arrays.fill(row, Integer.MAX_VALUE);

        // State: {distance, row, col} with path carried alongside; order by (dist, path)
        PriorityQueue<Object[]> pq = new PriorityQueue<>((a, b) -> {
            int da = (int) a[0], db = (int) b[0];
            if (da != db) return Integer.compare(da, db);
            return ((String) a[1]).compareTo((String) b[1]);
        });
        dist[start[0]][start[1]] = 0;
        pq.add(new Object[]{0, "", start[0], start[1]});

        while (!pq.isEmpty()) {
            Object[] cur = pq.poll();
            int cd = (int) cur[0]; String path = (String) cur[1];
            int x = (int) cur[2], y = (int) cur[3];
            if (x == pocket[0] && y == pocket[1]) return path;
            if (cd > dist[x][y]) continue;
            for (int k = 0; k < 4; k++) {
                int r = x, c = y, d = 0;
                while (r + dR[k] >= 0 && r + dR[k] < m && c + dC[k] >= 0 && c + dC[k] < n &&
                       grid[r + dR[k]][c + dC[k]] == 0) {
                    r += dR[k]; c += dC[k]; d++;
                    if (r == pocket[0] && c == pocket[1]) break;
                }
                if (d == 0) continue;
                int nd = cd + d;
                if (nd < dist[r][c]) {
                    dist[r][c] = nd;
                    pq.add(new Object[]{nd, path + chr[k], r, c});
                }
            }
        }
        return "impossible";
    }
}
C++
CPP
#include <vector>
#include <string>
#include <queue>
#include <climits>
using namespace std;

class Solution {
public:
    string shortestDropPath(vector<vector<int>>& grid, vector<int>& start, vector<int>& pocket) {
        int m = grid.size(), n = grid[0].size();
        int dR[4] = {1, 0, 0, -1}, dC[4] = {0, -1, 1, 0};
        char chr[4] = {'d', 'l', 'r', 'u'};

        vector<vector<int>> dist(m, vector<int>(n, INT_MAX));
        // min-heap ordered by (distance, path) so we always expand the best route first
        using State = tuple<int, string, int, int>; // dist, path, row, col
        priority_queue<State, vector<State>, greater<State>> pq;
        dist[start[0]][start[1]] = 0;
        pq.push({0, "", start[0], start[1]});

        while (!pq.empty()) {
            auto [cd, path, x, y] = pq.top(); pq.pop();
            if (x == pocket[0] && y == pocket[1]) return path;
            if (cd > dist[x][y]) continue;
            for (int k = 0; k < 4; k++) {
                int r = x, c = y, d = 0;
                while (r + dR[k] >= 0 && r + dR[k] < m && c + dC[k] >= 0 && c + dC[k] < n &&
                       grid[r + dR[k]][c + dC[k]] == 0) {
                    r += dR[k]; c += dC[k]; d++;
                    if (r == pocket[0] && c == pocket[1]) break;
                }
                if (d == 0) continue;
                int nd = cd + d;
                string np = path + chr[k];
                if (nd < dist[r][c]) {
                    dist[r][c] = nd;
                    pq.push({nd, np, r, c});
                }
            }
        }
        return "impossible";
    }
};
C
C
#include <stdlib.h>
#include <string.h>
#include <limits.h>

typedef struct { int dist; char* path; int r, c; } Node;
typedef struct { Node* a; int size, cap; } Heap;

/* Order by distance, then lexicographically by the instruction string. */
static int nodeLess(const Node* x, const Node* y) {
    if (x->dist != y->dist) return x->dist < y->dist;
    return strcmp(x->path, y->path) < 0;
}
static void heapPush(Heap* h, Node v) {
    if (h->size == h->cap) { h->cap = h->cap ? h->cap * 2 : 16;
        h->a = (Node*)realloc(h->a, sizeof(Node) * h->cap); }
    int i = h->size++; h->a[i] = v;
    while (i > 0) { int p = (i - 1) / 2;
        if (nodeLess(&h->a[i], &h->a[p])) { Node t = h->a[i]; h->a[i] = h->a[p]; h->a[p] = t; i = p; }
        else break; }
}
static Node heapPop(Heap* h) {
    Node top = h->a[0]; h->a[0] = h->a[--h->size]; int i = 0;
    while (1) { int l = 2*i+1, rr = 2*i+2, s = i;
        if (l < h->size && nodeLess(&h->a[l], &h->a[s])) s = l;
        if (rr < h->size && nodeLess(&h->a[rr], &h->a[s])) s = rr;
        if (s == i) break;
        Node t = h->a[i]; h->a[i] = h->a[s]; h->a[s] = t; i = s; }
    return top;
}

char* shortestDropPath(int** grid, int gridSize, int* gridColSize, int* start, int* pocket) {
    int m = gridSize, n = gridColSize[0];
    int dR[4] = {1, 0, 0, -1}, dC[4] = {0, -1, 1, 0};
    char chr[4] = {'d', 'l', 'r', 'u'};

    int** dist = (int**)malloc(sizeof(int*) * m);
    char*** best = (char***)malloc(sizeof(char**) * m);
    for (int i = 0; i < m; i++) {
        dist[i] = (int*)malloc(sizeof(int) * n);
        best[i] = (char**)malloc(sizeof(char*) * n);
        for (int j = 0; j < n; j++) { dist[i][j] = INT_MAX; best[i][j] = NULL; }
    }

    Heap h = {0};
    dist[start[0]][start[1]] = 0;
    Node s0; s0.dist = 0; s0.path = strdup(""); s0.r = start[0]; s0.c = start[1];
    heapPush(&h, s0);

    char* answer = NULL;
    while (h.size) {
        Node cur = heapPop(&h);
        if (cur.r == pocket[0] && cur.c == pocket[1]) { answer = strdup(cur.path); free(cur.path); break; }
        if (cur.dist > dist[cur.r][cur.c] ||
            (best[cur.r][cur.c] && strcmp(cur.path, best[cur.r][cur.c]) != 0)) { free(cur.path); continue; }
        for (int k = 0; k < 4; k++) {
            int r = cur.r, c = cur.c, d = 0;
            while (r + dR[k] >= 0 && r + dR[k] < m && c + dC[k] >= 0 && c + dC[k] < n &&
                   grid[r + dR[k]][c + dC[k]] == 0) {
                r += dR[k]; c += dC[k]; d++;
                if (r == pocket[0] && c == pocket[1]) break; /* dropped into the pocket */
            }
            if (d == 0) continue; /* immediately blocked: not a move */
            int nd = cur.dist + d;
            int len = (int)strlen(cur.path);
            char* np = (char*)malloc(len + 2);
            memcpy(np, cur.path, len); np[len] = chr[k]; np[len + 1] = '\0';
            if (nd < dist[r][c] || (nd == dist[r][c] && best[r][c] && strcmp(np, best[r][c]) < 0)) {
                dist[r][c] = nd;
                if (best[r][c]) free(best[r][c]);
                best[r][c] = strdup(np);
                Node nn; nn.dist = nd; nn.path = np; nn.r = r; nn.c = c;
                heapPush(&h, nn);
            } else free(np);
        }
        free(cur.path);
    }

    for (int i = 0; i < m; i++) { free(dist[i]);
        for (int j = 0; j < n; j++) if (best[i][j]) free(best[i][j]);
        free(best[i]); }
    free(dist); free(best);
    while (h.size) free(heapPop(&h).path);
    free(h.a);

    if (!answer) answer = strdup("impossible");
    return answer;
}

#### Follow-up
questions
Only the shortest distance is required, not the path — how does that simplify the state and the tie-break?
If instead you must return the lexicographically largest shortest path, what changes in the comparator?
Suppose the marble can also stop voluntarily at any open cell (friction is optional) — how does the graph and complexity change?
Handle multiple pockets and report which one is reachable most cheaply, with the smallest path.

