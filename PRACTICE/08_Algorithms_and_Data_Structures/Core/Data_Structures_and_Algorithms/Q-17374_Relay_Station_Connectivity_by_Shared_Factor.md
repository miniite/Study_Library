---
type: "practice"
difficulty: "Other"
---

# Relay Station Connectivity by Shared Factor

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Union Find (Disjoint Set Union)](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Union_Find_(Disjoint_Set_Union).md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

There are n relay stations, labeled from 1 to n. A direct fiber link joins two distinct stations a and b if and only if they share a common divisor that is strictly greater than a given gap value g. Formally, a link exists between a and b when there is an integer d such that all of the following hold:

a % d == 0,
b % d == 0, and
d > g.

Two stations are connected if they are joined by a direct link or by a chain of links passing through intermediate stations.

You are given the integers n and g, and a list pairs, where pairs[i] = [a_i, b_i]. For each query, decide whether stations a_i and b_i are connected (directly or indirectly).

Input
n: the number of stations.
g: the gap value; a shared divisor must strictly exceed it to create a link.
pairs: a list of [a, b] queries.
Output
An array answer of the same length as pairs, where answer[i] is true if stations a_i and b_i are connected, and false otherwise.
Notes
The query [a, b] is the same as [b, a], and duplicate queries may appear.
When g == 0, every pair of stations shares the divisor 1 > 0, so all stations are connected.

#### Cons
traints
2 <= n <= 10^4
0 <= g <= n
1 <= pairs.length <= 10^5
pairs[i].length == 2
1 <= a_i, b_i <= n
a_i != b_i

#### Example
s & test cases
TEXT
- Input: n = 8, g = 2, pairs = [[3,6],[4,8],[2,6],[3,8]] -> Output: [true,true,false,false]
  Why: with g=2 only divisors >2 create links. Multiples of 3 link {3,6}; multiples of 4 link {4,8}. So [3,6]=true and [4,8]=true, but station 2 has no divisor >2 so [2,6]=false, and {3,6} and {4,8} are separate so [3,8]=false.
- Input: n = 8, g = 0, pairs = [[1,7],[2,5]] -> Output: [true,true]
  Why: g=0 means the shared divisor only needs to exceed 0, so 1 qualifies and every station is mutually connected.
- Input: n = 8, g = 1, pairs = [[2,8],[3,4],[5,7],[1,2]] -> Output: [true,true,false,false]
  Why: divisors >1 link multiples of 2 into {2,4,6,8} and multiples of 3 into {3,6}; sharing 6 fuses them into {2,3,4,6,8}. So [2,8] and [3,4] are true; 5, 7 and 1 are isolated, so their queries are false.
- Input: n = 5, g = 1, pairs = [[4,5],[3,2],[3,4]] -> Output: [false,false,false]
  Why: only 2 and 4 share a divisor (2) greater than 1; every other pairing is disconnected.
- Input: n = 10, g = 3, pairs = [[4,8],[5,10],[6,9]] -> Output: [true,true,false]
  Why: divisors >3 link multiples of 4 into {4,8} and of 5 into {5,10}, so those two queries are true; but 6 and 9 share only the divisor 3, which is not strictly greater than 3, so [6,9]=false.
- Input: n = 6, g = 6, pairs = [[2,4],[3,6]] -> Output: [false,false]
  Why: no divisor can exceed 6 while still dividing a label in 1..6 that has another multiple, so no links exist and every station is isolated.

#### Approach

Two stations are directly linked exactly when they share a common divisor d > g. Key observation: every station that is a multiple of some d shares that divisor d, so all multiples of a fixed d form one clique and can be merged into a single connected group. Rather than testing pairs, iterate d = g+1, g+2, ..., n and, using a Disjoint Set Union (Union-Find) structure, union d with each of its multiples 2d, 3d, ... up to n. This is a sieve-style pass that captures indirect connectivity transitively (e.g. multiples of 2 and multiples of 3 fuse through 6). After preprocessing, each query is a constant-time check of whether the two labels share a root. When g == 0, the pass starts at d = 1 and unions everything, so all stations become connected — matching the definition since 1 divides every label.

Complexity

Time: O(n log n * α(n) + q), where q = number of queries. For each divisor z from g+1 to n we visit its multiples; the total number of union operations is n/(g+1) + n/(g+2) + ... ≈ O(n log n) (harmonic sum), each near-constant under union by rank with path compression (α is the inverse Ackermann function). Each query is answered by two finds in effectively O(1). Space: O(n) for the disjoint-set arrays.

Python
PYTHON
def are_connected(n, g, pairs):
    parent = list(range(n + 1))
    rank = [0] * (n + 1)

    def find(x):
        while parent[x] != x:
            parent[x] = parent[parent[x]]   # path halving
            x = parent[x]
        return x

    def unite(a, b):
        ra, rb = find(a), find(b)
        if ra == rb:
            return
        if rank[ra] < rank[rb]:
            ra, rb = rb, ra
        parent[rb] = ra
        if rank[ra] == rank[rb]:
            rank[ra] += 1

    for z in range(g + 1, n + 1):
        for m in range(2 * z, n + 1, z):
            unite(z, m)

    return [find(a) == find(b) for a, b in pairs]
JavaScript
JAVASCRIPT
function areConnected(n, g, pairs) {

#### cons
t parent = new Array(n + 1);

#### cons
t rank = new Array(n + 1).fill(0);
    for (let i = 0; i <= n; i++) parent[i] = i;

    function find(x) {
        while (parent[x] !== x) {
            parent[x] = parent[parent[x]];   // path halving
            x = parent[x];
        }
        return x;
    }

    function unite(a, b) {
        let ra = find(a), rb = find(b);
        if (ra === rb) return;
        if (rank[ra] < rank[rb]) { const t = ra; ra = rb; rb = t; }
        parent[rb] = ra;
        if (rank[ra] === rank[rb]) rank[ra]++;
    }

    for (let z = g + 1; z <= n; z++) {
        for (let m = 2 * z; m <= n; m += z) {
            unite(z, m);
        }
    }

    return pairs.map(([a, b]) => find(a) === find(b));
}
Java
JAVA
import java.util.*;

class Solution {
    private int[] parent, rank;

    private int find(int x) {
        while (parent[x] != x) { parent[x] = parent[parent[x]]; x = parent[x]; }
        return x;
    }

    private void unite(int a, int b) {
        int ra = find(a), rb = find(b);
        if (ra == rb) return;
        if (rank[ra] < rank[rb]) { int t = ra; ra = rb; rb = t; }
        parent[rb] = ra;
        if (rank[ra] == rank[rb]) rank[ra]++;
    }

    public List<Boolean> areConnected(int n, int g, int[][] pairs) {
        parent = new int[n + 1];
        rank = new int[n + 1];
        for (int i = 0; i <= n; i++) parent[i] = i;

        for (int z = g + 1; z <= n; z++)
            for (int m = 2 * z; m <= n; m += z)
                unite(z, m);

        List<Boolean> ans = new ArrayList<>(pairs.length);
        for (int[] q : pairs)
            ans.add(find(q[0]) == find(q[1]));
        return ans;
    }
}
C++
CPP
#include <vector>
using namespace std;

class Solution {
    vector<int> parent, rnk;
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
public:
    vector<bool> areConnected(int n, int g, vector<vector<int>>& pairs) {
        parent.resize(n + 1);
        rnk.assign(n + 1, 0);
        for (int i = 0; i <= n; i++) parent[i] = i;

        for (int z = g + 1; z <= n; z++)
            for (int m = 2 * z; m <= n; m += z)
                unite(z, m);

        vector<bool> ans;
        ans.reserve(pairs.size());
        for (auto& q : pairs)
            ans.push_back(find(q[0]) == find(q[1]));
        return ans;
    }
};
C
C
#include <stdlib.h>
#include <stdbool.h>

static int find(int* parent, int x) {
    while (parent[x] != x) { parent[x] = parent[parent[x]]; x = parent[x]; }
    return x;
}

static void unite(int* parent, int* rank, int a, int b) {
    int ra = find(parent, a), rb = find(parent, b);
    if (ra == rb) return;
    if (rank[ra] < rank[rb]) { int t = ra; ra = rb; rb = t; }
    parent[rb] = ra;
    if (rank[ra] == rank[rb]) rank[ra]++;
}

/* returns a malloc'd bool array of length pairsSize; caller frees */
bool* areConnected(int n, int g, int** pairs, int pairsSize, int* returnSize) {
    int* parent = (int*)malloc(sizeof(int) * (n + 1));
    int* rank = (int*)calloc(n + 1, sizeof(int));
    for (int i = 0; i <= n; i++) parent[i] = i;

    for (int z = g + 1; z <= n; z++)
        for (int m = 2 * z; m <= n; m += z)
            unite(parent, rank, z, m);

    bool* ans = (bool*)malloc(sizeof(bool) * pairsSize);
    for (int i = 0; i < pairsSize; i++)
        ans[i] = (find(parent, pairs[i][0]) == find(parent, pairs[i][1]));

    *returnSize = pairsSize;
    free(parent);
    free(rank);
    return ans;
}

#### Follow-up
questions
What changes if a link required a shared prime divisor greater than g? (Union only multiples of each prime > g.)
With up to 10^6 queries, is any per-query work avoidable? (No: each is already O(1) after the one-time DSU build.)
Report the number of connected components, or the size of the largest component, after building the structure.
Handle offline queries where g varies per query — can you sort queries and incrementally add divisor edges?

