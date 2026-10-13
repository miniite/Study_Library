---
type: "practice"
difficulty: "Deep"
---

# What is the space complexity of BFS and why does it depend on the graph's width?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Set Operation Complexity](<../../../../08_Algorithms_and_Data_Structures/Core/Complexity_Analysis/Set_Operation_Complexity.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

What the question is really asking: State the space complexity of Breadth-First Search and — more importantly — explain why it is governed by the graph's width (how many nodes sit at the same distance from the source) rather than its depth.

What the interviewer is probing: Do you understand BFS mechanically (the queue, the visited set), and can you connect the shape of the data structure at runtime to the shape of the graph? "Deep understanding" here means you should distinguish BFS from DFS on exactly this axis, quantify the worst case, and reason about real inputs (trees, grids, dense graphs) instead of reciting O(V).

#### Assumptions
/ clarifications I'd state:

Graph has V vertices and E edges; adjacency-list representation.
We want auxiliary space (extra memory BFS allocates), not the input graph's storage.
"Width" = the maximum number of vertices at any single BFS level (the largest frontier). For a tree this is the max nodes on one level; more generally it's the peak size of the queue.

Plan: walk through what BFS actually stores, derive the two contributions to memory (visited set + queue), show the queue is bounded by the width, then give the tight bound and concrete examples.

From first principles: what BFS keeps in memory

BFS explores the graph level by level. It maintains three things:

A queue of nodes to visit next — the "frontier."
A visited/seen set (or a dist/parent array) so we never enqueue a node twice.
The output/bookkeeping we care about (distances, parents), usually O(V).

The classic textbook answer is O(V) overall space, because in the worst case the visited set holds every vertex and the queue can hold up to V nodes. That's correct as an upper bound. But the interesting and practically dominant term — the one that actually spikes memory during a run — is the queue, and the queue's peak size equals the maximum width of the graph.

Why the queue size = the graph's width

Here is the key mechanic. BFS processes nodes in order of increasing distance from the source. At any moment, the queue holds nodes from at most two adjacent levels: the tail of the level currently being drained and the head of the next level being filled as we discover neighbors. So the queue never holds nodes from level 0, level 1, level 2 all at once — only the current frontier and its immediate successors.

Therefore the peak queue size is proportional to the largest level's node count — the maximum width W of the graph. Formally, auxiliary queue space is O(W), where W is the maximum number of vertices at any single distance from the source.

A wide, shallow graph (short but bushy) forces a huge frontier into the queue at once → high memory.
A narrow, deep graph (a long path/chain) keeps the queue tiny — never more than a couple of nodes — even though it has many vertices.

