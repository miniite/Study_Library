---
type: "practice"
difficulty: "Intermediate"
---

# Given a BST, find the lowest common ancestor of two nodes.

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Binary Search Trees](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Binary_Search_Trees.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage
— what is really being asked

The question: given a Binary Search Tree (BST) and two node values p and q, find their Lowest Common Ancestor (LCA) — the deepest node in the tree that has both p and q in its subtree (a node can be an ancestor of itself).

What the interviewer is probing at the "intermediate" level:

Do you recognize that this is a BST, not a generic binary tree? The BST ordering property lets you solve it in O(h) without visiting the whole tree — using the generic-tree algorithm here would work but signals you missed the key insight.
Can you write both an iterative (O(1) space) and recursive version and reason about complexity?
Do you handle assumptions cleanly (are p and q guaranteed present? are values unique?).

Clarifying questions I'd ask: Are all values unique (standard for LCA problems — yes)? Are p and q guaranteed to exist in the tree (usually yes; if not, we'd need a presence check)? Is it a valid BST (no duplicates violating the invariant)? I'll assume: valid BST, unique values, both nodes present.

First principles — the BST property is the whole trick

A BST invariant: for any node n, every value in n.left subtree is < n.val, and every value in n.right subtree is > n.val.

Now think about where p and q sit relative to some node n while we walk down from the root:

If both p and q are smaller than n.val, then both live entirely in the left subtree. The LCA cannot be n — it must be deeper on the left. So move n = n.left.
If both p and q are larger than n.val, both live in the right subtree — move n = n.right.
Otherwise the paths to p and q diverge at n (one goes left/equals, the other goes right/equals, or n is p or q). That node n is the split point — it is the LCA. Return it.

That "first node where the two values fall on opposite sides (or equal)" is precisely the lowest common ancestor. Because we only ever descend one branch, we never explore siblings, giving O(h) time.

Algorithm (iterative — the answer I'd write first)

Start at the root. Loop:

if p.val < n.val and q.val < n.val → n = n.left
else if p.val > n.val and q.val > n.val → n = n.right
else → return n

The else fires the moment the two are no longer on the same side, which includes the case where n equals p or q (then the other is in n's subtree, so n is the ancestor of both — correct, since a node is an ancestor of itself).

Complexity
Time: O(h) where h is the tree height. Balanced BST → O(log n); degenerate (a "linked list" BST) → O(n).
Space: O(1) iterative (no recursion stack, no auxiliary structure). The recursive version is O(h) stack space.

This beats the generic-binary-tree LCA (which is O(n) time, O(h) space because it must search both subtrees). The BST ordering is what buys us the shortcut.

Recursive version (equivalent)

Same three-way decision, expressed recursively. It's clean but uses O(h) stack. In an interview I'd present the iterative one as the "optimal" answer and mention the recursive one as a stylistic variant.

Edge cases & correctness
p == q: LCA is that node itself; the algorithm returns it (both comparisons put it on the "same side" until we hit it, then the else returns it).
One node is an ancestor of the other (e.g., p is on the path to q): when we reach p, p.val is neither strictly less nor strictly greater than itself, so we hit else and return p — correct.
Root itself is the LCA: handled — if p and q straddle the root's value, we return the root immediately.
Nodes not present: with the standard guarantee they exist, no check is needed. If not guaranteed, I'd first confirm both exist (or do a search along the descent and validate), otherwise the returned "split node" may be meaningless.

#### Common mistakes
/ follow-up probes
Using the generic binary-tree recursion (recurse left; recurse right; if both non-null return node). It's correct but O(n) — the interviewer wants you to exploit the BST.
Comparing node identity instead of values — fine if you have the node objects, but if you only have values, compare values. Be consistent.
Off-by-one on the equality case — forgetting that n can be p or q. Using strict < and > (not <=) so the equal case falls into else is what handles ancestor-is-a-node correctly.

#### Follow-up
"What if it's NOT a BST?" → switch to the O(n) generic recursion. "What if you have parent pointers?" → walk up from each node collecting ancestors (or use the two-pointer "align depths then move together" trick, O(h) time O(1) space). "Values not unique?" → BSTs with duplicates are ambiguous; clarify the duplicate convention first.
What a strong answer emphasizes

State the BST property, derive the three-way descent from it, give the O(h) time / O(1) space iterative solution, and correctly justify the equality/ancestor edge cases. Mention the fallback to O(n) generic LCA if the tree weren't a BST. That combination — right insight, tight complexity, clean edge-case reasoning — is a complete intermediate answer.

**Comparison:**

```
Approach	Uses BST property?	Time	Space	Notes
BST descent (iterative)	Yes	O(h) → O(log n) balanced, O(n) skewed	O(1)	Optimal; recommended
BST descent (recursive)	Yes	O(h)	O(h) stack	Clean but uses stack
Generic binary-tree LCA	No	O(n)	O(h)	Fallback when not a BST
Parent-pointer / align-depths	n/a	O(h)	O(1)	If parent pointers available
```

**Code:**

```
Definition of a BST node

class TreeNode: def init(self, val, left=None, right=None): self.val = val self.left = left self.right = right

Iterative — O(h) time, O(1) space (optimal)

def lowest_common_ancestor(root, p, q): node = root while node: if p.val < node.val and q.val < node.val: node = node.left # both in left subtree elif p.val > node.val and q.val > node.val: node = node.right # both in right subtree else: return node # split point (or node == p/q) -> LCA return None # only if p or q absent

Recursive — O(h) time, O(h) stack space (equivalent)

def lca_recursive(root, p, q): if root is None: return None if p.val < root.val and q.val < root.val: return lca_recursive(root.left, p, q) if p.val > root.val and q.val > root.val: return lca_recursive(root.right, p, q) return root

CODE
Copy
```java
// Java iterative version
class Solution {
    public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
        TreeNode node = root;
        while (node != null) {
            if (p.val < node.val && q.val < node.val)      node = node.left;
            else if (p.val > node.val && q.val > node.val) node = node.right;
            else return node;
        }
        return null;
    }
}
```

**Example input / output:**

```

Example BST:

CODE
Copy
            6
          /   \
         2     8
        / \   / \
       0   4 7   9
          / \
         3   5

Trace for p=2, q=8:

node=6: 2<6 and 8<6? no. 2>6 and 8>6? no. -> diverge -> return 6. ✅ LCA = 6

Trace for p=2, q=4:

node=6: both < 6 -> go left, node=2
node=2: 2<2? no (strict). 4<2? no. -> else -> return 2. ✅ LCA = 2 (This is the "ancestor is a node" case: 2 is an ancestor of 4.)

Trace for p=3, q=5:

node=6: both < 6 -> node=2
node=2: both > 2 -> node=4
node=4: 3<4 and 5<4? no. 3>4 and 5>4? no. -> return 4. ✅ LCA = 4

Trace for p=0, q=5:

node=6: both < 6 -> node=2
node=2: 0<2 and 5<2? no. 0>2 and 5>2? no. -> return 2. ✅ LCA = 2
```

**Flow:**

1. no
2. yes
3. yes
4. no
5. yes
6. no
7. node = root
8. node != null?
9. return null / absent
10. p.val < node.val AND q.val <node.val?
11. node = node.left
12. p.val > node.val AND q.val >node.val?
13. node = node.right
14. return node = LCA

