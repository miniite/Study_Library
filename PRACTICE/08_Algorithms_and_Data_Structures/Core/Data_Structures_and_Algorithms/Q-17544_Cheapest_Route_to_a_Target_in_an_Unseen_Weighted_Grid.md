---
type: "practice"
difficulty: "Other"
---

# Cheapest Route to a Target in an Unseen Weighted Grid

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Depth-First Search](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Depth-First_Search.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

This is an interactive problem. A robot sits on some cell of a rectangular grid, and you must move it from its (unknown) starting cell to a hidden target cell as cheaply as possible. Every cell is either blocked or open, and each open cell has a positive entry cost that you pay each time the robot moves onto it. The robot's starting cell is open, the target cell is open, and the two are different. You are not told the grid's size, the robot's position, or where the target is.

You interact only through a helper object called Navigator, which exposes three methods (a direction is one of the characters 'U', 'D', 'L', 'R' for up, down, left, right):

boolean canMove(direction) — returns true if the robot can move one step in direction (the neighboring cell exists and is open), otherwise false.
int move(direction) — moves the robot one step in direction and returns the entry cost of the cell it moved onto. If the step would leave the grid or enter a blocked cell, the move is ignored, the robot stays put, and it returns -1.
boolean isTarget() — returns true if the robot currently stands on the target cell.

Return the minimum total entry cost to bring the robot from its starting cell to the target cell. The starting cell's own cost is never charged (you pay only for cells you move onto). If no path from start to target exists, return -1.

How test data is described (you do NOT receive this directly): a case is given as a 2D grid plus start (r1, c1) and target (r2, c2), where grid[i][j] == 0 marks a blocked cell and grid[i][j] >= 1 gives an open cell's entry cost.

#### Cons
traints

1 <= m, n <= 100, where the grid is m rows by n columns.
0 <= grid[i][j] <= 100; 0 means blocked, >= 1 means open with that entry cost.
The start and target are distinct open cells.

#### Example
s & test cases
TEXT
- Input: `grid = [[5,4],[2,7]]`, start `(0,0)`, target `(1,1)` -> Output: `9`
  Why: going down then right costs `2 + 7 = 9`, cheaper than right then down (`4 + 7 = 11`); the start cell's `5` is not charged.
- Input: `grid = [[9,1],[8,8]]`, start `(0,0)`, target `(0,1)` -> Output: `1`
  Why: a single step right pays only the target cell's entry cost `1` (adjacent target).
- Input: `grid = [[100,1],[1,1]]`, start `(0,0)`, target `(1,1)` -> Output: `2`
  Why: either two-step path costs `1 + 1 = 2`; the expensive start cost `100` is excluded, proving the start's cost is never applied.
- Input: `grid = [[1,2,3],[4,5,6],[7,8,9]]`, start `(0,0)`, target `(2,2)` -> Output: `20`
  Why: the cheapest route is right, right, down, down entering cells `2, 3, 6, 9` for `2+3+6+9 = 20`; a greedy/BFS choice would miss it, so Dijkstra is required.
- Input: `grid = [[3,0],[0,6]]`, start `(0,0)`, target `(1,1)` -> Output: `-1`
  Why: both neighbors of the start are blocked (`0`), so the target is unreachable.

#### Approach

Two phases. Phase 1 — map the reachable region. Because the grid, start, and target are hidden, pin the start at local coordinate (0,0) and run a DFS through the Navigator: for each of the four directions, if canMove is true, call move (recording the returned entry cost against the neighbor's coordinate), recurse into it, then immediately move back the opposite way so the robot's real position always matches the DFS position. During the walk, whenever isTarget is true, remember that cell. This discovers every open cell reachable from the start together with its entry cost, and locates the target (if reachable). Phase 2 — cheapest path. With the weighted subgrid now known in memory, run Dijkstra from (0,0), where the weight of stepping onto a neighbor is that neighbor's recorded entry cost and the start contributes 0 (its own cost is never charged). The distance to the target cell is the answer; if the target was never seen, return -1. Note that plain BFS is insufficient because entry costs vary, so fewest steps need not be cheapest — Dijkstra is required. Pattern: interactive DFS exploration plus Dijkstra on a weighted grid.

Complexity

Time: O(V log V), where V is the number of reachable cells (at most m*n <= 10^4). Exploration visits each reachable cell once (each move is undone by a move back), and Dijkstra with a binary heap processes each cell and its up-to-4 edges in O(log V). Space: O(V) for the discovered-cost map, the distance table, and the heap. Justification: full mapping is linear; weighted shortest path with non-negative costs is dominated by the heap operations.

Python
PYTHON
import heapq

class Solution:
    # Provided interactive object `nav`: nav.canMove(d), nav.move(d), nav.isTarget()
    def findMinCost(self, nav: 'Navigator') -> int:
        MOVES = {'U': (-1, 0), 'D': (1, 0), 'L': (0, -1), 'R': (0, 1)}
        BACK = {'U': 'D', 'D': 'U', 'L': 'R', 'R': 'L'}
        cost = {(0, 0): 0}          # entry cost of each discovered cell
        target = None

        def explore(r, c):
            nonlocal target
            if nav.isTarget():
                target = (r, c)
            for d, (dr, dc) in MOVES.items():
                nr, nc = r + dr, c + dc
                if (nr, nc) in cost:
                    continue
                if nav.canMove(d):
                    cost[(nr, nc)] = nav.move(d)
                    explore(nr, nc)
                    nav.move(BACK[d])        # backtrack to restore position

        explore(0, 0)
        if target is None:
            return -1

        dist = {(0, 0): 0}
        pq = [(0, 0, 0)]                     # (distance, r, c)
        while pq:
            d, r, c = heapq.heappop(pq)
            if (r, c) == target:
                return d
            if d > dist.get((r, c), float('inf')):
                continue
            for dr, dc in ((-1, 0), (1, 0), (0, -1), (0, 1)):
                nr, nc = r + dr, c + dc
                if (nr, nc) in cost:
                    nd = d + cost[(nr, nc)]
                    if nd < dist.get((nr, nc), float('inf')):
                        dist[(nr, nc)] = nd
                        heapq.heappush(pq, (nd, nr, nc))
        return -1
JavaScript
JAVASCRIPT
/* Provided interactive object `nav`:
 *   nav.canMove(dir) -> boolean
 *   nav.move(dir)    -> number   (entry cost of the entered cell, or -1 if the move is ignored)
 *   nav.isTarget()   -> boolean
 */
function findMinCost(nav) {

#### cons
t DIR  = ['U', 'D', 'L', 'R'];

#### cons
t BACK = { U: 'D', D: 'U', L: 'R', R: 'L' };

#### cons
t STEP = { U: [-1, 0], D: [1, 0], L: [0, -1], R: [0, 1] };

#### cons
t cost = new Map();               // "r,c" -> entry cost
    let target = null;

#### cons
t key = (r, c) => r + ',' + c;

#### cons
t explore = (r, c) => {
        if (nav.isTarget()) target = [r, c];
        for (const d of DIR) {

#### cons
t nr = r + STEP[d][0], nc = c + STEP[d][1];
            if (cost.has(key(nr, nc))) continue;
            if (nav.canMove(d)) {
                cost.set(key(nr, nc), nav.move(d));
                explore(nr, nc);
                nav.move(BACK[d]);         // backtrack
            }
        }
    };
    cost.set(key(0, 0), 0);
    explore(0, 0);
    if (target === null) return -1;

    // Dijkstra with a binary min-heap of [dist, r, c]

#### cons
t heap = [];

#### cons
t swap = (i, j) => { const t = heap[i]; heap[i] = heap[j]; heap[j] = t; };

#### cons
t push = (node) => {
        heap.push(node);
        let i = heap.length - 1;
        while (i > 0) {

#### cons
t p = (i - 1) >> 1;
            if (heap[p][0] <= heap[i][0]) break;
            swap(p, i); i = p;
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
            for (;;) {
                let s = i, l = 2 * i + 1, r = 2 * i + 2;
                if (l < heap.length && heap[l][0] < heap[s][0]) s = l;
                if (r < heap.length && heap[r][0] < heap[s][0]) s = r;
                if (s === i) break;
                swap(s, i); i = s;
            }
        }
        return top;
    };

#### cons
t dist = new Map([[key(0, 0), 0]]);
    push([0, 0, 0]);
    while (heap.length) {

#### cons
t [d, r, c] = pop();
        if (r === target[0] && c === target[1]) return d;
        if (d > (dist.get(key(r, c)) ?? Infinity)) continue;
        for (const dir of DIR) {

#### cons
t nr = r + STEP[dir][0], nc = c + STEP[dir][1];

#### cons
t k = key(nr, nc);
            if (!cost.has(k)) continue;

#### cons
t nd = d + cost.get(k);
            if (nd < (dist.get(k) ?? Infinity)) {
                dist.set(k, nd);
                push([nd, nr, nc]);
            }
        }
    }
    return -1;
}
Java
JAVA
import java.util.Arrays;
import java.util.PriorityQueue;

/* Provided interactive class `Navigator`:
 *   boolean canMove(char d); int move(char d); boolean isTarget(); */
class Solution {
    private static final int SIZE = 210, OFF = 105;
    private final int[]  DR   = {-1, 1, 0, 0};
    private final int[]  DC   = {0, 0, -1, 1};
    private final char[] DIR  = {'U', 'D', 'L', 'R'};
    private final char[] BACK = {'D', 'U', 'R', 'L'};
    private final int[][] cost = new int[SIZE][SIZE];
    private int tr = -1, tc = -1;

    public int findMinCost(Navigator nav) {
        for (int[] row : cost) Arrays.fill(row, -1);
        cost[OFF][OFF] = 0;
        explore(nav, OFF, OFF);
        if (tr == -1) return -1;

        int[][] dist = new int[SIZE][SIZE];
        for (int[] row : dist) Arrays.fill(row, Integer.MAX_VALUE);
        dist[OFF][OFF] = 0;
        PriorityQueue<int[]> pq = new PriorityQueue<>((a, b) -> Integer.compare(a[0], b[0]));
        pq.offer(new int[]{0, OFF, OFF});
        while (!pq.isEmpty()) {
            int[] cur = pq.poll();
            int d = cur[0], r = cur[1], c = cur[2];
            if (r == tr && c == tc) return d;
            if (d > dist[r][c]) continue;
            for (int i = 0; i < 4; i++) {
                int nr = r + DR[i], nc = c + DC[i];
                if (cost[nr][nc] < 0) continue;
                int nd = d + cost[nr][nc];
                if (nd < dist[nr][nc]) {
                    dist[nr][nc] = nd;
                    pq.offer(new int[]{nd, nr, nc});
                }
            }
        }
        return -1;
    }

    private void explore(Navigator nav, int r, int c) {
        if (nav.isTarget()) { tr = r; tc = c; }
        for (int i = 0; i < 4; i++) {
            int nr = r + DR[i], nc = c + DC[i];
            if (cost[nr][nc] != -1) continue;
            if (nav.canMove(DIR[i])) {
                cost[nr][nc] = nav.move(DIR[i]);
                explore(nav, nr, nc);
                nav.move(BACK[i]);          // backtrack
            }
        }
    }
}
C++
CPP
#include <array>
#include <climits>
#include <queue>
#include <vector>
using namespace std;

/* Provided interactive class `Navigator`:
 *   bool canMove(char d); int move(char d); bool isTarget(); */
class Solution {
    static const int SIZE = 210, OFF = 105;

#### cons
t int  DR[4]   = {-1, 1, 0, 0};

#### cons
t int  DC[4]   = {0, 0, -1, 1};

#### cons
t char DIR[4]  = {'U', 'D', 'L', 'R'};

#### cons
t char BACK[4] = {'D', 'U', 'R', 'L'};
    vector<vector<int>> cost = vector<vector<int>>(SIZE, vector<int>(SIZE, -1));
    int tr = -1, tc = -1;

    void explore(Navigator &nav, int r, int c) {
        if (nav.isTarget()) { tr = r; tc = c; }
        for (int i = 0; i < 4; ++i) {
            int nr = r + DR[i], nc = c + DC[i];
            if (cost[nr][nc] != -1) continue;
            if (nav.canMove(DIR[i])) {
                cost[nr][nc] = nav.move(DIR[i]);
                explore(nav, nr, nc);
                nav.move(BACK[i]);              // backtrack
            }
        }
    }

 public:
    int findMinCost(Navigator &nav) {
        cost[OFF][OFF] = 0;
        explore(nav, OFF, OFF);
        if (tr == -1) return -1;

        vector<vector<int>> dist(SIZE, vector<int>(SIZE, INT_MAX));
        dist[OFF][OFF] = 0;
        priority_queue<array<int, 3>, vector<array<int, 3>>, greater<>> pq;
        pq.push({0, OFF, OFF});
        while (!pq.empty()) {
            auto [d, r, c] = pq.top(); pq.pop();
            if (r == tr && c == tc) return d;
            if (d > dist[r][c]) continue;
            for (int i = 0; i < 4; ++i) {
                int nr = r + DR[i], nc = c + DC[i];
                if (cost[nr][nc] < 0) continue;
                int nd = d + cost[nr][nc];
                if (nd < dist[nr][nc]) {
                    dist[nr][nc] = nd;
                    pq.push({nd, nr, nc});
                }
            }
        }
        return -1;
    }
};
C
C
#include <stdbool.h>
#include <limits.h>

/* Provided interactive interface:
 *   nav->canMove(nav, d)  -> bool
 *   nav->move(nav, d)     -> int   (entry cost of the entered cell, or -1 if the move is ignored)
 *   nav->isTarget(nav)    -> bool
 */
struct Navigator {
    bool (*canMove)(struct Navigator *, char);
    int  (*move)(struct Navigator *, char);
    bool (*isTarget)(struct Navigator *);
};

#define SIZE 210
#define OFF  105

static int gCost[SIZE][SIZE];      /* -1 = undiscovered/blocked, >=0 = entry cost */
static int gTr, gTc;               /* discovered target coordinates (offset) */

static const int  DR[4]   = {-1, 1, 0, 0};
static const int  DC[4]   = {0, 0, -1, 1};
static const char DIR[4]  = {'U', 'D', 'L', 'R'};
static const char BACK[4] = {'D', 'U', 'R', 'L'};

static void explore(struct Navigator *nav, int r, int c) {
    if (nav->isTarget(nav)) { gTr = r; gTc = c; }
    for (int i = 0; i < 4; ++i) {
        int nr = r + DR[i], nc = c + DC[i];
        if (gCost[nr][nc] != -1) continue;              /* already discovered */
        if (nav->canMove(nav, DIR[i])) {
            gCost[nr][nc] = nav->move(nav, DIR[i]);
            explore(nav, nr, nc);
            nav->move(nav, BACK[i]);                     /* backtrack to restore position */
        }
    }
}

/* binary min-heap of (dist, r, c) */
struct Node { int d, r, c; };
static struct Node heap[SIZE * SIZE];
static int heapSize;

static void heapPush(int d, int r, int c) {
    int i = heapSize++;
    heap[i].d = d; heap[i].r = r; heap[i].c = c;
    while (i > 0) {
        int p = (i - 1) / 2;
        if (heap[p].d <= heap[i].d) break;
        struct Node t = heap[p]; heap[p] = heap[i]; heap[i] = t;
        i = p;
    }
}

static struct Node heapPop(void) {
    struct Node top = heap[0];
    heap[0] = heap[--heapSize];
    int i = 0;
    for (;;) {
        int l = 2 * i + 1, rr = 2 * i + 2, s = i;
        if (l  < heapSize && heap[l].d  < heap[s].d) s = l;
        if (rr < heapSize && heap[rr].d < heap[s].d) s = rr;
        if (s == i) break;
        struct Node t = heap[s]; heap[s] = heap[i]; heap[i] = t;
        i = s;
    }
    return top;
}

int findMinCost(struct Navigator *nav) {
    for (int i = 0; i < SIZE; ++i)
        for (int j = 0; j < SIZE; ++j)
            gCost[i][j] = -1;
    gTr = gTc = -1;
    gCost[OFF][OFF] = 0;
    explore(nav, OFF, OFF);
    if (gTr == -1) return -1;

    static int dist[SIZE][SIZE];
    for (int i = 0; i < SIZE; ++i)
        for (int j = 0; j < SIZE; ++j)
            dist[i][j] = INT_MAX;
    dist[OFF][OFF] = 0;
    heapSize = 0;
    heapPush(0, OFF, OFF);
    while (heapSize > 0) {
        struct Node cur = heapPop();
        if (cur.r == gTr && cur.c == gTc) return cur.d;
        if (cur.d > dist[cur.r][cur.c]) continue;
        for (int i = 0; i < 4; ++i) {
            int nr = cur.r + DR[i], nc = cur.c + DC[i];
            if (gCost[nr][nc] < 0) continue;             /* undiscovered / blocked */
            int nd = cur.d + gCost[nr][nc];
            if (nd < dist[nr][nc]) {
                dist[nr][nc] = nd;
                heapPush(nd, nr, nc);
            }
        }
    }
    return -1;
}

#### Follow-up
questions
Why is Dijkstra necessary here rather than a plain BFS, and where exactly would BFS give a wrong answer?
How would you minimize the number of move/canMove calls during exploration if each interactive call were expensive?
What breaks if entry costs could be negative, and which algorithm would you switch to?
Suppose the robot could not backtrack (no reverse move) — how would the exploration strategy change?

