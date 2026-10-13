---
type: "practice"
difficulty: "Other"
---

# Communication Links Between Adjacent Occupied Rows

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ String](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/String.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given a rectangular grid described by an array of binary strings grid. Every string has the same length, so the grid is an m x n matrix of characters. Each character is either '0', meaning the cell is empty, or '1', meaning the cell holds a signal relay.

A communication link is formed between two relays when both of the following hold:

The two relays lie in two different rows r1 and r2 with r1 < r2.
Every row i strictly between them (r1 < i < r2) contains no relays at all.

Links are independent: a link between one pair of relays neither blocks nor merges with any other link. Note that column position does not matter — a relay in any column of one occupied row links to a relay in any column of the next occupied row below it.

Return the total number of communication links in the grid.

Input

grid: an array of m equal-length binary strings, each character '0' or '1'.

Output

An integer: the total number of communication links.

#### Cons
traints

m == grid.length
n == grid[i].length
1 <= m, n <= 500
Each grid[i][j] is either '0' or '1'.

#### Example
s & test cases
TEXT
- Input: `grid = ["011001","000000","010100","001000"]` -> Output: `8`
  Why: relay counts per row are `3, 0, 2, 1`; dropping the empty row leaves consecutive occupied rows with counts `3, 2, 1`, giving `3*2 + 2*1 = 8`.
- Input: `grid = ["000","111","000"]` -> Output: `0`
  Why: only one row is occupied, so no two relays sit in different occupied rows — there is nothing to link.
- Input: `grid = ["1","1","1"]` -> Output: `2`
  Why: counts are `1, 1, 1`; adjacent occupied rows contribute `1*1 + 1*1 = 2` (single-column grid still forms vertical links).
- Input: `grid = ["11","00","00","11"]` -> Output: `4`
  Why: counts are `2, 0, 0, 2`; the two empty rows collapse away, leaving `2` and `2` as consecutive occupied rows, so `2*2 = 4`.
- Input: `grid = ["0000"]` -> Output: `0`
  Why: a single all-empty row has no relays (minimum-size edge case), so the answer is `0`.
- Input: `grid = ["10001","00000","10001"]` -> Output: `4`
  Why: counts are `2, 0, 2`; the middle empty row is skipped and the two occupied rows link fully, `2*2 = 4`.

#### Approach

The key observation is that column positions are irrelevant: any relay in an occupied row links to any relay in the next occupied row below it, and empty rows simply vanish (they never break a link, they just don't create one). So the problem collapses to the sequence of relay counts of the occupied rows only. If two consecutive occupied rows hold a and b relays, they contribute a * b links (every top relay pairs with every bottom relay). The total is therefore the sum of count(prev) * count(cur) over each pair of consecutive occupied rows. Implement it as one pass: count '1's per row, and whenever a row is non-empty, add prev * cur to the answer and update prev. This is a counting / running-product technique — no matrix of pairs is ever built.

Complexity

Time: O(m * n) — each of the m rows is scanned once to count its relays, and each scan is O(n). Space: O(1) — only two integers (the running total and the previous occupied row's count) are kept, independent of grid size. Justification: the answer depends only on the sequence of per-row relay counts, so a single left-to-right pass suffices.

Python
PYTHON
from typing import List

class Solution:
    def numberOfLinks(self, grid: List[str]) -> int:
        total = 0
        prev = 0                      # relay count of the last occupied row seen
        for row in grid:
            cur = row.count('1')
            if cur:
                total += prev * cur   # every relay above links to every relay below
                prev = cur
        return total
JavaScript
JAVASCRIPT
function numberOfLinks(grid) {
    let total = 0;
    let prev = 0;
    for (const row of grid) {
        let cur = 0;
        for (const ch of row) if (ch === '1') cur++;
        if (cur > 0) {
            total += prev * cur;
            prev = cur;
        }
    }
    return total;
}
Java
JAVA
class Solution {
  public int numberOfLinks(String[] grid) {
    long total = 0;
    int prev = 0;
    for (String row : grid) {
      int cur = 0;
      for (int j = 0; j < row.length(); ++j)
        if (row.charAt(j) == '1') ++cur;
      if (cur > 0) {
        total += (long) prev * cur;
        prev = cur;
      }
    }
    return (int) total;
  }
}
C++
CPP
class Solution {
 public:
  int numberOfLinks(vector<string>& grid) {
    long long total = 0;
    int prev = 0;
    for (const string& row : grid) {
      int cur = (int)count(row.begin(), row.end(), '1');
      if (cur > 0) {
        total += (long long)prev * cur;
        prev = cur;
      }
    }
    return (int)total;
  }
};
C
C
int numberOfLinks(char** grid, int gridSize) {
    long long total = 0;   /* running total of links */
    int prev = 0;          /* relay count of the most recent occupied row (0 = none yet) */
    for (int i = 0; i < gridSize; ++i) {
        int cur = 0;
        for (const char* p = grid[i]; *p; ++p)
            if (*p == '1') ++cur;
        if (cur > 0) {
            total += (long long)prev * cur;  /* link each relay above with each below */
            prev = cur;
        }
    }
    return (int)total;
}

#### Follow-up
questions
What if links were also allowed between two relays in the same row? How would the count change?
Suppose the grid is streamed one row at a time and cannot be stored; can you still answer in O(1) memory? (Yes — the running-product pass already does this.)
Generalize to links that may skip up to k consecutive empty rows but no more; how do you adapt the pass?
How would you also return the specific pairs of columns that form links, not just the total?

