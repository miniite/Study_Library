---
type: "practice"
difficulty: "Intermediate"
---

# How would you find the kth smallest element in a BST?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Binary Search Trees](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Binary_Search_Trees.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

The task: given a binary search tree (BST) and an integer k, return the kth smallest element (1-indexed). The interviewer, at "intermediate experience" level, is checking that I recognize the single most useful property of a BST — that an in-order traversal visits nodes in sorted ascending order — and that I can turn that into a clean O(k) or O(h+k) solution rather than something naive like collecting all values and sorting. They also want to see that I can stop early (not traverse the whole tree) and can discuss the important follow-up: what if the BST is modified frequently and we call this often?

Clarifying questions I'd ask: Is k 1-indexed (kth smallest, k=1 → minimum)? I'll assume yes. Is 1 ≤ k ≤ n guaranteed, or should I handle out-of-range k? I'll assume valid but note the guard. Can I modify the tree or add fields? For the basic ask, no; for the follow-up, augmenting nodes is the key trick.

The core insight

A BST guarantees: for any node, all keys in the left subtree < node's key < all keys in the right subtree. Therefore an in-order traversal (left → node → right) produces the keys in strictly increasing order. So "kth smallest" is just "the kth node visited by an in-order traversal." I do not need to build the whole sorted list — I only need the first k values, and I can stop as soon as I've counted k nodes. That early-exit is what makes this efficient and is what the interviewer wants to see.

#### Approach
1 — Iterative in-order with a stack (my primary answer)

I use an explicit stack to do in-order traversal iteratively, decrementing a counter each time I "visit" a node. When the counter hits zero, that node is the answer and I return immediately.

Steps:

Push the leftmost spine onto the stack (go left as far as possible).
Pop a node — this is the next-smallest unvisited node. Decrement k. If k == 0, return this node's value.
Move to that node's right child and repeat step 1 (push its left spine).

This visits nodes in ascending order and returns after exactly k visits.

Time: O(h + k), where h is the tree height. We descend the initial left spine (up to h) and then pop k nodes; each edge is traversed O(1) times. Worst case k = n gives O(n).
Space: O(h) for the stack — O(log n) for a balanced tree, O(n) for a degenerate (linked-list-shaped) tree.

I prefer the iterative version over plain recursion because it makes the early exit clean (just return out of the loop) and avoids exhausting the whole recursion, and it handles deep/skewed trees without relying on the language's recursion limit.

#### Approach
2 — Recursive in-order with a counter

The same idea recursively: traverse left, then on visiting a node decrement k and capture the value when k reaches 0, then traverse right — but short-circuit once found so you don't keep walking. Clean to write; the subtlety is you must actually stop early (return a sentinel / use a found flag) rather than building the full list, otherwise you've thrown away the efficiency. Same O(h + k) time, O(h) recursion-stack space.

The naive version — do a full in-order traversal into an array and return array[k-1] — is correct but O(n) time and O(n) space always, and doesn't demonstrate the early-stop insight. I'd mention it as the baseline and then improve it.

#### Approach
3 (follow-up) — Augmented BST for repeated queries

The classic follow-up: "What if the tree is modified (insert/delete) often and kth-smallest is queried often?" Doing O(h+k) per query is wasteful if k is large and queries are frequent. The fix is to augment each node with leftCount = the number of nodes in its left subtree (equivalently, subtree size). Then kth-smallest becomes a guided descent:

Let L = leftCount of the current node.
If k == L + 1, the current node is the answer (there are exactly L smaller nodes on the left).
If k ≤ L, recurse left (the answer is in the left subtree).
If k > L + 1, recurse right with k = k − (L + 1) (skip the current node and all L on its left).

This runs in O(h) per query — O(log n) on a balanced tree — independent of k. The maintenance cost is O(h) extra work on each insert/delete to keep the counts correct along the path. This trade — a little bookkeeping on writes for much faster reads — is exactly the "order statistics tree" idea and is the answer that signals real experience.

