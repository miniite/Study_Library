---
type: "practice"
difficulty: "Other"
---

# Most Frequent Category Along a Dependency Path

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Topological Sort](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Topological_Sort.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given a directed graph that models a set of workstations connected by one-way dependency links. The graph has n workstations numbered from 0 to n - 1 and m directed links.

Each workstation carries a single category tag, given as a string labels where labels[i] is a lowercase English letter denoting the category of workstation i (0-indexed).

The links are given as a 2D array links, where links[j] = [u, v] means there is a directed link from workstation u to workstation v.

A route in the graph is a sequence of workstations x1 -> x2 -> x3 -> ... -> xk such that there is a directed link from x(i) to x(i+1) for every 1 <= i < k. A single workstation with no links is itself a route of length 1.

The score of a route is the number of workstations on that route that share the most frequently occurring category among all workstations on the route.

Return the largest score achievable by any valid route in the graph. If the graph contains a cycle (so that some route could repeat workstations indefinitely), return -1.

Input

labels: a string of n lowercase letters.
links: a list of m pairs [u, v] of directed links.

Output

An integer: the maximum route score, or -1 if the graph has a cycle.

#### Cons
traints

n == labels.length
m == links.length
1 <= n <= 10^5
0 <= m <= 10^5
labels consists only of lowercase English letters.
0 <= u, v < n
The same directed link may appear more than once and self-links [u, u] are allowed (a self-link is a cycle).

#### Example
s & test cases
TEXT
Input: labels = "xyxzx", links = [[0,1],[0,2],[2,3],[3,4]] -> Output: 3
Why: The route 0 -> 2 -> 3 -> 4 visits categories x, x, z, x; category 'x' appears 3 times, the maximum over all routes.

Input: labels = "a", links = [[0,0]] -> Output: -1
Why: The self-link 0 -> 0 forms a cycle, so no finite maximum exists.

Input: labels = "w", links = [] -> Output: 1
Why: A single isolated workstation is a route of length 1; its own category is counted once.

Input: labels = "bb", links = [[0,1]] -> Output: 2
Why: The route 0 -> 1 has two workstations both tagged 'b', giving a score of 2.

Input: labels = "abc", links = [[0,1],[1,2],[2,0]] -> Output: -1
Why: The three links form a directed cycle 0 -> 1 -> 2 -> 0.

Input: labels = "aabbb", links = [[0,1],[1,2],[2,3],[3,4]] -> Output: 3
Why: The full route 0 -> 1 -> 2 -> 3 -> 4 has categories a, a, b, b, b; 'b' occurs 3 times, beating 'a' (2).

#### Approach

The answer is undefined when a directed cycle exists, so the algorithm doubles as a cycle detector by using Kahn's topological sort. Build the adjacency list and in-degrees, then repeatedly remove nodes with in-degree 0. For each workstation u keep a length-26 array count[u], where count[u][c] is the maximum number of category-c stations over every route that ends at u, counting u itself. When u is finalized (all its predecessors already processed), first add 1 to count[u] for u's own category; that finalized bucket is the best route value ending at u, so update the global answer. Then push count[u] forward: for each successor v, take the element-wise maximum into count[v]. Because Kahn's order guarantees every predecessor of v is processed before v, count[v] is correct once v is dequeued. If fewer than n nodes are ever dequeued, some nodes lie on a cycle and the answer is -1. Pattern: DAG dynamic programming layered on a topological ordering (Kahn's algorithm) with a fixed-size frequency vector per node.

Complexity

Time: O((n + m) * 26). Each workstation is dequeued once and each link is relaxed once; every relaxation compares all 26 categories, so the alphabet factor is constant. Space: O(n * 26 + m) for the per-node category counts and the adjacency structure.

Python
PYTHON
from collections import deque

def largest_path_value(labels: str, links: list[list[int]]) -> int:
    n = len(labels)
    adj = [[] for _ in range(n)]
    indeg = [0] * n
    for u, v in links:
        adj[u].append(v)
        indeg[v] += 1

    # count[i][c] = max count of category c on any route ending at i (including i).
    count = [[0] * 26 for _ in range(n)]
    dq = deque(i for i in range(n) if indeg[i] == 0)

    visited = 0
    best = 0
    while dq:
        u = dq.popleft()
        visited += 1
        cu = ord(labels[u]) - 97
        count[u][cu] += 1                  # include u's own category
        if count[u][cu] > best:
            best = count[u][cu]
        cu_row = count[u]
        for v in adj[u]:
            cv_row = count[v]
            for c in range(26):
                if cu_row[c] > cv_row[c]:
                    cv_row[c] = cu_row[c]
            indeg[v] -= 1
            if indeg[v] == 0:
                dq.append(v)

    return best if visited == n else -1
JavaScript
JAVASCRIPT
function largestPathValue(labels, links) {

#### cons
t n = labels.length;

#### cons
t adj = Array.from({ length: n }, () => []);

#### cons
t indeg = new Array(n).fill(0);
    for (const [u, v] of links) {
        adj[u].push(v);
        indeg[v]++;
    }

    // count[i][c] = max count of category c on any route ending at i (including i).

#### cons
t count = Array.from({ length: n }, () => new Int32Array(26));

#### cons
t queue = [];
    for (let i = 0; i < n; i++) if (indeg[i] === 0) queue.push(i);

    let visited = 0, best = 0, head = 0;
    while (head < queue.length) {

#### cons
t u = queue[head++];
        visited++;

#### cons
t cu = labels.charCodeAt(u) - 97;
        count[u][cu]++;                     // include u's own category
        if (count[u][cu] > best) best = count[u][cu];
        for (const v of adj[u]) {

#### cons
t cuRow = count[u], cvRow = count[v];
            for (let c = 0; c < 26; c++)
                if (cuRow[c] > cvRow[c]) cvRow[c] = cuRow[c];
            if (--indeg[v] === 0) queue.push(v);
        }
    }
    return visited === n ? best : -1;
}
Java
JAVA
import java.util.*;

