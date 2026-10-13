---
type: "practice"
difficulty: "Other"
---

# Largest Consistent Set of Fully-Truthful Reviewers

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Bit Manipulation](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Bit_Manipulation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

There are n reviewers, numbered 0 to n - 1. Each reviewer is exactly one of two kinds:

honest — every remark this reviewer makes is guaranteed to be true.
unreliable — this reviewer's remarks carry no guarantee; each remark may independently be true or false.

You are given an n x n integer grid remarks, where remarks[i][j] describes what reviewer i says about reviewer j:

0 — reviewer i remarks that reviewer j is unreliable.
1 — reviewer i remarks that reviewer j is honest.
2 — reviewer i makes no remark about reviewer j.

No reviewer ever remarks about themselves, so remarks[i][i] = 2 for every i.

#### Cons
ider every possible way of labeling each reviewer as honest or unreliable. Such a labeling is consistent when every reviewer marked honest tells the complete truth: an honest reviewer never calls an honest reviewer "unreliable" and never calls an unreliable reviewer "honest". Reviewers marked unreliable impose no constraints, since their remarks may be true or false.

Return the maximum number of reviewers that can be honest across all consistent labelings.

#### Cons
traints

n == remarks.length == remarks[i].length
2 <= n <= 15
remarks[i][j] ∈ {0, 1, 2}
remarks[i][i] = 2

#### Example
s & test cases
TEXT
- Input: `remarks = [[2,1,2],[1,2,2],[2,0,2]]` -> Output: `2`
  Why: labeling reviewer 2 unreliable and reviewers 0 and 1 honest is consistent (0 and 1 vouch for each other truthfully); no labeling makes all three honest.
- Input: `remarks = [[2,0],[0,2]]` -> Output: `1`
  Why: each accuses the other of being unreliable; both cannot be honest (that would make an honest reviewer's accusation false), so at most one is honest.
- Input: `remarks = [[2,1],[1,2]]` -> Output: `2`
  Why: both call each other honest, so labeling both honest is fully consistent.
- Input: `remarks = [[2,2],[2,2]]` -> Output: `2`
  Why: nobody remarks about anybody, so all reviewers can be honest with no constraints (no-remark edge case).
- Input: `remarks = [[2,0,0],[0,2,0],[0,0,2]]` -> Output: `1`
  Why: everyone calls both others unreliable; any two honest reviewers would contradict each other, so the max honest count is 1.
- Input: `remarks = [[2,1,1],[1,2,1],[1,1,2]]` -> Output: `3`
  Why: everyone vouches for everyone as honest, so all three being honest is consistent (maximum possible).

#### Approach

Only honest reviewers must tell the truth; unreliable ones constrain nothing. So the entire consistency of a labeling is decided by which reviewers are honest — the honest set. Enumerate every subset of reviewers as a bitmask candidate for the honest set. A subset is valid iff, for each honest member i, every non-2 remark exactly matches the subset membership of its target: a remark of 1 (says honest) requires the target's bit to be set, and a remark of 0 (says unreliable) requires it to be unset — compactly, remarks[i][j] must equal bit j of the mask. Track the largest valid subset's population count. This is subset enumeration via bit manipulation (brute force over 2^n assignments), viable because n is tiny.

Complexity

Time: O(2^n · n^2) — there are 2^n candidate honest sets, and validating one scans every honest reviewer's row (up to n·n cells). With n ≤ 15 this is at most about 32768 · 225 ≈ 7.4M cell checks. Space: O(1) — only the running mask and best count are kept (no auxiliary structures beyond the input). Justification: the honest set fully determines validity, so enumerating subsets and checking each is both correct and cheap at these bounds.

Python
PYTHON
from typing import List

class Solution:
    def maxHonestReviewers(self, remarks: List[List[int]]) -> int:
        n = len(remarks)
        best = 0
        for mask in range(1 << n):                 # candidate honest set as a bitmask
            ok = True
            for i in range(n):
                if not (mask >> i) & 1:            # only honest reviewers constrain
                    continue
                for j in range(n):
                    if remarks[i][j] == 2:        # no remark => no constraint
                        continue
                    if remarks[i][j] != ((mask >> j) & 1):
                        ok = False
                        break
                if not ok:
                    break
            if ok:
                best = max(best, bin(mask).count("1"))
        return best
JavaScript
JAVASCRIPT
function maxHonestReviewers(remarks) {

#### cons
t n = remarks.length;
    let best = 0;
    for (let mask = 0; mask < (1 << n); ++mask) {
        let ok = true;
        for (let i = 0; i < n && ok; ++i) {
            if (!((mask >> i) & 1)) continue;
            for (let j = 0; j < n; ++j) {
                if (remarks[i][j] === 2) continue;
                if (remarks[i][j] !== ((mask >> j) & 1)) { ok = false; break; }
            }
        }
        if (ok) {
            let cnt = 0, m = mask;
            while (m) { m &= m - 1; ++cnt; }
            best = Math.max(best, cnt);
        }
    }
    return best;
}
Java
JAVA
class Solution {
  public int maxHonestReviewers(int[][] remarks) {
    int n = remarks.length;
    int best = 0;
    for (int mask = 0; mask < (1 << n); ++mask) {
      boolean ok = true;
      for (int i = 0; i < n && ok; ++i) {
        if (((mask >> i) & 1) == 0) continue;
        for (int j = 0; j < n; ++j) {
          if (remarks[i][j] == 2) continue;
          if (remarks[i][j] != ((mask >> j) & 1)) { ok = false; break; }
        }
      }
      if (ok) best = Math.max(best, Integer.bitCount(mask));
    }
    return best;
  }
}
C++
CPP
class Solution {
 public:
  int maxHonestReviewers(vector<vector<int>>& remarks) {
    int n = remarks.size();
    int best = 0;
    for (int mask = 0; mask < (1 << n); ++mask) {
      bool ok = true;
      for (int i = 0; i < n && ok; ++i) {
        if (!((mask >> i) & 1)) continue;
        for (int j = 0; j < n; ++j) {
          if (remarks[i][j] == 2) continue;
          if (remarks[i][j] != ((mask >> j) & 1)) { ok = false; break; }
        }
      }
      if (ok) best = max(best, __builtin_popcount(mask));
    }
    return best;
  }
};
C
C
static int popcount(int x) {
    int c = 0;
    while (x) { x &= (x - 1); ++c; }
    return c;
}

int maxHonestReviewers(int** remarks, int remarksSize, int* remarksColSize) {
    int n = remarksSize;
    int best = 0;
    for (int mask = 0; mask < (1 << n); ++mask) {   /* mask = candidate honest set */
        int ok = 1;
        for (int i = 0; i < n && ok; ++i) {
            if (!((mask >> i) & 1)) continue;        /* only honest reviewers constrain */
            for (int j = 0; j < n; ++j) {
                if (remarks[i][j] == 2) continue;    /* no remark => no constraint */
                if (remarks[i][j] != ((mask >> j) & 1)) { ok = 0; break; }
            }
        }
        if (ok) {
            int cnt = popcount(mask);
            if (cnt > best) best = cnt;
        }
    }
    return best;
}

#### Follow-up
questions
Can you prune the enumeration early — e.g., skip masks whose lowest honest reviewer already contradicts a fixed lower reviewer — to cut the constant factor?
How does the problem change if unreliable reviewers were required to always lie rather than "may lie either way"?
At what n does 2^n enumeration become infeasible, and is there any polynomial-time structure to exploit, or is the general problem inherently exponential?
Besides the maximum count, how would you also reconstruct and return one honest/unreliable labeling that achieves it?

