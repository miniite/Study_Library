---
type: "practice"
difficulty: "Other"
---

# Highest Combined Connectivity of a Station Pair

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Graph](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Graph.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given a communication backbone made up of n relay stations, labelled from 0 to n - 1. The stations are joined by a set of two-way cables described by the array links, where each links[i] = [u, v] means there is a single bidirectional cable directly connecting station u and station v.

For any two distinct stations x and y, define their combined connectivity as the total number of cables that are directly attached to either station. A cable that runs directly between x and y touches both stations, but it must be counted only once.

The peak connectivity of the whole backbone is the largest combined connectivity taken over every possible pair of distinct stations.

Given the integer n and the array links, return the peak connectivity of the backbone.

Note that the backbone does not have to be fully connected — some stations may sit in separate groups, and a station may have no cables at all.

Input

An integer n: the number of relay stations.
A 2D integer array links: each element is a pair [u, v] describing one cable.

Output

A single integer: the peak combined connectivity over all pairs of distinct stations.

#### Cons
traints

2 <= n <= 100
0 <= links.length <= n * (n - 1) / 2
links[i].length == 2
0 <= u, v <= n - 1
u != v (no cable connects a station to itself)
Every pair of stations is connected by at most one cable (no duplicate cables).

#### Example
s & test cases
TEXT
Input: n = 4, links = [[0,1],[0,2],[0,3]] -> Output: 3
Why: Station 0 has 3 cables; pairing it with any leaf gives 3 + 1 - 1 (shared cable) = 3.

Input: n = 5, links = [[0,1],[0,2],[1,2],[1,3],[2,4]] -> Output: 5
Why: Stations 1 (degree 3) and 2 (degree 3) share one cable, so 3 + 3 - 1 = 5, the maximum.

Input: n = 3, links = [] -> Output: 0
Why: Edge case with no cables at all; every station has degree 0, so every pair scores 0.

Input: n = 2, links = [[0,1]] -> Output: 1
Why: Minimum backbone. The single shared cable is counted once: 1 + 1 - 1 = 1.

Input: n = 6, links = [[0,1],[1,2],[3,4],[4,5]] -> Output: 4
Why: Two separate groups; best pair is stations 1 and 4 (degree 2 each, no shared cable): 2 + 2 - 0 = 4, showing stations need not be connected.

#### Approach

The combined connectivity of a pair (x, y) equals degree(x) + degree(y), minus 1 if a cable runs directly between them (that cable was counted in both degrees but should count once). So precompute each station's degree and a lookup for whether an edge exists between two stations, then brute-force every distinct pair taking the maximum of deg[x] + deg[y] - hasEdge(x, y). This is the "count degrees, then adjust for shared edge over all pairs" pattern. (An O(n) refinement exists — track only the two largest degrees and how many top-degree stations there are — but n <= 100 makes the O(n^2) scan the clean, safe choice.)

Complexity

Time: O(n^2) — first count degrees in O(links) then scan all O(n^2) distinct pairs; with n <= 100 this is trivially fast. Space: O(n^2) for the adjacency matrix (a degree array plus an edge set would reduce this to O(n + links) if desired).

Python
PYTHON
from typing import List

def peak_connectivity(n: int, links: List[List[int]]) -> int:
    deg = [0] * n
    adj = [[False] * n for _ in range(n)]
    for u, v in links:
        deg[u] += 1
        deg[v] += 1
        adj[u][v] = True
        adj[v][u] = True

    best = 0
    for i in range(n):
        for j in range(i + 1, n):
            rank = deg[i] + deg[j] - (1 if adj[i][j] else 0)
            if rank > best:
                best = rank
    return best
JavaScript
JAVASCRIPT
function peakConnectivity(n, links) {

#### cons
t deg = new Array(n).fill(0);

#### cons
t adj = Array.from({ length: n }, () => new Array(n).fill(false));
    for (const [u, v] of links) {
        deg[u]++;
        deg[v]++;
        adj[u][v] = true;
        adj[v][u] = true;
    }
    let best = 0;
    for (let i = 0; i < n; i++) {
        for (let j = i + 1; j < n; j++) {

#### cons
t rank = deg[i] + deg[j] - (adj[i][j] ? 1 : 0);
            if (rank > best) best = rank;
        }
    }
    return best;
}
Java
JAVA
class Solution {
    public int peakConnectivity(int n, int[][] links) {
        int[] deg = new int[n];
        boolean[][] adj = new boolean[n][n];
        for (int[] e : links) {
            int u = e[0], v = e[1];
            deg[u]++;
            deg[v]++;
            adj[u][v] = true;
            adj[v][u] = true;
        }
        int best = 0;
        for (int i = 0; i < n; ++i) {
            for (int j = i + 1; j < n; ++j) {
                int rank = deg[i] + deg[j] - (adj[i][j] ? 1 : 0);
                if (rank > best) best = rank;
            }
        }
        return best;
    }
}
C++
CPP
#include <vector>
using namespace std;

class Solution {
public:
    int peakConnectivity(int n, vector<vector<int>>& links) {
        vector<int> deg(n, 0);
        vector<vector<bool>> adj(n, vector<bool>(n, false));
        for (auto& e : links) {
            int u = e[0], v = e[1];
            deg[u]++;
            deg[v]++;
            adj[u][v] = adj[v][u] = true;
        }
        int best = 0;
        for (int i = 0; i < n; ++i) {
            for (int j = i + 1; j < n; ++j) {
                int rank = deg[i] + deg[j] - (adj[i][j] ? 1 : 0);
                if (rank > best) best = rank;
            }
        }
        return best;
    }
};
C
C
#include <stddef.h>

int peakConnectivity(int n, int** links, int linksSize, int* linksColSize) {
    int deg[100] = {0};
    int adj[100][100] = {{0}};
    for (int i = 0; i < linksSize; ++i) {
        int u = links[i][0], v = links[i][1];
        deg[u]++;
        deg[v]++;
        adj[u][v] = 1;
        adj[v][u] = 1;
    }
    int best = 0;
    for (int i = 0; i < n; ++i) {
        for (int j = i + 1; j < n; ++j) {
            int rank = deg[i] + deg[j] - adj[i][j];
            if (rank > best) best = rank;
        }
    }
    return best;
}

#### Follow-up
questions
What changes if a station could be linked to itself, or if duplicate cables between the same pair were allowed?
Can you achieve O(n + links) time and O(n) space instead of building the full adjacency matrix?
How would you also return the actual pair of stations that achieves the peak connectivity?
If the graph were enormous and sparse, which stations must the optimal pair come from, and why do the top two degrees (with a tie-break) suffice?

