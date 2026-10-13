---
type: "practice"
difficulty: "Other"
---

# Best Alignment of Two Binary Stencils

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Matrix](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Matrix.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given two binary stencils, first and second, each a square grid of size n x n. Every cell holds either 0 or 1.

You may slide one stencil over the other: shift all of its 1 cells the same number of units left, right, up, and/or down (any amount, including zero). Sliding is a pure translation — no rotation and no flipping is allowed. Any 1 that is pushed past the border of the grid is discarded.

After sliding, lay the moved stencil directly on top of the other one. The alignment score is the number of grid positions that hold a 1 in both stencils simultaneously.

Return the largest alignment score achievable over all possible slides.

Input

first: an n x n grid of 0s and 1s.
second: an n x n grid of 0s and 1s.

Output

A single integer: the maximum number of overlapping 1s after the best translation.

#### Cons
traints

n == first.length == first[i].length
n == second.length == second[i].length
1 <= n <= 30
Every cell of first and second is either 0 or 1.

#### Example
s & test cases
TEXT
- Input: `first = [[1,0,0],[1,1,0],[0,0,0]], second = [[0,0,0],[0,1,0],[0,1,1]]` -> Output: `3`
  Why: sliding `first` down one row and right one column lines up all three of its `1`s with `second`'s `1`s.
- Input: `first = [[1]], second = [[1]]` -> Output: `1`
  Why: both single cells are `1`; no slide needed (minimum non-zero case).
- Input: `first = [[0]], second = [[0]]` -> Output: `0`
  Why: neither grid has a `1`, so no overlap is possible.
- Input: `first = [[1,0],[0,0]], second = [[0,0],[0,1]]` -> Output: `1`
  Why: the lone `1`s are in opposite corners; sliding one onto the other yields exactly one overlap.
- Input: `first = [[1,1],[0,0]], second = [[0,0],[1,1]]` -> Output: `2`
  Why: the horizontal pair in the top row slides down one row to sit exactly on the bottom-row pair.

#### Approach

The key observation is that the answer only depends on the relative offset (translation vector) between the two stencils, not on the absolute slide. If a 1 at position a in the first grid must land on a 1 at position b in the second grid, the required translation is the vector a - b. Any two ones that share the same translation vector can be aligned simultaneously by a single slide. So the largest alignment score is simply the most frequent translation vector across all (one-in-first, one-in-second) pairs. Collect the coordinates of the ones in both grids, tally the vector (ar - br, ac - bc) for every pair using a hash map (encoding the two components into one integer key with an offset to stay collision-free), and return the maximum bucket count. This replaces the naive "try every one of the ~O(n^2) possible shifts and rescan the whole grid" (which is also polynomial but rescans redundantly) with a direct counting argument.

Complexity

Time: O(n^4) in the worst case — up to n^2 ones in each grid, and every pair of ones contributes one translation vector; with n <= 30 this is at most ~810k pairs. Space: O(n^2) for the coordinate lists and the map of distinct translation vectors.

Python
PYTHON
from collections import defaultdict
from typing import List

class Solution:
    def bestAlignment(self, first: List[List[int]], second: List[List[int]]) -> int:
        n = len(first)
        ones_a = [(i, j) for i in range(n) for j in range(n) if first[i][j] == 1]
        ones_b = [(i, j) for i in range(n) for j in range(n) if second[i][j] == 1]

        freq = defaultdict(int)
        best = 0
        for ai, aj in ones_a:
            for bi, bj in ones_b:
                key = (ai - bi, aj - bj)  # the translation that maps b onto a
                freq[key] += 1
                if freq[key] > best:
                    best = freq[key]
        return best
JavaScript
JAVASCRIPT
/**
 * @param {number[][]} first
 * @param {number[][]} second
 * @return {number}
 */
function bestAlignment(first, second) {

#### cons
t n = first.length;

#### cons
t onesA = [];

#### cons
t onesB = [];
    for (let i = 0; i < n; i++) {
        for (let j = 0; j < n; j++) {
            if (first[i][j] === 1)  onesA.push([i, j]);
            if (second[i][j] === 1) onesB.push([i, j]);
        }
    }

#### cons
t magic = 2 * n;

#### cons
t freq = new Map();
    let best = 0;
    for (const [ai, aj] of onesA) {
        for (const [bi, bj] of onesB) {

#### cons
t key = (ai - bi) * magic + (aj - bj);

#### cons
t f = (freq.get(key) || 0) + 1;
            freq.set(key, f);
            if (f > best) best = f;
        }
    }
    return best;
}
Java
JAVA
import java.util.*;

class Solution {
    public int bestAlignment(int[][] first, int[][] second) {
        int n = first.length;
        List<int[]> onesA = new ArrayList<>();
        List<int[]> onesB = new ArrayList<>();
        for (int i = 0; i < n; i++)
            for (int j = 0; j < n; j++) {
                if (first[i][j] == 1)  onesA.add(new int[]{i, j});
                if (second[i][j] == 1) onesB.add(new int[]{i, j});
            }

        final int magic = 2 * n;
        Map<Integer, Integer> freq = new HashMap<>();
        int best = 0;
        for (int[] a : onesA)
            for (int[] b : onesB) {
                int key = (a[0] - b[0]) * magic + (a[1] - b[1]);
                int f = freq.merge(key, 1, Integer::sum);
                best = Math.max(best, f);
            }
        return best;
    }
}
C++
CPP
#include <vector>
#include <unordered_map>
#include <algorithm>
using namespace std;

class Solution {
public:
    int bestAlignment(vector<vector<int>>& first, vector<vector<int>>& second) {
        int n = first.size();
        vector<pair<int,int>> onesA, onesB;
        for (int i = 0; i < n; i++)
            for (int j = 0; j < n; j++) {
                if (first[i][j] == 1)  onesA.emplace_back(i, j);
                if (second[i][j] == 1) onesB.emplace_back(i, j);
            }

        // Count each translation vector; encode (dr, dc) into a single key.

#### cons
t int magic = 2 * n;
        unordered_map<int,int> freq;
        int best = 0;
        for (auto& a : onesA)
            for (auto& b : onesB) {
                int key = (a.first - b.first) * magic + (a.second - b.second);
                best = max(best, ++freq[key]);
            }
        return best;
    }
};
C
C
#include <stdlib.h>

int bestAlignment(int** first, int firstSize, int* firstColSize,
                  int** second, int secondSize, int* secondColSize) {
    int n = firstSize;

    /* Collect coordinates of the 1s in each grid. */
    int* ar = (int*)malloc(n * n * sizeof(int));
    int* ac = (int*)malloc(n * n * sizeof(int));
    int* br = (int*)malloc(n * n * sizeof(int));
    int* bc = (int*)malloc(n * n * sizeof(int));
    int na = 0, nb = 0;
    for (int i = 0; i < n; i++)
        for (int j = 0; j < n; j++) {
            if (first[i][j] == 1)  { ar[na] = i; ac[na] = j; na++; }
            if (second[i][j] == 1) { br[nb] = i; bc[nb] = j; nb++; }
        }

    /* Frequency of each translation vector (dr, dc), dr,dc in [-(n-1), n-1].
       Encode with offset n into a (2n-1)*(2n-1) bucket table. */
    int span = 2 * n - 1;
    int* freq = (int*)calloc(span * span, sizeof(int));
    int best = 0;
    for (int a = 0; a < na; a++)
        for (int b = 0; b < nb; b++) {
            int dr = ar[a] - br[b] + (n - 1);
            int dc = ac[a] - bc[b] + (n - 1);
            int f = ++freq[dr * span + dc];
            if (f > best) best = f;
        }

    free(ar); free(ac); free(br); free(bc); free(freq);
    return best;
}

#### Follow-up
questions
What if the grids are rectangular (m x n) rather than square — does the offset-counting idea still hold?
How would you also return the exact translation (dr, dc) that achieves the best score, not just its value?
If rotations of 90/180/270 degrees were allowed, how would you extend the search?
For very sparse grids with only a handful of 1s among millions of cells, why is the coordinate-pair approach far better than shifting bitmasks?

