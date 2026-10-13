---
type: "practice"
difficulty: "Other"
---

# Counting Even Column Ranges Across Two Rows

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Dynamic Programming](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Dynamic_Programming.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given two integer arrays top and bottom, both of length n, describing the two rows of a grid with n columns.

#### Cons
ider a contiguous column range [l, r] with 0 <= l <= r < n. For that range you must, for every column i in [l, r], take exactly one of the two values in that column: either top[i] or bottom[i]. The range is called even if there is a choice of picks for which

(sum of the values you took from the top row) == (sum of the values you took from the bottom row)

where a row's sum is 0 if you never picked a value from that row inside the range.

Two even ranges are considered distinct if any of the following holds:

their left endpoints differ, or
their right endpoints differ, or
at some column i inside the range, one selection takes top[i] while the other takes bottom[i].

In other words, an even range is a specific interval together with a specific top/bottom selection at each of its columns that makes the two row-sums equal. Count the number of distinct even ranges. Because the count can be very large, return it modulo 10^9 + 7.

Input

Two integer arrays top and bottom of equal length n.

Output

The number of distinct even column ranges, taken modulo 10^9 + 7.

#### Cons
traints

n == top.length == bottom.length
1 <= n <= 100
0 <= top[i], bottom[i] <= 100

#### Example
s & test cases
TEXT
Input: top = [4,1,3], bottom = [2,3,1] -> Output: 3
Why: Range [1,2] is even two ways (pick top[1]=1 & bottom[2]=1, or bottom[1]=3 & top[2]=3), and range [0,2] is even one way (top[0]=4, bottom[1]=3, bottom[2]=1 give 4 = 3+1). No single column can balance. Total 2 + 1 = 3.

Input: top = [0,0], bottom = [0,0] -> Output: 8
Why: Every value is 0, so any selection balances (0 = 0). Range [0,0] has 2 selections, range [1,1] has 2, and range [0,1] has 4 selections; 2 + 2 + 4 = 8. All eight selections across the three ranges are even.

Input: top = [3,1,4], bottom = [1,4,2] -> Output: 2
Why: Range [0,1] balances once (bottom[0]=1 = top[1]=1) and range [1,2] balances once (bottom[1]=4 = top[2]=4); the full range [0,2] never balances. Total 2.

Input: top = [5], bottom = [5] -> Output: 0
Why: A single column forces you to take exactly one value, so one row-sum is a positive number and the other is 0 — they can never be equal. Zero even ranges.

#### Approach

Reduce a balanced selection to a signed sum. Taking top[i] adds +top[i] to a running difference D = (top picks) - (bottom picks), and taking bottom[i] adds -bottom[i]. A range is even precisely when some selection drives D to exactly 0. Fix the left endpoint l and grow the window to the right one column at a time, maintaining a DP indexed by the difference value: dp[D] = number of selections of the current window [l, i] that reach difference D. Each new column doubles paths by pushing every count to D + top[i] and D - bottom[i]. After absorbing column i, dp[0] counts the balanced selections of the range [l, i], so add it to the answer. An offset shifts the negative differences into non-negative array indices, and tracking the reachable [lo, hi] band keeps the work near-linear per column. Summing dp[0] over all l and all right endpoints, modulo 10^9 + 7, gives the count of distinct even ranges. This is a subset-sum / meet-the-target counting DP applied over every window.

Complexity

Time: O(n^2 * V), where n is the number of columns and V is the width of the reachable difference band (at most 2 * 100 * 100 + 1). For each of the n left endpoints we extend across up to n columns, each step touching O(V) difference values (in practice only the currently reachable band). Space: O(V) for the two rolling difference arrays.

Python
PYTHON
def number_of_even_ranges(top, bottom):
    MOD = 10**9 + 7
    n = len(top)
    MAX = 10000            # max one-sided sum: 100 columns * 100
    OFF = MAX              # index OFF encodes difference 0
    SIZE = 2 * MAX + 1

    ans = 0
    for l in range(n):
        dp = [0] * SIZE
        dp[OFF] = 1        # empty selection, difference 0
        lo = hi = OFF

        for i in range(l, n):
            t, b = top[i], bottom[i]
            nlo = max(0, lo - b)
            nhi = min(SIZE - 1, hi + t)
            ndp = [0] * SIZE

            for d in range(lo, hi + 1):
                v = dp[d]
                if v:
                    ndp[d + t] = (ndp[d + t] + v) % MOD   # take top value
                    ndp[d - b] = (ndp[d - b] + v) % MOD   # take bottom value

            dp = ndp
            lo, hi = nlo, nhi
            ans = (ans + dp[OFF]) % MOD   # balanced selections of range [l, i]

    return ans
JavaScript
JAVASCRIPT
function numberOfEvenRanges(top, bottom) {

#### cons
t MOD = 1000000007;

#### cons
t n = top.length;

#### cons
t MAX = 10000, OFF = MAX, SIZE = 2 * MAX + 1;

    let ans = 0;
    let dp  = new Array(SIZE).fill(0);
    let ndp = new Array(SIZE).fill(0);

    for (let l = 0; l < n; l++) {
        dp.fill(0);
        dp[OFF] = 1;              // difference 0, empty selection
        let lo = OFF, hi = OFF;

        for (let i = l; i < n; i++) {

#### cons
t t = top[i], b = bottom[i];

#### cons
t nlo = Math.max(0, lo - b);

#### cons
t nhi = Math.min(SIZE - 1, hi + t);
            for (let d = nlo; d <= nhi; d++) ndp[d] = 0;

            for (let d = lo; d <= hi; d++) {

#### cons
t v = dp[d];
                if (v) {
                    ndp[d + t] = (ndp[d + t] + v) % MOD;   // take top value
                    ndp[d - b] = (ndp[d - b] + v) % MOD;   // take bottom value
                }
            }
            [dp, ndp] = [ndp, dp];
            lo = nlo; hi = nhi;
            ans = (ans + dp[OFF]) % MOD;   // balanced selections of range [l, i]
        }
    }
    return ans;
}
Java
JAVA
import java.util.Arrays;

class Solution {
    public int numberOfEvenRanges(int[] top, int[] bottom) {
        final int MOD = 1_000_000_007;
        int n = top.length;
        final int MAX = 10000, OFF = MAX, SIZE = 2 * MAX + 1;

        long ans = 0;
        long[] dp  = new long[SIZE];
        long[] ndp = new long[SIZE];

        for (int l = 0; l < n; l++) {
            Arrays.fill(dp, 0);
            dp[OFF] = 1;                       // difference 0, empty selection
            int lo = OFF, hi = OFF;

            for (int i = l; i < n; i++) {
                int nlo = Math.max(0, lo - bottom[i]);
                int nhi = Math.min(SIZE - 1, hi + top[i]);
                for (int d = nlo; d <= nhi; d++) ndp[d] = 0;

                for (int d = lo; d <= hi; d++) {
                    long v = dp[d];
                    if (v != 0) {
                        ndp[d + top[i]]    = (ndp[d + top[i]]    + v) % MOD;
                        ndp[d - bottom[i]] = (ndp[d - bottom[i]] + v) % MOD;
                    }
                }
                long[] tmp = dp; dp = ndp; ndp = tmp;
                lo = nlo; hi = nhi;
                ans = (ans + dp[OFF]) % MOD;   // balanced selections of range [l, i]
            }
        }
        return (int) ans;
    }
}
C++
CPP
#include <vector>
#include <algorithm>
using namespace std;

class Solution {
public:
    int numberOfEvenRanges(vector<int>& top, vector<int>& bottom) {

#### cons
t int MOD = 1e9 + 7;
        int n = top.size();

#### cons
t int MAX = 10000, OFF = MAX, SIZE = 2 * MAX + 1;

        long long ans = 0;
        vector<long long> dp(SIZE), ndp(SIZE);

        for (int l = 0; l < n; l++) {
            fill(dp.begin(), dp.end(), 0);
            dp[OFF] = 1;                      // difference 0, empty selection
            int lo = OFF, hi = OFF;

            for (int i = l; i < n; i++) {
                int nlo = max(0, lo - bottom[i]);
                int nhi = min(SIZE - 1, hi + top[i]);
                for (int d = nlo; d <= nhi; d++) ndp[d] = 0;

                for (int d = lo; d <= hi; d++) {
                    long long v = dp[d];
                    if (v) {
                        ndp[d + top[i]]    = (ndp[d + top[i]]    + v) % MOD;
                        ndp[d - bottom[i]] = (ndp[d - bottom[i]] + v) % MOD;
                    }
                }
                swap(dp, ndp);
                lo = nlo; hi = nhi;
                ans = (ans + dp[OFF]) % MOD;  // balanced selections of range [l, i]
            }
        }
        return (int)ans;
    }
};
C
C
#include <stdlib.h>
#include <string.h>

#define MOD 1000000007

// For a fixed left endpoint l, extend r one column at a time and track, by the
// running difference (top picks minus bottom picks), how many selections reach
// each difference. A range is even exactly when that difference is 0.
int numberOfEvenRanges(int* top, int topSize, int* bottom, int bottomSize) {
    int n = topSize;

#### cons
t int MAX = 10000;          // max one-sided sum: 100 columns * 100

#### cons
t int OFF = MAX;            // index OFF represents difference 0

#### cons
t int SIZE = 2 * MAX + 1;

    long long ans = 0;
    long long* dp  = malloc(sizeof(long long) * SIZE);
    long long* ndp = malloc(sizeof(long long) * SIZE);

    for (int l = 0; l < n; l++) {
        memset(dp, 0, sizeof(long long) * SIZE);
        dp[OFF] = 1;                // empty prefix: difference 0, one way
        int lo = OFF, hi = OFF;     // band of reachable differences

        for (int i = l; i < n; i++) {
            int nlo = lo - bottom[i]; if (nlo < 0) nlo = 0;
            int nhi = hi + top[i];   if (nhi > SIZE - 1) nhi = SIZE - 1;
            for (int d = nlo; d <= nhi; d++) ndp[d] = 0;

            for (int d = lo; d <= hi; d++) {
                long long v = dp[d];
                if (v) {
                    ndp[d + top[i]]    = (ndp[d + top[i]]    + v) % MOD; // take top
                    ndp[d - bottom[i]] = (ndp[d - bottom[i]] + v) % MOD; // take bottom
                }
            }
            long long* tmp = dp; dp = ndp; ndp = tmp;
            lo = nlo; hi = nhi;
            ans = (ans + dp[OFF]) % MOD;   // selections for range [l, i] that balance
        }
    }
    free(dp); free(ndp);
    return (int)(ans % MOD);
}

#### Follow-up
questions
How does the difference-value DP keep the state bounded, and what is the exact range of reachable differences?
Can you count even ranges without re-running the DP for every left endpoint (for example, a two-pointer or divide-and-conquer merge)?
If values could be negative, how would the offset and band tracking change?
What if you only needed to know whether at least one even range exists, rather than counting all of them?

