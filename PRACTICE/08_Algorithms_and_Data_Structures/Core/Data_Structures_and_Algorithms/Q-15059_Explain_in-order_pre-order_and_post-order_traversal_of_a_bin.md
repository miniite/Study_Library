---
type: "practice"
difficulty: "Basic"
---

# Explain in-order, pre-order, and post-order traversal of a binary tree.

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Binary Tree Traversal](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Binary_Tree_Traversal.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage
— what is really being asked

The interviewer wants me to explain the three depth-first (DFS) traversals of a binary tree — in-order, pre-order, and post-order — clearly enough that they know I understand what order the nodes are visited in, why, and where each is used. This is a "basic concept" question, so the bar is: define a binary tree node, state the exact visit order of each traversal, show the recursion, dry-run one small tree, and mention practical uses and the complexity. A strong candidate also notes that all three are the same recursive skeleton — only the position of the "process this node" step moves.

Clarifying assumptions: a binary tree is a tree where every node has at most two children, a left child and a right child (order matters — left is not interchangeable with right). "Visit"/"process" a node usually means print or collect its value. I'll assume nodes can be null (empty subtree), which is the base case of every traversal.

Plan: define the node → give the recursive rule for each order → show the code → dry-run on one tree → note iterative forms, complexity, and real uses → summarize.

First principles: the shared skeleton

Every DFS traversal does exactly three things for a node: process the node itself (N), recurse into the left subtree (L), and recurse into the right subtree (R). The only difference between the three traversals is where N sits relative to L and R:

Pre-order = N, L, R — process the node before its subtrees ("root first").
In-order = L, N, R — process the node between its subtrees ("root in the middle").
Post-order = L, R, N — process the node after its subtrees ("root last").

The base case in all three is the same: if the node is null, do nothing and return. That single rule is what terminates the recursion at the leaves.

A memory aid: the prefix (pre/in/post) tells you when the root is touched — first, middle, or last — and L always comes before R.

Walking through each traversal

Pre-order (N → L → R): you output the current node, then everything in its left subtree, then everything in its right subtree. Because the root is emitted first and each subtree's root is emitted before its own children, pre-order gives you a top-down picture of the tree. This is exactly the order you'd use to copy/serialize a tree (you know the root before the children, so you can rebuild it), or to render a prefix expression / directory listing where the parent must be created before its contents.

In-order (L → N → R): you fully process the left subtree, then the node, then the right subtree. The key fact every interviewer wants to hear: for a Binary Search Tree (BST), in-order traversal visits the nodes in sorted (ascending) order. That is because in a BST everything on the left is smaller and everything on the right is larger, so L-N-R naturally yields sorted keys. In-order is also how you evaluate/print an infix arithmetic expression.

Post-order (L → R → N): you process both subtrees first, and the node last. This is the order you use when a node's work depends on its children being done — e.g. deleting/freeing a tree (free the children before the parent so you don't lose the pointers), computing the height/size of a subtree (you need the children's results before combining at the parent), or evaluating a postfix expression.

Code (recursive)

The recursive versions are near-identical — only the line where you visit the node moves. See code_blocks. The iterative form uses an explicit stack (because recursion is itself stack-based); in-order iterative and post-order iterative are slightly trickier because you must remember whether you've already visited the left/right side.

Complexity
Time: O(n) for all three — every node is visited exactly once.
Space: O(h) for the recursion/explicit stack, where h is the tree height. For a balanced tree h ≈ log n, so O(log n); for a degenerate (skewed) tree h = n, so O(n) worst case. (Morris traversal can do in-order in O(1) extra space by temporarily threading the tree — a good follow-up to mention.)
Dry-run

For the tree in input_output, the three orders come out distinct, and — crucially — because that tree happens to be a BST, its in-order result is sorted. Showing that connection out loud is what separates a memorized answer from an understood one.

Summary — what a strong answer emphasizes
All three are the same recursion; only the position of "visit node" changes: pre = N L R, in = L N R, post = L R N.
In-order on a BST = sorted order (the single most-tested fact).
Pre-order rebuilds/serializes a tree top-down; post-order is for bottom-up work (delete, height, evaluate-postfix).
All are O(n) time, O(h) space; Morris makes in-order O(1) space.

#### Common mistakes
/ follow-up probes: confusing which order gives sorted output (it's in-order, not pre-order); forgetting the null base case (causes crashes); saying pre-order can rebuild a tree alone (you need pre+in, or a null-marked pre-order); and not knowing the iterative/stack version, which interviewers often ask next.

**Comparison:**

```
Traversal	Visit order	On a BST gives	Typical use
Pre-order	N, L, R (root first)	root-before-children	Serialize/copy a tree, prefix expression, dir listing
In-order	L, N, R (root middle)	sorted ascending	Print BST in order, infix expression
Post-order	L, R, N (root last)	children-before-root	Delete/free tree, compute height/size, evaluate postfix
Aspect	All three DFS traversals
Time	O(n) — each node once
Space	O(h): O(log n) balanced, O(n) skewed
Base case	node is null → return
```

**Code:**

```

class Node: def init(self, val, left=None, right=None): self.val = val self.left = left self.right = right

def pre_order(node, out): # N, L, R if node is None: return out.append(node.val) # visit N pre_order(node.left, out) # L pre_order(node.right, out) # R

def in_order(node, out): # L, N, R if node is None: return in_order(node.left, out) # L out.append(node.val) # visit N in_order(node.right, out) # R

def post_order(node, out): # L, R, N if node is None: return post_order(node.left, out) # L post_order(node.right, out) # R out.append(node.val) # visit N

CODE
Copy

```python
# Iterative in-order using an explicit stack (O(h) space)
def in_order_iter(root):
    out, stack, cur = [], [], root
    while cur or stack:
        while cur:              # go as far left as possible
            stack.append(cur)
            cur = cur.left
        cur = stack.pop()       # leftmost unvisited node
        out.append(cur.val)     # visit N
        cur = cur.right         # then go right
    return out
```

**Example input / output:**

```text
Tree (also a BST):

            8
          /   \
         3     10
        / \      \
       1   6      14
          / \    /
         4   7  13

Pre-order  (N L R): 8, 3, 1, 6, 4, 7, 10, 14, 13
In-order   (L N R): 1, 3, 4, 6, 7, 8, 10, 13, 14   <-- SORTED (because it's a BST)
Post-order (L R N): 1, 4, 7, 6, 3, 13, 14, 10, 8

Trace of in-order at node 3:
  in(1) -> [1]
  visit 3 -> [1,3]
  in(6): in(4)->[...,4]; visit 6->[...,4,6]; in(7)->[...,4,6,7]
  => subtree(3) yields 1,3,4,6,7
```

**Flow:**

1. yes
2. no
3. traverse(node)
4. node == null?
5. return
6. Pre-order: visit N here
7. recurse LEFT
8. In-order: visit N here
9. recurse RIGHT
10. Post-order: visit N here

