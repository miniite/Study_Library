---
type: "practice"
difficulty: "Other"
---

# Fewest Phases to Finish All Training Modules

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Topological Sort](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Topological_Sort.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

An onboarding program is made up of k training modules, numbered from 1 to k. Some modules build on others, so they must be completed in a certain order.

You are given the integer k and a list deps, where each entry deps[i] = [a, b] means module a must be fully completed before module b can be started (a is a prerequisite of b).

The program runs in discrete phases. During a single phase you may complete any number of modules at the same time, as long as every prerequisite of each chosen module was already completed in an earlier phase (not the current one).

Return the minimum number of phases needed to complete all k modules. If the prerequisites contain a cycle so that some modules can never be started, return -1.

#### Cons
traints:

1 <= k <= 5000
0 <= deps.length <= 5000
deps[i].length == 2
1 <= a, b <= k
a != b
All pairs [a, b] are unique.

#### Example
s & test cases
TEXT
Input: k = 3, deps = [[1,3],[2,3]] -> Output: 2
Why: modules 1 and 2 have no prerequisites so both finish in phase 1; module 3 then finishes in phase 2.

Input: k = 3, deps = [[1,2],[2,3],[3,1]] -> Output: -1
Why: the dependencies form a cycle 1 -> 2 -> 3 -> 1, so no module ever has all prerequisites met and none can start.

Input: k = 1, deps = [] -> Output: 1
Why: a single module with no prerequisites completes in one phase; the answer is 1, not 0.

Input: k = 4, deps = [[1,2],[2,3],[3,4]] -> Output: 4
Why: a strict chain forces one module per phase, so all four phases are needed (this equals the longest dependency path).

Input: k = 5, deps = [[1,2],[1,3],[2,4],[3,4],[4,5]] -> Output: 4
Why: phase 1 does {1}, phase 2 does {2,3} together, phase 3 does {4}, phase 4 does {5} — a diamond followed by a tail.

Input: k = 2, deps = [] -> Output: 1
Why: two independent modules with no prerequisites are completed together in a single phase.

#### Approach

This is a layered (level-order) topological sort using Kahn's algorithm. Build an adjacency list and an in-degree count for every module. Seed a queue with all modules whose in-degree is zero — these have no prerequisites and can be done in the first phase. Then process the queue one full "layer" at a time: everything currently in the queue belongs to the same phase, so record its size, increment the phase counter, and pop exactly that many modules. As each module is completed, decrement the in-degree of its dependents; any dependent whose in-degree drops to zero becomes eligible and is pushed for the next phase. The number of layers processed is the answer, which is exactly the length of the longest prerequisite chain in the DAG. Finally, if the number of completed modules is less than k, some modules were trapped in a cycle and never reached in-degree zero, so return -1.

Complexity

Time: O(k + E), where E is the number of dependency pairs; building the graph touches every edge once and Kahn's BFS processes each node and each edge exactly once. Space: O(k + E) for the adjacency list, in-degree array, and queue.

Python
PYTHON
from collections import deque
from typing import List

class Solution:
    def minPhases(self, k: int, deps: List[List[int]]) -> int:
        indeg = [0] * (k + 1)
        adj = [[] for _ in range(k + 1)]
        for a, b in deps:
            adj[a].append(b)
            indeg[b] += 1

        q = deque(i for i in range(1, k + 1) if indeg[i] == 0)
        phases = 0
        done = 0
        while q:
            phases += 1
            for _ in range(len(q)):   # drain exactly one phase's worth
                u = q.popleft()
                done += 1
                for v in adj[u]:
                    indeg[v] -= 1
                    if indeg[v] == 0:
                        q.append(v)

        return phases if done == k else -1
JavaScript
JAVASCRIPT
function minPhases(k, deps) {

#### cons
t indeg = new Array(k + 1).fill(0);

#### cons
t adj = Array.from({ length: k + 1 }, () => []);
    for (const [a, b] of deps) {
        adj[a].push(b);
        indeg[b]++;
    }

#### cons
t queue = [];
    for (let i = 1; i <= k; i++) {
        if (indeg[i] === 0) queue.push(i);
    }

    let phases = 0;
    let done = 0;
    let head = 0;
    while (head < queue.length) {

#### cons
t levelEnd = queue.length; // freeze this phase's boundary
        phases++;
        while (head < levelEnd) {

#### cons
t u = queue[head++];
            done++;
            for (const v of adj[u]) {
                if (--indeg[v] === 0) queue.push(v);
            }
        }
    }

    return done === k ? phases : -1;
}
Java
JAVA
import java.util.*;

class Solution {
    public int minPhases(int k, int[][] deps) {
        int[] indeg = new int[k + 1];
        List<List<Integer>> adj = new ArrayList<>();
        for (int i = 0; i <= k; i++) adj.add(new ArrayList<>());
        for (int[] d : deps) {
            adj.get(d[0]).add(d[1]);
            indeg[d[1]]++;
        }

        Deque<Integer> q = new ArrayDeque<>();
        for (int i = 1; i <= k; i++)
            if (indeg[i] == 0) q.offer(i);

        int phases = 0, done = 0;
        while (!q.isEmpty()) {
            int size = q.size();   // one full level = one phase
            phases++;
            for (int i = 0; i < size; i++) {
                int u = q.poll();
                done++;
                for (int v : adj.get(u))
                    if (--indeg[v] == 0) q.offer(v);
            }
        }
        return done == k ? phases : -1;
    }
}
C++
CPP
#include <vector>
#include <queue>
using namespace std;

class Solution {
public:
    int minPhases(int k, vector<vector<int>>& deps) {
        vector<int> indeg(k + 1, 0);
        vector<vector<int>> adj(k + 1);
        for (auto& d : deps) {
            adj[d[0]].push_back(d[1]);
            indeg[d[1]]++;
        }

        queue<int> q;
        for (int i = 1; i <= k; i++)
            if (indeg[i] == 0) q.push(i);

        int phases = 0, done = 0;
        while (!q.empty()) {
            int size = (int)q.size();  // everything at this level = one phase
            phases++;
            while (size--) {
                int u = q.front(); q.pop();
                done++;
                for (int v : adj[u])
                    if (--indeg[v] == 0) q.push(v);
            }
        }
        return done == k ? phases : -1;
    }
};
C
C
#include <stdlib.h>

int minPhases(int k, int **deps, int depsSize, int *depsColSize) {
    int *indeg = calloc(k + 1, sizeof(int));
    int *head  = malloc((k + 1) * sizeof(int));
    for (int i = 0; i <= k; i++) head[i] = -1;

    int cap = depsSize > 0 ? depsSize : 1;
    int *to  = malloc(cap * sizeof(int));
    int *nxt = malloc(cap * sizeof(int));

    for (int i = 0; i < depsSize; i++) {
        int a = deps[i][0], b = deps[i][1];
        to[i]  = b;
        nxt[i] = head[a];
        head[a] = i;         /* adjacency as an intrusive linked list */
        indeg[b]++;
    }

    int *queue = malloc((k + 1) * sizeof(int));
    int qh = 0, qt = 0;
    for (int i = 1; i <= k; i++)
        if (indeg[i] == 0) queue[qt++] = i;

    int phases = 0, done = 0;
    while (qh < qt) {
        int levelEnd = qt;   /* freeze this phase's boundary */
        phases++;
        while (qh < levelEnd) {
            int u = queue[qh++];
            done++;
            for (int e = head[u]; e != -1; e = nxt[e]) {
                int v = to[e];
                if (--indeg[v] == 0) queue[qt++] = v;
            }
        }
    }

    free(indeg); free(head); free(to); free(nxt); free(queue);
    return done == k ? phases : -1;
}

#### Follow-up
questions
Return one valid phase-by-phase schedule (which modules run in each phase), not just the phase count.
Suppose each module takes a given number of days and phases may overlap once prerequisites finish — compute the earliest overall completion time (critical-path / longest weighted path).
When the answer is -1, identify and report the actual set of modules involved in the cycle.
How does the memory footprint change for a very sparse graph with millions of modules but few dependencies, and how would you keep it O(k + E)?

