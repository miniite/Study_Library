---
type: "practice"
difficulty: "Other"
---

# Deal a Sequence Into Two Buckets by Higher-Count Rule

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Segment Tree](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Segment_Tree.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given a 1-indexed sequence of integers seq of length n. You must deal out its elements one at a time, in order, into two initially empty buckets A and B, following a fixed rule.

Define a helper higher(bucket, x) = the number of elements currently in bucket that are strictly greater than x.

Dealing rule

Operation 1: place seq[1] into bucket A.
Operation 2: place seq[2] into bucket B.
For every operation i from 3 to n, let a = higher(A, seq[i]) and b = higher(B, seq[i]), then:
if a > b, place seq[i] into A;
if a < b, place seq[i] into B;
if a == b, place seq[i] into whichever bucket currently holds fewer elements;
if a == b and both buckets currently hold the same number of elements, place seq[i] into A.

After all n operations, form the answer by concatenating bucket A followed by bucket B. Each bucket preserves the order in which its elements were added. Return that concatenated array.

Input

seq: a 1-indexed array of n integers.

Output

The array obtained by appending bucket B to the end of bucket A.

#### Cons
traints

3 <= n <= 10^5
1 <= seq[i] <= 10^9

#### Example
s & test cases
TEXT
Input: seq = [4, 2, 6, 6] -> Output: [4, 6, 2, 6]
Why: A=[4], B=[2]. Op3 x=6: higher counts 0 and 0, lengths equal -> A=[4,6]. Op4 x=6: counts 0 and 0, A longer so B fewer -> B=[2,6]. Concatenate -> [4,6,2,6]. Exercises the equal-count length tie-breaks.

Input: seq = [7, 20, 4, 2, 3] -> Output: [7, 4, 2, 3, 20]
Why: A=[7], B=[20]. x=4: higher 1 vs 1 equal, equal length -> A. x=2: higher 2 vs 1 -> A. x=3: higher 2 vs 1 -> A. Everything after the seeds lands in A; B stays [20]. Normal case driven by the count comparison.

Input: seq = [5, 5, 5, 5] -> Output: [5, 5, 5, 5]
Why: all equal, so higher is always 0 for both; decisions fall to the length rule: A,B,A,B. Duplicate-heavy edge case.

Input: seq = [1, 3, 2] -> Output: [1, 3, 2]
Why: minimum length n=3. A=[1], B=[3]. x=2: higher(A)=0, higher(B)=1, so 0<1 -> B=[3,2]. Concatenate -> [1,3,2]. Smallest input plus a strict-inequality branch.

Input: seq = [1000000000, 1, 1000000000] -> Output: [1000000000, 1000000000, 1]
Why: A=[1e9], B=[1]. x=1e9: higher 0 vs 0, lengths equal -> A. Confirms large values and duplicates handled via coordinate compression.

#### Approach

The naive higher(bucket, x) scan is O(size) per element, giving O(n^2) overall. The key insight is that higher(bucket, x) = (number of elements in the bucket) minus (number of elements in the bucket that are <= x). "Count of elements <= x" is a prefix-count query answerable in O(log n) with a Binary Indexed Tree (Fenwick tree) if values are first coordinate-compressed to ranks 1..m. Maintain one Fenwick tree per bucket plus each bucket's running size. For each incoming value, compute its rank, query both trees to get the two "strictly greater" counts, apply the branch rules (count comparison, then fewer-elements tie-break, then default to A), append the value, and point-update the chosen bucket's tree. The pattern is order-statistics / ordered-frequency counting via a Fenwick tree over compressed coordinates.

Complexity

Time: O(n log n) — sorting for coordinate compression is O(n log n), and each of the n dealing steps does a constant number of Fenwick-tree operations, each O(log m) where m <= n. Space: O(n) — the compressed value table, the two Fenwick trees, and the two buckets are all linear in n.

Python
PYTHON
from typing import List
from bisect import bisect_left

class Fenwick:
    def __init__(self, n: int):
        self.n = n
        self.t = [0] * (n + 1)

    def update(self, i: int) -> None:
        while i <= self.n:
            self.t[i] += 1
            i += i & (-i)

    def query(self, i: int) -> int:      # prefix count of ranks [1..i]
        s = 0
        while i > 0:
            s += self.t[i]
            i -= i & (-i)
        return s

def result_array(seq: List[int]) -> List[int]:
    vals = sorted(set(seq))                       # coordinate compression
    m = len(vals)
    bit_a, bit_b = Fenwick(m), Fenwick(m)
    A, B = [], []
    for idx, v in enumerate(seq):
        r = bisect_left(vals, v) + 1              # 1-indexed rank
        if idx == 0:
            A.append(v); bit_a.update(r); continue
        if idx == 1:
            B.append(v); bit_b.update(r); continue
        g_a = len(A) - bit_a.query(r)             # strictly greater than v in A
        g_b = len(B) - bit_b.query(r)
        if g_a > g_b:
            A.append(v); bit_a.update(r)
        elif g_a < g_b:
            B.append(v); bit_b.update(r)
        elif len(A) <= len(B):
            A.append(v); bit_a.update(r)
        else:
            B.append(v); bit_b.update(r)
    return A + B
JavaScript
JAVASCRIPT
function resultArray(seq) {

#### cons
t vals = Array.from(new Set(seq)).sort((a, b) => a - b); // coordinate compression

#### cons
t m = vals.length;

#### cons
t rank = new Map();
    for (let i = 0; i < m; i++) rank.set(vals[i], i + 1);        // 1-indexed rank

#### cons
t tA = new Int32Array(m + 1);

#### cons
t tB = new Int32Array(m + 1);

#### cons
t update = (t, i) => { for (; i <= m; i += i & (-i)) t[i]++; };

#### cons
t query = (t, i) => { let s = 0; for (; i > 0; i -= i & (-i)) s += t[i]; return s; };

#### cons
t A = [], B = [];
    for (let idx = 0; idx < seq.length; idx++) {

#### cons
t v = seq[idx];

#### cons
t r = rank.get(v);
        if (idx === 0) { A.push(v); update(tA, r); continue; }
        if (idx === 1) { B.push(v); update(tB, r); continue; }

#### cons
t gA = A.length - query(tA, r);   // strictly greater than v in A

#### cons
t gB = B.length - query(tB, r);
        if (gA > gB) { A.push(v); update(tA, r); }
        else if (gA < gB) { B.push(v); update(tB, r); }
        else if (A.length <= B.length) { A.push(v); update(tA, r); }
        else { B.push(v); update(tB, r); }
    }
    return A.concat(B);
}
Java
JAVA
import java.util.*;

class Solution {
    static class Fenwick {
        int[] t; int n;
        Fenwick(int n) { this.n = n; t = new int[n + 1]; }
        void update(int i) { for (; i <= n; i += i & (-i)) t[i]++; }
        int query(int i) { int s = 0; for (; i > 0; i -= i & (-i)) s += t[i]; return s; }
    }

    public int[] resultArray(int[] seq) {
        int[] sorted = seq.clone();
        Arrays.sort(sorted);
        // build 1-indexed rank map over unique values
        HashMap<Integer, Integer> rank = new HashMap<>();
        int m = 0;
        for (int v : sorted) if (!rank.containsKey(v)) rank.put(v, ++m);

        Fenwick bitA = new Fenwick(m), bitB = new Fenwick(m);
        ArrayList<Integer> A = new ArrayList<>(), B = new ArrayList<>();
        for (int idx = 0; idx < seq.length; idx++) {
            int v = seq[idx];
            int r = rank.get(v);
            if (idx == 0)                  { A.add(v); bitA.update(r); continue; }
            if (idx == 1)                  { B.add(v); bitB.update(r); continue; }
            int gA = A.size() - bitA.query(r);   // strictly greater than v in A
            int gB = B.size() - bitB.query(r);
            if      (gA > gB)              { A.add(v); bitA.update(r); }
            else if (gA < gB)             { B.add(v); bitB.update(r); }
            else if (A.size() <= B.size()) { A.add(v); bitA.update(r); }
            else                          { B.add(v); bitB.update(r); }
        }
        int[] res = new int[seq.length];
        int k = 0;
        for (int x : A) res[k++] = x;
        for (int x : B) res[k++] = x;
        return res;
    }
}
C++
CPP
#include <vector>
#include <algorithm>
using namespace std;

class Fenwick {
    vector<int> t; int n;
public:
    Fenwick(int n = 0) : t(n + 1, 0), n(n) {}
    void update(int i) { for (; i <= n; i += i & (-i)) ++t[i]; }
    int query(int i) const { int s = 0; for (; i > 0; i -= i & (-i)) s += t[i]; return s; }
};

class Solution {
public:
    vector<int> resultArray(vector<int>& seq) {
        vector<int> vals(seq.begin(), seq.end());
        sort(vals.begin(), vals.end());
        vals.erase(unique(vals.begin(), vals.end()), vals.end());
        int m = (int)vals.size();

        Fenwick bitA(m), bitB(m);
        vector<int> A, B;
        for (int idx = 0; idx < (int)seq.size(); ++idx) {
            int v = seq[idx];
            int r = int(lower_bound(vals.begin(), vals.end(), v) - vals.begin()) + 1;
            if (idx == 0)                { A.push_back(v); bitA.update(r); continue; }
            if (idx == 1)                { B.push_back(v); bitB.update(r); continue; }
            int gA = (int)A.size() - bitA.query(r);   // strictly greater than v in A
            int gB = (int)B.size() - bitB.query(r);
            if      (gA > gB)            { A.push_back(v); bitA.update(r); }
            else if (gA < gB)            { B.push_back(v); bitB.update(r); }
            else if (A.size() <= B.size()) { A.push_back(v); bitA.update(r); }
            else                         { B.push_back(v); bitB.update(r); }
        }
        A.insert(A.end(), B.begin(), B.end());
        return A;
    }
};
C
C
#include <stdlib.h>
#include <string.h>

/* Fenwick (Binary Indexed Tree) over a flat array of counts, size m+1. */
static void bitUpdate(int* t, int m, int i) {
    for (; i <= m; i += i & (-i)) t[i]++;
}
static int bitQuery(int* t, int i) {          /* prefix count of ranks [1..i] */
    int s = 0;
    for (; i > 0; i -= i & (-i)) s += t[i];
    return s;
}

static int cmpInt(const void* a, const void* b) {
    int x = *(const int*)a, y = *(const int*)b;
    return (x > y) - (x < y);
}

/* 1-indexed rank of v among the sorted unique values vals[0..m-1]. */
static int rankOf(const int* vals, int m, int v) {
    int lo = 0, hi = m - 1, ans = 0;
    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2;
        if (vals[mid] < v) lo = mid + 1;
        else { ans = mid; hi = mid - 1; }
    }
    return ans + 1;
}

