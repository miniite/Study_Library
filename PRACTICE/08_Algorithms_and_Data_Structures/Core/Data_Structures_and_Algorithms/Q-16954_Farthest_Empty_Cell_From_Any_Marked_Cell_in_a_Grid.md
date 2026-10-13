---
type: "practice"
difficulty: "Other"
---

# Farthest Empty Cell From Any Marked Cell in a Grid

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Breadth-First Search](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Breadth-First_Search.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given a square grid board of size n x n. Every cell holds one of two values:

1 marks a filled cell.
0 marks an empty cell.

The distance between two cells (r0, c0) and (r1, c1) is their Manhattan distance, |r0 - r1| + |c0 - c1| (equivalently, the number of single steps taken while moving up, down, left, or right along the grid).

Among all empty cells, find the one whose distance to its nearest filled cell is as large as possible, and return that largest distance.

If the grid contains no filled cell at all, or no empty cell at all, then there is no valid answer, so return -1.

Input

board: an n x n matrix whose entries are each 0 or 1.

Output

A single integer: the maximum over all empty cells of the distance to the closest filled cell, or -1 when either value is missing from the grid.

#### Cons
traints

n == board.length
n == board[i].length
1 <= n <= 100
board[i][j] is either 0 or 1.

#### Example
s & test cases
TEXT
Input: board = [[0,0,0],[0,1,0],[0,0,0]] -> Output: 2
Why: A single filled cell sits in the center; each corner empty cell is 2 steps away, the maximum possible here.

Input: board = [[1,0,0,0],[0,0,0,0],[0,0,0,0],[0,0,0,0]] -> Output: 6
Why: The only filled cell is the top-left corner; the opposite corner (3,3) is |3-0|+|3-0| = 6 away.

Input: board = [[1,0,1],[0,0,0],[1,0,1]] -> Output: 2
Why: Four filled corners; the center empty cell (1,1) is 2 steps from every corner, the farthest any empty cell can be.

Input: board = [[1,1],[1,1]] -> Output: -1
Why: There is no empty cell in the grid, so no valid answer exists.

Input: board = [[0]] -> Output: -1
Why: There is no filled cell in the grid, so no valid answer exists.

Input: board = [[1,0],[0,0]] -> Output: 2
Why: The single filled cell is at (0,0); the farthest empty cell is (1,1) at distance 2.

#### Approach

This is a multi-source shortest-path problem on an unweighted grid, solved with a multi-source BFS. Instead of running a separate search from each empty cell to find its nearest filled cell (which would be expensive), we invert the process: seed the BFS queue with all filled cells at once (distance 0) and let the wavefront expand outward one ring at a time. Because BFS visits cells in nondecreasing distance order, the first time a wavefront reaches an empty cell it does so via the closest filled cell. The last level of the expansion to complete corresponds to the empty cell(s) that are hardest to reach, i.e. maximally far from every filled cell — exactly the answer. We mark visited empty cells as filled to avoid revisiting, and increment the distance counter once per completed level. The two degenerate inputs (no filled cells or no empty cells) are handled up front with a -1 return. Pattern: multi-source BFS / grid distance transform.

Complexity

Time: O(n^2). Every cell is enqueued and dequeued at most once during the multi-source breadth-first expansion, so the total work is proportional to the number of cells. Space: O(n^2). In the worst case the queue simultaneously holds a constant fraction of all cells, and the visited marking reuses the input grid.

Python
PYTHON
from collections import deque
from typing import List

def farthest_empty_cell(board: List[List[int]]) -> int:
    n = len(board)
    total = n * n
    q = deque()
    for i in range(n):
        for j in range(n):
            if board[i][j] == 1:
                q.append((i, j))

    if len(q) == 0 or len(q) == total:
        return -1

    dist = -1
    directions = ((0, 1), (0, -1), (1, 0), (-1, 0))
    while q:
        for _ in range(len(q)):
            x, y = q.popleft()
            for dx, dy in directions:
                nx, ny = x + dx, y + dy
                if 0 <= nx < n and 0 <= ny < n and board[nx][ny] == 0:
                    board[nx][ny] = 1
                    q.append((nx, ny))
        dist += 1
    return dist
JavaScript
JAVASCRIPT
function farthestEmptyCell(board) {

#### cons
t n = board.length;

#### cons
t total = n * n;
    let queue = [];
    for (let i = 0; i < n; i++) {
        for (let j = 0; j < n; j++) {
            if (board[i][j] === 1) queue.push([i, j]);
        }
    }

    if (queue.length === 0 || queue.length === total) return -1;

    let dist = -1;

#### cons
t dirs = [[0, 1], [0, -1], [1, 0], [-1, 0]];
    let head = 0;
    while (head < queue.length) {
        let levelSize = queue.length - head;
        while (levelSize-- > 0) {

#### cons
t [x, y] = queue[head++];
            for (const [dx, dy] of dirs) {

#### cons
t nx = x + dx, ny = y + dy;
                if (nx >= 0 && nx < n && ny >= 0 && ny < n && board[nx][ny] === 0) {
                    board[nx][ny] = 1;
                    queue.push([nx, ny]);
                }
            }
        }
        dist++;
    }
    return dist;
}
Java
JAVA
import java.util.ArrayDeque;
import java.util.Deque;

class Solution {
    public int farthestEmptyCell(int[][] board) {
        int n = board.length;
        Deque<int[]> q = new ArrayDeque<>();
        for (int i = 0; i < n; i++)
            for (int j = 0; j < n; j++)
                if (board[i][j] == 1) q.offer(new int[]{i, j});

        if (q.isEmpty() || q.size() == n * n) return -1;

        int dist = -1;
        int[] dx = {0, 0, 1, -1};
        int[] dy = {1, -1, 0, 0};

        while (!q.isEmpty()) {
            int levelSize = q.size();
            for (int s = 0; s < levelSize; s++) {
                int[] cell = q.poll();
                int x = cell[0], y = cell[1];
                for (int d = 0; d < 4; d++) {
                    int nx = x + dx[d], ny = y + dy[d];
                    if (nx >= 0 && nx < n && ny >= 0 && ny < n && board[nx][ny] == 0) {
                        board[nx][ny] = 1;
                        q.offer(new int[]{nx, ny});
                    }
                }
            }
            dist++;
        }
        return dist;
    }
}
C++
CPP
#include <vector>
#include <queue>
using namespace std;

class Solution {
public:
    int farthestEmptyCell(vector<vector<int>>& board) {
        int n = board.size();
        queue<pair<int,int>> q;
        for (int i = 0; i < n; i++)
            for (int j = 0; j < n; j++)
                if (board[i][j] == 1) q.push({i, j});

        if ((int)q.size() == 0 || (int)q.size() == n * n) return -1;

        int dist = -1;
        int dx[4] = {0, 0, 1, -1};
        int dy[4] = {1, -1, 0, 0};

        while (!q.empty()) {
            int levelSize = q.size();
            while (levelSize--) {
                auto [x, y] = q.front();
                q.pop();
                for (int d = 0; d < 4; d++) {
                    int nx = x + dx[d], ny = y + dy[d];
                    if (nx >= 0 && nx < n && ny >= 0 && ny < n && board[nx][ny] == 0) {
                        board[nx][ny] = 1;
                        q.push({nx, ny});
                    }
                }
            }
            dist++;
        }
        return dist;
    }
};
C
C
#include <stdlib.h>

// board is modified in place (visited empties are marked filled).
int farthestEmptyCell(int** board, int boardSize, int* boardColSize) {
    int n = boardSize;
    int total = n * n;
    int* qx = (int*)malloc(sizeof(int) * total);
    int* qy = (int*)malloc(sizeof(int) * total);
    int head = 0, tail = 0;

    for (int i = 0; i < n; i++)
        for (int j = 0; j < n; j++)
            if (board[i][j] == 1) { qx[tail] = i; qy[tail] = j; tail++; }

    if (tail == 0 || tail == total) { free(qx); free(qy); return -1; }

    int dist = -1;
    int dx[4] = {0, 0, 1, -1};
    int dy[4] = {1, -1, 0, 0};

    while (head < tail) {
        int levelSize = tail - head;
        for (int s = 0; s < levelSize; s++) {
            int x = qx[head], y = qy[head];
            head++;
            for (int d = 0; d < 4; d++) {
                int nx = x + dx[d], ny = y + dy[d];
                if (nx >= 0 && nx < n && ny >= 0 && ny < n && board[nx][ny] == 0) {
                    board[nx][ny] = 1;
                    qx[tail] = nx; qy[tail] = ny; tail++;
                }
            }
        }
        dist++;
    }

    free(qx); free(qy);
    return dist;
}

#### Follow-up
questions
What changes if the grid is a rectangle n x m rather than a square?
Instead of the distance, return the coordinates of the farthest empty cell.
What if movement is 8-directional (diagonals allowed), so distance becomes Chebyshev distance?
Can you achieve the same result with a two-pass dynamic-programming distance transform instead of BFS, and how do the tradeoffs compare?