Contrast with DFS, whose stack depth is bounded by the longest path (the graph's height/depth), not its width. This is the crisp symmetry an interviewer wants: BFS ↔ width, DFS ↔ depth.

Concrete numbers
Complete binary tree of n nodes: the last level holds ~n/2 nodes. BFS queue peaks at ~n/2 → O(n) space. DFS peaks at the height, O(log n). This is exactly why BFS on a wide tree is memory-hungry and DFS isn't.
Path graph (a—b—c—…): width is 1 at every level; BFS queue holds ≤ 2 nodes → O(1) queue, and the O(V) comes only from the visited array.
Grid / maze m×n: the frontier is the "wavefront" of the flood fill. On a large grid the diagonal frontier can be O(min(m,n)) or more, which is why grid-BFS memory scales with grid dimension, not just cell count intuition.
Dense/star graph: source connected to V−1 nodes → first expansion enqueues V−1 neighbors → queue is O(V) immediately.
The precise statement
Visited set / dist array: always O(V) — unavoidable, you must remember every vertex to avoid revisiting.
Queue (the width-dependent part): O(W) where W = maximum frontier width. Worst case W = O(V) (wide graph), best case O(1) (path).
Total auxiliary space: O(V + W) = O(V) as a clean upper bound, but the runtime-dominant, input-sensitive term is the width W.

The reason people say "BFS space depends on width" is that the visited-set O(V) term is a fixed floor you pay regardless, whereas the queue term is what varies with graph shape and is what makes BFS blow up on wide graphs and what makes DFS preferable when memory is tight and the graph is wide but shallow.

Why this matters in practice
Choosing BFS vs DFS: if you need shortest paths (unweighted) you must use BFS, but if the graph is very wide you should anticipate large memory and possibly use bidirectional BFS (search from both ends, meeting in the middle — each frontier is smaller, dramatically cutting peak width) or iterative-deepening DFS to trade time for space.
Memory blowups in the wild: web crawlers, social-graph "friends within k hops," and puzzle solvers (15-puzzle, word ladders) are wide-frontier problems where BFS memory, not time, is the limiting resource. Estimating peak frontier size up front tells you whether BFS will even fit in RAM.
Summary — what a strong answer emphasizes
BFS auxiliary space = visited set O(V) (fixed floor) + queue O(W) (varies with graph shape).
The queue holds only the current + next frontier, so its peak = the maximum width W of the graph — the number of nodes at the widest level.
Worst case O(V) (wide/dense), best case O(1) queue (path); the width is the input-sensitive term.
Crisp contrast: BFS memory ∝ width; DFS memory ∝ depth. Mention bidirectional BFS as the mitigation when width is the problem.

#### Common mistakes
saying only "O(V)" without explaining the width dependency; confusing the visited-set term with the queue term; claiming the queue can hold the whole graph across all levels (it holds at most two adjacent levels); forgetting the dist/visited O(V) floor and claiming a path graph is O(1) overall (its queue is O(1), but bookkeeping is still O(V)).

#### Follow-up
probes you should be ready for: "How does bidirectional BFS cut this?" (each side searches to depth d/2, frontier ≈ b^(d/2) instead of b^d). "What if you can't store all visited nodes?" (external-memory BFS, or accept re-expansion). "0-1 BFS / multi-source BFS" — same width logic, just multiple starting frontier nodes.

**Comparison:**

```
Aspect	BFS	DFS
Frontier data structure	Queue (FIFO)	Stack / recursion
Peak auxiliary structure bounded by	Max width W (widest level)	Max depth D (longest path)
Bookkeeping floor	O(V) visited/dist	O(V) visited
Best case (path graph) queue/stack	O(1)	O(V)
Best case (wide bushy tree)	O(V) (bad)	O(D) = O(log V) (good)
Overall space	O(V + W) = O(V)	O(V + D) = O(V)
Mitigation when this term explodes	Bidirectional BFS	Iterative deepening
```

**Code:**

```python
from collections import deque

def bfs_with_peak_width(adj, src):
    """Standard BFS that also reports peak queue size == graph's max width."""
    visited = {src}                 # O(V) floor: must remember every node
    q = deque([src])
    dist = {src: 0}
    peak_width = 1                   # track the largest frontier we ever hold

    while q:
        peak_width = max(peak_width, len(q))   # queue peak == max width W
        node = q.popleft()
        for nxt in adj[node]:
            if nxt not in visited:
                visited.add(nxt)
                dist[nxt] = dist[node] + 1
                q.append(nxt)

    return dist, peak_width           # peak_width is the O(W) queue term
```

**Example input / output:**

```text
Two graphs, same vertex count (V = 7), opposite shapes:

PATH (narrow, deep):  1-2-3-4-5-6-7
  adj = {1:[2],2:[1,3],3:[2,4],4:[3,5],5:[4,6],6:[5,7],7:[6]}
  bfs_with_peak_width(adj, 1) -> dist covers all 7,  peak_width = 2   (queue O(1))

STAR (wide, shallow): center 1 joined to 2..7
  adj = {1:[2,3,4,5,6,7], 2:[1],3:[1],4:[1],5:[1],6:[1],7:[1]}
  bfs_with_peak_width(adj, 1) -> dist covers all 7,  peak_width = 6   (queue O(V))

Same V, same visited-set cost O(V); the QUEUE term swings from 2 to 6 purely
because the STAR's frontier (width) is huge and the PATH's is tiny.
```

**Flow:**

1. peak queue size == widthW
2. Queue at peak holds only ~2 adjacent levels
3. draining current level
4. filling next level
5. Source
6. Level 1 frontier
7. Level 2 frontier - WIDEST
8. Level 3 frontier

