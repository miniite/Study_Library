---
type: "practice"
difficulty: "Other"
---

# Count Fully Meshed Device Clusters

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Union Find](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Union_Find.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

A data center contains deviceCount machines, labeled from 0 to deviceCount - 1. Some pairs of machines are joined by a bidirectional network cable. You are given a list cables, where each entry cables[i] = [u, v] means there is a cable directly connecting machine u and machine v. Cables are two-way, no cable connects a machine to itself, and no pair of machines is joined by more than one cable.

Group the machines into clusters: a cluster is a maximal set of machines such that you can travel from any machine in the set to any other machine in the set by following cables, and no machine in the set is cabled to a machine outside the set. (In graph terms, a cluster is a connected component.)

A cluster is called fully meshed if every pair of machines inside that cluster is joined by a direct cable — there are no missing links. A cluster consisting of a single isolated machine is trivially fully meshed (it has no pairs to connect).

Return the number of fully meshed clusters in the network.

Input
deviceCount: an integer, the number of machines.
cables: a list of pairs [u, v] describing the direct connections.
Output
An integer: the count of clusters in which every pair of machines is directly cabled.

#### Cons
traints
1 <= deviceCount <= 50
0 <= cables.length <= deviceCount * (deviceCount - 1) / 2
cables[i].length == 2
0 <= u, v <= deviceCount - 1
u != v
No pair of machines appears more than once in cables.

#### Example
s & test cases
TEXT
Input: deviceCount = 5, cables = [[0,1],[1,2],[0,2],[3,4]] -> Output: 2
Why: Cluster {0,1,2} has 3 cables and 3*2/2 = 3 possible pairs, so it is fully meshed; cluster {3,4} has 1 cable = 2*1/2, also meshed. Both count.

Input: deviceCount = 6, cables = [[0,1],[0,2],[1,2],[3,4],[3,5]] -> Output: 1
Why: {0,1,2} is fully meshed (3 cables). {3,4,5} has only 2 cables but needs 3 (no cable between 4 and 5), so it is not meshed. Only one cluster qualifies.

Input: deviceCount = 4, cables = [] -> Output: 4
Why: With no cables, each machine is its own isolated cluster; a single-machine cluster is trivially fully meshed, so all 4 count.

Input: deviceCount = 4, cables = [[0,1],[1,2],[2,3],[3,0]] -> Output: 0
Why: One cluster of 4 machines forming a ring has 4 cables but needs 4*3/2 = 6 for a full mesh, so it does not qualify.

Input: deviceCount = 3, cables = [[0,1],[1,2],[0,2]] -> Output: 1
Why: A triangle: 3 machines, 3 cables, every pair connected — a single fully meshed cluster.

Input: deviceCount = 1, cables = [] -> Output: 1
Why: A single isolated machine is a fully meshed cluster on its own (no pairs to satisfy).

#### Approach

A cluster (connected component) with k machines is fully meshed exactly when it contains all k*(k-1)/2 possible cables among its members. So the answer needs, per component: how many machines it holds and how many cables lie inside it. Build the components with a Disjoint Set Union (Union-Find): union the two endpoints of every cable. Then make two linear passes keyed by each element's representative root — one counting machines per root, one counting cables per root (each cable's endpoints share a root). Finally, a component is complete iff its edge count equals nodeCount*(nodeCount-1)/2. The underlying pattern is Union-Find for connected components plus the combinatorial completeness check (a complete graph on k vertices has exactly k choose 2 edges). A DFS/BFS per component that verifies every vertex has degree size - 1 is an equivalent alternative.

Complexity

Time: O((V + E) * alpha(V)), effectively O(V + E), where V = deviceCount and E = cables.length. Union-Find with path compression makes each find/union near-constant (inverse-Ackermann amortized); grouping node and edge counts is linear. Space: O(V) for the parent array and the two per-root count maps.

Python
PYTHON
from collections import defaultdict

def count_fully_meshed_clusters(device_count, cables):
    parent = list(range(device_count))

    def find(x):
        while parent[x] != x:
            parent[x] = parent[parent[x]]  # path halving
            x = parent[x]
        return x

    for a, b in cables:
        ra, rb = find(a), find(b)
        if ra != rb:
            parent[ra] = rb

    node_count = defaultdict(int)
    edge_count = defaultdict(int)
    for v in range(device_count):
        node_count[find(v)] += 1
    for a, _ in cables:
        edge_count[find(a)] += 1

    result = 0
    for root, cnt in node_count.items():
        if edge_count[root] == cnt * (cnt - 1) // 2:
            result += 1
    return result

if __name__ == "__main__":
    print(count_fully_meshed_clusters(6, [[0, 1], [0, 2], [1, 2], [3, 4], [3, 5]]))  # 1
