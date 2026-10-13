---
type: "practice"
difficulty: "Other"
---

# Count Distinct GCD Values Over All Subsequences

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Math](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Math.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given an array arr of positive integers.

The GCD (greatest common divisor) of a non-empty sequence of numbers is the largest integer that divides every number in the sequence. For example, the GCD of [4, 6, 16] is 2.

A subsequence of arr is any sequence obtained by deleting zero or more elements while keeping the relative order of the remaining elements (for the purpose of a GCD, order and duplicates do not matter — only which values are chosen).

#### Cons
ider the GCD of every non-empty subsequence of arr. Return the number of distinct values that occur as such a GCD.

Input

arr: an array of positive integers.

Output

A single integer: how many distinct GCD values are achievable across all non-empty subsequences.

#### Cons
traints

1 <= arr.length <= 10^5
1 <= arr[i] <= 2 * 10^5

#### Example
s & test cases
TEXT
- Input: `arr = [4,9,6]` -> Output: `6`
  Why: single elements give `4, 9, 6`; pairs give `gcd(4,9)=1`, `gcd(4,6)=2`, `gcd(9,6)=3`; the triple gives `1`. Distinct set `{1,2,3,4,6,9}` has `6` values.
- Input: `arr = [12,15,20]` -> Output: `7`
  Why: achievable GCDs are `1` (whole array), `3` (from 12,15), `4` (from 12,20), `5` (from 15,20), and the three singletons `12, 15, 20` — the set `{1,3,4,5,12,15,20}`.
- Input: `arr = [2,4,8,16]` -> Output: `4`
  Why: every subset's GCD is a power of two and the smallest reachable GCD is `2` (no subset yields `1`); the distinct values are `{2,4,8,16}`.
- Input: `arr = [7,7,7]` -> Output: `1`
  Why: only the value `7` is present, so every non-empty subsequence has GCD `7`; duplicates add nothing new.
- Input: `arr = [8]` -> Output: `1`
  Why: the sole non-empty subsequence is `[8]`, whose GCD is `8` (minimum-size input).

#### Approach

A value g is realizable as the GCD of some subsequence iff the GCD of the set of ALL elements of arr that are multiples of g equals g. Reason: any subsequence whose GCD is a multiple of g must consist entirely of multiples of g, and among all such choices the one with the smallest possible GCD is the one that takes every present multiple of g (adding more elements can only shrink the GCD, never grow it). If even that smallest reachable GCD is strictly greater than g, then no subsequence has GCD exactly g. So: mark which values are present, then for each candidate g from 1 to max, walk g, 2g, 3g, ..., fold the present ones into a running GCD, and stop early the moment the running GCD hits g (it can never drop below its own divisor). Count the g for which the folded GCD equals g. This is a number-theory + sieve/harmonic-enumeration pattern rather than any per-subsequence enumeration.

Complexity

Time: O(M log M) where M is the maximum array value — the outer loop runs M times and the inner "walk the multiples of g" loops M/g times, and sum over g of M/g = M * H(M) = O(M log M); each GCD step is O(log M) but amortizes into the same bound because of the early break. Space: O(M) — a boolean presence table over 1..M. Justification: enumerating multiples of every candidate is the classic harmonic-sum (sieve-style) cost, independent of n beyond the initial presence marking.

Python
PYTHON
from math import gcd
from typing import List

class Solution:
    def countDistinctGCDs(self, arr: List[int]) -> int:
        max_val = max(arr)
        present = [False] * (max_val + 1)
        for x in arr:
            present[x] = True

        count = 0
        for g in range(1, max_val + 1):
            cur = 0                                   # gcd(0, x) == x
            for mult in range(g, max_val + 1, g):
                if present[mult]:
                    cur = gcd(cur, mult)
                    if cur == g:                      # cannot go below g
                        break
            if cur == g:
                count += 1
        return count
JavaScript
JAVASCRIPT
function countDistinctGCDs(arr) {
    let maxVal = 0;
    for (const x of arr) if (x > maxVal) maxVal = x;

#### cons
t present = new Uint8Array(maxVal + 1);
    for (const x of arr) present[x] = 1;

#### cons
t gcd = (a, b) => { while (b) { const t = a % b; a = b; b = t; } return a; };

    let count = 0;
    for (let g = 1; g <= maxVal; ++g) {
        let cur = 0;                                  // gcd(0, x) == x
        for (let mult = g; mult <= maxVal; mult += g) {
            if (present[mult]) {
                cur = gcd(cur, mult);
                if (cur === g) break;                 // already minimal
            }
        }
        if (cur === g) ++count;
    }
    return count;
}
Java
JAVA
class Solution {
  public int countDistinctGCDs(int[] arr) {
    int maxVal = 0;
    for (int x : arr) maxVal = Math.max(maxVal, x);

    boolean[] present = new boolean[maxVal + 1];
    for (int x : arr) present[x] = true;

    int count = 0;
    for (int g = 1; g <= maxVal; ++g) {
      int cur = 0;                                    // gcd(0, x) == x
      for (int mult = g; mult <= maxVal; mult += g) {
        if (present[mult]) {
          cur = gcd(cur, mult);
          if (cur == g) break;                        // already minimal
        }
      }
      if (cur == g) ++count;
    }
    return count;
  }

  private int gcd(int a, int b) {
    while (b != 0) { int t = a % b; a = b; b = t; }
    return a;
  }
}
C++
CPP
#include <numeric>

class Solution {
 public:
  int countDistinctGCDs(vector<int>& arr) {
    int maxVal = *max_element(arr.begin(), arr.end());
    vector<char> present(maxVal + 1, 0);
    for (int x : arr) present[x] = 1;

    int count = 0;
    for (int g = 1; g <= maxVal; ++g) {
      int cur = 0;                                    // gcd(0, x) == x
      for (int mult = g; mult <= maxVal; mult += g) {
        if (present[mult]) {
          cur = std::gcd(cur, mult);
          if (cur == g) break;                        // already minimal
        }
      }
      if (cur == g) ++count;
    }
    return count;
  }
};
C
C
#include <stdlib.h>

static int gcd_int(int a, int b) {
    while (b) { int t = a % b; a = b; b = t; }
    return a;
}

int countDistinctGCDs(int* arr, int n) {
    int maxVal = 0;
    for (int i = 0; i < n; ++i) if (arr[i] > maxVal) maxVal = arr[i];

    char* present = (char*)calloc(maxVal + 1, sizeof(char));
    for (int i = 0; i < n; ++i) present[arr[i]] = 1;

    int count = 0;
    for (int g = 1; g <= maxVal; ++g) {
        int cur = 0;                                  /* gcd(0, x) == x */
        for (int mult = g; mult <= maxVal; mult += g) {
            if (present[mult]) {
                cur = gcd_int(cur, mult);
                if (cur == g) break;                  /* can't shrink below g */
            }
        }
        if (cur == g) ++count;                        /* g is realizable as a subsequence GCD */
    }
    free(present);
    return count;
}

#### Follow-up
questions
Why is it optimal, for a fixed g, to fold in every present multiple of g rather than some subset? Prove that this yields the minimum reachable GCD.
Return the actual sorted list of achievable GCDs instead of only their count.
What if arr[i] can be as large as 10^9? The O(M) presence table no longer fits — how would you restructure the search around the (at most n) distinct values?
Restrict to subsequences of length at least k. How does the realizability test change?

