---
type: "practice"
difficulty: "Other"
---

# Minimum Round-Trip Cost to Acquire One Unit

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Graph](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Graph.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

There are n depots numbered from 0 to n - 1. You are given a list corridors, where corridors[i] = [u, v, w] means there is a bidirectional corridor connecting depots u and v whose one-way traversal cost is w.

Every depot sells the exact same interchangeable unit, but at different prices. You are given an array price of length n, where price[d] is the cost of buying one unit at depot d.

For a chosen home depot you do the following:

Start at the home depot.
Travel along corridors to some depot (possibly the home depot itself, meaning no travel) and buy exactly one unit there.
Return from that depot back to the home depot. On this return leg, every corridor's traversal cost is multiplied by a given integer factor k.

The total cost of a trip is: (cost of the outbound travel) + (unit price where you bought) + (k times the cost of the return travel). You may choose any paths and any purchase depot to minimize this total.

Return an array answer of length n, where answer[d] is the minimum total trip cost when the home depot is d. If a depot is isolated, the answer for it is simply its own unit price (buy at home, no travel).

Input

n: number of depots.
corridors: list of [u, v, w] bidirectional corridors.
price: array of length n of unit prices per depot.
k: the return-trip cost multiplier.

Output

An array answer of length n with the minimum total trip cost per home depot.

#### Cons
traints

2 <= n <= 1000
1 <= corridors.length <= 1000
corridors[i].length == 3
0 <= u, v < n and u != v
1 <= w <= 100000
price.length == n
1 <= price[d] <= 100000
1 <= k <= 100
No duplicate corridor connects the same pair of depots.

#### Example
s & test cases
TEXT
Input: n = 3, corridors = [[0,1,3],[1,2,4],[0,2,10]], price = [20,5,8], k = 1 -> Output: [11,5,8]
Why: From depot 0, walk to depot 1 (cost 3), buy for 5, return (3 * 1 = 3): total 11, cheaper than buying at home for 20. Depot 1 buys at home for 5. Depot 2 buys at home for 8.

Input: n = 2, corridors = [[0,1,100]], price = [7,9], k = 5 -> Output: [7,9]
Why: the return multiplier is so large that any travel is never worth it, so each depot just buys at home.

Input: n = 4, corridors = [[0,1,4],[1,2,2],[1,3,5],[2,3,1],[0,2,4]], price = [56,42,102,301], k = 2 -> Output: [54,42,48,51]
Why: cheapest unit is 42 at depot 1; every depot compares buying locally against reaching depot 1 with round-trip factor (k+1)=3, e.g. depot 3 goes 3->2->1 (dist 3): 42 + 3*3 = 51.

Input: n = 3, corridors = [[0,1,2]], price = [10,3,4], k = 1 -> Output: [7,3,4]
Why: depot 2 is isolated so it buys at home for 4. Depot 0 reaches depot 1 (dist 2), 3 + 2*2 = 7, beating 10. Depot 1 buys at home for 3.

Input: n = 2, corridors = [[0,1,1]], price = [100,1], k = 100 -> Output: [100,1]
Why: the unit next door costs only 1, but fetching it from depot 0 costs 1 (walk out) + 1 (buy) + 100*1 (walk back) = 102, which is worse than buying at home for 100, so answer[0] = min(100, 102) = 100. Depot 1 buys at home for 1.

#### Approach

Because corridors are undirected and costs are non-negative, the shortest outbound path from a home depot d to a purchase depot j has the same length as the return path, call it dist(d, j). So the cost of buying at j from home d is price[j] + dist(d, j) + k * dist(d, j) = price[j] + (k + 1) * dist(d, j). Therefore answer[d] = min over all j of price[j] + (k + 1) * dist(d, j). Instead of computing this per home depot, invert it: treat every depot j as a source pre-loaded with distance price[j], and run one multi-source Dijkstra where each corridor of cost w is relaxed as (k + 1) * w. The final settled distance at depot d equals exactly min_j (price[j] + (k + 1) * dist(d, j)), which is answer[d]. Isolated depots keep their seed price[d] (buy at home). Pattern: shortest-path with a super-source / multi-source Dijkstra plus an edge-reweighting trick.

Complexity

Time: O((n + E) log n), where E = corridors.length, from a single multi-source Dijkstra over a graph with n nodes and 2E directed edges. Space: O(n + E) for the adjacency structure, the distance array, and the priority queue. This replaces the naive "run Dijkstra once per depot" (O(n * (n + E) log n)) with one combined pass.

Python
PYTHON
import heapq
from typing import List

def min_acquire_cost(n: int, corridors: List[List[int]],
                     price: List[int], k: int) -> List[int]:
    graph = [[] for _ in range(n)]
    for u, v, w in corridors:
        graph[u].append((v, w))
        graph[v].append((u, w))

    mult = k + 1                 # outbound (1x) + return (k x)
    dist = list(price)           # seed every depot with its own price
    pq = [(price[i], i) for i in range(n)]
    heapq.heapify(pq)

    while pq:
        d, u = heapq.heappop(pq)
        if d > dist[u]:
            continue
        for v, w in graph[u]:
            nd = d + mult * w
            if nd < dist[v]:
                dist[v] = nd
                heapq.heappush(pq, (nd, v))
    return dist
JavaScript
JAVASCRIPT
class MinHeap {

#### cons
tructor() { this.a = []; }
    size() { return this.a.length; }
    push(x) {

#### cons
t a = this.a;
        a.push(x);
        let i = a.length - 1;
        while (i > 0) {

#### cons
t p = (i - 1) >> 1;
            if (a[p][0] <= a[i][0]) break;
            [a[p], a[i]] = [a[i], a[p]];
            i = p;
        }
    }
    pop() {

#### cons
t a = this.a;

#### cons
t top = a[0];

#### cons
t last = a.pop();
        if (a.length) {
            a[0] = last;
            let i = 0;

#### cons
t n = a.length;
            for (;;) {
                let l = 2 * i + 1, r = 2 * i + 2, s = i;
                if (l < n && a[l][0] < a[s][0]) s = l;
                if (r < n && a[r][0] < a[s][0]) s = r;
                if (s === i) break;
                [a[s], a[i]] = [a[i], a[s]];
                i = s;
            }
        }
        return top;
    }
}

function minAcquireCost(n, corridors, price, k) {

#### cons
t graph = Array.from({ length: n }, () => []);
    for (const [u, v, w] of corridors) {
        graph[u].push([v, w]);
        graph[v].push([u, w]);
    }

#### cons
t mult = k + 1;              // outbound (1x) + return (k x)

#### cons
t dist = price.slice();      // seed every depot with its own price

#### cons
t pq = new MinHeap();
    for (let i = 0; i < n; i++) pq.push([price[i], i]);

    while (pq.size()) {

#### cons
t [d, u] = pq.pop();
        if (d > dist[u]) continue;
        for (const [v, w] of graph[u]) {

#### cons
t nd = d + mult * w;
            if (nd < dist[v]) { dist[v] = nd; pq.push([nd, v]); }
        }
    }
    return dist;
}
Java
JAVA
import java.util.*;

class Solution {
    public long[] minAcquireCost(int n, int[][] corridors, int[] price, int k) {
        List<long[]>[] graph = new List[n];
        for (int i = 0; i < n; i++) graph[i] = new ArrayList<>();
        for (int[] e : corridors) {
            graph[e[0]].add(new long[]{e[1], e[2]});
            graph[e[1]].add(new long[]{e[0], e[2]});
        }

        long mult = (long) k + 1;   // outbound (1x) + return (k x)
        long[] dist = new long[n];
        PriorityQueue<long[]> pq =
            new PriorityQueue<>((a, b) -> Long.compare(a[0], b[0]));

        // Seed every depot with its own price (multi-source Dijkstra).
        for (int i = 0; i < n; i++) { dist[i] = price[i]; pq.offer(new long[]{dist[i], i}); }

        while (!pq.isEmpty()) {
            long[] cur = pq.poll();
            long d = cur[0];
            int u = (int) cur[1];
            if (d > dist[u]) continue;
            for (long[] nb : graph[u]) {
                int v = (int) nb[0];
                long nd = d + mult * nb[1];
                if (nd < dist[v]) { dist[v] = nd; pq.offer(new long[]{nd, v}); }
            }
        }
        return dist;
    }
}
C++
CPP
#include <vector>
#include <queue>
using namespace std;

class Solution {
public:
    vector<long long> minAcquireCost(int n, vector<vector<int>> &corridors,
                                     vector<int> &price, int k) {
        vector<vector<pair<int, long long>>> graph(n);
        for (auto &e : corridors) {
            graph[e[0]].push_back({e[1], e[2]});
            graph[e[1]].push_back({e[0], e[2]});
        }

        long long mult = (long long)k + 1;  // outbound (1x) + return (k x)
        vector<long long> dist(n);
        priority_queue<pair<long long, int>, vector<pair<long long, int>>,
                       greater<pair<long long, int>>> pq;

        // Seed every depot with its own price (multi-source).
        for (int i = 0; i < n; i++) { dist[i] = price[i]; pq.push({dist[i], i}); }

        while (!pq.empty()) {
            auto [d, u] = pq.top(); pq.pop();
            if (d > dist[u]) continue;
            for (auto &[v, w] : graph[u]) {
                long long nd = d + mult * w;
                if (nd < dist[v]) { dist[v] = nd; pq.push({nd, v}); }
            }
        }
        return dist;
    }
};
C
C
#include <stdlib.h>

typedef struct { long long key; int node; } HeapItem;
typedef struct { HeapItem *a; int size; int cap; } Heap;

static void heapPush(Heap *h, long long key, int node) {
    if (h->size == h->cap) {
        h->cap = h->cap ? h->cap * 2 : 16;
        h->a = (HeapItem *)realloc(h->a, h->cap * sizeof(HeapItem));
    }
    int i = h->size++;
    h->a[i].key = key;
    h->a[i].node = node;
    while (i > 0) {
        int p = (i - 1) / 2;
        if (h->a[p].key <= h->a[i].key) break;
        HeapItem t = h->a[p]; h->a[p] = h->a[i]; h->a[i] = t;
        i = p;
    }
}

static HeapItem heapPop(Heap *h) {
    HeapItem top = h->a[0];
    h->a[0] = h->a[--h->size];
    int i = 0;
    for (;;) {
        int l = 2 * i + 1, r = 2 * i + 2, s = i;
        if (l < h->size && h->a[l].key < h->a[s].key) s = l;
        if (r < h->size && h->a[r].key < h->a[s].key) s = r;
        if (s == i) break;
        HeapItem t = h->a[s]; h->a[s] = h->a[i]; h->a[i] = t;
        i = s;
    }
    return top;
}

/* corridors: corridorsSize rows of {u, v, w}, depots 0-indexed.
   Returns a malloc'd long long[n]; caller frees. */
long long *minAcquireCost(int n, int corridors[][3], int corridorsSize,

#### cons
t int *price, int k) {
    int *head = (int *)malloc(n * sizeof(int));
    for (int i = 0; i < n; i++) head[i] = -1;

    int m2 = corridorsSize * 2;
    int cap = m2 ? m2 : 1;
    int *to = (int *)malloc(cap * sizeof(int));
    int *nxt = (int *)malloc(cap * sizeof(int));
    long long *w = (long long *)malloc(cap * sizeof(long long));
    int ec = 0;
    for (int i = 0; i < corridorsSize; i++) {
        int u = corridors[i][0], v = corridors[i][1];
        long long c = corridors[i][2];
        to[ec] = v; w[ec] = c; nxt[ec] = head[u]; head[u] = ec++;
        to[ec] = u; w[ec] = c; nxt[ec] = head[v]; head[v] = ec++;
    }

    long long *dist = (long long *)malloc(n * sizeof(long long));
    Heap h; h.a = NULL; h.size = 0; h.cap = 0;

    /* Multi-source Dijkstra: seed every depot with its own price, then
       relax edges weighted by (k + 1) = outbound + k*return. */
    long long mult = (long long)k + 1;
    for (int i = 0; i < n; i++) { dist[i] = price[i]; heapPush(&h, dist[i], i); }

    while (h.size > 0) {
        HeapItem it = heapPop(&h);
        long long d = it.key; int u = it.node;
        if (d > dist[u]) continue;
        for (int e = head[u]; e != -1; e = nxt[e]) {
            int v = to[e];
            long long nd = d + mult * w[e];
            if (nd < dist[v]) { dist[v] = nd; heapPush(&h, nd, v); }
        }
    }

    free(head); free(to); free(nxt); free(w); free(h.a);
    return dist;
}

#### Follow-up
questions
What if only a subset of depots actually sells the unit? (Seed Dijkstra only from those sources.)
What if you do NOT have to return, or the return uses a different graph or multiplier?
What if corridors were directed, so outbound and return distances differ? (Run one Dijkstra on the graph and one on its reverse.)
What if you must acquire q units, or buy at up to two different depots to minimize total cost?

