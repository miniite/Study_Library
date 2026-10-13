---
type: "practice"
difficulty: "Other"
---

# Feasibility of a Layered Rectangle Stamping Machine

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Topological Sort](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Topological_Sort.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

A stamping machine paints on a grid with rows rows and cols columns, initially blank. In a single operation the machine picks any axis-aligned rectangular region of cells and floods that entire rectangle with one color, painting over whatever those cells previously held. There is one restriction: each color may be used in at most one operation — once a color has been stamped, it can never be used again.

You are given a fully painted target grid grid, where grid[r][c] is the color id at row r, column c. Starting from a blank grid, decide whether some sequence of stamping operations can reproduce exactly this target grid.

Return true if the target is achievable, and false otherwise.

#### Cons
traints:

rows == grid.length
cols == grid[0].length
1 <= rows, cols <= 60
1 <= grid[r][c] <= 60
Every cell of the target grid is painted (no blank cells remain).

#### Example
s & test cases
TEXT
Input: grid = [[3,3,3],[3,7,3],[3,3,3]] -> Output: true
Why: stamp color 3 over the whole 3x3, then stamp color 7 on the center cell; the concentric layering is legal.

Input: grid = [[2,2,2,2],[2,2,5,5],[2,2,5,6],[8,8,2,6]] -> Output: true
Why: colors 6 and 8 sit in solid corners, color 5 above them, and color 2 underneath everything; peeling from the last stamp inward succeeds.

Input: grid = [[4,9,4],[9,4,9],[4,9,4]] -> Output: false
Why: colors 4 and 9 interleave in a checkerboard; each color's bounding box covers the whole grid and contains the other, so no color can be the final stamp.

Input: grid = [[6,6],[6,6]] -> Output: true
Why: a single color filling the grid needs exactly one stamp.

Input: grid = [[1,2],[2,1]] -> Output: false
Why: a 2x2 two-color checkerboard; both bounding boxes span the entire grid and block each other, so it is impossible.

Input: grid = [[5]] -> Output: true
Why: a single cell is produced by one stamp of that color.

#### Approach

Think of the stamps in reverse. The very last stamp of any color must appear in the final grid as a solid rectangle equal to that color's bounding box, because nothing painted after it. So a color can be "the most recent remaining stamp" only if every cell inside its bounding box is either that same color or a cell that was later overwritten (which we model as already cleared). Compute each color's bounding box (min/max row and column). Then repeatedly scan the colors: whenever a color's box contains only its own color or cleared cells, it is a legal final stamp — clear its entire box (mark those cells transparent, since earlier stamps may hide underneath) and drop the color. Repeat in passes; if a full pass clears nothing while colors remain, the grid is impossible and we return false; if every color is cleared, it is achievable. This is a reverse-simulation / topological peeling over the "painted-before" dependencies between overlapping rectangles — the checkerboard cases fail because the dependencies form a cycle.

Complexity

Let C be the number of distinct colors (at most 60) and A = rows * cols the grid area. Each peeling pass tests every still-present color against its bounding box in O(A) worst case, so O(C * A) per pass, and at least one color is removed per successful pass, giving up to C passes: O(C^2 * A) time overall. With all dimensions bounded by 60 this is trivially fast. Space is O(A + C) for the working grid copy and the per-color bounds.

Python
PYTHON
def can_stamp(grid):
    rows, cols = len(grid), len(grid[0])
    g = [row[:] for row in grid]          # work on a copy; cleared cells become -1

    bounds = {}                           # color -> [minR, minC, maxR, maxC]
    for r in range(rows):
        for c in range(cols):
            color = g[r][c]
            if color not in bounds:
                bounds[color] = [r, c, r, c]
            else:
                b = bounds[color]
                b[0] = min(b[0], r); b[1] = min(b[1], c)
                b[2] = max(b[2], r); b[3] = max(b[3], c)

    remaining = set(bounds)
    while remaining:
        progress = False
        for color in list(remaining):
            r0, c0, r1, c1 = bounds[color]
            ok = True
            for r in range(r0, r1 + 1):
                if not ok:
                    break
                for c in range(c0, c1 + 1):
                    if g[r][c] != -1 and g[r][c] != color:
                        ok = False
                        break
            if ok:
                for r in range(r0, r1 + 1):
                    for c in range(c0, c1 + 1):
                        g[r][c] = -1
                remaining.discard(color)
                progress = True
        if not progress:
            return False
    return True

if __name__ == "__main__":
    print(can_stamp([[3, 3, 3], [3, 7, 3], [3, 3, 3]]))  # True
JavaScript
JAVASCRIPT
function canStamp(grid) {

#### cons
t rows = grid.length, cols = grid[0].length;

#### cons
t g = grid.map(row => row.slice());   // copy; cleared cells become -1

#### cons
t bounds = new Map();                  // color -> [minR, minC, maxR, maxC]
    for (let r = 0; r < rows; r++)
        for (let c = 0; c < cols; c++) {

#### cons
t color = g[r][c];
            if (!bounds.has(color)) bounds.set(color, [r, c, r, c]);
            else {

#### cons
t b = bounds.get(color);
                b[0] = Math.min(b[0], r); b[1] = Math.min(b[1], c);
                b[2] = Math.max(b[2], r); b[3] = Math.max(b[3], c);
            }
        }

    let remaining = [...bounds.keys()];
    while (remaining.length) {
        let progress = false;

#### cons
t nxt = [];
        for (const color of remaining) {

#### cons
t [r0, c0, r1, c1] = bounds.get(color);
            let ok = true;
            for (let r = r0; r <= r1 && ok; r++)
                for (let c = c0; c <= c1; c++)
                    if (g[r][c] !== -1 && g[r][c] !== color) { ok = false; break; }
            if (ok) {
                for (let r = r0; r <= r1; r++)
                    for (let c = c0; c <= c1; c++) g[r][c] = -1;
                progress = true;
            } else {
                nxt.push(color);
            }
        }
        if (!progress) return false;
        remaining = nxt;
    }
    return true;
}

#### cons
ole.log(canStamp([[3, 3, 3], [3, 7, 3], [3, 3, 3]])); // true
Java
JAVA
import java.util.*;

public class Solution {
    public boolean canStamp(int[][] grid) {   // grid is mutated in place
        int rows = grid.length, cols = grid[0].length;
        final int MAXC = 61;
        int[] minR = new int[MAXC], minC = new int[MAXC], maxR = new int[MAXC], maxC = new int[MAXC];
        boolean[] present = new boolean[MAXC];
        Arrays.fill(minR, Integer.MAX_VALUE);
        Arrays.fill(minC, Integer.MAX_VALUE);
        Arrays.fill(maxR, -1);
        Arrays.fill(maxC, -1);

        for (int r = 0; r < rows; r++)
            for (int c = 0; c < cols; c++) {
                int col = grid[r][c];
                present[col] = true;
                minR[col] = Math.min(minR[col], r);
                minC[col] = Math.min(minC[col], c);
                maxR[col] = Math.max(maxR[col], r);
                maxC[col] = Math.max(maxC[col], c);
            }

        List<Integer> remaining = new ArrayList<>();
        for (int col = 1; col < MAXC; col++)
            if (present[col]) remaining.add(col);

        while (!remaining.isEmpty()) {
            boolean progress = false;
            List<Integer> nxt = new ArrayList<>();
            for (int col : remaining) {
                boolean ok = true;
                for (int r = minR[col]; r <= maxR[col] && ok; r++)
                    for (int c = minC[col]; c <= maxC[col]; c++)
                        if (grid[r][c] != -1 && grid[r][c] != col) { ok = false; break; }
                if (ok) {
                    for (int r = minR[col]; r <= maxR[col]; r++)
                        for (int c = minC[col]; c <= maxC[col]; c++)
                            grid[r][c] = -1;
                    progress = true;
                } else {
                    nxt.add(col);
                }
            }
            if (!progress) return false;
            remaining = nxt;
        }
        return true;
    }

    public static void main(String[] args) {
        int[][] g = {{3, 3, 3}, {3, 7, 3}, {3, 3, 3}};
        System.out.println(new Solution().canStamp(g)); // true
    }
}
C++
CPP
#include <bits/stdc++.h>
using namespace std;

bool canStamp(vector<vector<int>> grid) {   // grid taken by value; we mutate the copy
    int rows = (int)grid.size(), cols = (int)grid[0].size();

#### cons
t int MAXC = 61;
    vector<int> minR(MAXC, INT_MAX), minC(MAXC, INT_MAX), maxR(MAXC, -1), maxC(MAXC, -1);
    vector<bool> present(MAXC, false);
    for (int r = 0; r < rows; r++)
        for (int c = 0; c < cols; c++) {
            int col = grid[r][c];
            present[col] = true;
            minR[col] = min(minR[col], r);
            minC[col] = min(minC[col], c);
            maxR[col] = max(maxR[col], r);
            maxC[col] = max(maxC[col], c);
        }

    vector<int> remaining;
    for (int col = 1; col < MAXC; col++)
        if (present[col]) remaining.push_back(col);

    while (!remaining.empty()) {
        bool progress = false;
        vector<int> nxt;
        for (int col : remaining) {
            bool ok = true;
            for (int r = minR[col]; r <= maxR[col] && ok; r++)
                for (int c = minC[col]; c <= maxC[col]; c++)
                    if (grid[r][c] != -1 && grid[r][c] != col) { ok = false; break; }
            if (ok) {
                for (int r = minR[col]; r <= maxR[col]; r++)
                    for (int c = minC[col]; c <= maxC[col]; c++)
                        grid[r][c] = -1;
                progress = true;
            } else {
                nxt.push_back(col);
            }
        }
        if (!progress) return false;
        remaining = move(nxt);
    }
    return true;
}

int main() {
    vector<vector<int>> g = {{3, 3, 3}, {3, 7, 3}, {3, 3, 3}};
    cout << (canStamp(g) ? "true" : "false") << "\n"; // true
    return 0;
}
C
C
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <stdbool.h>
#include <limits.h>

/* grid is mutated (cleared cells become -1). Colors are 1..60. */
bool canStamp(int **grid, int rows, int cols) {
    int minR[61], minC[61], maxR[61], maxC[61];
    bool present[61];
    for (int i = 0; i < 61; i++) {
        minR[i] = INT_MAX; minC[i] = INT_MAX; maxR[i] = -1; maxC[i] = -1;
        present[i] = false;
    }
    for (int r = 0; r < rows; r++)
        for (int c = 0; c < cols; c++) {
            int col = grid[r][c];
            present[col] = true;
            if (r < minR[col]) minR[col] = r;
            if (c < minC[col]) minC[col] = c;
            if (r > maxR[col]) maxR[col] = r;
            if (c > maxC[col]) maxC[col] = c;
        }

    int remaining[61], rn = 0;
    for (int col = 1; col <= 60; col++)
        if (present[col]) remaining[rn++] = col;

    while (rn > 0) {
        bool progress = false;
        int nxt[61], nn = 0;
        for (int k = 0; k < rn; k++) {
            int col = remaining[k];
            bool ok = true;
            for (int r = minR[col]; r <= maxR[col] && ok; r++)
                for (int c = minC[col]; c <= maxC[col]; c++)
                    if (grid[r][c] != -1 && grid[r][c] != col) { ok = false; break; }
            if (ok) {
                for (int r = minR[col]; r <= maxR[col]; r++)
                    for (int c = minC[col]; c <= maxC[col]; c++)
                        grid[r][c] = -1;
                progress = true;
            } else {
                nxt[nn++] = col;
            }
        }
        if (!progress) return false;
        memcpy(remaining, nxt, nn * sizeof(int));
        rn = nn;
    }
    return true;
}

int main(void) {
    int r0[] = {3, 3, 3}, r1[] = {3, 7, 3}, r2[] = {3, 3, 3};
    int *g[] = {r0, r1, r2};
    printf("%s\n", canStamp(g, 3, 3) ? "true" : "false"); /* true */
    return 0;
}

#### Follow-up
questions
Return an actual valid ordering of stamps (a topological order) instead of just true/false.
How does feasibility change if a color may be reused any number of times?
Build the dependency graph explicitly (color A before color B when B intrudes into A's box) and detect the cycle that proves impossibility.
What if colors are not guaranteed to form a single connected region — does the bounding-box argument still hold?