JavaScript
JAVASCRIPT
function countFullyMeshedClusters(deviceCount, cables) {

#### cons
t parent = Array.from({ length: deviceCount }, (_, i) => i);

#### cons
t find = (x) => {
    while (parent[x] !== x) {
      parent[x] = parent[parent[x]]; // path halving
      x = parent[x];
    }
    return x;
  };

  for (const [a, b] of cables) {

#### cons
t ra = find(a), rb = find(b);
    if (ra !== rb) parent[ra] = rb;
  }

#### cons
t nodeCount = new Map();

#### cons
t edgeCount = new Map();
  for (let v = 0; v < deviceCount; v++) {

#### cons
t r = find(v);
    nodeCount.set(r, (nodeCount.get(r) || 0) + 1);
  }
  for (const [a] of cables) {

#### cons
t r = find(a);
    edgeCount.set(r, (edgeCount.get(r) || 0) + 1);
  }

  let result = 0;
  for (const [root, cnt] of nodeCount) {
    if ((edgeCount.get(root) || 0) === (cnt * (cnt - 1)) / 2) result++;
  }
  return result;
}

// console.log(countFullyMeshedClusters(6, [[0,1],[0,2],[1,2],[3,4],[3,5]])); // 1
Java
JAVA
import java.util.*;

class Solution {
    private int[] parent;

    private int findRoot(int x) {
        while (parent[x] != x) { parent[x] = parent[parent[x]]; x = parent[x]; }
        return x;
    }

    public int countFullyMeshedClusters(int deviceCount, int[][] cables) {
        parent = new int[deviceCount];
        for (int i = 0; i < deviceCount; i++) parent[i] = i;

        for (int[] e : cables) {
            int ra = findRoot(e[0]), rb = findRoot(e[1]);
            if (ra != rb) parent[ra] = rb;
        }

        Map<Integer, Long> nodeCount = new HashMap<>();
        Map<Integer, Long> edgeCount = new HashMap<>();
        for (int v = 0; v < deviceCount; v++) nodeCount.merge(findRoot(v), 1L, Long::sum);
        for (int[] e : cables) edgeCount.merge(findRoot(e[0]), 1L, Long::sum);

        int result = 0;
        for (Map.Entry<Integer, Long> en : nodeCount.entrySet()) {
            long cnt = en.getValue();
            long ec = edgeCount.getOrDefault(en.getKey(), 0L);
            if (ec == cnt * (cnt - 1) / 2) result++;
        }
        return result;
    }

    public static void main(String[] args) {
        int[][] cables = {{0,1},{0,2},{1,2},{3,4},{3,5}};
        System.out.println(new Solution().countFullyMeshedClusters(6, cables)); // 1
    }
}
C++
CPP
#include <bits/stdc++.h>
using namespace std;

class Solution {
    vector<int> parent;
    int findRoot(int x) {
        while (parent[x] != x) { parent[x] = parent[parent[x]]; x = parent[x]; }
        return x;
    }
public:
    int countFullyMeshedClusters(int deviceCount, vector<vector<int>>& cables) {
        parent.resize(deviceCount);
        iota(parent.begin(), parent.end(), 0);
        for (auto& e : cables) {
            int ra = findRoot(e[0]), rb = findRoot(e[1]);
            if (ra != rb) parent[ra] = rb;
        }
        unordered_map<int, long long> nodeCount, edgeCount;
        for (int v = 0; v < deviceCount; v++) nodeCount[findRoot(v)]++;
        for (auto& e : cables) edgeCount[findRoot(e[0])]++;

        int result = 0;
        for (auto& [root, cnt] : nodeCount)
            if (edgeCount[root] == cnt * (cnt - 1) / 2) result++;
        return result;
    }
};

int main() {
    vector<vector<int>> cables = {{0,1},{0,2},{1,2},{3,4},{3,5}};
    cout << Solution().countFullyMeshedClusters(6, cables) << "\n"; // 1
}
C
C
#include <stdio.h>
#include <stdlib.h>

static int findRoot(int *parent, int x) {
    while (parent[x] != x) {
        parent[x] = parent[parent[x]]; /* path halving */
        x = parent[x];
    }
    return x;
}

int countFullyMeshedClusters(int deviceCount, int cables[][2], int cableCount) {
    int *parent = malloc(sizeof(int) * deviceCount);
    long *nodeCount = calloc(deviceCount, sizeof(long));
    long *edgeCount = calloc(deviceCount, sizeof(long));
    for (int i = 0; i < deviceCount; i++) parent[i] = i;

    for (int i = 0; i < cableCount; i++) {
        int ra = findRoot(parent, cables[i][0]);
        int rb = findRoot(parent, cables[i][1]);
        if (ra != rb) parent[ra] = rb;
    }

    for (int v = 0; v < deviceCount; v++) nodeCount[findRoot(parent, v)]++;
    for (int i = 0; i < cableCount; i++) edgeCount[findRoot(parent, cables[i][0])]++;

    int result = 0;
    for (int v = 0; v < deviceCount; v++) {
        if (parent[v] == v) { /* v is a cluster root */
            long c = nodeCount[v];
            if (edgeCount[v] == c * (c - 1) / 2) result++;
        }
    }
    free(parent); free(nodeCount); free(edgeCount);
    return result;
}

int main(void) {
    int cables[][2] = {{0,1},{0,2},{1,2},{3,4},{3,5}};
    printf("%d\n", countFullyMeshedClusters(6, cables, 5)); /* 1 */
    return 0;
}

#### Follow-up
questions
Return the size of the largest fully meshed cluster instead of the total count.
The input may contain duplicate cables or self-loops; how do you keep the completeness check correct?
Support online edge insertions and answer "how many complete clusters now?" after each addition efficiently.
How would the approach change for a graph with millions of vertices spread across machines (distributed / streaming edges)?

