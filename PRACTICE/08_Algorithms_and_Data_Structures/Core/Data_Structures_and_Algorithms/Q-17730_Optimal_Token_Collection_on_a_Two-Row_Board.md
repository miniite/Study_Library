---
type: "practice"
difficulty: "Other"
---

# Optimal Token Collection on a Two-Row Board

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Prefix Sum](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Prefix_Sum.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given a 2-row grid board of size 2 x n. The value board[r][c] is the number of tokens sitting on the cell at row r, column c.

Two travellers play a game on this grid. Both start on the top-left cell (0, 0) and must reach the bottom-right cell (1, n - 1). From any cell a traveller may step only to the right, (r, c) -> (r, c + 1), or downward, (r, c) -> (r + 1, c). Because the grid has exactly two rows, every valid trip walks along the top row for a while, steps down exactly once, and then walks along the bottom row to the end.

The first traveller moves first, from (0, 0) to (1, n - 1), and picks up every token on the cells of its path. Each cell the first traveller visits then has its token count set to 0. Afterwards the second traveller makes the same kind of trip from (0, 0) to (1, n - 1) and collects whatever tokens remain on the cells of its own path. The two paths are allowed to overlap.

The first traveller wants to minimise the number of tokens the second traveller ends up with. The second traveller wants to maximise its own haul. Assuming both play optimally, return the number of tokens collected by the second traveller.

#### Cons
traints:

board.length == 2
n == board[0].length == board[1].length
1 <= n <= 5 * 10^4
1 <= board[r][c] <= 10^5

#### Example
s & test cases
TEXT
Input: board = [[1,3,2],[4,2,1]] -> Output: 4
Why: The first traveller steps down at column 1, clearing the top row up to column 1 and the bottom row from column 1 on. What remains is the top-right cell (2) and the bottom-left cell (4); the second traveller takes the larger block, 4.

Input: board = [[3,1,4,2],[2,5,1,3]] -> Output: 6
Why: The first traveller descends at column 1. The leftover top-right block {4,2} sums to 6 and the bottom-left block {2} sums to 2; the second traveller grabs the bigger one, 6.

Input: board = [[2,2,1],[6,4,3]] -> Output: 3
Why: The first traveller drops down immediately at column 0 and sweeps the whole bottom row, zeroing it. Only the top-right block {2,1} = 3 is left, so the second traveller collects 3 (a normal case where descending early is best).

Input: board = [[7],[9]] -> Output: 0
Why: Edge case n = 1. The first traveller's path covers both cells (0,0) and (1,0), zeroing everything, so nothing remains for the second traveller.

Input: board = [[5,5,5],[5,5,5]] -> Output: 5
Why: A symmetric board with duplicate values; whichever column the first traveller uses, the two leftover blocks are balanced and the minimised maximum is 5.

#### Approach

On a 2-row grid the first traveller's entire route is determined by the single column p at which it steps down: it zeroes board[0][0..p] and board[1][p..n-1]. After that, tokens survive only in two disjoint blocks — the top-right run board[0][p+1..n-1] and the bottom-left run board[1][0..p-1]. The second traveller can fully claim exactly one of these blocks (run all the way across the top then drop at the very end to take the top-right, or drop immediately then run across the bottom to take the bottom-left), so it collects max(topRight, bottomLeft). The first traveller therefore picks the column p that minimises this maximum. Sweep p from left to right while maintaining topRight (subtract the element that leaves the right block) and bottomLeft (add the element that enters the left block); track the minimum of max(topRight, bottomLeft). This is a prefix/suffix-sum minimax over a single dimension. Use 64-bit accumulators, since a block sum can reach about 5 * 10^4 * 10^5 = 5 * 10^9.

Complexity

Time: O(n) — one pass builds the initial top-right sum and a second pass sweeps every descent column, updating both running sums in O(1). Space: O(1) beyond the input, since only two accumulators and a running minimum are kept.

Python
PYTHON
def second_traveller_score(board):
    n = len(board[0])
    top_right = sum(board[0][1:])   # tokens strictly right of the descent column, top row
    bottom_left = 0                 # tokens strictly left of the descent column, bottom row
    best = float('inf')
    for p in range(n):
        best = min(best, max(top_right, bottom_left))
        if p + 1 < n:
            top_right -= board[0][p + 1]
        bottom_left += board[1][p]
    return best
JavaScript
JAVASCRIPT
function secondTravellerScore(board) {

#### cons
t n = board[0].length;
    let topRight = 0, bottomLeft = 0;   // leftover blocks around the descent column
    for (let c = 1; c < n; c++) topRight += board[0][c];

    let best = Infinity;
    for (let p = 0; p < n; p++) {
        best = Math.min(best, Math.max(topRight, bottomLeft));
        if (p + 1 < n) topRight -= board[0][p + 1];
        bottomLeft += board[1][p];
    }
    return best;
}
Java
JAVA
class Solution {
    public long secondTravellerScore(int[][] board) {
        int n = board[0].length;
        long topRight = 0, bottomLeft = 0;   // leftover blocks around the descent column
        for (int c = 1; c < n; c++) topRight += board[0][c];

        long best = Long.MAX_VALUE;
        for (int p = 0; p < n; p++) {
            best = Math.min(best, Math.max(topRight, bottomLeft));
            if (p + 1 < n) topRight -= board[0][p + 1];
            bottomLeft += board[1][p];
        }
        return best;
    }
}
C++
CPP
#include <bits/stdc++.h>
using namespace std;

long long secondTravellerScore(const vector<vector<int>>& board) {
    int n = board[0].size();
    long long topRight = 0, bottomLeft = 0;   // leftover blocks around the descent column
    for (int c = 1; c < n; c++) topRight += board[0][c];

    long long best = LLONG_MAX;
    for (int p = 0; p < n; p++) {
        best = min(best, max(topRight, bottomLeft));
        if (p + 1 < n) topRight -= board[0][p + 1];
        bottomLeft += board[1][p];
    }
    return best;
}
C
C
#include <stdio.h>
#include <stdlib.h>
#include <limits.h>

/*
 * board is a 2 x n grid of non-negative token counts.
 * Returns the number of tokens the second traveller collects
 * when both travellers play optimally.
 */
long long secondTravellerScore(int** board, int n) {
    long long topRight = 0;    /* sum of board[0][c] for c in (p, n-1] */
    long long bottomLeft = 0;  /* sum of board[1][c] for c in [0, p)   */
    for (int c = 1; c < n; c++) topRight += board[0][c];

    long long best = LLONG_MAX;
    for (int p = 0; p < n; p++) {
        long long second = topRight > bottomLeft ? topRight : bottomLeft;
        if (second < best) best = second;
        if (p + 1 < n) topRight -= board[0][p + 1];
        bottomLeft += board[1][p];
    }
    return best;
}

int main(void) {
    int r0[] = {3, 1, 4, 2};
    int r1[] = {2, 5, 1, 3};
    int* board[2] = { r0, r1 };
    printf("%lld\n", secondTravellerScore(board, 4)); /* 6 */
    return 0;
}

#### Follow-up
questions
What changes if the grid has more than two rows? (The single-descent-column trick breaks; you need full DP over paths.)
What if both travellers cooperate to maximise the combined total instead of competing?
Generalise to k travellers moving in sequence, each zeroing its path.
How do you guard against integer overflow, and what are the exact bounds on a block sum?

