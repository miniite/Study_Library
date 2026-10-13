---
type: "practice"
difficulty: "Intermediate"
---

# What is the time complexity of BFS and DFS on a graph with V vertices and E edges?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Set Operation Complexity](<../../../../08_Algorithms_and_Data_Structures/Core/Complexity_Analysis/Set_Operation_Complexity.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

The question asks for the time complexity of BFS (breadth-first search) and DFS (depth-first search) on a graph with V vertices and E edges. Both are full-graph traversals, and the expected, universally accepted answer is O(V + E) — but the interviewer wants more than the formula. They're probing whether you can derive it: where does the V term come from, where does the E term come from, and why is it a sum rather than a product like O(V*E).

#### Assumptions
to state: V = number of vertices, E = number of edges. The complexity O(V + E) assumes the graph is stored as an adjacency list. (With an adjacency matrix the traversal becomes O(V^2), which I'll note explicitly, because the representation changes the answer.) I'll assume we traverse the whole graph, handling disconnected components too.

Plan: describe what BFS/DFS do, account for the per-vertex work (the V term), account for the per-edge work (the E term), explain why they add, give space complexity, then cover the matrix variant and edge cases.

Deriving O(V + E) from first principles

Both BFS and DFS visit every reachable vertex exactly once and, from each visited vertex, examine each of its outgoing edges exactly once. The mechanism differs — BFS uses a queue (FIFO) and explores level by level; DFS uses a stack or recursion and goes deep first — but the amount of work is accounted for identically.

The key device that guarantees "each vertex once" is a visited/marked array. Before we recurse or enqueue a vertex, we check the marker; if already visited, we skip. This turns what could be exponential re-exploration into linear work.

The V term — visiting vertices

Every vertex is dequeued (BFS) or popped/entered (DFS) exactly once, because after the first visit it is marked and never re-processed. That's V units of constant work for the visit bookkeeping (mark it, record distance/parent, etc.). To also start traversal in every disconnected component, we loop for each vertex: if not visited, run BFS/DFS from it — that outer loop is also V iterations. Both contribute the O(V) term.

The E term — scanning edges

When we process a vertex u, we walk its adjacency list to look at every neighbor v. The cost of processing u is proportional to degree(u) (the number of edges incident to u). Summing over all vertices:

Undirected graph: sum of degree(u) over all u = 2E (each edge contributes to two endpoints' lists). So total edge-scanning work = O(2E) = O(E).
Directed graph: sum of out-degree(u) = E (each directed edge appears in exactly one list). Total = O(E).

Either way, across the entire traversal we examine each edge a constant number of times, giving the O(E) term. Crucially, each edge is examined only a fixed number of times — not once per vertex — because the visited check stops re-expansion.

Why it's O(V + E), not O(V * E)

This is the trap. The vertex work and the edge work are separate, additive budgets: V vertex-visits plus 2E (or E) edge-inspections. We don't do E edge work for each vertex; we do a total of 2E edge inspections spread across all vertices. The correct combination is a sum: O(V + E).

A useful way to see it: total work = (work to touch every vertex once) + (work to touch every edge a constant number of times) = O(V) + O(E) = O(V + E).

Space complexity
BFS: the queue can hold up to O(V) vertices in the worst case (e.g., a wide level near the root, or a star graph). Plus the visited array O(V). So space = O(V).
DFS: the recursion stack (or explicit stack) can go as deep as O(V) in the worst case (a path/skewed graph). Plus visited array O(V). So space = O(V).

Both are O(V) auxiliary space (the adjacency list storage of O(V + E) is the input, not extra).

The representation matters: matrix vs list

The O(V + E) result assumes an adjacency list. If the graph is stored as an adjacency matrix, finding a vertex's neighbors requires scanning an entire row of V cells (most of which are zeros). Doing that for all V vertices costs V x V = V^2, so:

BFS/DFS on adjacency list → O(V + E).
BFS/DFS on adjacency matrix → O(V^2).

Always mention which representation you assume; a sharp interviewer will pin you on it.

Sanity numbers
Sparse graph, V = 1,000,000, E = 3,000,000 (avg degree 3): O(V + E) ≈ 4,000,000 ops — fast. On a matrix it'd be 10^12 — infeasible.
Dense graph, E ≈ V^2: O(V + E) = O(V + V^2) = O(V^2) — here list and matrix converge because there genuinely are ~V^2 edges.
Summary — what a strong answer emphasizes
BFS and DFS are both O(V + E) on an adjacency-list graph.
The V term = each vertex visited once (guarded by a visited array) plus the outer loop over components.
The E term = each edge scanned a constant number of times (2E undirected, E directed) while expanding adjacency lists.
It's additive (V + E), not multiplicative (V*E) — separate budgets.
Space is O(V) for both (queue for BFS, recursion/stack for DFS, plus visited).
On an adjacency matrix both become O(V^2) because neighbor lookup costs O(V) per vertex.

#### Common mistakes
answering O(V*E); forgetting the visited array (leads to exponential blowup, and the wrong complexity); saying O(E) alone (ignores isolated vertices / the vertex-visiting cost); not stating the representation assumption. Follow-up probes: "Prove why not O(V^2) for the list" (sum of degrees = 2E, not V per vertex); "What changes for a matrix?" (O(V^2)); "Space?" (O(V)); "BFS vs DFS when memory-constrained?" (DFS recursion depth vs BFS queue width, both O(V) worst case but differ by graph shape).

**Comparison:**

```
Aspect	BFS	DFS
Time (adjacency list)	O(V + E)	O(V + E)
Time (adjacency matrix)	O(V^2)	O(V^2)
Auxiliary space	O(V) (queue)	O(V) (recursion/stack)
Data structure	Queue (FIFO)	Stack / recursion (LIFO)
Explores	Level by level	Deep first
Term	Source	Cost
V	each vertex visited once + outer component loop	O(V)
E	each edge scanned; sum of degrees = 2E (undirected) / E (directed)	O(E)
Total	additive, not multiplicative	O(V + E)
```

**Code:**

```python
from collections import deque

def bfs(adj, start, V):
    """adj: adjacency list (list of lists). O(V + E) time, O(V) space."""
    visited = [False] * V
    order = []
    q = deque([start])
    visited[start] = True          # mark BEFORE enqueue -> each vertex once
    while q:                        # each vertex dequeued once -> O(V)
        u = q.popleft()
        order.append(u)
        for v in adj[u]:            # scans each edge once -> total O(E)
            if not visited[v]:
                visited[v] = True
                q.append(v)
    return order

def dfs(adj, start, V):
    """Recursive DFS. O(V + E) time, O(V) recursion-stack space."""
    visited = [False] * V
    order = []
    def go(u):
        visited[u] = True          # each vertex entered once -> O(V)
        order.append(u)
        for v in adj[u]:           # each edge examined once -> total O(E)
            if not visited[v]:
                go(v)
    go(start)
    return order

def full_traverse(adj, V):
    """Handles disconnected components: outer loop is the extra O(V)."""
    visited = [False] * V
    for u in range(V):             # V iterations
        if not visited[u]:
            # start a BFS/DFS from u (marks its whole component)
            ...
```

**Example input / output:**

```text
Input: adjacency list above, start = 0, V = 5.

BFS dry run (queue shown):
- enqueue 0 → visit 0, enqueue 1,2 → queue [1,2]
- dequeue 1 → visit 1, enqueue 3 (0 already visited) → queue [2,3]
- dequeue 2 → visit 2, enqueue 4 → queue [3,4]
- dequeue 3 → visit 3 (1 visited) → queue [4]
- dequeue 4 → visit 4 (2 visited) → queue []
- Output order: [0, 1, 2, 3, 4]

DFS dry run from 0: 0 → 1 → 3 (back) → 2 → 4 → Output order: [0, 1, 3, 2, 4]

Work accounting: 5 vertex visits (V=5) + 8 edge inspections (2E=8) = 13 units → O(V + E).
```

**Data:**

```

Adjacency list for a 5-vertex, 4-edge undirected graph (edges 0-1, 0-2, 1-3, 2-4):

CODE
Copy
adj[0] = [1, 2]
adj[1] = [0, 3]
adj[2] = [0, 4]
adj[3] = [1]
adj[4] = [2]

Sum of list lengths = 2 + 2 + 2 + 1 + 1 = 8 = 2E (E = 4). Confirms edge-scan work = 2E = O(E).
```

**Flow:**

1. yes
2. no
3. yes
4. no
5. list exhausted
6. Pick start vertex, mark visited
7. Container empty?
8. Traversal done: O of V plus E
9. Take next vertex u
10. Scan adj list of u -> degree uedges
11. Neighbor v unvisited?
12. Mark v, add to container
13. Skip v
14. BFS uses a queue for the container; DFS uses a stack/recursion. Same work accounting either way.

