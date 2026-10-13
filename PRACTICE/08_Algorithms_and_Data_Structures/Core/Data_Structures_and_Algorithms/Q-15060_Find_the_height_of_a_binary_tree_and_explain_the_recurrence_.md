---
type: "practice"
difficulty: "Intermediate"
---

# Find the height of a binary tree and explain the recurrence you use.

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Binary Tree Traversal](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Binary_Tree_Traversal.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage
— what is really being asked

This is an "intermediate experience" question, so the interviewer isn't just asking me to recite a one-liner — they want the recurrence stated precisely, the base case pinned down, the height-vs-depth definition clarified, both recursive and iterative implementations, and the complexity with edge cases. The real probe is: do you reason about tree problems the standard way — "solve for the children, then combine at the parent" — and do you get the off-by-one and the empty-tree convention right? Those two details (base case value and the +1) are exactly where people slip.

Clarifying question I'd raise first: how do we define the height of an empty tree, and do we count nodes or edges? Definitions differ, and interviewers respect candidates who surface this:

Edge-based (most common, and what I'll use): height = number of edges on the longest root-to-leaf path. A single node has height 0; an empty tree has height -1.
Node-based: height = number of nodes on that path. A single node has height 1; an empty tree has height 0.

I'll present the edge-based version and note exactly how to switch to node-based (change only the two base-case constants). Height is measured downward from a node to its deepest descendant; depth is measured upward from a node to the root. The question asks for height, so I compute it for the root.

First principles: the recurrence

A tree is recursive by nature — a node's subtrees are themselves trees — so height has a clean recursive definition. The height of a node is 1 more than the taller of its two children's heights, because to reach the deepest leaf you take one edge down into whichever subtree is taller, then continue optimally within it:

CODE
height(node) = -1                                   if node is null      (base case)
height(node) = 1 + max(height(left), height(right)) otherwise

Why this is correct: the longest downward path from node must go through one of its children (or nowhere, if it's a leaf). If it goes through the left child, its length is 1 + height(left); through the right, 1 + height(right). The longest path is the max of these two options, hence 1 + max(...).

Why the base case is -1 (edge convention): consider a leaf node. Both its children are null, so height(leaf) = 1 + max(height(null), height(null)) = 1 + max(-1, -1) = 1 + (-1) = 0. That correctly gives a leaf height 0. If I instead defined the empty tree as 0, a leaf would come out as 1, which is the node-based convention. So the single constant in the base case is what selects the convention — that's a great thing to say out loud, because it shows you understand why the number is what it is, not just that it "is -1."

Deriving it as an algorithm (post-order)

Notice the structure: to compute a node's height I must already have both children's heights — I can only combine after recursing into both subtrees. That is exactly a post-order (L, R, N) traversal: solve left, solve right, then compute self. This is the canonical shape of "tree DP" / bottom-up tree problems (size, diameter, balanced-check, sum) — same skeleton, different combine step. Being able to name this pattern is what "intermediate experience" is checking for.

Complexity
Time: O(n) — every node is visited exactly once and does O(1) work (a compare and an add).
Space: O(h) for the recursion stack, where h is the height itself. For a balanced tree that's O(log n); for a skewed/degenerate tree (a linked list) it's O(n) — worth calling out because a very deep tree can stack-overflow the recursion. The mitigation is the iterative BFS/level-count version (see code_blocks), which uses a queue and counts levels; its space is O(w) where w is the max width. So recursion trades better cache/simplicity for stack-depth risk; BFS avoids the deep-recursion crash. In production I'd use the iterative version if inputs can be adversarially deep, or convert to an explicit-stack DFS.
Edge cases to state
Empty tree → -1 (edge) or 0 (node) — must match your stated convention.
Single node → 0 (edge).
Skewed tree of n nodes → height n-1 (edge); this is also the worst case for recursion depth.
Perfectly balanced tree of n nodes → height ⌊log2 n⌋.
Dry-run

See input_output — I trace the recursion bottom-up on a small tree and show that a leaf resolves to 0 and the root to 2, confirming the +1/max combine.

Summary — what a strong answer emphasizes
State the recurrence exactly: height(node) = 1 + max(height(left), height(right)), base height(null) = -1.
Justify the base-case constant by the leaf case — it selects the edge-vs-node convention.
Identify it as a post-order / bottom-up computation ("solve children, combine at parent").
O(n) time, O(h) space; skewed tree → O(n) stack → overflow risk → use iterative BFS.
Clarify height vs depth and the empty-tree convention up front.

#### Common mistakes
/ follow-ups: off-by-one from a wrong base case; confusing height with depth; forgetting the skewed-tree stack blow-up; and the natural follow-up — "now compute the tree's diameter" — which reuses this exact height function (max of left-height + right-height + 2 over all nodes, computed in the same single post-order pass).

**Comparison:**

```
Case	Height (edge convention)	Note
Empty tree (null)	-1	base case
Single node	0	leaf: 1 + max(-1,-1)
n-node skewed tree	n - 1	worst case; O(n) recursion depth
n-node perfectly balanced	⌊log2 n⌋	best/typical
Approach	Time	Space	When to prefer
Recursive (post-order)	O(n)	O(h) stack	Simple; fine for balanced/bounded-depth trees
Iterative BFS level-count	O(n)	O(w) queue	Deep/adversarial trees — avoids stack overflow
```

**Code:**

```

class Node: def init(self, val, left=None, right=None): self.val, self.left, self.right = val, left, right

Recursive (post-order). Edge convention: height(null) = -1, leaf = 0.

def height(node): if node is None: return -1 # base case; use 0 here for node-count convention return 1 + max(height(node.left), height(node.right))

CODE
Copy

```python
from collections import deque

# Iterative BFS level-count — avoids deep recursion / stack overflow.
# Edge convention: returns levels - 1. (Return `levels` for node convention.)
def height_iter(root):
    if root is None:
        return -1
    q = deque([root])
    levels = 0
    while q:
        for _ in range(len(q)):        # drain exactly one level
            n = q.popleft()
            if n.left:  q.append(n.left)
            if n.right: q.append(n.right)
        levels += 1
    return levels - 1
```

**Example input / output:**

```text
Tree:
            A            depth 0
          /   \
         B     C         depth 1
        / \
       D   E             depth 2

height(D) = 1 + max(height(null), height(null)) = 1 + max(-1,-1) = 0   (leaf)
height(E) = 0                                                          (leaf)
height(C) = 0                                                          (leaf)
height(B) = 1 + max(height(D), height(E)) = 1 + max(0,0) = 1
height(A) = 1 + max(height(B), height(C)) = 1 + max(1,0) = 2   <-- answer

Edge convention: height = 2 (longest path A->B->D has 2 edges).
Node convention: height = 3 (3 nodes on that path).

Empty tree -> -1 (edge) / 0 (node).
Skewed A->B->C->D (4 nodes) -> height 3 (edge).
```

**Flow:**

1. yes
2. no
3. height(node)
4. node == null?
5. return -1
6. hL = height(node.left)
7. hR = height(node.right)
8. return 1 + max(hL, hR)

