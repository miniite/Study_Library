---
type: "practice"
difficulty: "Other"
---

# Nearest Guards That Can Strike the Target on a Grid

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Matrix](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Matrix.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Cons
ider an 8 x 8 grid whose rows and columns are indexed 0 through 7. Somewhere on the grid sits a single target cell, and there are one or more guards placed on distinct cells (none of them shares the target's cell).

A guard can strike the target if it lies in the same row, the same column, or on one of the two diagonals through the target — but only if it is the first guard encountered when walking outward from the target along that straight line. In other words, along each of the eight compass directions (up, down, left, right, and the four diagonals), the closest guard blocks every guard behind it; only that closest guard can strike the target from that direction.

You are given:

guards: a list where each element is a pair [row, col] giving a guard's position.
target: a pair [row, col] giving the target's position.

Return the positions of all guards that can directly strike the target. The results may be returned in any order.

Input

guards: a list of [row, col] pairs on an 8 x 8 grid, all distinct and none equal to target.
target: a single [row, col] pair.

Output

A list of [row, col] positions of the guards able to strike the target (any order).

#### Cons
traints

1 <= guards.length < 64
Each coordinate satisfies 0 <= row, col < 8.
All guard positions are distinct, and none coincides with target.

#### Example
s & test cases
TEXT
- Input: `guards = [[4,1],[4,6],[1,4],[7,4],[2,2],[6,6],[1,1],[0,4]], target = [4,4]` -> Output: `[[4,1],[4,6],[1,4],[7,4],[2,2],[6,6]]`
  Why: one guard strikes from each of six directions; `[1,1]` sits behind `[2,2]` on the up-left diagonal and `[0,4]` sits behind `[1,4]` in the up column, so both are blocked.
- Input: `guards = [[0,0]], target = [3,3]` -> Output: `[[0,0]]`
  Why: `[0,0]` is on the up-left diagonal from `[3,3]` and is the only guard, so it strikes.
- Input: `guards = [[1,2]], target = [4,4]` -> Output: `[]`
  Why: `[1,2]` shares neither row, nor column, nor a diagonal with `[4,4]` (row diff 3, col diff 2), so no strike is possible.
- Input: `guards = [[4,0],[4,7]], target = [4,4]` -> Output: `[[4,0],[4,7]]`
  Why: same-row guards on opposite sides; each is nearest along its own direction.
- Input: `guards = [[4,3],[4,1]], target = [4,4]` -> Output: `[[4,3]]`
  Why: both are to the left in row 4, but `[4,3]` is closer and blocks `[4,1]`.
- Input: `guards = [[3,3],[5,5],[3,5],[5,3]], target = [4,4]` -> Output: `[[3,3],[5,5],[3,5],[5,3]]`
  Why: the four immediate diagonal neighbours each strike from a distinct diagonal direction.

#### Approach

Mark every guard on a fixed 8 x 8 occupancy grid (or a hash set of coordinates) so any cell can be tested in O(1). Then, from the target, walk outward one step at a time along each of the 8 directions (dr, dc) ∈ {-1,0,1}^2 \ {(0,0)}. The first occupied cell reached in a direction is the nearest guard there — record it and stop scanning that direction, since every guard further along is blocked. Directions where you run off the board without hitting a guard contribute nothing. This is a grid simulation / directional ray-cast pattern: cast one ray per direction and keep only the closest hit — the same "line of sight" technique used for visibility and attack-range problems.

Complexity

Time: O(g) — building the occupancy board scans the g guards once; the outward walk covers 8 directions of at most 7 steps each, a fixed 56-step constant. Space: O(1) beyond the output — the board is a fixed 8 x 8 array (or an O(g) hash set); the returned list holds at most 8 positions. Justification: because the grid is a constant 8 x 8, the striking-guard scan is bounded work independent of input size.

Python
PYTHON
class Solution:
    def nearestStrikingGuards(self, guards, target):
        occupied = {(r, c) for r, c in guards}
        dirs = [(-1, -1), (-1, 0), (-1, 1), (0, -1),
                (0, 1), (1, -1), (1, 0), (1, 1)]
        res = []
        for dr, dc in dirs:
            r, c = target[0] + dr, target[1] + dc
            while 0 <= r < 8 and 0 <= c < 8:
                if (r, c) in occupied:      # first guard along this direction
                    res.append([r, c])
                    break
                r += dr
                c += dc
        return res
JavaScript
JAVASCRIPT
function nearestStrikingGuards(guards, target) {

#### cons
t occupied = new Set(guards.map(([r, c]) => r * 8 + c));

#### cons
t dirs = [[-1,-1],[-1,0],[-1,1],[0,-1],[0,1],[1,-1],[1,0],[1,1]];

#### cons
t res = [];
    for (const [dr, dc] of dirs) {
        let r = target[0] + dr, c = target[1] + dc;
        while (r >= 0 && r < 8 && c >= 0 && c < 8) {
            if (occupied.has(r * 8 + c)) { res.push([r, c]); break; }
            r += dr;
            c += dc;
        }
    }
    return res;
}
Java
JAVA
class Solution {
  public List<List<Integer>> nearestStrikingGuards(int[][] guards, int[] target) {
    boolean[][] board = new boolean[8][8];
    for (int[] g : guards) board[g[0]][g[1]] = true;

    int[][] dirs = {{-1,-1},{-1,0},{-1,1},{0,-1},{0,1},{1,-1},{1,0},{1,1}};
    List<List<Integer>> res = new ArrayList<>();
    for (int[] d : dirs) {
      int r = target[0] + d[0], c = target[1] + d[1];
      while (r >= 0 && r < 8 && c >= 0 && c < 8) {
        if (board[r][c]) { res.add(Arrays.asList(r, c)); break; }
        r += d[0];
        c += d[1];
      }
    }
    return res;
  }
}
C++
CPP
class Solution {
 public:
  vector<vector<int>> nearestStrikingGuards(vector<vector<int>>& guards,
                                            vector<int>& target) {
    bool board[8][8] = {};
    for (auto& g : guards) board[g[0]][g[1]] = true;

    int dirs[8][2] = {{-1,-1},{-1,0},{-1,1},{0,-1},{0,1},{1,-1},{1,0},{1,1}};
    vector<vector<int>> res;
    for (auto& dir : dirs) {
      int r = target[0] + dir[0], c = target[1] + dir[1];
      while (r >= 0 && r < 8 && c >= 0 && c < 8) {
        if (board[r][c]) { res.push_back({r, c}); break; }
        r += dir[0];
        c += dir[1];
      }
    }
    return res;
  }
};
C
C
/**
 * Returns a newly allocated array of [row, col] pairs of striking guards.
 * *returnSize receives the count; *returnColumnSizes receives an array of 2s.
 */
int** nearestStrikingGuards(int** guards, int guardsSize, int* guardsColSize,
                            int* target, int targetSize,
                            int* returnSize, int** returnColumnSizes) {
    int board[8][8] = {0};
    for (int i = 0; i < guardsSize; ++i) board[guards[i][0]][guards[i][1]] = 1;

    int dirs[8][2] = {{-1,-1},{-1,0},{-1,1},{0,-1},{0,1},{1,-1},{1,0},{1,1}};
    int** res = (int**)malloc(sizeof(int*) * 8);
    *returnColumnSizes = (int*)malloc(sizeof(int) * 8);
    int cnt = 0;

    for (int d = 0; d < 8; ++d) {
        int r = target[0] + dirs[d][0];
        int c = target[1] + dirs[d][1];
        while (r >= 0 && r < 8 && c >= 0 && c < 8) {
            if (board[r][c]) {                       /* first guard in this direction */
                res[cnt] = (int*)malloc(sizeof(int) * 2);
                res[cnt][0] = r;
                res[cnt][1] = c;
                (*returnColumnSizes)[cnt] = 2;
                ++cnt;
                break;
            }
            r += dirs[d][0];
            c += dirs[d][1];
        }
    }
    *returnSize = cnt;
    return res;
}

#### Follow-up
questions
Generalize to an arbitrary N x N grid — what is the new time complexity of the outward walks?
What if some cells contain walls that block a strike even when no guard is between? How do you incorporate obstacles into each ray?
Instead of the nearest guard per direction, return every guard aligned with the target regardless of blocking; how does the algorithm simplify?
If guards and targets stream in with updates over time, how would you answer strike queries efficiently without rescanning all rays each time?

