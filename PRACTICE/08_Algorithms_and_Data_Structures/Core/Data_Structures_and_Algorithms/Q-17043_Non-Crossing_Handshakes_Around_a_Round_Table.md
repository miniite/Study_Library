---
type: "practice"
difficulty: "Other"
---

# Non-Crossing Handshakes Around a Round Table

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Dynamic Programming](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Dynamic_Programming.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

An even number of guests, count, are seated at equal spacing around a circular table. At a signal, they all pair up and reach across the table to shake hands, so that exactly count / 2 handshakes happen and every guest is part of exactly one handshake.

If you draw each handshake as a straight line (chord) between the two guests, count the number of distinct ways the guests can pair up such that no two handshake lines cross each other.

Because the number of arrangements can be enormous, return the answer modulo 1000000007 (10^9 + 7).

Input

count: an even integer, the number of guests around the table.

Output

The number of non-crossing pairings, taken modulo 10^9 + 7.

#### Cons
traints

2 <= count <= 1000
count is even.

#### Example
s & test cases
TEXT
- Input: `count = 2` -> Output: `1`
  Why: only two guests, so the single handshake between them is the one and only pairing (minimum input).
- Input: `count = 4` -> Output: `2`
  Why: guests `1..4` around the table can pair as `{(1,2),(3,4)}` or `{(1,4),(2,3)}`; pairing `(1,3)` and `(2,4)` would cross, so it is excluded.
- Input: `count = 6` -> Output: `5`
  Why: this is the 3rd Catalan number; five non-crossing perfect matchings exist on six points.
- Input: `count = 8` -> Output: `14`
  Why: the count follows the Catalan sequence (1, 2, 5, 14, ...); with 8 guests there are 14 valid arrangements.
- Input: `count = 10` -> Output: `42`
  Why: the 5th Catalan number; confirms the pattern continues to grow.
- Input: `count = 12` -> Output: `132`
  Why: the 6th Catalan number; still well within `int` before the modulo matters, validating the recurrence for larger inputs.

#### Approach

The number of non-crossing perfect matchings of 2n points on a circle is exactly the nth Catalan number. The recurrence comes from fixing one reference guest and choosing their handshake partner: that chord splits the remaining guests into two arcs that must each be matched internally (a chord may never cross the fixed one). If the reference guest shakes the j-th person along, one side holds j - 1 pairs and the other side holds i - j pairs, and the two sides are independent — giving dp[i] = sum over j of dp[j-1] * dp[i-j], with dp[0] = 1. Evaluate this bottom-up modulo 10^9 + 7. Pattern: Catalan-number dynamic programming (the same recurrence that counts balanced parenthesizations and full binary trees); products are kept in 64-bit / BigInt to avoid overflow before the modulo.

Complexity

Let n = count / 2 be the number of handshakes. Time: O(n^2) — the outer loop runs n times and the inner convolution runs up to i times, summing to ~n^2 / 2 operations. Space: O(n) for the dp array. Justification: this is a one-dimensional DP whose value at index i is a convolution of previously computed values.

Python
PYTHON
class Solution:
    def numberOfWays(self, count: int) -> int:
        MOD = 1_000_000_007
        n = count // 2
        dp = [0] * (n + 1)
        dp[0] = 1                                   # one way to pair up zero guests
        for i in range(1, n + 1):
            total = 0
            for j in range(1, i + 1):               # reference guest's partner splits the arc
                total += dp[j - 1] * dp[i - j]
            dp[i] = total % MOD
        return dp[n]
JavaScript
JAVASCRIPT
function numberOfWays(count) {

#### cons
t MOD = 1000000007n;

#### cons
t n = count / 2;

#### cons
t dp = new Array(n + 1).fill(0n);
    dp[0] = 1n;
    for (let i = 1; i <= n; ++i) {
        for (let j = 1; j <= i; ++j) {
            dp[i] = (dp[i] + dp[j - 1] * dp[i - j]) % MOD;   // BigInt keeps products exact
        }
    }
    return Number(dp[n]);
}
Java
JAVA
class Solution {
    public int numberOfWays(int count) {
        final long MOD = 1_000_000_007L;
        int n = count / 2;
        long[] dp = new long[n + 1];
        dp[0] = 1;
        for (int i = 1; i <= n; ++i)
            for (int j = 1; j <= i; ++j)
                dp[i] = (dp[i] + dp[j - 1] * dp[i - j]) % MOD;
        return (int) dp[n];
    }
}
C++
CPP
#include <bits/stdc++.h>
using namespace std;

class Solution {
 public:
  int numberOfWays(int count) {

#### cons
t long long MOD = 1000000007LL;
    int n = count / 2;
    vector<long long> dp(n + 1, 0);
    dp[0] = 1;
    for (int i = 1; i <= n; ++i)
      for (int j = 1; j <= i; ++j)
        dp[i] = (dp[i] + dp[j - 1] * dp[i - j]) % MOD;
    return (int)dp[n];
  }
};
C
C
#include <stdlib.h>

int numberOfWays(int count) {

#### cons
t long MOD = 1000000007L;
    int n = count / 2;                  /* number of handshakes = nth Catalan number */
    long *dp = (long *)calloc(n + 1, sizeof(long));
    dp[0] = 1;                          /* one way to pair up zero guests */
    for (int i = 1; i <= n; ++i) {
        long s = 0;
        for (int j = 1; j <= i; ++j)    /* partner of the reference guest splits the arc */
            s = (s + dp[j - 1] * dp[i - j]) % MOD;
        dp[i] = s;
    }
    long ans = dp[n];
    free(dp);
    return (int)ans;
}

#### Follow-up
questions
Derive the closed form C(2n, n) / (n + 1) and compute it in O(n) using precomputed factorials and a modular inverse (Fermat's little theorem).
How many pairings are there if crossings ARE allowed? (All perfect matchings: (2n - 1)!!.)
Explain the bijection between these non-crossing handshakes and balanced parenthesis strings / full binary trees.
What if the modulus were not prime — how would you compute the nth Catalan number modulo a composite?

