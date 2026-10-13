---
type: "practice"
difficulty: "Other"
---

# Is the Given Order the Only Shortest Supersequence

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Topological Sort](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Topological_Sort.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given an integer array order of length n that is a permutation of the integers 1, 2, ..., n. You are also given a list fragments, where each fragments[i] is a subsequence of order.

A supersequence of fragments is any sequence that contains every fragment as a subsequence (a subsequence keeps the relative order of its elements but may skip elements in between). A shortest supersequence is one of minimum possible length. In general there may be more than one shortest supersequence.

Your task: decide whether order is the one and only shortest supersequence of fragments. Return true if order is uniquely the shortest supersequence, and false otherwise.

Intuition: each fragment [a, b, c, ...] asserts the relative order a before b before c .... Collecting the adjacent "before" relations across all fragments defines a set of ordering constraints among the values 1..n. order is the unique shortest supersequence exactly when those constraints pin down a single total ordering of all n values, and that ordering is precisely order. If at any point two different values could each come next, more than one shortest supersequence exists.

Input:

order: a permutation of 1..n.
fragments: a list of integer arrays, each a subsequence of order.

Output: a boolean — true if order is the unique shortest supersequence of all fragments, else false.

#### Cons
traints:

n == order.length
1 <= n <= 10^4
order is a permutation of every integer in [1, n].
1 <= fragments.length <= 10^4
1 <= fragments[i].length <= 10^4
1 <= sum of all fragments[i].length <= 10^5
1 <= fragments[i][j] <= n
All arrays in fragments are distinct.
Each fragments[i] is a subsequence of order.

#### Example
s & test cases
TEXT
Input: order = [3,1,2], fragments = [[3,1],[3,2]] -> Output: false
Why: the constraints fix that 3 comes first but leave 1 and 2's relative order free, so both [3,1,2] and [3,2,1] are shortest supersequences -> not unique.

Input: order = [3,1,2], fragments = [[3,1]] -> Output: false
Why: the only constraint is 3 before 1; the shortest supersequence is just [3,1], so the length-3 order is not even shortest.

Input: order = [3,1,2], fragments = [[3,1],[3,2],[1,2]] -> Output: true
Why: constraints 3->1, 3->2, 1->2 force exactly one total order [3,1,2].

Input: order = [1,2,3,4], fragments = [[1,2],[2,3],[3,4]] -> Output: true
Why: the chained constraints 1->2->3->4 admit a single ordering equal to order.

Input: order = [1,2,3,4], fragments = [[1,2],[3,4]] -> Output: false
Why: nothing ties the {1,2} pair to the {3,4} pair, so at the start both 1 and 3 could come first -> multiple shortest supersequences.

Input: order = [2,1], fragments = [[2,1]] -> Output: true
Why: the single constraint 2 before 1 uniquely determines [2,1].

#### Approach

Model each fragment's adjacent elements as directed "must come before" edges over the values 1..n, forming a constraint graph. Run Kahn's topological sort: seed a queue with every in-degree-0 node. order is the unique shortest supersequence iff two conditions hold at every step: (1) the queue holds exactly one node — if it ever holds two or more, two different values could legally come next, so multiple shortest supersequences exist; and (2) that single node matches the corresponding element of order. Peeling nodes one at a time and decrementing successors' in-degrees, we must consume all n nodes in exactly the sequence order. If the traversal stalls before covering all n values (a cycle) or ever offers a choice or a mismatch, the answer is false. (Note: a purely position-based endpoint check is insufficient — it verifies each fragment is a subsequence of order but not that order is the unique shortest supersequence, which is why the topological-uniqueness test is required.)

Complexity

Time: O(V + E) where V = n and E = sum over fragments of (length - 1) — building the graph touches every adjacent pair once, and Kahn's traversal visits each node once and relaxes each edge once. Space: O(V + E) for the adjacency lists, in-degree array, and queue.

Python
PYTHON
from typing import List
from collections import deque

class Solution:
    def sequenceReconstruction(self, order: List[int], fragments: List[List[int]]) -> bool:
        if not fragments:
            return False

        n = len(order)
        graph = [[] for _ in range(n)]
        in_degree = [0] * n

        # Build the constraint graph from adjacent pairs in each fragment.
        for frag in fragments:
            for u, v in zip(frag, frag[1:]):
                graph[u - 1].append(v - 1)
                in_degree[v - 1] += 1

        q = deque(i for i, d in enumerate(in_degree) if d == 0)
        idx = 0

        while q:
            if len(q) > 1:            # more than one choice -> not unique
                return False
            u = q.popleft()
            if u != order[idx] - 1:
                return False
            idx += 1
            for v in graph[u]:
                in_degree[v] -= 1
                if in_degree[v] == 0:
                    q.append(v)

        return idx == n
JavaScript
JAVASCRIPT
function sequenceReconstruction(order, fragments) {
    if (fragments.length === 0) return false;

#### cons
t n = order.length;

#### cons
t graph = Array.from({ length: n }, () => []);

#### cons
t inDegree = new Array(n).fill(0);

    // Build the constraint graph from adjacent pairs in each fragment.
    for (const frag of fragments) {
        for (let i = 0; i + 1 < frag.length; i++) {

#### cons
t u = frag[i] - 1, v = frag[i + 1] - 1;
            graph[u].push(v);
            inDegree[v]++;
        }
    }

#### cons
t queue = [];
    for (let i = 0; i < n; i++) if (inDegree[i] === 0) queue.push(i);

    let idx = 0, head = 0;
    while (head < queue.length) {
        if (queue.length - head > 1) return false;   // ambiguous next element

#### cons
t u = queue[head++];
        if (u !== order[idx] - 1) return false;
        idx++;
        for (const v of graph[u]) {
            if (--inDegree[v] === 0) queue.push(v);
        }
    }
    return idx === n;
}
Java
JAVA
import java.util.*;

class Solution {
    public boolean sequenceReconstruction(int[] order, List<List<Integer>> fragments) {
        if (fragments.isEmpty()) return false;
        final int n = order.length;
        List<Integer>[] graph = new List[n];
        int[] inDegree = new int[n];
        for (int i = 0; i < n; i++) graph[i] = new ArrayList<>();

        // Build the constraint graph from adjacent pairs in each fragment.
        for (List<Integer> frag : fragments)
            for (int i = 0; i + 1 < frag.size(); i++) {
                int u = frag.get(i) - 1, v = frag.get(i + 1) - 1;
                graph[u].add(v);
                inDegree[v]++;
            }

        Deque<Integer> q = new ArrayDeque<>();
        for (int i = 0; i < n; i++)
            if (inDegree[i] == 0) q.offer(i);

        int idx = 0;
        while (!q.isEmpty()) {
            if (q.size() > 1) return false;       // ambiguous next element
            int u = q.poll();
            if (u != order[idx] - 1) return false;
            idx++;
            for (int v : graph[u])
                if (--inDegree[v] == 0) q.offer(v);
        }
        return idx == n;
    }
}
C++
CPP
#include <vector>
#include <queue>
using namespace std;

class Solution {
public:
    bool sequenceReconstruction(vector<int>& order, vector<vector<int>>& fragments) {
        if (fragments.empty()) return false;

#### cons
t int n = order.size();
        vector<vector<int>> graph(n);
        vector<int> inDegree(n, 0);

        // Build the constraint graph from adjacent pairs in each fragment.
        for (const auto& frag : fragments)
            for (int i = 0; i + 1 < (int)frag.size(); ++i) {
                int u = frag[i] - 1, v = frag[i + 1] - 1;
                graph[u].push_back(v);
                ++inDegree[v];
            }

        queue<int> q;
        for (int i = 0; i < n; ++i)
            if (inDegree[i] == 0) q.push(i);

        int idx = 0;
        while (!q.empty()) {
            if (q.size() > 1) return false;      // ambiguous next element
            int u = q.front(); q.pop();
            if (u != order[idx] - 1) return false;
            ++idx;
            for (int v : graph[u])
                if (--inDegree[v] == 0) q.push(v);
        }
        return idx == n;
    }
};
C
C
#include <stdbool.h>
#include <stdlib.h>

/* order is a permutation of 1..n; fragments given as seqs with per-row sizes. */
bool sequenceReconstruction(int* order, int orderSize,
                            int** fragments, int fragmentsSize, int* fragmentColSize) {
    if (fragmentsSize == 0) return false;
    int n = orderSize;

    int* inDegree = (int*)calloc(n, sizeof(int));
    int* head = (int*)malloc(n * sizeof(int));   /* head[u] = first edge index from u, or -1 */
    for (int i = 0; i < n; i++) head[i] = -1;

    long totalEdges = 0;
    for (int i = 0; i < fragmentsSize; i++)
        if (fragmentColSize[i] > 1) totalEdges += fragmentColSize[i] - 1;

    long cap = totalEdges > 0 ? totalEdges : 1;
    int* to = (int*)malloc(cap * sizeof(int));
    int* nxt = (int*)malloc(cap * sizeof(int));
    int ec = 0;

    /* Build the constraint graph from adjacent pairs in each fragment. */
    for (int s = 0; s < fragmentsSize; s++) {
        for (int j = 0; j + 1 < fragmentColSize[s]; j++) {
            int u = fragments[s][j] - 1;
            int v = fragments[s][j + 1] - 1;
            to[ec] = v;
            nxt[ec] = head[u];
            head[u] = ec;
            ec++;
            inDegree[v]++;
        }
    }

    int* q = (int*)malloc(n * sizeof(int));
    int qh = 0, qt = 0;
    for (int i = 0; i < n; i++)
        if (inDegree[i] == 0) q[qt++] = i;

    int idx = 0;
    bool ok = true;
    while (qh < qt) {
        if (qt - qh > 1) { ok = false; break; }   /* more than one choice -> not unique */
        int u = q[qh++];
        if (u != order[idx] - 1) { ok = false; break; }
        idx++;
        for (int e = head[u]; e != -1; e = nxt[e]) {
            int v = to[e];
            if (--inDegree[v] == 0) q[qt++] = v;
        }
    }
    if (ok && idx != n) ok = false;

    free(inDegree); free(head); free(to); free(nxt); free(q);
    return ok;
}

#### Follow-up
questions
Instead of a yes/no, return the reconstructed order when it is unique, or report "ambiguous" versus "impossible" (cyclic constraints) as distinct outcomes.
If a fragment contains a repeated value or the constraints form a cycle, detect it and explain why no valid supersequence exists.
When the order is not unique, count how many distinct shortest supersequences the constraints allow.
Stream the fragments online and answer whether uniqueness still holds after each new fragment arrives.

