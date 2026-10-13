---
type: "practice"
difficulty: "Intermediate"
---

# Perform level-order traversal of a binary tree and explain why a queue is used.

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Recursion and DP Judgment](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Recursion_and_DP_Judgment.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage
— what is really being asked

Two things are bundled here: (1) perform a level-order traversal (visit the tree level by level, top to bottom, left to right within each level), and (2) explain the "why" — why a queue is the right data structure, as opposed to the stack/recursion used for DFS. This is "intermediate experience," so I should give a clean BFS implementation, the FIFO argument, the complexity, the important "grouped by level" variant (the version interviewers almost always follow up with), and the contrast with DFS. The real probe is whether I understand that the traversal order a data structure produces is dictated by that structure's ordering discipline — FIFO gives breadth-first, LIFO gives depth-first.

#### Assumptions
/clarifications: binary tree, left child before right child within a level. "Visit" = output the value. I'll produce a flat left-to-right order, then show how to emit it as a list-of-levels, since that's the more useful and more commonly tested form.

Plan: state the algorithm → give code → make the FIFO argument rigorously → show the level-grouping trick → complexity and edge cases → dry-run → contrast with DFS → summary.

The algorithm (BFS)

Level-order traversal is breadth-first search (BFS) on a tree:

If the root is null, return (empty).
Put the root into a queue.
While the queue is non-empty: dequeue a node from the front, visit it, then enqueue its left child then its right child (skipping null children).

That's the whole thing. Nodes come out of the queue in the exact order we want to visit them.

Why a queue — the core explanation

A queue is FIFO (First-In, First-Out). That ordering discipline is exactly what level-order requires, and here's the precise reason:

Level ordering is preserved. When we process a node on level k, we enqueue its children (level k+1) at the back of the queue. All nodes of level k were enqueued before any node of level k+1, so — because the queue is FIFO — every level-k node is dequeued before any level-k+1 node. The queue never lets a deeper node "cut in line" ahead of a shallower one. This is what guarantees strict top-to-bottom order.
Left-to-right within a level is preserved too. We always enqueue left child before right child, and a parent that is to the left is dequeued (and thus enqueues its children) before a parent to its right. FIFO carries that left-to-right order down to the next level automatically.

Contrast — why not a stack/recursion? A stack is LIFO; the most recently added node is processed next. If you enqueued children onto a stack, you'd immediately dive into the last child's subtree before finishing the current level — that produces depth-first, not breadth-first, order. Recursion is implicitly a stack, so plain recursion also gives DFS. The single design choice — FIFO queue vs LIFO stack — is what flips a traversal between breadth-first and depth-first. Stating that crisply is the heart of a strong answer.

(There is a recursive way to do level-order — recurse with a level index and append into a per-level list — but it still relies on level bookkeeping to reconstruct FIFO order and is essentially DFS emulating BFS; the queue is the natural, canonical tool.)

The "grouped by level" variant

The most useful production form emits a list of levels (e.g. [[root], [level1...], [level2...]]). The trick: snapshot the queue size at the start of each outer iteration — that size is exactly the number of nodes on the current level. Drain exactly that many nodes into the current level's list, enqueuing their children for the next round. See code_blocks. This is the pattern behind "right-side view," "average of levels," "zigzag traversal," "minimum depth," and "connect level pointers" — all are one small tweak on this loop, which is why interviewers love it.

Complexity
Time: O(n) — each node is enqueued once and dequeued once; O(1) work each.
Space: O(w) where w is the maximum width of the tree (max nodes on any level) — that's the peak queue size. For a perfect/complete tree the bottom level holds up to n/2 nodes, so space is O(n) in the worst case. Contrast: DFS uses O(h) space (height). So on a wide, shallow tree DFS is more memory-efficient; on a deep, narrow (skewed) tree BFS is more memory-efficient and also avoids recursion stack-overflow. That trade-off is a good thing to volunteer.

#### Implementation
note: use an actual FIFO queue with O(1) ends — collections.deque in Python, ArrayDeque/LinkedList in Java. Using a Python list with pop(0) makes each dequeue O(n) and the whole thing O(n²); mentioning this shows practical maturity.

Edge cases
Empty tree → return [] (guard the root).
Single node → [[root]].
Skewed tree → each level has one node; queue holds ≤1 node; still O(n) time.
Dry-run

See input_output — I show the queue's contents step by step, demonstrating that level-1 nodes are fully drained before any level-2 node emerges, which is the FIFO guarantee in action.

Summary — what a strong answer emphasizes
Level-order = BFS: root into queue, then loop {dequeue front, visit, enqueue left then right}.
Queue = FIFO, and FIFO is precisely what preserves top-to-bottom, left-to-right order — children enqueued behind all current-level nodes can never be dequeued before them.
A stack/recursion is LIFO → depth-first; the FIFO-vs-LIFO choice is the whole difference.
Snapshot len(queue) each iteration to group nodes by level — unlocks the whole BFS-on-tree family.
O(n) time, O(w) space (up to O(n) on a wide tree); use a real O(1) deque, not list.pop(0).

#### Common mistakes
/ follow-ups: using list.pop(0) (O(n²)); forgetting the null-children skip; not knowing how to split into levels; and the follow-ups — zigzag (reverse alternate levels) and right-side view (last node of each level) — both reuse this exact loop.

**Comparison:**

```
Aspect	Level-order (BFS, queue)	Pre/in/post-order (DFS, stack/recursion)
Data structure	Queue — FIFO	Stack / recursion — LIFO
Visit pattern	Breadth-first, level by level	Depth-first, subtree by subtree
Time	O(n)	O(n)
Space	O(w) — max width (up to n/2 on wide tree)	O(h) — height
Best when	Deep/narrow trees; need levels/shortest path	Wide/shallow trees; recursive structure
Edge case	Result
Empty tree	[]
Single node	[[root]]
Skewed tree (n nodes)	n levels, each with 1 node
```

**Code:**

```

from collections import deque

class Node: def init(self, val, left=None, right=None): self.val, self.left, self.right = val, left, right

Flat left-to-right level order

def level_order(root): if root is None: return [] out, q = [], deque([root]) while q: node = q.popleft() # FIFO: take from the FRONT out.append(node.val) # visit if node.left: q.append(node.left) # enqueue left then right at BACK if node.right: q.append(node.right) return out

CODE
Copy

```python
# Grouped by level: snapshot queue size = current level's node count
def level_order_grouped(root):
    if root is None:
        return []
    levels, q = [], deque([root])
    while q:
        n = len(q)                  # exactly the nodes on this level
        level = []
        for _ in range(n):
            node = q.popleft()
            level.append(node.val)
            if node.left:  q.append(node.left)
            if node.right: q.append(node.right)
        levels.append(level)
    return levels
```

**Example input / output:**

```text
Tree:
            1
          /   \
         2     3
        / \      \
       4   5      6

Flat level-order:  [1, 2, 3, 4, 5, 6]
Grouped by level:  [[1], [2, 3], [4, 5, 6]]

Queue trace (front ... back), showing FIFO preserves level order:
  start:            [1]
  visit 1, enq 2,3: [2, 3]
  visit 2, enq 4,5: [3, 4, 5]      <- level-1 node 3 still ahead of level-2 nodes
  visit 3, enq 6:   [4, 5, 6]      <- level 1 fully drained before any level-2 visit
  visit 4:          [5, 6]
  visit 5:          [6]
  visit 6:          []
  output order = 1,2,3,4,5,6  (top-to-bottom, left-to-right)
```

**Flow:**

1. yes
2. no
3. yes
4. no
5. root == null?
6. return empty
7. enqueue root
8. queue empty?
9. done
10. node = dequeue FRONT (FIFO)
11. visit node
12. enqueue left child (if any)
13. enqueue right child (if any)

