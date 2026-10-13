---
type: "practice"
difficulty: "Other"
---

# Running Weighted Sums Over a Toggleable Bit Array

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Segment Tree](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Segment_Tree.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given two 0-indexed integer arrays of equal length, bits and scores, and a list of operations queries. Every entry of bits is either 0 or 1. Each operation is a triple [type, x, y] and is one of three kinds:

Type 1 — [1, l, r]: toggle every value of bits in the index range l .. r inclusive (each 0 becomes 1 and each 1 becomes 0). Here x = l and y = r.
Type 2 — [2, m, 0]: for every index i, add bits[i] * m to scores[i]. In effect, scores[i] += bits[i] * m for all i. Here x = m and y is unused (always 0).
Type 3 — [3, 0, 0]: record the current sum of all elements of scores. Here x and y are unused.

Process the operations in order and return an array containing the answer to every type 3 operation, in the order they occurred.

#### Cons
traints

1 <= bits.length == scores.length <= 10^5
1 <= queries.length <= 10^5
Each queries[i] has exactly 3 elements.
For a type 1 operation: 0 <= l <= r <= bits.length - 1.
For a type 2 operation: 0 <= m <= 10^6.
0 <= bits[i] <= 1 and 0 <= scores[i] <= 10^9.
Sums can grow large (up to roughly 10^16), so accumulate them in a 64-bit integer type.

#### Example
s & test cases
TEXT
Input: bits = [0,1,1], scores = [2,3,4], queries = [[3,0,0],[1,0,0],[2,2,0],[3,0,0]] -> Output: [9, 15]
Why: first type 3 reports 2+3+4 = 9. Type 1 toggles index 0 → bits = [1,1,1]. Type 2 with m=2 adds bits[i]*2 → scores = [4,5,6]. Final type 3 reports 15.

Input: bits = [1,0,1,0,1], scores = [9,9,9,9,9], queries = [[3,0,0],[1,1,3],[2,4,0],[3,0,0]] -> Output: [45, 61]
Why: first type 3 = 45. Toggle indices 1..3 → bits = [1,1,0,1,1] (four ones). Type 2 with m=4 adds 4 ones × 4 = 16 to the total → 61.

Input: bits = [1,1,0,0], scores = [0,0,0,0], queries = [[2,5,0],[3,0,0],[1,0,3],[2,5,0],[3,0,0]] -> Output: [10, 20]
Why: two ones, m=5 adds 2×5 = 10 → first type 3 = 10. Toggling the whole array flips [1,1,0,0] to [0,0,1,1] — still two ones — so the next m=5 adds another 10 → 20 (shows range toggle changing which bits are set while keeping the count).

Input: bits = [1], scores = [7], queries = [[2,0,0],[3,0,0]] -> Output: [7]
Why: single-element edge case; type 2 with m=0 adds nothing, so the sum stays 7.

Input: bits = [0,0], scores = [10,20], queries = [[2,7,0],[3,0,0]] -> Output: [30]
Why: no bit is set, so a type 2 operation adds `0` regardless of m; the total remains 10+20 = 30 (all-zeros edge case).

#### Approach

The key observation is that a type-2 operation adds m to scores[i] exactly for the indices where bits[i] == 1, so its effect on the total is simply m * (number of 1s in bits). Therefore you never need to touch scores element-by-element: keep a single 64-bit running total (initialized to the sum of scores) and, on each type-2 query, do total += ones * m.

All that remains is to maintain ones, the count of 1s in bits, under range toggles. A segment tree with lazy propagation does this: each node stores how many 1s are in its segment, and a range flip on [l, r] replaces a node's count by segmentLength - count and records a boolean "flip" lazy flag pushed down on demand. The root always holds the current total number of ones in O(1). This is the standard lazy-segment-tree range-flip / range-count pattern; a Binary Indexed Tree cannot express the flip lazily, which is why the segment tree is the natural fit. Because sums can reach ~10^16, accumulate them in 64-bit integers (and BigInt in JavaScript to stay exact beyond 2^53).

Complexity

Let n = array length and Q = number of queries. Time: O(n + Q log n) — building the segment tree is O(n), each type-1 range toggle is O(log n) via lazy propagation, and type-2 / type-3 queries are O(1) using the maintained running total and the current count of ones. Space: O(n) for the segment tree (counts + lazy flags).

Python
PYTHON
from typing import List

class _FlipTree:
    """Segment tree over a 0/1 array: range-toggle with lazy flip, total count of 1s."""

    def __init__(self, arr: List[int]):
        self.n = len(arr)
        self.cnt = [0] * (4 * self.n)
        self.flip = [False] * (4 * self.n)
        self._build(1, 0, self.n - 1, arr)

    def _build(self, node, l, r, arr):
        if l == r:
            self.cnt[node] = arr[l]
            return
        mid = (l + r) // 2
        self._build(2 * node, l, mid, arr)
        self._build(2 * node + 1, mid + 1, r, arr)
        self.cnt[node] = self.cnt[2 * node] + self.cnt[2 * node + 1]

    def _apply(self, node, l, r):
        self.cnt[node] = (r - l + 1) - self.cnt[node]
        self.flip[node] = not self.flip[node]

    def _push(self, node, l, r):
        if self.flip[node]:
            mid = (l + r) // 2
            self._apply(2 * node, l, mid)
            self._apply(2 * node + 1, mid + 1, r)
            self.flip[node] = False

    def flip_range(self, ql, qr, node=1, l=0, r=None):
        if r is None:
            r = self.n - 1
        if qr < l or r < ql:
            return
        if ql <= l and r <= qr:
            self._apply(node, l, r)
            return
        self._push(node, l, r)
        mid = (l + r) // 2
        self.flip_range(ql, qr, 2 * node, l, mid)
        self.flip_range(ql, qr, 2 * node + 1, mid + 1, r)
        self.cnt[node] = self.cnt[2 * node] + self.cnt[2 * node + 1]

    def total_ones(self):
        return self.cnt[1]

class Solution:
    def runningTotals(self, bits: List[int], scores: List[int],
                      queries: List[List[int]]) -> List[int]:
        tree = _FlipTree(bits)
        total = sum(scores)
        ones = tree.total_ones()
        out = []
        for kind, x, _y in queries:
            if kind == 1:
                tree.flip_range(x, _y)
                ones = tree.total_ones()
            elif kind == 2:
                total += ones * x
            else:
                out.append(total)
        return out
JavaScript
JAVASCRIPT
/**
 * @param {number[]} bits    // 0/1 array
 * @param {number[]} scores
 * @param {number[][]} queries
 * @return {bigint[]}        // BigInt to keep exact sums (they can exceed 2^53)
 */
function runningTotals(bits, scores, queries) {

#### cons
t n = bits.length;

#### cons
t cnt = new Int32Array(4 * n);   // count of 1s per node

#### cons
t flip = new Uint8Array(4 * n);  // lazy toggle flag

#### cons
t build = (node, l, r) => {
        if (l === r) { cnt[node] = bits[l]; return; }

#### cons
t mid = (l + r) >> 1;
        build(2 * node, l, mid);
        build(2 * node + 1, mid + 1, r);
        cnt[node] = cnt[2 * node] + cnt[2 * node + 1];
    };

#### cons
t applyFlip = (node, l, r) => {
        cnt[node] = (r - l + 1) - cnt[node];
        flip[node] ^= 1;
    };

#### cons
t push = (node, l, r) => {
        if (flip[node]) {

#### cons
t mid = (l + r) >> 1;
            applyFlip(2 * node, l, mid);
            applyFlip(2 * node + 1, mid + 1, r);
            flip[node] = 0;
        }
    };

#### cons
t update = (node, l, r, ql, qr) => {
        if (qr < l || r < ql) return;
        if (ql <= l && r <= qr) { applyFlip(node, l, r); return; }
        push(node, l, r);

#### cons
t mid = (l + r) >> 1;
        update(2 * node, l, mid, ql, qr);
        update(2 * node + 1, mid + 1, r, ql, qr);
        cnt[node] = cnt[2 * node] + cnt[2 * node + 1];
    };

    build(1, 0, n - 1);
    let total = 0n;
    for (const s of scores) total += BigInt(s);
    let ones = BigInt(cnt[1]);

#### cons
t out = [];
    for (const q of queries) {
        if (q[0] === 1) { update(1, 0, n - 1, q[1], q[2]); ones = BigInt(cnt[1]); }
        else if (q[0] === 2) { total += ones * BigInt(q[1]); }
        else out.push(total);
    }
    return out;
}
Java
JAVA
import java.util.ArrayList;
import java.util.List;

class Solution {
    private long[] cnt;      // count of 1s in each segment
    private boolean[] flip;  // lazy toggle flag
    private int N;

    private void build(int node, int l, int r, int[] a) {
        if (l == r) { cnt[node] = a[l]; return; }
        int mid = (l + r) / 2;
        build(2 * node, l, mid, a);
        build(2 * node + 1, mid + 1, r, a);
        cnt[node] = cnt[2 * node] + cnt[2 * node + 1];
    }
    private void applyFlip(int node, int l, int r) {
        cnt[node] = (long)(r - l + 1) - cnt[node];
        flip[node] = !flip[node];
    }
    private void push(int node, int l, int r) {
        if (flip[node]) {
            int mid = (l + r) / 2;
            applyFlip(2 * node, l, mid);
            applyFlip(2 * node + 1, mid + 1, r);
            flip[node] = false;
        }
    }
    private void update(int node, int l, int r, int ql, int qr) {
        if (qr < l || r < ql) return;
        if (ql <= l && r <= qr) { applyFlip(node, l, r); return; }
        push(node, l, r);
        int mid = (l + r) / 2;
        update(2 * node, l, mid, ql, qr);
        update(2 * node + 1, mid + 1, r, ql, qr);
        cnt[node] = cnt[2 * node] + cnt[2 * node + 1];
    }

    public long[] runningTotals(int[] bits, int[] scores, int[][] queries) {
        N = bits.length;
        cnt = new long[4 * N];
        flip = new boolean[4 * N];
        build(1, 0, N - 1, bits);

        long total = 0;
        for (int s : scores) total += s;
        long ones = cnt[1];

        List<Long> out = new ArrayList<>();
        for (int[] q : queries) {
            if (q[0] == 1) { update(1, 0, N - 1, q[1], q[2]); ones = cnt[1]; }
            else if (q[0] == 2) { total += ones * (long) q[1]; }
            else out.add(total);
        }
        long[] res = new long[out.size()];
        for (int i = 0; i < res.length; i++) res[i] = out.get(i);
        return res;
    }
}
C++
CPP
#include <vector>
using namespace std;

class Solution {
    vector<long long> cnt;   // count of 1s in each segment
    vector<char>      flip;  // lazy toggle flag
    int N;

    void build(int node, int l, int r, const vector<int>& a) {
        if (l == r) { cnt[node] = a[l]; return; }
        int mid = (l + r) / 2;
        build(2 * node, l, mid, a);
        build(2 * node + 1, mid + 1, r, a);
        cnt[node] = cnt[2 * node] + cnt[2 * node + 1];
    }
    void applyFlip(int node, int l, int r) {
        cnt[node] = (long long)(r - l + 1) - cnt[node];
        flip[node] = !flip[node];
    }
    void push(int node, int l, int r) {
        if (flip[node]) {
            int mid = (l + r) / 2;
            applyFlip(2 * node, l, mid);
            applyFlip(2 * node + 1, mid + 1, r);
            flip[node] = 0;
        }
    }
    void update(int node, int l, int r, int ql, int qr) {
        if (qr < l || r < ql) return;
        if (ql <= l && r <= qr) { applyFlip(node, l, r); return; }
        push(node, l, r);
        int mid = (l + r) / 2;
        update(2 * node, l, mid, ql, qr);
        update(2 * node + 1, mid + 1, r, ql, qr);
        cnt[node] = cnt[2 * node] + cnt[2 * node + 1];
    }

public:
    vector<long long> runningTotals(vector<int>& bits, vector<int>& scores,
                                    vector<vector<int>>& queries) {
        N = (int)bits.size();
        cnt.assign(4 * N, 0);
        flip.assign(4 * N, 0);
        build(1, 0, N - 1, bits);

        long long total = 0;
        for (int s : scores) total += s;
        long long ones = cnt[1];

        vector<long long> out;
        for (auto& q : queries) {
            if (q[0] == 1) { update(1, 0, N - 1, q[1], q[2]); ones = cnt[1]; }
            else if (q[0] == 2) { total += ones * (long long)q[1]; }
            else out.push_back(total);
        }
        return out;
    }
};
C
C
#include <stdlib.h>

/* Segment tree over `bits`, tracking count of 1s per node with a lazy flip flag. */
static long long *segCnt;
static char      *segFlip;

static void segBuild(int node, int l, int r, const int* a) {
    if (l == r) { segCnt[node] = a[l]; return; }
    int mid = (l + r) / 2;
    segBuild(2 * node, l, mid, a);
    segBuild(2 * node + 1, mid + 1, r, a);
    segCnt[node] = segCnt[2 * node] + segCnt[2 * node + 1];
}
static void segApply(int node, int l, int r) {
    segCnt[node] = (long long)(r - l + 1) - segCnt[node];  /* ones <-> zeros */
    segFlip[node] = !segFlip[node];
}
static void segPush(int node, int l, int r) {
    if (segFlip[node]) {
        int mid = (l + r) / 2;
        segApply(2 * node, l, mid);
        segApply(2 * node + 1, mid + 1, r);
        segFlip[node] = 0;
    }
}
static void segUpdate(int node, int l, int r, int ql, int qr) {
    if (qr < l || r < ql) return;
    if (ql <= l && r <= qr) { segApply(node, l, r); return; }
    segPush(node, l, r);
    int mid = (l + r) / 2;
    segUpdate(2 * node, l, mid, ql, qr);
    segUpdate(2 * node + 1, mid + 1, r, ql, qr);
    segCnt[node] = segCnt[2 * node] + segCnt[2 * node + 1];
}

/* Returns a heap array of type-3 answers; caller frees it. */
long long* runningTotals(int* bits, int bitsSize, int* scores, int scoresSize,
                         int** queries, int queriesSize, int* queriesColSize,
                         int* returnSize) {
    int n = bitsSize;
    segCnt  = (long long*)calloc((size_t)(4 * n), sizeof(long long));
    segFlip = (char*)     calloc((size_t)(4 * n), sizeof(char));
    segBuild(1, 0, n - 1, bits);

    long long total = 0;
    for (int i = 0; i < scoresSize; i++) total += scores[i];
    long long ones = segCnt[1];

    long long* out = (long long*)malloc((size_t)(queriesSize ? queriesSize : 1) * sizeof(long long));
    int cnt = 0;
    for (int i = 0; i < queriesSize; i++) {
        int type = queries[i][0];
        if (type == 1) {
            segUpdate(1, 0, n - 1, queries[i][1], queries[i][2]);
            ones = segCnt[1];
        } else if (type == 2) {
            total += ones * (long long)queries[i][1];
        } else {
            out[cnt++] = total;
        }
    }
    free(segCnt);
    free(segFlip);
    *returnSize = cnt;
    return out;
}

#### Follow-up
questions
Why can't you use a plain Binary Indexed Tree (Fenwick) for the range-toggle count, and what makes the lazy segment tree necessary here?
Suppose type 2 instead multiplied scores[i] by a factor (or added m only where bits[i] == 0) — how would the running-total shortcut change?
Add a fourth operation that queries the sum of scores over an index range [l, r]. What extra structure would you maintain, and at what cost?
If bits values could be arbitrary non-negative integers rather than 0/1, is the "delta = ones * m" trick still valid? What would you track instead?

