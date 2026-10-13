---
type: "practice"
difficulty: "Other"
---

# Largest Self-Spacing Arrangement of Numbered Tiles

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Backtracking](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Backtracking.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given a positive integer m. Build a row of 2 * m - 1 slots and fill every slot with a number from the range [1, m] so that all of the following hold:

The number 1 appears exactly once in the row.
Every number k with 2 <= k <= m appears exactly twice in the row.
For every number k with 2 <= k <= m, the two slots holding k are separated by a gap of exactly k. Formally, if k sits at slot indices i and j (0-indexed), then |i - j| = k.

Among all rows that satisfy these rules, return the one that is lexicographically largest. A row a is lexicographically larger than a row b of the same length if, at the first index where they differ, a holds a strictly greater number than b.

It is guaranteed that at least one valid row always exists for the given m.

Input
m: a positive integer.
Output
An integer array of length 2 * m - 1 describing the lexicographically largest valid row.

#### Cons
traints
1 <= m <= 20

#### Example
s & test cases
TEXT
Input: m = 3 -> Output: [3, 1, 2, 3, 2]
Why: Length 5. The 3's are 3 apart (indices 0 and 3), the 2's are 2 apart (indices 2 and 4), and 1 appears once. `[2,3,2,1,3]` is also valid but starts with 2, so it is smaller.

Input: m = 5 -> Output: [5, 3, 1, 4, 3, 5, 2, 4, 2]
Why: Length 9. Each of 2..5 appears twice at the required spacing (5's at 0 & 5, 4's at 3 & 7, 3's at 1 & 4, 2's at 6 & 8) and 1 appears once; greedily choosing the largest number at each earliest empty slot yields this maximum row.

Input: m = 1 -> Output: [1]
Why: The smallest case. Length `2*1-1 = 1`, and the single slot must hold the number 1.

Input: m = 2 -> Output: [2, 1, 2]
Why: Length 3. The two 2's must be exactly 2 apart (indices 0 and 2), leaving index 1 for the single 1.

Input: m = 4 -> Output: [4, 2, 3, 2, 4, 3, 1]
Why: Length 7. 4's are 4 apart (0 & 4), 3's are 3 apart (2 & 5), 2's are 2 apart (1 & 3), and 1 fills the last free slot; this is the lexicographically greatest feasible arrangement.

#### Approach

Backtracking with a greedy largest-first ordering. Walk the row from the leftmost empty slot. To make the result lexicographically largest, always try to place the largest still-unused number at the current slot first. Placing a number k >= 2 also pins its partner k slots to the right, so it is only allowed when both slot idx and slot idx + k are empty and in range; the number 1 occupies just the current slot. After each tentative placement, recurse to the next slot; if the recursion fails, undo the placement (reset both slots and the used flag) and try the next smaller number. Because we always attempt the greatest feasible value first, the first complete arrangement the search reaches is automatically the lexicographically largest, so we can stop immediately. The core patterns are constraint-satisfaction backtracking and greedy ordering for lexicographic optimization.

Complexity

Time: exponential in the worst case, roughly O(m!) branching bounded by the constraint checks, because it is a backtracking search over placements; with m <= 20 and strong pruning (fixed distance for each value plus greedy largest-first ordering that finds the answer quickly) it runs comfortably. Space: O(m) for the arrangement and the used array plus O(m) recursion depth. One-line justification: each slot tries at most m candidate values, but placing a value pins a second slot too, so feasible branches collapse fast under the exact-distance constraint.

Python
PYTHON
from typing import List

def largest_arrangement(m: int) -> List[int]:
    L = 2 * m - 1
    row = [0] * L
    used = [False] * (m + 1)

    def build(idx: int) -> bool:
        if idx == L:
            return True
        if row[idx] != 0:                       # slot already filled
            return build(idx + 1)
        for num in range(m, 0, -1):             # largest first for lexicographic max
            if used[num]:
                continue
            if num == 1:
                row[idx] = 1
                used[1] = True
                if build(idx + 1):
                    return True
                row[idx] = 0
                used[1] = False
            elif idx + num < L and row[idx + num] == 0:
                row[idx] = num
                row[idx + num] = num
                used[num] = True
                if build(idx + 1):
                    return True
                row[idx] = 0
                row[idx + num] = 0
                used[num] = False
        return False

    build(0)
    return row

if __name__ == "__main__":
    print(largest_arrangement(3))   # [3, 1, 2, 3, 2]
    print(largest_arrangement(5))   # [5, 3, 1, 4, 3, 5, 2, 4, 2]
JavaScript
JAVASCRIPT
function largestArrangement(m) {

#### cons
t L = 2 * m - 1;

#### cons
t row = new Array(L).fill(0);

#### cons
t used = new Array(m + 1).fill(false);

  function build(idx) {
    if (idx === L) return true;
    if (row[idx] !== 0) return build(idx + 1);       // slot already filled
    for (let num = m; num >= 1; num--) {             // largest first for lexicographic max
      if (used[num]) continue;
      if (num === 1) {
        row[idx] = 1; used[1] = true;
        if (build(idx + 1)) return true;
        row[idx] = 0; used[1] = false;
      } else if (idx + num < L && row[idx + num] === 0) {
        row[idx] = num; row[idx + num] = num; used[num] = true;
        if (build(idx + 1)) return true;
        row[idx] = 0; row[idx + num] = 0; used[num] = false;
      }
    }
    return false;
  }

  build(0);
  return row;
}

#### cons
ole.log(largestArrangement(3));  // [3, 1, 2, 3, 2]

#### cons
ole.log(largestArrangement(5));  // [5, 3, 1, 4, 3, 5, 2, 4, 2]
Java
JAVA
import java.util.Arrays;

public class LargestArrangement {
    private int L, M;
    private int[] row;
    private boolean[] used;

    private boolean build(int idx) {
        if (idx == L) return true;
        if (row[idx] != 0) return build(idx + 1);       // slot already filled
        for (int num = M; num >= 1; num--) {            // largest first for lexicographic max
            if (used[num]) continue;
            if (num == 1) {
                row[idx] = 1; used[1] = true;
                if (build(idx + 1)) return true;
                row[idx] = 0; used[1] = false;
            } else if (idx + num < L && row[idx + num] == 0) {
                row[idx] = num; row[idx + num] = num; used[num] = true;
                if (build(idx + 1)) return true;
                row[idx] = 0; row[idx + num] = 0; used[num] = false;
            }
        }
        return false;
    }

    public int[] largestArrangement(int m) {
        M = m;
        L = 2 * m - 1;
        row = new int[L];
        used = new boolean[m + 1];
        build(0);
        return row;
    }

    public static void main(String[] args) {
        System.out.println(Arrays.toString(new LargestArrangement().largestArrangement(5)));
    }
}
C++
CPP
#include <bits/stdc++.h>
using namespace std;

class Solution {
    int L, M;
    vector<int> row;
    vector<bool> used;

    bool build(int idx) {
        if (idx == L) return true;
        if (row[idx] != 0) return build(idx + 1);      // slot already taken
        for (int num = M; num >= 1; --num) {           // try largest first
            if (used[num]) continue;
            if (num == 1) {
                row[idx] = 1; used[1] = true;
                if (build(idx + 1)) return true;
                row[idx] = 0; used[1] = false;
            } else if (idx + num < L && row[idx + num] == 0) {
                row[idx] = num; row[idx + num] = num; used[num] = true;
                if (build(idx + 1)) return true;
                row[idx] = 0; row[idx + num] = 0; used[num] = false;
            }
        }
        return false;
    }

public:
    vector<int> largestArrangement(int m) {
        M = m;
        L = 2 * m - 1;
        row.assign(L, 0);
        used.assign(m + 1, false);
        build(0);
        return row;
    }
};

int main() {

#### Solution
s;
    for (int x : s.largestArrangement(5)) cout << x << ' ';
    cout << "\n";
}
C
C
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

static int L;          // total slots = 2*m - 1
static int M;          // the value m
static int *row;       // the arrangement being built
static int *used;      // used[k] = 1 if number k is already placed

// Try to fill from slot `idx` onward. Returns 1 on success.
static int build(int idx) {
    if (idx == L) return 1;
    if (row[idx] != 0) return build(idx + 1);   // already filled, move on
    for (int num = M; num >= 1; num--) {         // largest first for lexicographic max
        if (used[num]) continue;
        if (num == 1) {
            row[idx] = 1;
            used[1] = 1;
            if (build(idx + 1)) return 1;
            row[idx] = 0;
            used[1] = 0;
        } else if (idx + num < L && row[idx + num] == 0) {
            row[idx] = num;
            row[idx + num] = num;
            used[num] = 1;
            if (build(idx + 1)) return 1;
            row[idx] = 0;
            row[idx + num] = 0;
            used[num] = 0;
        }
    }
    return 0;
}

int* largestArrangement(int m, int* returnSize) {
    M = m;
    L = 2 * m - 1;
    row = (int *)calloc(L, sizeof(int));
    used = (int *)calloc(m + 1, sizeof(int));
    build(0);
    free(used);
    *returnSize = L;
    return row;
}

int main(void) {
    int n;
    int *r = largestArrangement(5, &n);
    for (int i = 0; i < n; i++) printf("%d%c", r[i], i + 1 < n ? ' ' : '\n');
    free(r);
    return 0;
}

#### Follow-up
questions
Instead of the lexicographically largest arrangement, return the lexicographically smallest — what single change to the ordering achieves it?
Count the total number of valid arrangements for a given m rather than returning one.
How would you prune more aggressively (e.g. bitmask of used numbers, forward-checking) to speed up larger inputs?
What if some numbers were required to appear a different number of times, or the spacing rule were |i - j| = f(k) for an arbitrary function f?