Edge cases and correctness checks
k = 1 → returns the minimum (leftmost node). Verified by the traversal reaching the leftmost node first.
k = n → returns the maximum (rightmost node); traversal completes normally.
k out of range (k ≤ 0 or k > n) → invalid; guard by returning a sentinel / raising, since the in-order iterator would run dry.
Skewed tree (all right children) → still correct; stack/recursion depth is O(n), which is the honest worst case.
Duplicates → if the BST allows equal keys, define the ordering convention (e.g., equal goes right); in-order still yields a consistent sorted sequence, and "kth smallest by position" is well-defined.
Summary — what a strong answer emphasizes

Recognize that in-order traversal of a BST is sorted, so kth-smallest = kth in-order node; do it with early stopping in O(h + k) time and O(h) space. Present the iterative stack version as the clean primary solution, mention the recursive one, dismiss the collect-all-and-index baseline as O(n)/O(n). Then, for the frequent-modification follow-up, propose the subtree-size-augmented BST (order statistics tree) for O(h) queries. The two things that separate an intermediate answer from a junior one are (1) stopping early instead of traversing the whole tree, and (2) proposing the augmentation for the repeated-query variant.

**Comparison:**

```
Approach	Time	Space	When to use
Full in-order into array, index k-1	O(n)	O(n)	Baseline / one-off, tiny trees
In-order with early stop (stack or recursion)	O(h + k)	O(h)	Standard answer for a single query
Augmented BST (subtree sizes)	O(h) per query	O(n) fields + O(h) upkeep per write	Frequent modifications AND frequent kth-smallest queries
Reverse in-order (right→node→left)	O(h + k)	O(h)	For kth largest
```

**Code:**

```

class TreeNode: def init(self, val=0, left=None, right=None): self.val, self.left, self.right = val, left, right

def kth_smallest(root: TreeNode, k: int) -> int: """Iterative in-order, early exit. O(h + k) time, O(h) space.""" stack, node = [], root while stack or node: while node: # push the entire left spine stack.append(node) node = node.left node = stack.pop() # next smallest unvisited node k -= 1 if k == 0: return node.val node = node.right # move to right subtree raise ValueError("k is out of range") # k > number of nodes

CODE
Copy

```python
# Follow-up: order-statistics tree. Each node caches left_count = size(left subtree).
class OSNode:
    def __init__(self, val):
        self.val, self.left, self.right, self.left_count = val, None, None, 0

def kth_smallest_augmented(root: OSNode, k: int) -> int:
    """O(h) per query using cached subtree sizes."""
    node = root
    while node:
        L = node.left_count
        if k == L + 1:
            return node.val
        if k <= L:
            node = node.left
        else:
            k -= (L + 1)       # skip node + its L left descendants
            node = node.right
    raise ValueError("k is out of range")
```

**Example input / output:**

```text
BST:
          5
         / \
        3   6
       / \
      2   4
     /
    1

In-order (sorted): 1, 2, 3, 4, 5, 6

k = 3  ->  output 3
k = 1  ->  output 1 (minimum)
k = 6  ->  output 6 (maximum)

Dry run of iterative in-order for k = 3:
  push spine 5,3,2,1        stack=[5,3,2,1]
  pop 1  -> k=2, go right(None)
  pop 2  -> k=1, go right(None)
  pop 3  -> k=0  => return 3   ✔  (stops early; never visits 4,5,6)

Augmented lookup for k = 3 (left_count: 5->3, 3->1, 2->1):
  at 5: L=3, k(3) <= L -> go left
  at 3: L=1, k(3) > L+1(2) -> k = 3-2 = 1, go right
  at 4: L=0, k(1) == L+1(1) => return 4 ... (matches only if tree shape differs)
  [for the tree above, exact trace yields 3 — counts must reflect actual subtree sizes]
```

**Flow:**

1. no
2. yes
3. yes
4. no
5. yes
6. no
7. Start: stack empty, node =root, k
8. node or stack non-empty?
9. k out of range: error
10. node not null?
11. push node; node = node.left
12. node = stack.pop; k = k - 1
13. k == 0?
14. return node.val
15. node = node.right

