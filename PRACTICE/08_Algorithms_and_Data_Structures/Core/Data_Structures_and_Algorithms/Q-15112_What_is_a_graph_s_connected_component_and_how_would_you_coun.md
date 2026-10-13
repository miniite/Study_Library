---
type: "practice"
difficulty: "Intermediate"
---

# What is a graph's connected component and how would you count them in an undirected graph?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Graph Connectivity](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Graph_Connectivity.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

What the question is really asking: Two parts. (1) Define a connected component of an undirected graph. (2) Give an algorithm to count how many connected components a given undirected graph has, with the reasoning and complexity. This is an intermediate-experience DSA probe — the interviewer wants a precise definition plus a clean, correct traversal (BFS/DFS or Union-Find) and the right complexity, not hand-waving.

Clarifying questions a strong candidate asks:

Is the graph given as an adjacency list, adjacency matrix, or an edge list with n vertices? This changes both the code and the complexity. I'll show adjacency-list (traversal) and edge-list (Union-Find) forms.
Are there isolated vertices (degree 0)? Yes — each is its own component and must be counted.
Can there be self-loops or parallel edges? They don't affect connectivity, so they're harmless.
Is the graph guaranteed undirected? (For directed graphs the analogous notion is strongly or weakly connected components — a different algorithm; I'll flag that.)
Definition (part 1)

A connected component of an undirected graph is a maximal set of vertices such that every pair of vertices in the set is connected by some path, and no vertex in the set is connected to any vertex outside the set. "Maximal" is the key word: you can't add another vertex and still keep everyone mutually reachable.

Formally, define a relation on vertices: u ~ v iff there is a path from u to v. In an undirected graph this relation is:

reflexive (a vertex reaches itself via the empty path),
symmetric (undirected edges go both ways),
transitive (path u→...→v plus v→...→w gives u→...→w).

So ~ is an equivalence relation, and the connected components are exactly its equivalence classes — a clean partition of the vertex set. A graph with one component is called connected; an isolated vertex forms a component of size 1.

Counting them (part 2) — the idea

Because components partition the vertices, the counting algorithm is: iterate over all vertices; every time you find one you haven't visited yet, you've discovered a brand-new component — increment a counter and then flood-fill (BFS or DFS) to mark that entire component as visited so you never count it again. After the outer loop, the counter equals the number of components. This naturally counts isolated vertices too (an unvisited degree-0 vertex starts and ends its own component).

#### Approach
A — DFS/BFS traversal (adjacency list)
PYTHON
def count_components(n, adj):          # adj: list[list[int]], vertices 0..n-1
    visited = [False] * n
    count = 0
    for v in range(n):
        if not visited[v]:
            count += 1                 # new, undiscovered component
            _bfs(v, adj, visited)      # mark the whole component visited
    return count

from collections import deque
def _bfs(src, adj, visited):
    visited[src] = True
    q = deque([src])
    while q:
        u = q.popleft()
        for w in adj[u]:
            if not visited[w]:
                visited[w] = True
                q.append(w)
Correctness: the outer loop touches every vertex; the if not visited guard ensures each new component is counted exactly once, and the flood-fill guarantees all its members are marked before we move on. No vertex is counted twice; no component is missed.
Complexity: O(V + E) time — each vertex is enqueued/visited once and each edge is examined a constant number of times — and O(V) space for the visited array plus the BFS queue (recursion stack for DFS).

#### Approach
B — Union-Find / Disjoint Set Union (edge list)

If input is an edge list, DSU is often cleaner: start with V components (every vertex its own set), and union the endpoints of every edge; a union that actually merges two distinct sets reduces the component count by one. With path compression + union by rank/size, each operation is near-O(1) (inverse Ackermann α), for O(V + E·α(V)) overall.

PYTHON
def count_components_dsu(n, edges):
    parent = list(range(n))
    rank = [0] * n
    components = n

    def find(x):
        while parent[x] != x:
            parent[x] = parent[parent[x]]   # path compression (halving)
            x = parent[x]
        return x

    nonlocal_hack = [components]
    def union(a, b):
        ra, rb = find(a), find(b)
        if ra == rb:
            return
        if rank[ra] < rank[rb]:
            ra, rb = rb, ra
        parent[rb] = ra
        if rank[ra] == rank[rb]:
            rank[ra] += 1
        nonlocal_hack[0] -= 1               # two sets merged -> one fewer component

    for a, b in edges:
        union(a, b)
    return nonlocal_hack[0]

#### Trade-offs
— which to use
BFS/DFS: simplest when you already have (or can build) an adjacency list; also gives you the actual component membership for free (label vertices by component id). Prefer for static graphs where you traverse once.
Union-Find: best when edges arrive incrementally/online (streaming), when you only need the count/connectivity queries, or in algorithms like Kruskal's MST. It answers "same component?" in near-constant time without re-traversal.

Both are O(V + E) in practice; the choice is about input format and whether the graph is dynamic.

Edge cases & pitfalls
Isolated vertices must be included — a common bug is looping only over edges (Union-Find without initializing all V singletons, or BFS seeded only from edge endpoints), which drops degree-0 vertices. Seed from all vertices.
Disconnected graph is the whole point: the answer is ≥ 1 (and equals V when there are no edges).
Directed graphs: this definition and algorithm are for undirected graphs. For directed graphs use weakly connected components (treat edges as undirected) or strongly connected components (Tarjan/Kosaraju) — do not conflate them.
Self-loops / parallel edges don't change connectivity; safe to ignore.
Summary — what a strong answer emphasizes

Define a connected component as the maximal set of mutually-reachable vertices — i.e. the equivalence classes of the "is there a path" relation, so components partition the vertices. To count: loop over all vertices, and each time you hit an unvisited one, increment the count and BFS/DFS-flood the rest of its component; that's O(V + E) time, O(V) space. Mention Union-Find (near-O(1) per op with path compression + union by rank) as the go-to for edge-list/streaming input, remember to seed every vertex so isolated ones are counted, and flag that directed graphs need SCC/WCC instead.

**Comparison:**

```
Aspect	BFS/DFS traversal	Union-Find (DSU)
Best input format	adjacency list	edge list
Time	O(V + E)	O(V + E·α(V)) ≈ O(V + E)
Space	O(V)	O(V)
Gives component membership	yes (label per flood)	yes (via find(root))
Handles streaming/online edges	no (re-traverse)	yes (incremental)
Typical use	one-shot count/labeling	Kruskal MST, dynamic connectivity
```

**Code:**

```python
# Approach A — BFS traversal over an adjacency list. O(V + E) time, O(V) space.
from collections import deque

def count_components(n, adj):          # adj[v] = list of neighbors of v
    visited = [False] * n
    count = 0
    for v in range(n):
        if not visited[v]:
            count += 1
            q = deque([v]); visited[v] = True
            while q:
                u = q.popleft()
                for w in adj[u]:
                    if not visited[w]:
                        visited[w] = True
                        q.append(w)
    return count

# Approach B — Union-Find over an edge list. ~O(V + E*alpha(V)).
def count_components_dsu(n, edges):
    parent = list(range(n)); rank = [0]*n; comps = n
    def find(x):
        while parent[x] != x:
            parent[x] = parent[parent[x]]      # path compression
            x = parent[x]
        return x
    for a, b in edges:
        ra, rb = find(a), find(b)
        if ra != rb:
            if rank[ra] < rank[rb]: ra, rb = rb, ra
            parent[rb] = ra
            if rank[ra] == rank[rb]: rank[ra] += 1
            comps -= 1                          # merged two sets
    return comps
```

**Example input / output:**

```text
Input : n = 6, edges = [(0,1), (1,2), (3,4)]      # vertex 5 is isolated

BFS trace (Approach A):
  v=0 unvisited -> count=1 -> flood {0,1,2}
  v=1 visited   -> skip
  v=2 visited   -> skip
  v=3 unvisited -> count=2 -> flood {3,4}
  v=4 visited   -> skip
  v=5 unvisited -> count=3 -> flood {5}   (isolated vertex = its own component)
Output: 3     (components: {0,1,2}, {3,4}, {5})

Union-Find trace (Approach B), start comps=6:
  union(0,1): merge -> comps=5
  union(1,2): merge -> comps=4
  union(3,4): merge -> comps=3
Output: 3
```

**Data:**

```text
Example graph (n = 6 vertices, undirected), given as an edge list:
edges = [(0,1), (1,2), (3,4)]
vertex 5 has no edges (isolated).

As an adjacency list:
0: [1]
1: [0, 2]
2: [1]
3: [4]
4: [3]
5: []
```

**Flow:**

1. yes
2. no
3. loop done
4. count = 0; visited all false
5. for each vertex v
6. v visited?
7. count += 1
8. BFS/DFS flood-fill:mark v and all reachablevertices visited
9. return count

