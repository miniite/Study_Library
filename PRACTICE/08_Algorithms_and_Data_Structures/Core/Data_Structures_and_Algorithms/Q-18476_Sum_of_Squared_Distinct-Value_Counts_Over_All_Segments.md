---
type: "practice"
difficulty: "Other"
---

# Sum of Squared Distinct-Value Counts Over All Segments

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Segment Tree](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Segment_Tree.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given a 0-indexed integer array a of length n.

For any contiguous segment a[i..j] (with 0 <= i <= j < n), define its variety as the number of distinct values that appear inside that segment. For example, the segment [4, 7, 4] has a variety of 2, because it contains only the two distinct values 4 and 7.

#### Cons
ider every non-empty contiguous segment of a. For each such segment, square its variety, and add all of these squares together. Return this total.

Because the result can be extremely large, return it modulo 1_000_000_007.

Input: an integer array a of length n.

Output: a single integer — the sum, over all non-empty contiguous segments, of (variety of the segment)^2, taken modulo 1_000_000_007.

#### Cons
traints:

1 <= n <= 10^5
1 <= a[i] <= 10^5

#### Example
s & test cases
TEXT
- Input: `a = [4, 7, 4]` -> Output: `15`
  Why: The six segments have varieties `1,1,1` (singletons) and `2,2,2` (`[4,7]`, `[7,4]`, `[4,7,4]`), so the sum of squares is `1+1+1+4+4+4 = 15`. A basic case with one repeated value.
- Input: `a = [5, 5]` -> Output: `3`
  Why: Segments `[5]`, `[5]`, `[5,5]` all have variety `1`, giving `1+1+1 = 3`. Tests full duplication (variety never grows).
- Input: `a = [9]` -> Output: `1`
  Why: The only segment has variety `1`; `1^2 = 1`. Minimal single-element case.
- Input: `a = [3, 1, 3, 2]` -> Output: `38`
  Why: Varieties over all 10 segments are `1,2,2,3 | 1,2,3 | 1,2 | 1`; squares sum to `18+14+5+1 = 38`. A mixed case where a value repeats away from its first occurrence.
- Input: `a = [8, 8, 8, 8]` -> Output: `10`
  Why: Every segment has variety `1`, and there are `4*5/2 = 10` segments, so the total is `10`. Stress case for many repeats.

#### Approach

Fix the right endpoint j and think of the variety of every segment a[i..j] for all left endpoints i <= j. When you extend from j-1 to j by appending a[j], the new value is newly distinct only for left endpoints i in the window (lastOccurrence[a[j]], j]; for those, the variety rises by exactly 1 (a range increment of size j - l + 1, where l = lastOccurrence[a[j]] + 1). Everything else is unchanged. To keep a running sum of squares under a range +1, use the identity Σ((c+1)^2 - c^2) = Σ(2c + 1) = 2·(rangeSum of old varieties) + count. A pair of Fenwick trees supports range-update with range-sum in O(log n), supplying that range sum; a running accumulator holds the current sum of squares and is added to the answer after each step. This is the contribution / "fix one endpoint and maintain a range-updated array" pattern.

Complexity

Time: O(n log n) — each of the n right endpoints performs a constant number of Fenwick range-add / range-sum operations, each O(log n). Space: O(n) for the two Fenwick trees plus O(maxValue) for the last-occurrence table.

Python
PYTHON
def sum_of_squared_varieties(a):
    MOD = 1_000_000_007
    n = len(a)
    N = n + 1
    B1 = [0] * (N + 2)   # BIT for the difference array
    B2 = [0] * (N + 2)   # BIT weighted by index

    def bit_add(tree, i, v):
        while i <= N:
            tree[i] += v
            i += i & (-i)

    def bit_query(tree, i):
        s = 0
        while i > 0:
            s += tree[i]
            i -= i & (-i)
        return s

    last = {}
    total = 0          # current sum of squared varieties (segments ending at j)
    ans = 0
    for j in range(n):
        l = last.get(a[j], -1) + 1
        pref_j = bit_query(B1, j) * j - bit_query(B2, j)
        pref_l = bit_query(B1, l) * l - bit_query(B2, l)
        rs = pref_j - pref_l                       # sum of varieties over [l, j-1]
        delta = (2 * (rs % MOD) + (j - l + 1)) % MOD
        total = (total + delta) % MOD

        L, R = l + 1, j + 1                         # range-add +1 over [L, R]
        bit_add(B1, L, 1);      bit_add(B1, R + 1, -1)
        bit_add(B2, L, L - 1);  bit_add(B2, R + 1, -R)

        ans = (ans + total) % MOD
        last[a[j]] = j
    return ans % MOD
JavaScript
JS
function sumOfSquaredVarieties(a) {

#### cons
t MOD = 1000000007;

#### cons
t n = a.length;

#### cons
t N = n + 1;

#### cons
t B1 = new Array(N + 2).fill(0);

#### cons
t B2 = new Array(N + 2).fill(0);

#### cons
t bitAdd = (tree, i, v) => {
        for (; i <= N; i += i & (-i)) tree[i] += v;
    };

#### cons
t bitQuery = (tree, i) => {
        let s = 0;
        for (; i > 0; i -= i & (-i)) s += tree[i];
        return s;
    };

#### cons
t last = new Map();
    let total = 0, ans = 0;
    for (let j = 0; j < n; j++) {

#### cons
t l = (last.has(a[j]) ? last.get(a[j]) : -1) + 1;

#### cons
t prefJ = bitQuery(B1, j) * j - bitQuery(B2, j);

#### cons
t prefL = bitQuery(B1, l) * l - bitQuery(B2, l);

#### cons
t rs = prefJ - prefL;                    // sum of varieties over [l, j-1]

#### cons
t delta = (2 * (rs % MOD) + (j - l + 1)) % MOD;
        total = (total + delta) % MOD;

#### cons
t L = l + 1, R = j + 1;                  // range-add +1 over [L, R]
        bitAdd(B1, L, 1);      bitAdd(B1, R + 1, -1);
        bitAdd(B2, L, L - 1);  bitAdd(B2, R + 1, -R);

        ans = (ans + total) % MOD;
        last.set(a[j], j);
    }
    return ans % MOD;
}
Java
JAVA
import java.util.Arrays;

class Solution {
    private long[] B1, B2;
    private int N;

    private void bitAdd(long[] t, int i, long v) {
        for (; i <= N; i += i & (-i)) t[i] += v;
    }
    private long bitQuery(long[] t, int i) {
        long s = 0;
        for (; i > 0; i -= i & (-i)) s += t[i];
        return s;
    }

    public int sumOfSquaredVarieties(int[] a) {
        final long MOD = 1000000007L;
        int n = a.length;
        N = n + 1;
        B1 = new long[N + 2];
        B2 = new long[N + 2];

        int maxV = 0;
        for (int x : a) maxV = Math.max(maxV, x);
        int[] last = new int[maxV + 1];
        Arrays.fill(last, -1);

        long total = 0, ans = 0;
        for (int j = 0; j < n; j++) {
            int l = last[a[j]] + 1;
            long prefJ = bitQuery(B1, j) * (long) j - bitQuery(B2, j);
            long prefL = bitQuery(B1, l) * (long) l - bitQuery(B2, l);
            long rs = prefJ - prefL;
            long delta = (2 * (rs % MOD) + (long) (j - l + 1)) % MOD;
            total = (total + delta) % MOD;

            int L = l + 1, R = j + 1;
            bitAdd(B1, L, 1);                bitAdd(B1, R + 1, -1);
            bitAdd(B2, L, (long) (L - 1));   bitAdd(B2, R + 1, -(long) R);

            ans = (ans + total) % MOD;
            last[a[j]] = j;
        }
        return (int) (ans % MOD);
    }
}
C++
CPP
#include <vector>
using namespace std;

class Solution {
    vector<long long> B1, B2;
    int N;
    void bitAdd(vector<long long>& t, int i, long long v) {
        for (; i <= N; i += i & (-i)) t[i] += v;
    }
    long long bitQuery(vector<long long>& t, int i) {
        long long s = 0;
        for (; i > 0; i -= i & (-i)) s += t[i];
        return s;
    }
public:
    int sumOfSquaredVarieties(vector<int>& a) {

#### cons
t long long MOD = 1000000007LL;
        int n = (int)a.size();
        N = n + 1;
        B1.assign(N + 2, 0);
        B2.assign(N + 2, 0);

        int maxV = 0;
        for (int x : a) maxV = max(maxV, x);
        vector<int> last(maxV + 1, -1);

        long long total = 0, ans = 0;
        for (int j = 0; j < n; j++) {
            int l = last[a[j]] + 1;
            long long prefJ = bitQuery(B1, j) * (long long)j - bitQuery(B2, j);
            long long prefL = bitQuery(B1, l) * (long long)l - bitQuery(B2, l);
            long long rs = prefJ - prefL;
            long long delta = (2 * (rs % MOD) + (long long)(j - l + 1)) % MOD;
            total = (total + delta) % MOD;

            int L = l + 1, R = j + 1;
            bitAdd(B1, L, 1);                 bitAdd(B1, R + 1, -1);
            bitAdd(B2, L, (long long)(L - 1)); bitAdd(B2, R + 1, -(long long)R);

            ans = (ans + total) % MOD;
            last[a[j]] = j;
        }
        return (int)(ans % MOD);
    }
};
C
C
#include <stdlib.h>

/* Two Fenwick (BIT) trees give O(log n) range-add + range-sum. */
static void bitAdd(long long* tree, int N, int i, long long v) {
    for (; i <= N; i += i & (-i)) tree[i] += v;
}
static long long bitQuery(long long* tree, int i) {
    long long s = 0;
    for (; i > 0; i -= i & (-i)) s += tree[i];
    return s;
}

int sumOfSquaredVarieties(int* a, int n) {

#### cons
t long long MOD = 1000000007LL;
    int N = n + 1;                                 /* 1-indexed positions 1..n+1 */
    long long* B1 = (long long*)calloc(N + 2, sizeof(long long));
    long long* B2 = (long long*)calloc(N + 2, sizeof(long long));

    int maxV = 0;
    for (int i = 0; i < n; i++) if (a[i] > maxV) maxV = a[i];
    int* last = (int*)malloc((maxV + 1) * sizeof(int));
    for (int v = 0; v <= maxV; v++) last[v] = -1;

    long long total = 0, ans = 0;                  /* total = current sum of squares */
    for (int j = 0; j < n; j++) {
        int l = last[a[j]] + 1;                    /* left boundary of the +1 window */
        /* sum of varieties over 0-indexed [l, j-1] = prefix(j) - prefix(l) */
        long long prefJ = bitQuery(B1, j) * (long long)j - bitQuery(B2, j);
        long long prefL = bitQuery(B1, l) * (long long)l - bitQuery(B2, l);
        long long rs = prefJ - prefL;
        long long delta = (2 * (rs % MOD) + (long long)(j - l + 1)) % MOD;
        total = (total + delta) % MOD;

        int L = l + 1, R = j + 1;                  /* range-add +1 over 1-indexed [L,R] */
        bitAdd(B1, N, L, 1);                bitAdd(B1, N, R + 1, -1);
        bitAdd(B2, N, L, (long long)(L - 1)); bitAdd(B2, N, R + 1, -(long long)R);

        ans = (ans + total) % MOD;
        last[a[j]] = j;
    }
    free(B1); free(B2); free(last);
    return (int)(ans % MOD);
}

#### Follow-up
questions
Return the sum of the varieties themselves (not squared) — this needs only a running counter, no Fenwick tree, and runs in O(n).
Return the sum of cubes of the varieties; derive the incremental delta Σ((c+1)^3 - c^3) and decide what range statistics you must maintain.
Support point updates to the array between queries — how do the maintained structures change?
What if you only consider segments of a fixed window length w, or only segments starting at index 0?