class Solution {
    public int largestPathValue(String labels, int[][] links) {
        int n = labels.length();
        List<List<Integer>> adj = new ArrayList<>();
        for (int i = 0; i < n; i++) adj.add(new ArrayList<>());
        int[] indeg = new int[n];
        for (int[] e : links) {
            adj.get(e[0]).add(e[1]);
            indeg[e[1]]++;
        }

        // count[i][c] = max count of category c on any route ending at i (including i).
        int[][] count = new int[n][26];
        Deque<Integer> queue = new ArrayDeque<>();
        for (int i = 0; i < n; i++)
            if (indeg[i] == 0) queue.offer(i);

        int visited = 0, best = 0;
        while (!queue.isEmpty()) {
            int u = queue.poll();
            visited++;
            int cu = labels.charAt(u) - 'a';
            count[u][cu]++;                 // include u's own category
            best = Math.max(best, count[u][cu]);
            for (int v : adj.get(u)) {
                for (int c = 0; c < 26; c++)
                    count[v][c] = Math.max(count[v][c], count[u][c]);
                if (--indeg[v] == 0) queue.offer(v);
            }
        }
        return visited == n ? best : -1;
    }
}
C++
CPP
#include <string>
#include <vector>
#include <array>
#include <queue>
#include <algorithm>
using namespace std;

int largestPathValue(string labels, vector<vector<int>>& links) {
    int n = (int)labels.size();
    vector<vector<int>> adj(n);
    vector<int> indeg(n, 0);
    for (auto& e : links) {
        adj[e[0]].push_back(e[1]);
        indeg[e[1]]++;
    }

    // count[i][c] = max count of category c on any route ending at i (including i).
    vector<array<int, 26>> count(n);
    for (auto& row : count) row.fill(0);

    queue<int> q;
    for (int i = 0; i < n; i++)
        if (indeg[i] == 0) q.push(i);

    int visited = 0, best = 0;
    while (!q.empty()) {
        int u = q.front(); q.pop();
        visited++;
        int cu = labels[u] - 'a';
        count[u][cu]++;                    // include u's own category
        best = max(best, count[u][cu]);
        for (int v : adj[u]) {
            for (int c = 0; c < 26; c++)
                count[v][c] = max(count[v][c], count[u][c]);
            if (--indeg[v] == 0) q.push(v);
        }
    }
    return visited == n ? best : -1;
}
C
C
#include <stdlib.h>
#include <string.h>

// Returns the largest route score, or -1 if the graph contains a cycle.
int largestPathValue(char* labels, int** links, int linksSize, int* linksColSize) {
    int n = (int)strlen(labels);
    int m = linksSize;

    // Adjacency stored as a head/next linked list over the edge indices.
    int* head  = malloc(sizeof(int) * n);
    int* nxt   = malloc(sizeof(int) * (m > 0 ? m : 1));
    int* dest  = malloc(sizeof(int) * (m > 0 ? m : 1));
    int* indeg = calloc(n, sizeof(int));
    for (int i = 0; i < n; i++) head[i] = -1;
    for (int j = 0; j < m; j++) {
        int u = links[j][0], v = links[j][1];
        dest[j] = v;
        nxt[j] = head[u];
        head[u] = j;
        indeg[v]++;
    }

    // count[i * 26 + c] = max number of category-c stations on any route ending at i (incl. i).
    int* count = calloc((long)n * 26, sizeof(int));
    int* queue = malloc(sizeof(int) * n);
    int qh = 0, qt = 0;
    for (int i = 0; i < n; i++)
        if (indeg[i] == 0) queue[qt++] = i;

    int visited = 0, best = 0;
    while (qh < qt) {
        int u = queue[qh++];
        visited++;
        int cu = labels[u] - 'a';
        long ubase = (long)u * 26;
        count[ubase + cu]++;               // include u's own category
        if (count[ubase + cu] > best) best = count[ubase + cu];
        for (int e = head[u]; e != -1; e = nxt[e]) {
            int v = dest[e];
            long vbase = (long)v * 26;
            for (int c = 0; c < 26; c++)
                if (count[ubase + c] > count[vbase + c])
                    count[vbase + c] = count[ubase + c];
            if (--indeg[v] == 0) queue[qt++] = v;
        }
    }

    int ans = (visited == n) ? best : -1;
    free(head); free(nxt); free(dest); free(indeg); free(count); free(queue);
    return ans;
}

#### Follow-up
questions
If the graph is guaranteed acyclic, can you drop the cycle check and simplify to a plain topological DP?
Instead of the best score, reconstruct and return one actual route that achieves it.
Memory is tight for very large n; can you avoid materializing a full 26-int array for every node, e.g., by processing categories independently?
How would you adapt this if categories were arbitrary integers (not just 26 letters) rather than a small fixed alphabet?

