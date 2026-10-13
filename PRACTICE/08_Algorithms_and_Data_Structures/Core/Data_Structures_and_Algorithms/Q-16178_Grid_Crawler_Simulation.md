---
type: "practice"
difficulty: "Other"
---

# Grid Crawler Simulation

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Queue](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Queue.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Design a simulation of a single crawler that moves around a rectangular board and grows as it consumes treats.

The board has rows rows and cols columns. Cells are addressed by (r, c) with 0 <= r < rows and 0 <= c < cols. The crawler is a chain of one or more occupied cells. It starts as a single cell at the top-left corner (0, 0), so its initial length is 1.

You are given a list treats, where treats[i] = [ri, ci] is the cell holding the i-th treat. Treats become active one at a time: the next treat is not placed on the board until the current one has been consumed. It is guaranteed that whenever a treat becomes active it is placed on a cell that is not currently occupied by the crawler. The score starts at 0; each time the crawler's head lands on the active treat, both the score and the crawler's length increase by 1, and the following treat (if any) becomes active.

Implement a class with the following operations:

GridCrawler(int cols, int rows, int[][] treats) — initialize the board with cols columns and rows rows, and record the ordered list of treat positions. The crawler begins as one cell at (0, 0).
int step(String heading) — advance the crawler one cell in the given heading, where heading is one of "U" (up, row -1), "D" (down, row +1), "L" (left, column -1), or "R" (right, column +1). The head moves first and the rest of the body follows it. Return the current score after the move, or -1 if the move ends the game.

A move ends the game (return -1) when either:

the head would leave the board (cross any wall), or
the head would move into a cell currently occupied by the crawler's own body after the tail has advanced. Because the tail cell is vacated on a non-growing move, the head is allowed to move into the exact cell the tail is leaving; a chain of length L cannot collide with the position its own tail just abandoned.

When the head lands on the active treat, the crawler grows (the tail does not advance that turn), so no self-collision check is needed for that move (the treat is guaranteed not to sit on the body).

Input/Output: each step call takes a one-character heading and returns an integer — the score after the move, or -1 on game over. Once the game is over, no further meaningful moves are made.

#### Cons
traints:

1 <= cols, rows <= 10^4
0 <= treats.length <= 50
treats[i].length == 2
0 <= ri < rows and 0 <= ci < cols
heading has length 1 and is one of "U", "D", "L", "R".
At most 10^4 calls are made to step.

#### Example
s & test cases
TEXT
Input: GridCrawler(cols=4, rows=3, treats=[[0,2],[2,3]]); step("R"),step("R"),step("D"),step("D"),step("R"),step("D") -> Output: 0,1,1,1,2,-1
Why: normal moves keep the score, the two treats at (0,2) and (2,3) raise it to 2, and the final downward move crosses the bottom wall so the game ends.

Input: GridCrawler(cols=3, rows=3, treats=[[1,0],[2,0],[2,1]]); step("D"),step("D"),step("R"),step("U"),step("D") -> Output: 1,2,3,3,-1
Why: three treats grow the chain to length 4, then the last move drives the head into a body cell that is NOT the vacating tail, ending the game.

Input: GridCrawler(cols=2, rows=2, treats=[[0,1]]); step("R"),step("D"),step("U") -> Output: 1,1,1
Why: after eating one treat (length 2), the final upward move enters the exact cell the tail is vacating, which is allowed, so it returns the score instead of -1.

Input: GridCrawler(cols=2, rows=2, treats=[]); step("U") -> Output: -1
Why: from the top-left corner an upward move leaves the board immediately; with no treats the chain never grows.

Input: GridCrawler(cols=3, rows=1, treats=[]); step("R"),step("R"),step("R") -> Output: 0,0,-1
Why: on a single-row board the crawler slides right twice, then the third move crosses the right wall.

#### Approach

Simulate the crawler with two synchronized structures: a deque of cell ids representing the body (front = head, back = tail) and a hash set of the same ids for O(1) occupancy tests. Encode each cell (r, c) as a single integer r * cols + c. On each step, compute the new head from the current head and the heading; if it leaves the board, the game is over. If the new head equals the active treat, grow: push the new head and keep the tail, bump the score, and advance the treat index (no collision check needed because treats never spawn on the body). Otherwise it is a normal move: check whether the new head cell is occupied while explicitly ignoring the current tail cell — because on a normal move the tail advances and frees that cell, so the head is allowed to enter it. If it is occupied by any other body cell, the game ends; otherwise remove the tail from both structures and push the new head. The core pattern is a deque-plus-hash-set state machine (queue simulation with O(1) membership).

Complexity

Time: O(1) per step — a constant amount of work (head arithmetic, one hash-set membership test, one insert, one erase, and one deque push and pop). Space: O(L) where L is the current crawler length, bounded by treats.length + 1 <= 51, for the deque of body cells plus the hash set of occupied cells.

Python
PYTHON
from collections import deque
from typing import List

class GridCrawler:
    def __init__(self, cols: int, rows: int, treats: List[List[int]]):
        self.cols = cols
        self.rows = rows
        self.treats = treats
        self.score = 0
        self.k = 0                       # index of the next pending treat
        start = self._cell_id(0, 0)
        self.occupied = {start}
        self.body = deque([start])       # left = head, right = tail

    def step(self, heading: str) -> int:
        r, c = divmod(self.body[0], self.cols)

        if heading == "U":
            r -= 1
            if r < 0:
                return -1
        elif heading == "D":
            r += 1
            if r == self.rows:
                return -1
        elif heading == "L":
            c -= 1
            if c < 0:
                return -1
        elif heading == "R":
            c += 1
            if c == self.cols:
                return -1

        head = self._cell_id(r, c)

        # Eat the active treat: grow (keep the tail).
        if self.k < len(self.treats) and r == self.treats[self.k][0] and c == self.treats[self.k][1]:
            self.occupied.add(head)
            self.body.appendleft(head)
            self.k += 1
            self.score += 1
            return self.score

        # Normal move: body collision ends the game, but the vacating tail cell is allowed.
        if head != self.body[-1] and head in self.occupied:
            return -1

        self.occupied.discard(self.body[-1])
        self.body.pop()
        self.occupied.add(head)
        self.body.appendleft(head)
        return self.score

    def _cell_id(self, r: int, c: int) -> int:
        return r * self.cols + c
JavaScript
JAVASCRIPT
class GridCrawler {

#### cons
tructor(cols, rows, treats) {
        this.cols = cols;
        this.rows = rows;
        this.treats = treats;
        this.score = 0;
        this.k = 0;                        // index of the next pending treat
        this.body = [[0, 0]];              // index 0 = tail, last = head
        this.occupied = new Set(['0,0']);
        this.dr = { U: -1, D: 1, L: 0, R: 0 };
        this.dc = { U: 0, D: 0, L: -1, R: 1 };
    }

    step(heading) {

#### cons
t [hr, hc] = this.body[this.body.length - 1];

#### cons
t r = hr + this.dr[heading];

#### cons
t c = hc + this.dc[heading];

        if (r < 0 || r >= this.rows || c < 0 || c >= this.cols) return -1;

#### cons
t key = r + ',' + c;

#### cons
t eat = this.k < this.treats.length &&
                    this.treats[this.k][0] === r && this.treats[this.k][1] === c;

        // Eat the active treat: grow (keep the tail).
        if (eat) {
            this.body.push([r, c]);
            this.occupied.add(key);
            this.score++;
            this.k++;
            return this.score;
        }

        // Normal move: the head may enter the cell the tail is vacating.

#### cons
t tail = this.body[0];

#### cons
t tailKey = tail[0] + ',' + tail[1];
        if (key !== tailKey && this.occupied.has(key)) return -1;

        this.body.shift();
        this.occupied.delete(tailKey);
        this.body.push([r, c]);
        this.occupied.add(key);
        return this.score;
    }
}
Java
JAVA
import java.util.*;

class GridCrawler {
    private int cols, rows, score = 0, k = 0;
    private int[][] treats;
    private Set<Integer> occupied = new HashSet<>();
    private Deque<Integer> body = new ArrayDeque<>();   // first = head, last = tail

    public GridCrawler(int cols, int rows, int[][] treats) {
        this.cols = cols;
        this.rows = rows;
        this.treats = treats;
        occupied.add(cellId(0, 0));
        body.offerFirst(cellId(0, 0));
    }

    public int step(String heading) {
        int r = body.peekFirst() / cols;
        int c = body.peekFirst() % cols;

        if (heading.equals("U") && --r < 0)      return -1;
        if (heading.equals("D") && ++r == rows)  return -1;
        if (heading.equals("L") && --c < 0)      return -1;
        if (heading.equals("R") && ++c == cols)  return -1;

        final int head = cellId(r, c);

        // Eat the active treat: grow (keep the tail).
        if (k < treats.length && r == treats[k][0] && c == treats[k][1]) {
            occupied.add(head);
            body.offerFirst(head);
            ++k;
            return ++score;
        }

        // Normal move: body collision ends the game, but the vacating tail cell is allowed.
        if (head != body.peekLast() && occupied.contains(head)) return -1;

        occupied.remove(body.peekLast());
        body.pollLast();
        occupied.add(head);
        body.offerFirst(head);
        return score;
    }

    private int cellId(int r, int c) { return r * cols + c; }
}
C++
CPP
#include <deque>
#include <string>
#include <unordered_set>
#include <vector>
using namespace std;

class GridCrawler {
public:
    GridCrawler(int cols, int rows, vector<vector<int>>& treats)
        : cols(cols), rows(rows), treats(treats) {
        int start = cellId(0, 0);
        occupied.insert(start);
        body.push_back(start);            // front = head, back = tail
    }

    int step(string heading) {
        int r = body.front() / cols;
        int c = body.front() % cols;

        if (heading == "U" && --r < 0)        return -1;
        if (heading == "D" && ++r == rows)    return -1;
        if (heading == "L" && --c < 0)        return -1;
        if (heading == "R" && ++c == cols)    return -1;

#### cons
t int head = cellId(r, c);

        // Eat the active treat: grow (keep the tail).
        if (k < (int)treats.size() && r == treats[k][0] && c == treats[k][1]) {
            occupied.insert(head);
            body.push_front(head);
            ++k;
            return ++score;
        }

        // Normal move: body collision ends the game, but entering the cell the
        // tail is vacating is allowed.
        if (head != body.back() && occupied.count(head)) return -1;

        occupied.erase(body.back());
        body.pop_back();
        occupied.insert(head);
        body.push_front(head);
        return score;
    }

private:
    int cols, rows, score = 0, k = 0;
    vector<vector<int>> treats;
    unordered_set<int> occupied;
    deque<int> body;

    int cellId(int r, int c) { return r * cols + c; }
};
C
C
#include <stdlib.h>
#include <string.h>

#define HS_BUCKETS 1024

typedef struct HNode { int key; struct HNode* next; } HNode;   /* chained hash set */
typedef struct DNode { int id; struct DNode* prev; struct DNode* next; } DNode; /* body deque */

typedef struct {
    int cols, rows;
    int** treats;
    int treatCount;
    int score;
    int k;                       /* index of the next pending treat */
    HNode* buckets[HS_BUCKETS];  /* occupied cells, O(1) membership */
    DNode* head;                 /* front = crawler head */
    DNode* tail;                 /* back  = crawler tail */
} GridCrawler;

static int cellId(GridCrawler* g, int r, int c) { return r * g->cols + c; }

static void hsInsert(GridCrawler* g, int key) {
    int b = key % HS_BUCKETS;
    HNode* n = (HNode*)malloc(sizeof(HNode));
    n->key = key; n->next = g->buckets[b]; g->buckets[b] = n;
}
static int hsContains(GridCrawler* g, int key) {
    for (HNode* n = g->buckets[key % HS_BUCKETS]; n; n = n->next)
        if (n->key == key) return 1;
    return 0;
}
static void hsErase(GridCrawler* g, int key) {
    HNode** pp = &g->buckets[key % HS_BUCKETS];
    while (*pp) {
        if ((*pp)->key == key) { HNode* d = *pp; *pp = d->next; free(d); return; }
        pp = &(*pp)->next;
    }
}
static void pushHead(GridCrawler* g, int id) {
    DNode* n = (DNode*)malloc(sizeof(DNode));
    n->id = id; n->prev = NULL; n->next = g->head;
    if (g->head) g->head->prev = n;
    g->head = n;
    if (!g->tail) g->tail = n;
}
static void popTail(GridCrawler* g) {
    DNode* old = g->tail;
    g->tail = old->prev;
    if (g->tail) g->tail->next = NULL; else g->head = NULL;
    free(old);
}

GridCrawler* gridCrawlerCreate(int cols, int rows, int** treats, int treatCount, int* treatColSize) {
    GridCrawler* g = (GridCrawler*)calloc(1, sizeof(GridCrawler));
    g->cols = cols; g->rows = rows;
    g->treats = treats; g->treatCount = treatCount;
    pushHead(g, 0);          /* start cell (0,0) */
    hsInsert(g, 0);
    return g;
}

int gridCrawlerStep(GridCrawler* g, char* heading) {
    int r = g->head->id / g->cols;
    int c = g->head->id % g->cols;
    switch (heading[0]) {
        case 'U': if (--r < 0)          return -1; break;
        case 'D': if (++r == g->rows)   return -1; break;
        case 'L': if (--c < 0)          return -1; break;
        case 'R': if (++c == g->cols)   return -1; break;
    }
    int nid = cellId(g, r, c);

    /* Eat the active treat: grow, keep the tail. */
    if (g->k < g->treatCount && r == g->treats[g->k][0] && c == g->treats[g->k][1]) {
        pushHead(g, nid);
        hsInsert(g, nid);
        g->k++;
        return ++g->score;
    }

    /* Normal move: colliding with the body ends the game, but the cell the
       tail is about to vacate is allowed. */
    if (nid != g->tail->id && hsContains(g, nid)) return -1;

    hsErase(g, g->tail->id);
    popTail(g);
    pushHead(g, nid);
    hsInsert(g, nid);
    return g->score;
}

void gridCrawlerFree(GridCrawler* g) {
    for (DNode* d = g->head; d; ) { DNode* nx = d->next; free(d); d = nx; }
    for (int i = 0; i < HS_BUCKETS; i++)
        for (HNode* n = g->buckets[i]; n; ) { HNode* nx = n->next; free(n); n = nx; }
    free(g);
}

#### Follow-up
questions
Support wrap-around walls (a torus board) so leaving one edge re-enters from the opposite edge.
Add static obstacle cells that also end the game on contact.
Return the full list of body coordinates after each step (for rendering), still in O(L) per query.
Handle very large boards efficiently — argue why the hash-set/deque approach stays O(1) per move regardless of rows and cols.

