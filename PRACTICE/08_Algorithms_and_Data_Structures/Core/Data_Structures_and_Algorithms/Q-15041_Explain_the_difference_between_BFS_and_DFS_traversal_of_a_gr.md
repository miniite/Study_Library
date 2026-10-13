---
type: "practice"
difficulty: "Intermediate"
---

# Explain the difference between BFS and DFS traversal of a graph.

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Shortest Paths](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Shortest_Paths.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

What's really being asked: Compare BFS (Breadth-First Search) and DFS (Depth-First Search) — the two fundamental graph/tree traversals — across how they explore, what data structure drives them, their complexity, and when to pick which. At "intermediate experience" level (Google/Meta/Uber L1, Ola/Flipkart/Atlassian L2), the interviewer isn't testing whether you can recite definitions; they're probing whether you understand the structural reason each behaves the way it does and whether you can map a real problem to the right traversal. A weak answer says "BFS uses a queue, DFS uses a stack." A strong answer explains why that data-structure choice produces breadth vs depth ordering, states complexity precisely, and gives crisp decision rules.

#### Assumptions
/ clarifying questions I'd raise: Is the graph directed or undirected? Weighted or unweighted? Represented as adjacency list or matrix? Can it have cycles (so I need a visited set)? For this answer I assume a general graph (possibly cyclic, unweighted) stored as an adjacency list — the most common interview setup.

Plan: (1) define each traversal from first principles, (2) show the mechanics with the same graph, (3) compare data structure, memory, and complexity, (4) give the decision matrix of when to use each, (5) note pitfalls.

From first principles

Both BFS and DFS are systematic ways to visit every reachable node exactly once, starting from a source. The only difference is the order in which they expand frontier nodes — and that order is dictated entirely by the data structure that holds the "to-visit" nodes.

BFS — explore level by level

BFS visits the start node, then all its immediate neighbors, then all their unvisited neighbors, and so on — expanding outward in concentric "rings" (levels) of increasing distance from the source. It achieves this with a FIFO queue: because the queue returns nodes in the order they were discovered, all distance-1 nodes come out before any distance-2 node. This "oldest-discovered-first" ordering is exactly what produces level-by-level exploration.

The key consequence: in an unweighted graph, BFS finds the shortest path (fewest edges) from the source to every node. The first time BFS reaches a node, it has done so via a minimum number of hops — a property DFS does not have.

DFS — go deep, then backtrack

DFS picks a neighbor, then a neighbor of that, plunging as deep as possible down one path before it backtracks and tries alternatives. It uses a LIFO stack — either an explicit stack or, more naturally, the call stack via recursion. "Newest-discovered-first" ordering means the most recently seen node is expanded next, which drives the traversal deep along a single branch before unwinding.

DFS is the natural fit for problems about structure and reachability: cycle detection, topological sort, connected components, finding a path (not the shortest), and exhaustive backtracking search.

Why visited matters

In any graph with cycles, both traversals must track a visited set, or they loop forever. In BFS you typically mark a node visited when you enqueue it (to avoid enqueuing duplicates); in DFS you mark when you first enter it.

Complexity

For a graph with V vertices and E edges stored as an adjacency list, both run in O(V + E) time — each vertex is dequeued/entered once and each edge is examined once. With an adjacency matrix it becomes O(V²) for both, because finding neighbors costs O(V) per vertex.

The space cost differs and is often the deciding factor:

BFS space: O(V), but the practical peak is the maximum width of the graph — the largest level. In a wide, bushy graph (e.g., a social graph where one node has millions of neighbors), the queue can balloon.
DFS space: O(V) worst case too, but the practical peak is the maximum depth (recursion/stack depth). In a deep, skinny graph, DFS is cheaper; but on a very deep graph, recursive DFS risks a stack overflow — a real production bug — so you switch to an explicit-stack iterative version.

So the rule of thumb: wide graph → DFS is memory-lighter; deep graph → BFS is memory-lighter.

When to use which
Shortest path in an unweighted graph / minimum number of steps → BFS (word ladder, shortest maze path, "min moves"). For weighted shortest path you upgrade to Dijkstra/0-1 BFS, not plain BFS.
Level-order processing (print tree level by level, "nodes at distance k") → BFS.
Cycle detection, topological sort, connected components, "does a path exist" → DFS (its recursion mirrors the recursive structure of these problems).
Exhaustive/backtracking search (permutations, N-Queens, Sudoku, maze enumeration) → DFS, because it naturally explores one full candidate before backtracking with minimal state.
Web crawling / finding closest results first → BFS. Detecting deadlocks in a resource graph → DFS.

#### Common mistakes
& follow-up probes
Saying BFS finds the shortest path in any graph — it only does so for unweighted edges (or uniform weights). Weighted → Dijkstra.
Forgetting the visited set → infinite loops on cyclic graphs.
Marking BFS nodes visited on dequeue instead of on enqueue → the same node gets enqueued many times, blowing up memory and time.
Using recursive DFS on a huge/deep graph and hitting a stack overflow — mention the iterative fallback.
A great follow-up you should preempt: "How would you find the shortest path if edges have weights?" → Dijkstra (or 0-1 BFS with a deque if weights are 0/1).
Summary — what a strong answer emphasizes

BFS and DFS both visit all reachable nodes once in O(V+E); the difference is order, driven by the container: a FIFO queue gives level-by-level (breadth) exploration and the crucial unweighted-shortest-path property, while a LIFO stack/recursion gives deep-first exploration ideal for reachability, cycles, topological sort, and backtracking. Choose by the question ("shortest hops?" → BFS; "structure/exhaustive search?" → DFS) and by graph shape (wide → DFS memory, deep → BFS memory, with iterative DFS to dodge stack overflow).

**Comparison:**

```
Aspect	BFS	DFS
Exploration order	Level by level (breadth)	Deep along one branch, then backtrack
Data structure	FIFO queue	LIFO stack / recursion (call stack)
Time (adj. list)	O(V + E)	O(V + E)
Time (adj. matrix)	O(V²)	O(V²)
Space (practical peak)	O(max width / level size)	O(max depth)
Shortest path (unweighted)	Yes — first visit = fewest edges	No
Risk on huge graphs	Queue blows up on wide graphs	Stack overflow on deep graphs (use iterative)
Best for	Shortest hops, level-order, closest-first	Cycle detection, topo sort, components, backtracking
```

**Code:**

```python
from collections import deque

# Graph as adjacency list
graph = {
    'A': ['B', 'C'],
    'B': ['A', 'D', 'E'],
    'C': ['A', 'F'],
    'D': ['B'],
    'E': ['B', 'F'],
    'F': ['C', 'E'],
}

def bfs(graph, start):
    """Level-by-level. Queue (FIFO). Finds shortest #edges in unweighted graph."""
    visited = {start}          # mark on enqueue to avoid duplicates
    queue = deque([start])
    order = []
    while queue:
        node = queue.popleft() # oldest-first -> breadth
        order.append(node)
        for nbr in graph[node]:
            if nbr not in visited:
                visited.add(nbr)
                queue.append(nbr)
    return order

def dfs_recursive(graph, start, visited=None, order=None):
    """Deep-first via the call stack."""
    if visited is None: visited, order = set(), []
    visited.add(start)
    order.append(start)
    for nbr in graph[start]:
        if nbr not in visited:
            dfs_recursive(graph, nbr, visited, order)
    return order

def dfs_iterative(graph, start):
    """Same as above but explicit stack -> no recursion-depth limit."""
    visited, stack, order = set(), [start], []
    while stack:
        node = stack.pop()     # newest-first -> depth
        if node not in visited:
            visited.add(node)
            order.append(node)
            for nbr in reversed(graph[node]):  # reversed -> match recursive order
                if nbr not in visited:
                    stack.append(nbr)
    return order
```

**Example input / output:**

```

Graph:

CODE
Copy
    A
   / \
  B   C
 / \   \
D   E---F

adjacency: A:[B,C] B:[A,D,E] C:[A,F] D:[B] E:[B,F] F:[C,E]

CODE
Copy
bfs(graph, 'A')            -> ['A', 'B', 'C', 'D', 'E', 'F']
   # ring 0: A | ring 1: B,C | ring 2: D,E,F  (level by level)

dfs_recursive(graph, 'A')  -> ['A', 'B', 'D', 'E', 'F', 'C']
   # A->B->D (dead end, backtrack) ->E->F->C(visited), backtrack, C(visited). goes DEEP first

BFS dry-run of the queue:

CODE
Copy
enqueue A            queue=[A]              order=[]
pop A, add B,C       queue=[B,C]            order=[A]
pop B, add D,E       queue=[C,D,E]          order=[A,B]
pop C, add F         queue=[D,E,F]          order=[A,B,C]
pop D                queue=[E,F]            order=[A,B,C,D]
pop E (F already in) queue=[F]              order=[A,B,C,D,E]
pop F                queue=[]               order=[A,B,C,D,E,F]
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Very wide
6. Very deep
7. Need fewest edges /shortest hops /level-by-level?
8. Use BFS (queue)
9. Structure problem?cycles / topo-sort /components / backtracking
10. Use DFS (stack / recursion)
11. Graph shape?
12. Prefer DFS (less memory)
13. Prefer BFS(or iterative DFS toavoid stack overflow)

