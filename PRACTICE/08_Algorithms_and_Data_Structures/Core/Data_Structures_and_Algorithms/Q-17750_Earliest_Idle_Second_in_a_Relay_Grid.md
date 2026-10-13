---
type: "practice"
difficulty: "Other"
---

# Earliest Idle Second in a Relay Grid

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Breadth-First Search](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Breadth-First_Search.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

A monitoring grid is made of n relay stations, labelled from 0 to n - 1. You are given a 2D integer array links, where links[i] = [a, b] means there is a two-way channel between station a and station b. Along any single channel, any number of packets can travel in either direction in exactly one second. You are also given a 0-indexed integer array cooldown of length n.

The grid is connected: a packet can travel from any station to any other station, directly or through a chain of channels.

Station 0 is the coordinator. Every other station is a field station. Each field station must deliver a packet to the coordinator and wait for an acknowledgement. Packets always travel optimally, i.e. every packet reaches the coordinator in the fewest possible seconds. The coordinator processes each newly arrived packet instantly and immediately sends an acknowledgement back to the originating station along the exact reverse of the path the packet took.

At the beginning of second 0, every field station sends its packet. Then, starting from second 1, at the beginning of every second, each field station checks whether it has received an acknowledgement for its packet (including any acknowledgement arriving at that very second):

If it has not yet received one, it will resend the packet on a fixed cadence: field station i resends whenever cooldown[i] second(s) have elapsed since the last time it sent a packet. (So it resends at seconds cooldown[i], 2 * cooldown[i], 3 * cooldown[i], ... until an acknowledgement arrives.)
If it has received an acknowledgement, it stops — no further resends ever happen from that station.

The grid becomes idle at a given second when there are no packets in transit along any channel and no packets or acknowledgements arriving at any station.

Return the earliest second, starting from which the grid is idle.

Input
links: a list of [a, b] pairs describing two-way channels (each traversable in 1 second).
cooldown: an integer array of length n; cooldown[0] = 0 (the coordinator never sends), and cooldown[i] >= 1 for every field station i.
Output
A single integer: the earliest second from which the entire grid is idle.

#### Cons
traints
n == cooldown.length
1 <= n <= 10^5
0 <= links.length <= min(10^5, n * (n - 1) / 2)
links[i].length == 2
0 <= a, b < n and a != b
There are no duplicate channels.
The grid is connected.
cooldown[0] == 0
1 <= cooldown[i] <= 10^5 for 1 <= i < n

#### Example
s & test cases
TEXT
- Input: `links = [[0,1],[0,3],[1,2],[3,2]], cooldown = [0,3,1,2]` -> Output: `8`
  Why: shortest distances from `0` are `dist=[0,1,1,2]`; round-trip times are `2*dist`. Station 2 has round trip `4` and cooldown `1`, so its last resend is at second `3`, whose reply returns at `3+4 = 7`; the grid is idle from `8`. Other stations finish earlier, so the maximum wins.
- Input: `links = [[0,1],[0,2],[1,2]], cooldown = [0,7,7]` -> Output: `3`
  Why: both field stations are distance `1` (round trip `2`) and their cooldown `7` exceeds the round trip, so they never resend; the only reply returns at second `2`, idle from `3`. Edge case: cooldown larger than the round trip means no resends.
- Input: `links = [[0,1]], cooldown = [0,1]` -> Output: `4`
  Why: single field station at distance `1`, round trip `2`; with cooldown `1` it resends at second `1`, and that packet's reply returns at `1+2 = 3`, so idle from `4`. Smallest non-trivial resend case.
- Input: `links = [], cooldown = [0]` -> Output: `1`
  Why: only the coordinator exists, no field station ever sends anything; the grid is idle from second `1`. Minimal / empty case.
- Input: `links = [[0,2],[2,1]], cooldown = [0,1,4]` -> Output: `8`
  Why: chain `0-2-1` gives `dist=[0,1,2]` from station 0. Station 1 (distance 2, round trip 4, cooldown 1) resends at seconds 1,2,3; that last packet's reply returns at `3+4 = 7`, idle from 8. Station 2 (distance 1, round trip 2, cooldown 4) never resends, reply at 2. The maximum, 8, wins — verifies the formula on a longer path.

#### Approach

Because every channel takes exactly one second, the fastest a packet from field station i can reach the coordinator is dist[i] seconds, where dist[i] is the unweighted shortest-path distance from station 0 — computed with a single BFS. The acknowledgement returns along the reverse path, so the full round trip takes rt = 2 * dist[i] seconds; the very first acknowledgement always arrives at second rt. Until that moment the station keeps resending on its cadence, at seconds cooldown[i], 2*cooldown[i], ... The last resend that happens before the reply arrives is at the largest multiple of cooldown[i] that is strictly less than rt, namely lastSend = floor((rt - 1) / cooldown[i]) * cooldown[i]. That final packet's own round trip completes at lastSend + rt, so station i stops contributing traffic at second lastSend + rt and is idle from lastSend + rt + 1. The whole grid is idle from the maximum of that quantity over all field stations (at least 1 when there are none). The underlying pattern is BFS shortest paths plus per-node arithmetic on periodic events.

Complexity

Time: O(n + E), where E = links.length. A single BFS from station 0 computes every shortest distance in linear time, and a final linear scan applies the closed-form timing formula per station. Space: O(n + E) for the adjacency list, distance array, and BFS queue.

Python
PYTHON
from collections import deque
from typing import List

class Solution:
    def earliestIdleSecond(self, links: List[List[int]], cooldown: List[int]) -> int:
        n = len(cooldown)
        adj = [[] for _ in range(n)]
        for a, b in links:
            adj[a].append(b)
            adj[b].append(a)

        # BFS shortest distance from the coordinator (station 0).
        dist = [-1] * n
        dist[0] = 0
        q = deque([0])
        while q:
            u = q.popleft()
            for w in adj[u]:
                if dist[w] == -1:
                    dist[w] = dist[u] + 1
                    q.append(w)

        ans = 1
        for i in range(1, n):
            rt = 2 * dist[i]                                  # round-trip time
            last_send = ((rt - 1) // cooldown[i]) * cooldown[i]  # last resend before reply
            ans = max(ans, last_send + rt + 1)
        return ans
JavaScript
JAVASCRIPT
function earliestIdleSecond(links, cooldown) {

#### cons
t n = cooldown.length;

#### cons
t adj = Array.from({ length: n }, () => []);
    for (const [a, b] of links) {
        adj[a].push(b);
        adj[b].push(a);
    }

    // BFS shortest distance from the coordinator (station 0).

#### cons
t dist = new Array(n).fill(-1);
    dist[0] = 0;

#### cons
t queue = [0];
    let qh = 0;
    while (qh < queue.length) {

#### cons
t u = queue[qh++];
        for (const w of adj[u]) {
            if (dist[w] === -1) {
                dist[w] = dist[u] + 1;
                queue.push(w);
            }
        }
    }

    let ans = 1;
    for (let i = 1; i < n; i++) {

#### cons
t rt = 2 * dist[i];                                     // round trip

#### cons
t lastSend = Math.floor((rt - 1) / cooldown[i]) * cooldown[i]; // last resend < rt
        ans = Math.max(ans, lastSend + rt + 1);
    }
    return ans;
}
Java
JAVA
import java.util.*;

class Solution {
    public int earliestIdleSecond(int[][] links, int[] cooldown) {
        int n = cooldown.length;
        List<List<Integer>> adj = new ArrayList<>();
        for (int i = 0; i < n; i++) adj.add(new ArrayList<>());
        for (int[] e : links) {
            adj.get(e[0]).add(e[1]);
            adj.get(e[1]).add(e[0]);
        }

        // BFS shortest distance from the coordinator.
        int[] dist = new int[n];
        Arrays.fill(dist, -1);
        ArrayDeque<Integer> q = new ArrayDeque<>();
        dist[0] = 0; q.offer(0);
        while (!q.isEmpty()) {
            int u = q.poll();
            for (int w : adj.get(u))
                if (dist[w] == -1) { dist[w] = dist[u] + 1; q.offer(w); }
        }

        int ans = 1;
        for (int i = 1; i < n; i++) {
            int rt = 2 * dist[i];                                 // round trip
            int lastSend = ((rt - 1) / cooldown[i]) * cooldown[i]; // last resend < rt
            ans = Math.max(ans, lastSend + rt + 1);
        }
        return ans;
    }
}
C++
CPP
#include <vector>
#include <queue>
#include <algorithm>
using namespace std;

class Solution {
public:
    int earliestIdleSecond(vector<vector<int>>& links, vector<int>& cooldown) {
        int n = cooldown.size();
        vector<vector<int>> adj(n);
        for (auto& e : links) {
            adj[e[0]].push_back(e[1]);
            adj[e[1]].push_back(e[0]);
        }

        // BFS shortest distance from the coordinator (station 0).
        vector<int> dist(n, -1);
        queue<int> q;
        dist[0] = 0; q.push(0);
        while (!q.empty()) {
            int u = q.front(); q.pop();
            for (int w : adj[u])
                if (dist[w] == -1) { dist[w] = dist[u] + 1; q.push(w); }
        }

        int ans = 1;
        for (int i = 1; i < n; i++) {
            int rt = 2 * dist[i];                                // round trip
            int lastSend = ((rt - 1) / cooldown[i]) * cooldown[i]; // last resend < rt
            ans = max(ans, lastSend + rt + 1);
        }
        return ans;
    }
};
C
C
#include <stdlib.h>

int earliestIdleSecond(int** links, int linksSize, int* linksColSize,
                       int* cooldown, int cooldownSize) {
    int n = cooldownSize;

    /* Build an adjacency list in CSR form. */
    int* deg = (int*)calloc(n, sizeof(int));
    for (int i = 0; i < linksSize; i++) { deg[links[i][0]]++; deg[links[i][1]]++; }
    int* head = (int*)malloc((n + 1) * sizeof(int));
    head[0] = 0;
    for (int i = 0; i < n; i++) head[i + 1] = head[i] + deg[i];
    int* cur = (int*)malloc(n * sizeof(int));
    for (int i = 0; i < n; i++) cur[i] = head[i];
    int* adj = (int*)malloc((size_t)2 * linksSize * sizeof(int) + 1);
    for (int i = 0; i < linksSize; i++) {
        int u = links[i][0], v = links[i][1];
        adj[cur[u]++] = v;
        adj[cur[v]++] = u;
    }

    /* BFS from station 0 -> shortest distance (edge weight = 1). */
    int* dist = (int*)malloc(n * sizeof(int));
    for (int i = 0; i < n; i++) dist[i] = -1;
    int* queue = (int*)malloc(n * sizeof(int));
    int qh = 0, qt = 0;
    dist[0] = 0; queue[qt++] = 0;
    while (qh < qt) {
        int u = queue[qh++];
        for (int e = head[u]; e < head[u + 1]; e++) {
            int w = adj[e];
            if (dist[w] == -1) { dist[w] = dist[u] + 1; queue[qt++] = w; }
        }
    }

    int ans = 1;
    for (int i = 1; i < n; i++) {
        int rt = 2 * dist[i];                          /* round-trip time */
        int lastSend = ((rt - 1) / cooldown[i]) * cooldown[i]; /* last resend before reply */
        int idle = lastSend + rt + 1;
        if (idle > ans) ans = idle;
    }

    free(deg); free(head); free(cur); free(adj); free(dist); free(queue);
    return ans;
}

#### Follow-up
questions
What if channels have different transmission times (weighted edges)? The shortest path is no longer BFS — switch to Dijkstra to get dist[i], keeping the same timing formula.
What if the grid could be disconnected? Decide the semantics for unreachable stations (they never get a reply, so the grid may never become idle) and handle them explicitly.
What if a field station gives up after a fixed maximum number of resends instead of waiting for the reply? Cap the number of send events and recompute the last-send time.
Can you report the idle second per station rather than only the global maximum, without changing the asymptotic cost?

