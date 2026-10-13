---
type: "practice"
difficulty: "Other"
---

# Fast Kth Ancestor Queries in a Rooted Hierarchy

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Tree](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Tree.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given a rooted hierarchy of n nodes labelled from 0 to n - 1. The structure is described by an integer array parent of length n, where parent[v] is the label of the immediate parent of node v. Node 0 is always the root of the hierarchy, and it has no parent, so parent[0] == -1.

An ancestor path of a node is the sequence of nodes you visit when you repeatedly move from a node to its parent until you reach the root. The k-th ancestor of a node is the node you land on after taking exactly k upward steps along this path. The 1st ancestor of a node is its direct parent, the 2nd ancestor is its grandparent, and so on. If following k steps upward would move past the root (that is, the node does not have k ancestors), the answer is -1.

Design a data structure that is built once from the hierarchy and can then answer many ancestor queries quickly:

LineageTree(int n, int[] parent) — build the structure from the given number of nodes and the parent array.
int queryAncestor(int node, int k) — return the label of the k-th ancestor of node, or -1 if it does not exist.

Because the number of queries can be large, a naive walk of up to k steps per query is too slow in the worst case; you should answer each query in logarithmic time.

#### Cons
traints

1 <= k <= n <= 5 * 10^4
parent.length == n
parent[0] == -1
0 <= parent[i] < n for every i with 0 < i < n
0 <= node < n
The parent array always describes a valid tree rooted at node 0.
There will be at most 5 * 10^4 calls to queryAncestor.

#### Example
s & test cases
TEXT

#### Cons
ider the hierarchy built from `n = 8`, `parent = [-1, 0, 0, 1, 2, 2, 4, 4]`. This gives the tree: node 0 is the root; nodes 1 and 2 are its children; node 3 is a child of 1; nodes 4 and 5 are children of 2; nodes 6 and 7 are children of 4.

- Input: `queryAncestor(6, 1)` -> Output: `4`
  Why: one step up from node 6 is its direct parent, node 4 (normal case).
- Input: `queryAncestor(6, 2)` -> Output: `2`
  Why: two steps up from 6 is 6 -> 4 -> 2 (multi-step climb).
- Input: `queryAncestor(6, 3)` -> Output: `0`
  Why: three steps up reaches the root, 6 -> 4 -> 2 -> 0.
- Input: `queryAncestor(3, 5)` -> Output: `-1`
  Why: node 3 has only 2 ancestors (1 and 0); asking for the 5th walks past the root (edge case).
- Input: `queryAncestor(0, 1)` -> Output: `-1`
  Why: the root has no ancestor at all (root edge case).
- Input: `queryAncestor(7, 3)` -> Output: `0`
  Why: 7 -> 4 -> 2 -> 0, exactly reaching the root on the last step (boundary).

#### Approach

The technique is binary lifting (a sparse-table over ancestors, a form of dynamic programming on trees). Precompute up[j][v], the 2^j-th ancestor of every node v: the base level up[0][v] is the direct parent, and each higher level is composed from the one below via up[j][v] = up[j-1][up[j-1][v]] (jump halfway, then jump again). Any k decomposes into a sum of distinct powers of two given by its binary representation, so a k-step climb is answered by taking one jump for each set bit of k, moving from low bits to high. If a jump ever falls off the top of the tree, the stored value is -1 and it propagates, correctly yielding -1 for missing ancestors. This turns a potentially O(k) walk into an O(log k) query after O(n log n) preprocessing.

Complexity

Preprocessing: time and space O(n log n) to fill the binary-lifting table of log n levels over n nodes. Each query is O(log k) since it consumes at most one jump per set bit of k. This comfortably handles up to 5*10^4 nodes and queries.

Python
PYTHON
class LineageTree:
    def __init__(self, n, parent):
        LOG = 1
        while (1 << LOG) < n:
            LOG += 1
        LOG += 1                                  # headroom
        self.LOG = LOG
        # up[j][v] = (2^j)-th ancestor of v, or -1 if it does not exist
        self.up = [[-1] * n for _ in range(LOG)]
        self.up[0] = list(parent)
        for j in range(1, LOG):
            prev, cur = self.up[j - 1], self.up[j]
            for v in range(n):
                mid = prev[v]
                cur[v] = -1 if mid == -1 else prev[mid]

    def query_ancestor(self, node, k):
        for j in range(self.LOG):
            if node == -1:
                break
            if k & (1 << j):
                node = self.up[j][node]
        return node
JavaScript
JAVASCRIPT
class LineageTree {

#### cons
tructor(n, parent) {
        let LOG = 1;
        while ((1 << LOG) < n) LOG++;
        LOG++;                                   // headroom
        this.LOG = LOG;
        // up[j][v] = (2^j)-th ancestor of v, or -1
        this.up = Array.from({ length: LOG }, () => new Int32Array(n));
        for (let v = 0; v < n; v++) this.up[0][v] = parent[v];
        for (let j = 1; j < LOG; j++) {
            for (let v = 0; v < n; v++) {

#### cons
t mid = this.up[j - 1][v];
                this.up[j][v] = mid === -1 ? -1 : this.up[j - 1][mid];
            }
        }
    }

    queryAncestor(node, k) {
        for (let j = 0; j < this.LOG && node !== -1; j++) {
            if (k & (1 << j)) node = this.up[j][node];
        }
        return node;
    }
}
Java
JAVA
class LineageTree {
    private final int LOG;
    private final int[][] up;   // up[j][v] = (2^j)-th ancestor of v, or -1

    public LineageTree(int n, int[] parent) {
        int lg = 1;
        while ((1 << lg) < n) lg++;
        lg++;                                   // headroom
        LOG = lg;
        up = new int[LOG][n];
        for (int v = 0; v < n; v++) up[0][v] = parent[v];
        for (int j = 1; j < LOG; j++)
            for (int v = 0; v < n; v++) {
                int mid = up[j - 1][v];
                up[j][v] = (mid == -1) ? -1 : up[j - 1][mid];
            }
    }

    public int queryAncestor(int node, int k) {
        for (int j = 0; j < LOG && node != -1; j++)
            if ((k & (1 << j)) != 0) node = up[j][node];
        return node;
    }
}
C++
CPP
#include <vector>
using namespace std;

class LineageTree {
    int LOG;
    vector<vector<int>> up;   // up[j][v] = (2^j)-th ancestor of v, or -1
public:
    LineageTree(int n, vector<int>& parent) {
        LOG = 1;
        while ((1 << LOG) < n) ++LOG;
        ++LOG;                                  // headroom
        up.assign(LOG, vector<int>(n));
        for (int v = 0; v < n; ++v) up[0][v] = parent[v];
        for (int j = 1; j < LOG; ++j)
            for (int v = 0; v < n; ++v) {
                int mid = up[j - 1][v];
                up[j][v] = (mid == -1) ? -1 : up[j - 1][mid];
            }
    }

    int queryAncestor(int node, int k) {
        for (int j = 0; j < LOG && node != -1; ++j)
            if (k & (1 << j)) node = up[j][node];
        return node;
    }
};
C
C
#include <stdlib.h>

/* Binary-lifting table: up[j][v] = the (2^j)-th ancestor of v, or -1 if none. */
typedef struct {
    int n;
    int LOG;
    int **up;
} LineageTree;

LineageTree* lineageTreeCreate(int n, int* parent) {
    LineageTree* t = (LineageTree*)malloc(sizeof(LineageTree));
    t->n = n;
    int LOG = 1;
    while ((1 << LOG) < n) LOG++;
    LOG++;                       /* headroom so k up to n fits in the bits */
    t->LOG = LOG;
    t->up = (int**)malloc(sizeof(int*) * LOG);
    for (int j = 0; j < LOG; j++)
        t->up[j] = (int*)malloc(sizeof(int) * n);

    for (int v = 0; v < n; v++)  /* base level: direct parent */
        t->up[0][v] = parent[v];

    for (int j = 1; j < LOG; j++) {
        for (int v = 0; v < n; v++) {
            int mid = t->up[j - 1][v];           /* 2^(j-1)-th ancestor */
            t->up[j][v] = (mid == -1) ? -1 : t->up[j - 1][mid];
        }
    }
    return t;
}

int lineageTreeQueryAncestor(LineageTree* t, int node, int k) {
    for (int j = 0; j < t->LOG && node != -1; j++)
        if (k & (1 << j))                        /* consume this power-of-two jump */
            node = t->up[j][node];
    return node;
}

void lineageTreeFree(LineageTree* t) {
    for (int j = 0; j < t->LOG; j++) free(t->up[j]);
    free(t->up);
    free(t);
}

#### Follow-up
questions
Extend the structure to answer the lowest common ancestor (LCA) of two nodes using the same table plus depth balancing.
Support the k-th node on the path between two arbitrary nodes, not just toward the root.
Handle a dynamic tree where new leaves are appended over time — can you keep queries logarithmic?
Reduce memory: discuss when an Euler-tour + sparse-table RMQ or heavy-path decomposition would be preferable to O(n log n) storage.

