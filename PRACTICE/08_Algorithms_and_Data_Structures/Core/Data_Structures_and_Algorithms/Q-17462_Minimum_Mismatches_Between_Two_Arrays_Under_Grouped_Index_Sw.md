---
type: "practice"
difficulty: "Other"
---

# Minimum Mismatches Between Two Arrays Under Grouped Index Swaps

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Union Find](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Union_Find.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given two integer arrays first and second, both of length n. You are also given a list swaps, where each entry swaps[i] = [a, b] means you are permitted to exchange the elements currently sitting at index a and index b of the array first (indices are 0-indexed). You may perform any allowed swap as many times as you like, in any order.

Because a swap can be applied repeatedly and swaps can be chained, the set of allowed index pairs partitions the indices of first into connected groups: within a single group you can rearrange the values held at those indices into any permutation you want. Values at indices in different groups can never mix.

The mismatch count between first and second is the number of positions i (for 0 <= i <= n-1) at which first[i] != second[i].

Rearrange the values of first using any sequence of allowed swaps so as to minimize the mismatch count, and return that minimum possible value. The array second is fixed and never changes.

Input

first: an integer array of length n.
second: an integer array of length n.
swaps: a list of index pairs [a, b] (possibly empty), each allowing a swap within first.

Output

A single integer: the minimum achievable number of positions where first and second differ after optimally applying the allowed swaps.

#### Cons
traints

n == first.length == second.length.
1 <= n <= 10^5.
1 <= first[i], second[i] <= 10^5.
0 <= swaps.length <= 10^5.
Each swaps[i] has exactly two entries a and b with 0 <= a, b <= n - 1 and a != b.

#### Example
s & test cases
TEXT
Input: first = [3,4,1,2], second = [4,3,2,7], swaps = [[0,1],[2,3]] -> Output: 1
Why: Group {0,1} holds values {3,4} and needs {4,3} — fully matchable (0 mismatches). Group {2,3} holds {1,2} but needs {2,7}; value 2 can match, value 1 cannot cover 7, leaving 1 mismatch. Total 1.

Input: first = [5,6,7,8], second = [5,7,6,8], swaps = [] -> Output: 2
Why: No swaps, so every index is its own group and must match position-for-position. Positions 1 and 2 differ (6 vs 7, 7 vs 6), positions 0 and 3 already match. Answer 2. Edge case: empty swaps.

Input: first = [9,2,3,7,4], second = [2,9,7,3,4], swaps = [[0,4],[4,2],[1,3],[1,4]] -> Output: 0
Why: The swaps connect all five indices into one group. That group holds the multiset {9,2,3,7,4}, exactly what `second` requires, so a perfect arrangement exists. Answer 0. Fully connected case.

Input: first = [4,4,5], second = [5,4,4], swaps = [[0,2]] -> Output: 0
Why: Group {0,2} holds {4,5} and can supply the needed 5 at index 0 and 4 at index 2; index 1 already holds 4 which matches. All positions satisfiable. Shows duplicate values handled by multiset matching.

Input: first = [1], second = [2], swaps = [] -> Output: 1
Why: Single element, no swap possible, and 1 != 2, so exactly one mismatch. Minimum-size edge case.

Input: first = [6,6,6], second = [6,6,9], swaps = [[0,1],[1,2]] -> Output: 1
Why: All three indices form one group holding {6,6,6}; `second` needs {6,6,9}. Two of the three 6-values match the two required 6s, but no value equals 9, leaving 1 unavoidable mismatch. Duplicate-heavy case.

#### Approach

The allowed swaps make positions freely rearrangeable within each connected component of indices, so model the swaps as edges of a graph and build the connected components with a Disjoint Set Union (union-find) structure using path compression and union by rank. Within one component the values of first form a multiset that can be assigned to that component's positions in any order; the positions of second in that component form the required multiset. The number of positions that can be made equal in a component equals, summed over each distinct value, the minimum of how many copies first supplies and how many second demands — a multiset intersection. Concretely, for each component tally the values first provides; then scan the positions and, for each second[i], consume one matching value from that component's tally if available, incrementing a match counter. The answer is n minus the total matches. The core patterns are union-find for connectivity and greedy multiset matching by value.

Complexity

Time: O((n + m) · α(n)) where m = swaps.length and α is the inverse-Ackermann function from union-find with path compression and union by rank — effectively linear. The C variant sorts encoded keys instead, giving O(n log n). Space: O(n) for the disjoint-set arrays plus the per-group value-count maps.

Python
PYTHON
from collections import defaultdict, Counter

class Solution:
    def minMismatch(self, first, second, swaps):
        n = len(first)
        parent = list(range(n))

        def find(x):
            while parent[x] != x:
                parent[x] = parent[parent[x]]  # path halving
                x = parent[x]
            return x

        def unite(a, b):
            ra, rb = find(a), find(b)
            if ra != rb:
                parent[rb] = ra

        for a, b in swaps:
            unite(a, b)

        # For each group, the multiset of values `first` can supply.
        supply = defaultdict(Counter)
        for i in range(n):
            supply[find(i)][first[i]] += 1

        matches = 0
        for i in range(n):
            bucket = supply[find(i)]
            if bucket[second[i]] > 0:
                bucket[second[i]] -= 1
                matches += 1
        return n - matches
JavaScript
JAVASCRIPT
function minMismatch(first, second, swaps) {

#### cons
t n = first.length;

#### cons
t parent = Array.from({ length: n }, (_, i) => i);

    function find(x) {
        while (parent[x] !== x) { parent[x] = parent[parent[x]]; x = parent[x]; }
        return x;
    }
    function unite(a, b) {

#### cons
t ra = find(a), rb = find(b);
        if (ra !== rb) parent[rb] = ra;
    }

    for (const [a, b] of swaps) unite(a, b);

    // group root -> Map(value -> available count) supplied by `first`

#### cons
t supply = new Map();
    for (let i = 0; i < n; i++) {

#### cons
t r = find(i);
        if (!supply.has(r)) supply.set(r, new Map());

#### cons
t m = supply.get(r);
        m.set(first[i], (m.get(first[i]) || 0) + 1);
    }

    let matches = 0;
    for (let i = 0; i < n; i++) {

#### cons
t m = supply.get(find(i));

#### cons
t c = m.get(second[i]) || 0;
        if (c > 0) { m.set(second[i], c - 1); matches++; }
    }
    return n - matches;
}
Java
JAVA
import java.util.*;

class Solution {
    private int[] parent, rank_;

    private int find(int x) {
        while (parent[x] != x) { parent[x] = parent[parent[x]]; x = parent[x]; }
        return x;
    }

    private void unite(int a, int b) {
        int ra = find(a), rb = find(b);
        if (ra == rb) return;
        if (rank_[ra] < rank_[rb]) { int t = ra; ra = rb; rb = t; }
        parent[rb] = ra;
        if (rank_[ra] == rank_[rb]) rank_[ra]++;
    }

    public int minMismatch(int[] first, int[] second, int[][] swaps) {
        int n = first.length;
        parent = new int[n];
        rank_ = new int[n];
        for (int i = 0; i < n; i++) parent[i] = i;
        for (int[] s : swaps) unite(s[0], s[1]);

        // group root -> (value -> available count) supplied by `first`
        Map<Integer, Map<Integer, Integer>> supply = new HashMap<>();
        for (int i = 0; i < n; i++) {
            supply.computeIfAbsent(find(i), z -> new HashMap<>())
                  .merge(first[i], 1, Integer::sum);
        }

        int matches = 0;
        for (int i = 0; i < n; i++) {
            Map<Integer, Integer> m = supply.get(find(i));
            Integer c = m.get(second[i]);
            if (c != null && c > 0) { m.put(second[i], c - 1); matches++; }
        }
        return n - matches;
    }
}
C++
CPP
#include <vector>
#include <unordered_map>
using namespace std;

class DSU {
    vector<int> parent, rnk;
public:
    DSU(int n) : parent(n), rnk(n, 0) {
        for (int i = 0; i < n; ++i) parent[i] = i;
    }
    int find(int x) {
        while (parent[x] != x) { parent[x] = parent[parent[x]]; x = parent[x]; }
        return x;
    }
    void unite(int a, int b) {
        int ra = find(a), rb = find(b);
        if (ra == rb) return;
        if (rnk[ra] < rnk[rb]) swap(ra, rb);
        parent[rb] = ra;
        if (rnk[ra] == rnk[rb]) rnk[ra]++;
    }
};

class Solution {
public:
    int minMismatch(vector<int>& first, vector<int>& second, vector<vector<int>>& swaps) {
        int n = first.size();
        DSU dsu(n);
        for (auto& s : swaps) dsu.unite(s[0], s[1]);

        // For each group, count how many of each value `first` can supply.
        unordered_map<int, unordered_map<int,int>> supply;
        for (int i = 0; i < n; ++i) supply[dsu.find(i)][first[i]]++;

        int matches = 0;
        for (int i = 0; i < n; ++i) {
            auto& m = supply[dsu.find(i)];
            auto it = m.find(second[i]);
            if (it != m.end() && it->second > 0) { it->second--; ++matches; }
        }
        return n - matches;
    }
};
C
C
#include <stdlib.h>

static int* g_parent;
static int* g_rank;

static int dsuFind(int x) {
    while (g_parent[x] != x) {
        g_parent[x] = g_parent[g_parent[x]]; /* path halving */
        x = g_parent[x];
    }
    return x;
}

static void dsuUnite(int a, int b) {
    int ra = dsuFind(a), rb = dsuFind(b);
    if (ra == rb) return;
    if (g_rank[ra] < g_rank[rb]) { int t = ra; ra = rb; rb = t; }
    g_parent[rb] = ra;
    if (g_rank[ra] == g_rank[rb]) g_rank[ra]++;
}

static int cmpLL(const void* p, const void* q) {
    long long a = *(const long long*)p, b = *(const long long*)q;
    return (a > b) - (a < b);
}

int minMismatch(int* first, int firstSize, int* second, int secondSize,
                int** swaps, int swapsSize, int* swapsColSize) {
    int n = firstSize;
    g_parent = (int*)malloc(sizeof(int) * n);
    g_rank   = (int*)calloc(n, sizeof(int));
    for (int i = 0; i < n; i++) g_parent[i] = i;
    for (int i = 0; i < swapsSize; i++) dsuUnite(swaps[i][0], swaps[i][1]);

    /* Encode (group root, value) as one sortable key; BASE exceeds any value. */

#### cons
t long long BASE = 200003LL;
    long long* aKeys = (long long*)malloc(sizeof(long long) * n);
    long long* bKeys = (long long*)malloc(sizeof(long long) * n);
    for (int i = 0; i < n; i++) {
        long long root = dsuFind(i);
        aKeys[i] = root * BASE + first[i];
        bKeys[i] = root * BASE + second[i];
    }
    qsort(aKeys, n, sizeof(long long), cmpLL);
    qsort(bKeys, n, sizeof(long long), cmpLL);

    /* Merge the two sorted key lists; per identical key, matches += min(counts). */
    int matches = 0, i = 0, j = 0;
    while (i < n && j < n) {
        if (aKeys[i] == bKeys[j]) {
            long long key = aKeys[i];
            int ca = 0, cb = 0;
            while (i < n && aKeys[i] == key) { ca++; i++; }
            while (j < n && bKeys[j] == key) { cb++; j++; }
            matches += ca < cb ? ca : cb;
        } else if (aKeys[i] < bKeys[j]) {
            i++;
        } else {
            j++;
        }
    }

    free(g_parent); free(g_rank); free(aKeys); free(bKeys);
    return n - matches;
}

#### Follow-up
questions
Prove why matching by value within each component is optimal — why can't crossing a component boundary ever help?
If the swaps arrived as a stream and you had to answer the minimum mismatch after each new swap, how would you maintain it incrementally?
Change the objective to actually output one optimal arrangement of first, not just the count. How do you reconstruct it?
Analyze memory and speed if values could be as large as 10^9 or negative — would you switch from hashing to coordinate compression or sorting?

