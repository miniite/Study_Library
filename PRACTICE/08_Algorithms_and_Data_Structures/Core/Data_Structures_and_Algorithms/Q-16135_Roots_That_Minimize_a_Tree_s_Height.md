---
type: "practice"
difficulty: "Other"
---

# Roots That Minimize a Tree's Height

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Breadth-First Search](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Breadth-First_Search.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

A tree is an undirected graph in which every pair of vertices is joined by exactly one path — equivalently, a connected graph with no cycles.

You are given an integer n and a list connections, describing a tree with n vertices labelled 0 to n - 1. There are exactly n - 1 entries in connections; each connections[i] = [a, b] means there is an undirected edge between vertex a and vertex b.

You may pick any vertex x and treat it as the root. Rooting the tree at x gives the tree a height: the number of edges on the longest downward path from the root to any leaf. Different choices of root generally give different heights.

Among all n possible root choices, some produce the minimum possible height. Return the list of labels of all vertices that achieve this minimum height. You may return the labels in any order.

Input
n: the number of vertices.
connections: a list of n - 1 edges, each a pair of distinct vertex labels.
Output
A list of all vertex labels whose rooted tree has the minimum height. There will always be either one or two such labels.

#### Cons
traints
1 <= n <= 2 * 10^4
connections.length == n - 1
0 <= a, b < n and a != b for every edge
All edge pairs are distinct.
The input is guaranteed to form a valid tree (connected, acyclic, no repeated edges).

#### Example
s & test cases
TEXT
Input: n = 4, connections = [[0,1],[1,2],[2,3]] -> Output: [1,2]
Why: This is a straight chain 0-1-2-3. Rooting at either middle vertex (1 or 2) gives height 2, the minimum; rooting at an end gives height 3. Even-length diameter yields two centers.

Input: n = 6, connections = [[0,1],[0,2],[0,3],[3,4],[4,5]] -> Output: [3]
Why: The longest path runs 1-0-3-4-5 (four edges). Its single midpoint is vertex 3, which minimizes height; a unique center exists here.

Input: n = 1, connections = [] -> Output: [0]
Why: A single vertex is its own root with height 0; it is trivially the only minimum-height root.

Input: n = 2, connections = [[0,1]] -> Output: [0,1]
Why: With two vertices, rooting at either gives height 1, so both are minimum-height roots.

Input: n = 5, connections = [[0,1],[0,2],[0,3],[0,4]] -> Output: [0]
Why: A star centered at 0. Rooting at 0 gives height 1 (best); rooting at any leaf gives height 2. The hub is the unique center.

Input: n = 5, connections = [[0,1],[1,2],[2,3],[3,4]] -> Output: [2]
Why: A chain of five vertices 0-1-2-3-4 has an odd-length diameter, so its single midpoint vertex 2 is the unique minimum-height root.

#### Approach

The minimum-height roots are exactly the center(s) of the tree — the one or two vertices in the middle of any longest path (diameter). Rather than root the tree at every vertex, we peel the tree inward layer by layer, like removing the outer ring of an onion. Every current leaf (degree 1) is the endpoint of some longest path, so it can never be a center; we remove all leaves simultaneously, decrement their neighbors' degrees, and any neighbor that drops to degree 1 becomes a leaf for the next round. We repeat until only one or two vertices remain — those survivors are the centers and thus the minimum-height roots. This is a topological-sort / BFS-from-the-boundary technique on a tree. A tree always has exactly one center (odd diameter) or two adjacent centers (even diameter), which is why the answer size is always 1 or 2. The n <= 2 cases are handled directly.

Complexity

Time: O(n) — building adjacency is O(n) (there are n-1 edges), and the trimming visits each vertex and each edge a constant number of times. Space: O(n) — the adjacency lists, degree array, and leaf queue are all linear in the number of vertices.

Python
PYTHON
from typing import List
from collections import deque

class Solution:
    def findMinHeightRoots(self, n: int, connections: List[List[int]]) -> List[int]:
        if n <= 2:
            return list(range(n))

        adj = [[] for _ in range(n)]
        degree = [0] * n
        for a, b in connections:
            adj[a].append(b)
            adj[b].append(a)
            degree[a] += 1
            degree[b] += 1

        leaves = deque(i for i in range(n) if degree[i] == 1)
        remaining = n

        while remaining > 2:
            layer = len(leaves)
            remaining -= layer
            for _ in range(layer):
                leaf = leaves.popleft()
                for nb in adj[leaf]:
                    degree[nb] -= 1
                    if degree[nb] == 1:
                        leaves.append(nb)

        return list(leaves)
JavaScript
JAVASCRIPT
function findMinHeightRoots(n, connections) {
    if (n <= 2) {

#### cons
t res = [];
        for (let i = 0; i < n; i++) res.push(i);
        return res;
    }

#### cons
t adj = Array.from({ length: n }, () => []);

#### cons
t degree = new Array(n).fill(0);
    for (const [a, b] of connections) {
        adj[a].push(b);
        adj[b].push(a);
        degree[a]++;
        degree[b]++;
    }

    let leaves = [];
    for (let i = 0; i < n; i++)
        if (degree[i] === 1) leaves.push(i);

    let remaining = n;
    while (remaining > 2) {
        remaining -= leaves.length;

#### cons
t next = [];
        for (const leaf of leaves) {
            for (const nb of adj[leaf]) {
                if (--degree[nb] === 1) next.push(nb);
            }
        }
        leaves = next;
    }

    return leaves;
}
Java
JAVA
import java.util.*;

class Solution {
    public List<Integer> findMinHeightRoots(int n, int[][] connections) {
        if (n <= 2) {
            List<Integer> res = new ArrayList<>();
            for (int i = 0; i < n; i++) res.add(i);
            return res;
        }

        List<List<Integer>> adj = new ArrayList<>();
        for (int i = 0; i < n; i++) adj.add(new ArrayList<>());
        int[] degree = new int[n];
        for (int[] e : connections) {
            adj.get(e[0]).add(e[1]);
            adj.get(e[1]).add(e[0]);
            degree[e[0]]++;
            degree[e[1]]++;
        }

        Queue<Integer> leaves = new ArrayDeque<>();
        for (int i = 0; i < n; i++)
            if (degree[i] == 1) leaves.offer(i);

        int remaining = n;
        while (remaining > 2) {
            int layer = leaves.size();
            remaining -= layer;
            for (int s = 0; s < layer; s++) {
                int leaf = leaves.poll();
                for (int nb : adj.get(leaf))
                    if (--degree[nb] == 1) leaves.offer(nb);
            }
        }

        return new ArrayList<>(leaves);
    }
}
C++
CPP
#include <vector>
#include <queue>
using namespace std;

class Solution {
 public:
  vector<int> findMinHeightRoots(int n, vector<vector<int>>& connections) {
    if (n <= 2) {
      vector<int> res;
      for (int i = 0; i < n; ++i) res.push_back(i);
      return res;
    }

    vector<vector<int>> adj(n);
    vector<int> degree(n, 0);
    for (const auto& e : connections) {
      adj[e[0]].push_back(e[1]);
      adj[e[1]].push_back(e[0]);
      ++degree[e[0]];
      ++degree[e[1]];
    }

    queue<int> leaves;
    for (int i = 0; i < n; ++i)
      if (degree[i] == 1) leaves.push(i);

    int remaining = n;
    while (remaining > 2) {

#### cons
t int layer = leaves.size();
      remaining -= layer;
      for (int s = 0; s < layer; ++s) {

#### cons
t int leaf = leaves.front();
        leaves.pop();
        for (const int nb : adj[leaf])
          if (--degree[nb] == 1) leaves.push(nb);
      }
    }

    vector<int> ans;
    while (!leaves.empty()) {
      ans.push_back(leaves.front());
      leaves.pop();
    }
    return ans;
  }
};
C
C
#include <stdlib.h>

/*
 * Returns the labels of all minimum-height roots (tree centers).
 * connectionsColSize is unused (each edge has 2 endpoints). Sets *returnSize.
 */
int* findMinHeightRoots(int n, int** connections, int connectionsSize,
                        int* connectionsColSize, int* returnSize) {
    if (n <= 2) {
        int* res = (int*)malloc((n > 0 ? n : 1) * sizeof(int));
        for (int i = 0; i < n; ++i) res[i] = i;
        *returnSize = n;
        return res;
    }

    /* Build a CSR adjacency structure and a degree array. */
    int* degree = (int*)calloc(n, sizeof(int));
    for (int i = 0; i < connectionsSize; ++i) {
        degree[connections[i][0]]++;
        degree[connections[i][1]]++;
    }
    int* head = (int*)malloc((n + 1) * sizeof(int));
    head[0] = 0;
    for (int i = 0; i < n; ++i) head[i + 1] = head[i] + degree[i];
    int* pos = (int*)malloc(n * sizeof(int));
    for (int i = 0; i < n; ++i) pos[i] = head[i];
    int* adj = (int*)malloc((size_t)2 * connectionsSize * sizeof(int));
    for (int i = 0; i < connectionsSize; ++i) {
        int a = connections[i][0], b = connections[i][1];
        adj[pos[a]++] = b;
        adj[pos[b]++] = a;
    }

    /* Working degree copy for leaf trimming. */
    int* deg = (int*)malloc(n * sizeof(int));
    for (int i = 0; i < n; ++i) deg[i] = degree[i];

    int* queue = (int*)malloc(n * sizeof(int));
    int qh = 0, qt = 0;
    for (int i = 0; i < n; ++i)
        if (deg[i] == 1) queue[qt++] = i;

    int remaining = n;
    while (remaining > 2) {
        int layer = qt - qh;
        remaining -= layer;
        for (int s = 0; s < layer; ++s) {
            int leaf = queue[qh++];
            for (int k = head[leaf]; k < head[leaf + 1]; ++k) {
                int nb = adj[k];
                if (--deg[nb] == 1) queue[qt++] = nb;
            }
        }
    }

    int cnt = qt - qh;
    int* res = (int*)malloc((cnt > 0 ? cnt : 1) * sizeof(int));
    for (int i = 0; i < cnt; ++i) res[i] = queue[qh + i];
    *returnSize = cnt;

    free(degree); free(head); free(pos); free(adj); free(deg); free(queue);
    return res;
}

#### Follow-up
questions
Prove that a tree has at most two centers, and that when there are two they are adjacent.
Return the actual minimum height along with the center labels.
If the graph might not be a tree (could have cycles or be disconnected), how would you validate the input first?
Solve it instead by two BFS/DFS passes to find the diameter and take its midpoint(s); compare the trade-offs versus leaf-trimming.

