---
type: "practice"
difficulty: "Other"
---

# Counting Fastest Routes Through a Network

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Graph](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Graph.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

A transit network has n stations, labeled 0 through n - 1. Some pairs of stations are joined by two-way links. The network is built so that you can travel between any two stations (possibly through intermediate stations), and no two stations are joined by more than one link.

You are given the integer n and a list links, where each entry links[i] = [a, b, t] means there is a two-way link directly connecting station a and station b, and traveling across it takes exactly t minutes.

Starting at station 0, you want to reach station n - 1. Among all possible routes, consider only those that take the least total time. Count how many distinct routes achieve that minimum total travel time.

Because the count can be enormous, return it modulo 10^9 + 7.

Input
n: the number of stations.
links: a list of [a, b, t] triples describing the two-way links and their travel times.
Output
A single integer: the number of minimum-total-time routes from station 0 to station n - 1, taken modulo 10^9 + 7.

#### Cons
traints
1 <= n <= 200
n - 1 <= links.length <= n * (n - 1) / 2
links[i].length == 3
0 <= a, b <= n - 1 and a != b
1 <= t <= 10^9
There is at most one link between any pair of stations.
Every station is reachable from every other station.
A single route's total time can be as large as about 2 * 10^11, so accumulate distances in a 64-bit integer type; only the final count is taken modulo 10^9 + 7.

#### Example
s & test cases
TEXT
- Input: `n = 4, links = [[0,1,1],[0,2,1],[1,3,1],[2,3,1]]` -> Output: `2`
  Why: the fastest trip from `0` to `3` costs `2` minutes, achievable two ways: `0->1->3` and `0->2->3`.
- Input: `n = 2, links = [[0,1,7]]` -> Output: `1`
  Why: the only route is the single direct link; exactly one fastest route.
- Input: `n = 3, links = [[0,1,1],[1,2,1],[0,2,5]] ` -> Output: `1`
  Why: `0->1->2` costs `2` while the direct link costs `5`; only the `2`-minute route is shortest, so the count is `1` (a slower alternative is not counted).
- Input: `n = 5, links = [[0,1,1],[0,2,1],[1,3,1],[2,3,1],[3,4,1]]` -> Output: `2`
  Why: the fastest way to `4` costs `3` minutes; both shortest routes pass through station `3` (`0->1->3->4` and `0->2->3->4`), so the count of `2` shortest routes into `3` carries through to `4`.
- Input: `n = 3, links = [[0,1,1000000000],[1,2,1000000000],[0,2,1000000000]]` -> Output: `1`
  Why: the two-hop route costs `2*10^9` while the direct link costs only `10^9`; only the direct route is shortest. Large weights require 64-bit distance accumulation.

#### Approach

Run Dijkstra's algorithm from station 0, but carry a second array ways[] alongside the usual dist[]. ways[v] records how many distinct shortest routes reach v, taken modulo 10^9 + 7. Initialize dist[0] = 0 and ways[0] = 1. Each time a node u is finalized (popped from the min-heap with its true shortest distance), relax every neighbor v: if going through u gives a strictly smaller distance, overwrite dist[v] and set ways[v] = ways[u] (the old routes are obsolete); if it exactly ties the current best distance to v, add ways[u] into ways[v] modulo the prime. Because Dijkstra finalizes nodes in non-decreasing distance order, every predecessor on a shortest path to v is fully counted before v is processed, so the counts compose correctly. Distances must be summed in 64-bit integers (weights up to 10^9 over up to n-1 hops reach ~2*10^11); only the route count is reduced modulo the prime. This is the "Dijkstra plus shortest-path DAG counting" technique.

Complexity

Time: O(E log V) where V = n and E = links.length — a standard Dijkstra with a binary heap, since each edge is relaxed a constant number of times and every heap operation costs O(log V). Space: O(V + E) for the adjacency list, the distance and count arrays, and the heap.

Python
PYTHON
import heapq

class Solution:
    def countFastestRoutes(self, n: int, links: list[list[int]]) -> int:
        MOD = 10 ** 9 + 7
        graph = [[] for _ in range(n)]
        for a, b, t in links:
            graph[a].append((b, t))
            graph[b].append((a, t))

        dist = [float('inf')] * n
        ways = [0] * n
        dist[0] = 0
        ways[0] = 1
        pq = [(0, 0)]  # (distance, node)

        while pq:
            d, u = heapq.heappop(pq)
            if d > dist[u]:
                continue  # stale entry
            for v, w in graph[u]:
                nd = d + w
                if nd < dist[v]:            # strictly shorter path
                    dist[v] = nd
                    ways[v] = ways[u]
                    heapq.heappush(pq, (nd, v))
                elif nd == dist[v]:         # equally short path
                    ways[v] = (ways[v] + ways[u]) % MOD

        return ways[n - 1] % MOD
JavaScript
JAVASCRIPT
function countFastestRoutes(n, links) {

#### cons
t MOD = 1000000007;

#### cons
t g = Array.from({ length: n }, () => []);
    for (const [a, b, t] of links) {
        g[a].push([b, t]);
        g[b].push([a, t]);
    }

#### cons
t dist = new Array(n).fill(Infinity);

#### cons
t ways = new Array(n).fill(0);
    dist[0] = 0;
    ways[0] = 1;

    // Binary min-heap of [distance, node].

#### cons
t heap = [[0, 0]];

#### cons
t swap = (i, j) => { const t = heap[i]; heap[i] = heap[j]; heap[j] = t; };

#### cons
t push = (item) => {
        heap.push(item);
        let i = heap.length - 1;
        while (i > 0) {

#### cons
t p = (i - 1) >> 1;
            if (heap[p][0] <= heap[i][0]) break;
            swap(i, p); i = p;
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

#### cons
t m = heap.length;
            while (true) {
                let s = i;

#### cons
t l = 2 * i + 1, r = 2 * i + 2;
                if (l < m && heap[l][0] < heap[s][0]) s = l;
                if (r < m && heap[r][0] < heap[s][0]) s = r;
                if (s === i) break;
                swap(i, s); i = s;
            }
        }
        return top;
    };

    while (heap.length) {

#### cons
t [d, u] = pop();
        if (d > dist[u]) continue;          // stale entry
        for (const [v, w] of g[u]) {

#### cons
t nd = d + w;
            if (nd < dist[v]) {             // strictly shorter path
                dist[v] = nd;
                ways[v] = ways[u];
                push([nd, v]);
            } else if (nd === dist[v]) {     // equally short path
                ways[v] = (ways[v] + ways[u]) % MOD;
            }
        }
    }
    return ways[n - 1] % MOD;
}
Java
JAVA
import java.util.*;

class Solution {
    public int countFastestRoutes(int n, int[][] links) {
        final long MOD = 1_000_000_007L;
        List<long[]>[] g = new List[n];
        for (int i = 0; i < n; i++) g[i] = new ArrayList<>();
        for (int[] e : links) {
            g[e[0]].add(new long[]{e[1], e[2]});
            g[e[1]].add(new long[]{e[0], e[2]});
        }
        long[] dist = new long[n];
        long[] ways = new long[n];
        Arrays.fill(dist, Long.MAX_VALUE);
        dist[0] = 0; ways[0] = 1;
        // Min-heap ordered by distance: {dist, node}.
        PriorityQueue<long[]> pq = new PriorityQueue<>((x, y) -> Long.compare(x[0], y[0]));
        pq.offer(new long[]{0, 0});
        while (!pq.isEmpty()) {
            long[] cur = pq.poll();
            long d = cur[0];
            int u = (int) cur[1];
            if (d > dist[u]) continue;              // stale entry
            for (long[] e : g[u]) {
                int v = (int) e[0];
                long nd = d + e[1];
                if (nd < dist[v]) {                 // strictly shorter path
                    dist[v] = nd;
                    ways[v] = ways[u];
                    pq.offer(new long[]{nd, v});
                } else if (nd == dist[v]) {          // equally short path
                    ways[v] = (ways[v] + ways[u]) % MOD;
                }
            }
        }
        return (int) (ways[n - 1] % MOD);
    }
}
C++
CPP
#include <vector>
#include <queue>
#include <climits>
using namespace std;

class Solution {
public:
    int countFastestRoutes(int n, vector<vector<int>>& links) {

#### cons
t long long MOD = 1000000007LL;
        vector<vector<pair<int,long long>>> g(n);
        for (auto& e : links) {
            g[e[0]].push_back({e[1], (long long)e[2]});
            g[e[1]].push_back({e[0], (long long)e[2]});
        }
        vector<long long> dist(n, LLONG_MAX), ways(n, 0);
        dist[0] = 0; ways[0] = 1;
        priority_queue<pair<long long,int>, vector<pair<long long,int>>, greater<>> pq;
        pq.push({0, 0});
        while (!pq.empty()) {
            auto [d, u] = pq.top(); pq.pop();
            if (d > dist[u]) continue;             // stale
            for (auto& [v, w] : g[u]) {
                long long nd = d + w;
                if (nd < dist[v]) {                // strictly shorter
                    dist[v] = nd;
                    ways[v] = ways[u];
                    pq.push({nd, v});
                } else if (nd == dist[v]) {         // equally short
                    ways[v] = (ways[v] + ways[u]) % MOD;
                }
            }
        }
        return (int)(ways[n - 1] % MOD);
    }
};
C
C
#include <stdlib.h>
#include <limits.h>

typedef struct { long long d; int u; } HeapNode;

typedef struct { HeapNode* a; int size; int cap; } MinHeap;

static void hpush(MinHeap* h, long long d, int u) {
    if (h->size == h->cap) { h->cap *= 2; h->a = (HeapNode*)realloc(h->a, h->cap * sizeof(HeapNode)); }
    int i = h->size++;
    h->a[i].d = d; h->a[i].u = u;
    while (i > 0) {
        int p = (i - 1) / 2;
        if (h->a[p].d <= h->a[i].d) break;
        HeapNode t = h->a[p]; h->a[p] = h->a[i]; h->a[i] = t; i = p;
    }
}

static HeapNode hpop(MinHeap* h) {
    HeapNode top = h->a[0];
    h->a[0] = h->a[--h->size];
    int i = 0;
    while (1) {
        int l = 2 * i + 1, r = 2 * i + 2, s = i;
        if (l < h->size && h->a[l].d < h->a[s].d) s = l;
        if (r < h->size && h->a[r].d < h->a[s].d) s = r;
        if (s == i) break;
        HeapNode t = h->a[s]; h->a[s] = h->a[i]; h->a[i] = t; i = s;
    }
    return top;
}

int countFastestRoutes(int n, int** links, int linksSize, int* linksColSize) {

#### cons
t long long MOD = 1000000007LL;
    /* Adjacency via a linked-list (forward-star) representation. */
    int* head = (int*)malloc(n * sizeof(int));
    for (int i = 0; i < n; i++) head[i] = -1;
    int ne = linksSize * 2;
    int* to = (int*)malloc(ne * sizeof(int));
    long long* wt = (long long*)malloc(ne * sizeof(long long));
    int* nxt = (int*)malloc(ne * sizeof(int));
    int ec = 0;
    for (int i = 0; i < linksSize; i++) {
        int a = links[i][0], b = links[i][1]; long long w = links[i][2];
        to[ec] = b; wt[ec] = w; nxt[ec] = head[a]; head[a] = ec++;
        to[ec] = a; wt[ec] = w; nxt[ec] = head[b]; head[b] = ec++;
    }
    long long* dist = (long long*)malloc(n * sizeof(long long));
    long long* ways = (long long*)malloc(n * sizeof(long long));
    for (int i = 0; i < n; i++) { dist[i] = LLONG_MAX; ways[i] = 0; }
    dist[0] = 0; ways[0] = 1;
    MinHeap h; h.cap = 16; h.size = 0; h.a = (HeapNode*)malloc(h.cap * sizeof(HeapNode));
    hpush(&h, 0, 0);
    while (h.size > 0) {
        HeapNode cur = hpop(&h);
        if (cur.d > dist[cur.u]) continue;   /* stale entry */
        for (int e = head[cur.u]; e != -1; e = nxt[e]) {
            int v = to[e];
            long long nd = cur.d + wt[e];
            if (nd < dist[v]) {              /* found a strictly shorter path */
                dist[v] = nd;
                ways[v] = ways[cur.u];
                hpush(&h, nd, v);
            } else if (nd == dist[v]) {      /* another shortest path */
                ways[v] = (ways[v] + ways[cur.u]) % MOD;
            }
        }
    }
    int result = (int)(ways[n - 1] % MOD);
    free(head); free(to); free(wt); free(nxt); free(dist); free(ways); free(h.a);
    return result;
}

#### Follow-up
questions
Why is it safe to accumulate counts the moment distances tie, and how does Dijkstra's finalization order guarantee no shortest predecessor is missed?
What if the links were one-directional (a directed graph) — does the same algorithm still work unchanged?
If some link times could be 0, how would that affect correctness of the counting, and how would you handle it?
Besides counting them, also return one actual shortest route, or the length of that shortest time itself.