int* resultArray(int* seq, int seqSize, int* returnSize) {
    /* coordinate compression: sorted unique values */
    int* vals = (int*)malloc(seqSize * sizeof(int));
    memcpy(vals, seq, seqSize * sizeof(int));
    qsort(vals, seqSize, sizeof(int), cmpInt);
    int m = 0;
    for (int i = 0; i < seqSize; i++)
        if (i == 0 || vals[i] != vals[i - 1]) vals[m++] = vals[i];

    int* bitA = (int*)calloc(m + 1, sizeof(int));
    int* bitB = (int*)calloc(m + 1, sizeof(int));
    int* A = (int*)malloc(seqSize * sizeof(int));
    int* B = (int*)malloc(seqSize * sizeof(int));
    int sa = 0, sb = 0;

    for (int idx = 0; idx < seqSize; idx++) {
        int v = seq[idx];
        int r = rankOf(vals, m, v);
        if (idx == 0)      { A[sa++] = v; bitUpdate(bitA, m, r); continue; }
        if (idx == 1)      { B[sb++] = v; bitUpdate(bitB, m, r); continue; }
        int gA = sa - bitQuery(bitA, r);   /* count strictly greater than v in A */
        int gB = sb - bitQuery(bitB, r);
        if      (gA > gB)  { A[sa++] = v; bitUpdate(bitA, m, r); }
        else if (gA < gB)  { B[sb++] = v; bitUpdate(bitB, m, r); }
        else if (sa <= sb) { A[sa++] = v; bitUpdate(bitA, m, r); }
        else               { B[sb++] = v; bitUpdate(bitB, m, r); }
    }

    int* result = (int*)malloc(seqSize * sizeof(int));
    int k = 0;
    for (int i = 0; i < sa; i++) result[k++] = A[i];
    for (int i = 0; i < sb; i++) result[k++] = B[i];
    *returnSize = k;

    free(vals); free(bitA); free(bitB); free(A); free(B);
    return result;
}

#### Follow-up
questions
How does the query change if you needed the count of elements strictly less than x instead of strictly greater?
Could you support the buckets growing indefinitely as an online stream, answering interleaved higher queries between insertions?
Would a balanced BST or an order-statistics tree give the same asymptotics, and what are the constant-factor trade-offs versus a Fenwick tree?
If the tie-break flipped to "append to the larger bucket," which comparison lines change and does correctness still hold?

