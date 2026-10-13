---
type: "practice"
difficulty: "Other"
---

# Most Reliable Route Through a Relay Network

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Graph](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Graph.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

A communication network is made up of n relay stations, labeled from 0 to n - 1. Certain pairs of stations are connected by a bidirectional link. Every link is imperfect: when a message is pushed across a link it succeeds only with a given probability. Different links have different reliabilities.

The reliability of an entire route is the product of the success probabilities of every link along that route (each hop must succeed for the message to arrive). A route that uses no link at all — i.e. the source station is also the destination — has reliability 1.

You are given:

an integer n, the number of stations,
a list links, where links[i] = [a, b] describes a bidirectional link between station a and station b,
a list reliability of the same length, where reliability[i] is the probability (a value between 0 and 1) that a message crossing links[i] succeeds,
two station indices source and target.

Return the maximum possible reliability of any route that carries a message from source to target. If no route connects them, return 0. An answer is considered correct if it is within 1e-5 of the true value.

There is at most one link between any pair of stations.

#### Cons
traints

2 <= n <= 10^4
0 <= source, target < n
source != target
0 <= links.length == reliability.length <= 2 * 10^4
links[i].length == 2
0 <= a, b < n and a != b
0 <= reliability[i] <= 1
At most one link exists between any two stations.

#### Example
s & test cases
TEXT
Input: n = 4, links = [[0,1],[1,3],[0,2],[2,3]], reliability = [0.9,0.8,0.6,0.5], source = 0, target = 3 -> Output: 0.72000
Why: Route 0->1->3 gives 0.9*0.8 = 0.72; route 0->2->3 gives 0.6*0.5 = 0.30. The larger product wins.

Input: n = 3, links = [[0,1],[1,2],[0,2]], reliability = [0.5,0.5,0.4], source = 0, target = 2 -> Output: 0.40000
Why: The direct link 0->2 (0.40) beats the two-hop 0->1->2 (0.25); a direct edge is not always best but here it is.

Input: n = 3, links = [[0,1]], reliability = [0.7], source = 0, target = 2 -> Output: 0.00000
Why: Station 2 is isolated; no route exists, so the answer is 0.

Input: n = 2, links = [[0,1]], reliability = [1.0], source = 0, target = 1 -> Output: 1.00000
Why: A single perfectly reliable link gives full reliability (upper bound edge case).

Input: n = 3, links = [[0,1],[1,2]], reliability = [0.0,0.9], source = 0, target = 2 -> Output: 0.00000
Why: The only route passes through a link with probability 0, so the product collapses to 0 even though a path exists.

#### Approach

This is a shortest-path problem with a twist: instead of minimizing a sum of edge weights, we maximize a product of edge probabilities. Because every probability lies in [0, 1], multiplying by an extra edge can only keep a route the same or make it less reliable — the product is monotonically non-increasing along a route, exactly the "no negative edges" property Dijkstra needs. So we run Dijkstra with a max-heap: best[v] holds the highest reliability found so far to reach station v, seeded with best[source] = 1. We always expand the most-reliable frontier node; when we pop the target, that popped value is provably optimal and we return it. A node popped with a stale (smaller) probability is skipped. If the heap empties without reaching target, no route exists and we return 0. (Equivalently, one could take -log(prob) per edge to turn products into additive non-negative weights and run standard shortest-path.)

Complexity

Time: O((V + E) log V) — a Dijkstra-style traversal where every link is relaxed at most once and each heap operation costs O(log V), with V = n stations and E = number of links. Space: O(V + E) for the adjacency list, the best-probability array, and the heap.

Python
PYTHON
import heapq
from collections import defaultdict

def max_reliability(n, links, reliability, source, target):
    graph = defaultdict(list)
    for i, (a, b) in enumerate(links):
        graph[a].append((b, reliability[i]))
        graph[b].append((a, reliability[i]))

    best = [0.0] * n
    best[source] = 1.0
    # Python's heapq is a min-heap, so store negated probabilities.
    heap = [(-1.0, source)]
    while heap:
        neg_p, u = heapq.heappop(heap)
        p = -neg_p
        if u == target:
            return p
        if p < best[u]:
            continue
        for v, w in graph[u]:
            np = p * w
            if np > best[v]:
                best[v] = np
                heapq.heappush(heap, (-np, v))
    return 0.0
JavaScript
JAVASCRIPT
function maxReliability(n, links, reliability, source, target) {

#### cons
t graph = Array.from({ length: n }, () => []);
    for (let i = 0; i < links.length; i++) {

#### cons
t [a, b] = links[i];
        graph[a].push([b, reliability[i]]);
        graph[b].push([a, reliability[i]]);
    }

#### cons
t best = new Array(n).fill(0);
    best[source] = 1;

    // Binary max-heap of [prob, node].

#### cons
t heap = [[1, source]];

#### cons
t swap = (i, j) => { const t = heap[i]; heap[i] = heap[j]; heap[j] = t; };

#### cons
t push = (item) => {
        heap.push(item);
        let i = heap.length - 1;
        while (i > 0) {

#### cons
t p = (i - 1) >> 1;
            if (heap[p][0] >= heap[i][0]) break;
            swap(p, i); i = p;
        }
    };

#### cons
t pop = () => {

#### cons
t top = heap[0];

#### cons
t last = heap.pop();
        if (heap.length) {
            heap[0] = last;
            let i = 0;
            while (true) {
                let l = 2 * i + 1, r = 2 * i + 2, m = i;
                if (l < heap.length && heap[l][0] > heap[m][0]) m = l;
                if (r < heap.length && heap[r][0] > heap[m][0]) m = r;
                if (m === i) break;
                swap(m, i); i = m;
            }
        }
        return top;
    };

    while (heap.length) {

#### cons
t [p, u] = pop();
        if (u === target) return p;
        if (p < best[u]) continue;
        for (const [v, w] of graph[u]) {

#### cons
t np = p * w;
            if (np > best[v]) { best[v] = np; push([np, v]); }
        }
    }
    return 0;
}
Java
JAVA
import java.util.*;

class Solution {
    public double maxReliability(int n, int[][] links, double[] reliability,
                                 int source, int target) {
        List<double[]>[] graph = new List[n];
        for (int i = 0; i < n; i++) graph[i] = new ArrayList<>();
        for (int i = 0; i < links.length; i++) {
            int a = links[i][0], b = links[i][1];
            graph[a].add(new double[]{b, reliability[i]});
            graph[b].add(new double[]{a, reliability[i]});
        }
        double[] best = new double[n];
        best[source] = 1.0;
        // Max-heap ordered by probability.
        PriorityQueue<double[]> pq =
            new PriorityQueue<>((x, y) -> Double.compare(y[0], x[0]));
        pq.offer(new double[]{1.0, source});
        while (!pq.isEmpty()) {
            double[] cur = pq.poll();
            double p = cur[0];
            int u = (int) cur[1];
            if (u == target) return p;
            if (p < best[u]) continue;
            for (double[] e : graph[u]) {
                int v = (int) e[0];
                double np = p * e[1];
                if (np > best[v]) { best[v] = np; pq.offer(new double[]{np, v}); }
            }
        }
        return 0.0;
    }
}
C++
CPP
#include <vector>
#include <queue>
using namespace std;

class Solution {
public:
    double maxReliability(int n, vector<vector<int>>& links,
                          vector<double>& reliability, int source, int target) {
        vector<vector<pair<int, double>>> graph(n);
        for (size_t i = 0; i < links.size(); ++i) {
            int a = links[i][0], b = links[i][1];
            graph[a].push_back({b, reliability[i]});
            graph[b].push_back({a, reliability[i]});
        }
        vector<double> best(n, 0.0);
        best[source] = 1.0;
        priority_queue<pair<double, int>> pq; // max-heap by probability
        pq.push({1.0, source});
        while (!pq.empty()) {
            auto [p, u] = pq.top(); pq.pop();
            if (u == target) return p;
            if (p < best[u]) continue;
            for (auto& [v, w] : graph[u]) {
                double np = p * w;
                if (np > best[v]) { best[v] = np; pq.push({np, v}); }
            }
        }
        return 0.0;
    }
};
C
C
#include <stdlib.h>

typedef struct { double p; int node; } HeapItem;
typedef struct { HeapItem *data; int size, cap; } Heap;

static void heapPush(Heap *h, double p, int node) {
    if (h->size == h->cap) {
        h->cap = h->cap ? h->cap * 2 : 16;
        h->data = (HeapItem *)realloc(h->data, h->cap * sizeof(HeapItem));
    }
    int i = h->size++;
    h->data[i].p = p; h->data[i].node = node;
    while (i > 0) {
        int parent = (i - 1) / 2;
        if (h->data[parent].p >= h->data[i].p) break;
        HeapItem t = h->data[parent]; h->data[parent] = h->data[i]; h->data[i] = t;
        i = parent;
    }
}

static HeapItem heapPop(Heap *h) {
    HeapItem top = h->data[0];
    h->data[0] = h->data[--h->size];
    int i = 0;
    while (1) {
        int l = 2 * i + 1, r = 2 * i + 2, largest = i;
        if (l < h->size && h->data[l].p > h->data[largest].p) largest = l;
        if (r < h->size && h->data[r].p > h->data[largest].p) largest = r;
        if (largest == i) break;
        HeapItem t = h->data[largest]; h->data[largest] = h->data[i]; h->data[i] = t;
        i = largest;
    }
    return top;
}

double maxReliability(int n, int **links, int linksSize, int *linksColSize,
                      double *reliability, int source, int target) {
    /* Build a CSR adjacency list. */
    int *deg = (int *)calloc(n, sizeof(int));
    for (int i = 0; i < linksSize; i++) { deg[links[i][0]]++; deg[links[i][1]]++; }
    int *head = (int *)malloc((n + 1) * sizeof(int));
    head[0] = 0;
    for (int i = 0; i < n; i++) head[i + 1] = head[i] + deg[i];
    int total = head[n];
    int *adjNode = (int *)malloc((total ? total : 1) * sizeof(int));
    double *adjW = (double *)malloc((total ? total : 1) * sizeof(double));
    int *pos = (int *)malloc(n * sizeof(int));
    for (int i = 0; i < n; i++) pos[i] = head[i];
    for (int i = 0; i < linksSize; i++) {
        int a = links[i][0], b = links[i][1];
        adjNode[pos[a]] = b; adjW[pos[a]] = reliability[i]; pos[a]++;
        adjNode[pos[b]] = a; adjW[pos[b]] = reliability[i]; pos[b]++;
    }
    double *best = (double *)calloc(n, sizeof(double));
    best[source] = 1.0;
    Heap h = { NULL, 0, 0 };
    heapPush(&h, 1.0, source);
    double ans = 0.0;
    while (h.size > 0) {
        HeapItem it = heapPop(&h);
        if (it.node == target) { ans = it.p; break; }
        if (it.p < best[it.node]) continue;
        for (int e = head[it.node]; e < head[it.node + 1]; e++) {
            int v = adjNode[e];
            double np = it.p * adjW[e];
            if (np > best[v]) { best[v] = np; heapPush(&h, np, v); }
        }
    }
    free(deg); free(head); free(adjNode); free(adjW);
    free(pos); free(best); free(h.data);
    return ans;
}

#### Follow-up
questions
What changes if the links are directed instead of bidirectional? (Only the source-side adjacency is added; the Dijkstra logic is unchanged.)
How would you also reconstruct and return the actual most-reliable route, not just its probability? (Store a parent pointer whenever best[v] improves.)
If you had to answer many (source, target) queries on the same fixed network, how would you preprocess or amortize the work?
Reformulate the problem so a standard min-sum shortest-path routine can solve it. (Use weight = -log(probability) and minimize the total.)

