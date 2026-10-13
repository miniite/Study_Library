---
type: "practice"
difficulty: "Intermediate"
---

# Check if a binary tree is a valid binary search tree.

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Binary Search Trees](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Binary_Search_Trees.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

What's really being asked: Given the root of a binary tree, return true if it is a valid Binary Search Tree (BST) and false otherwise. The interviewer is probing whether you understand the global BST invariant — not just the local parent/child relationship — plus your ability to pick a clean traversal and handle edge cases (empty tree, duplicates, integer bounds).

The BST definition (the crux): A binary tree is a BST if for every node:

all nodes in its left subtree are strictly less than the node, and
all nodes in its right subtree are strictly greater than the node, and
both subtrees are themselves BSTs.

The word "all" is what trips people up. It is not enough to check node.left.val < node.val < node.right.val. That naive check passes trees that are clearly invalid because a grandchild can violate an ancestor's bound.

Clarifying questions I'd ask:

Are duplicate values allowed? Standard definition says no (strictly less / strictly greater). I'll assume strict inequality; I'll mention how to flex it.
What's the value range? If values can be INT_MIN/INT_MAX, using those as sentinel bounds is buggy — I'll use nullable bounds (or long) instead.
Can the tree be empty? Yes — an empty tree is a valid BST, return true.

Plan: I'll present the classic counter-example that kills the naive approach, then give the range/bounds recursive solution (the canonical answer), then the in-order traversal solution (elegant alternative), discuss complexity and edge cases, and finish with follow-ups.

Why the naive local check fails

#### Cons
ider:

CODE
        5
       / \
      1   6
         / \
        4   7

Every parent/child pair looks fine locally: 1 < 5, 6 > 5, 4 < 6, 7 > 6. But 4 sits in the right subtree of 5, so it must be > 5 — and it isn't. This tree is not a BST. A correct algorithm must carry down the inherited constraint that everything under 5's right must stay above 5.

#### Approach
1 — Range (min/max bounds) recursion (recommended)

The insight: as we descend, each node is constrained to an open interval (low, high) dictated by its ancestors. The root has no bound: (-∞, +∞). When we go left, the value becomes the new upper bound; when we go right, it becomes the new lower bound.

Algorithm for validate(node, low, high):

If node is null → return true (empty subtree is valid).
If node.val <= low or node.val >= high → return false (out of range).
Recurse: left subtree must fit (low, node.val), right subtree must fit (node.val, high). Return the AND.

Using nullable bounds (or long ±∞) avoids the INT_MIN/INT_MAX sentinel bug where a legitimate node value equals the sentinel.

Complexity: O(n) time — each node visited once. Space O(h) for the recursion stack where h is tree height: O(log n) for a balanced tree, O(n) worst case (skewed).

#### Approach
2 — In-order traversal must be strictly increasing

Key theorem: an in-order traversal (left → node → right) of a valid BST yields values in strictly ascending order. So: traverse in-order, keep the previously visited value prev; if the current value is <= prev, it's not a BST.

You can do this with an explicit stack (iterative) and short-circuit the moment a violation is found — no need to materialize the whole array. This is O(n) time, O(h) space, and reads very cleanly. Interviewers often love this because it directly encodes the BST property.

Common bug: initializing prev to Long.MIN_VALUE and using <= is safe; using Integer.MIN_VALUE breaks if a node legitimately holds Integer.MIN_VALUE. Prefer a nullable prev (Integer/null) and only compare when non-null.

Handling duplicates

If the problem allows equal values on, say, the right subtree, relax one comparison: replace node.val >= high with node.val > high (or adjust the in-order check to < prev instead of <= prev). Always confirm the exact rule with the interviewer; the default LeetCode-style definition forbids duplicates entirely, so I keep strict inequalities.

Edge cases to call out
Empty tree (root == null) → true.
Single node → true.
Extreme values — a node holding INT_MIN or INT_MAX; solved by nullable/long bounds.
Skewed tree (linked-list shape) — still O(n), but recursion depth O(n); if stack overflow is a concern, the iterative in-order version avoids deep recursion (bounded by an explicit stack, but still O(n) in the worst case — mention Morris traversal for O(1) space if pushed).
What a strong answer emphasizes
Correctly stating the global invariant and killing the naive local check with a concrete counter-example.
Choosing the bounds recursion or in-order traversal — both O(n) — and explaining the trade-off (bounds recursion is the most intuitive; in-order is elegant and generalizes to "find kth smallest").
Avoiding the INT_MIN/INT_MAX sentinel pitfall (nullable or long bounds).
O(n) time, O(h) space, and being explicit about balanced vs. skewed.

#### Follow-up
probes I'd be ready for: convert to an iterative solution (I have it); do it with O(1) extra space (Morris in-order); adapt for duplicates; and "what if the tree is huge and doesn't fit in memory / is streamed?" (in-order streaming with a single prev).

Summary: Validate a BST by propagating an (low, high) interval down the tree (bounds recursion) or by asserting the in-order sequence is strictly increasing. Both are O(n) time, O(h) space. The whole difficulty is remembering that the constraint is inherited from all ancestors, not just the immediate parent.

**Comparison:**

```
Approach	Idea	Time	Space	Notes / pitfalls
Naive local check	left < node < right per node only	O(n)	O(h)	WRONG — ignores inherited ancestor bounds
Bounds recursion	Pass down (low, high) interval	O(n)	O(h)	Use nullable/long bounds; most intuitive
In-order traversal	Sequence must be strictly increasing	O(n)	O(h)	Elegant; short-circuit on first violation
Morris in-order	Threaded in-order, no stack	O(n)	O(1)	Advanced; mutates then restores pointers

(h = tree height: O(log n) balanced, O(n) skewed.)
```

**Code:**

```

// Approach 1: Range / bounds recursion (nullable Long bounds avoid INT_MIN/MAX bug) class Solution { public boolean isValidBST(TreeNode root) { return validate(root, null, null); } private boolean validate(TreeNode node, Long low, Long high) { if (node == null) return true; // empty subtree is valid if (low != null && node.val <= low) return false; if (high != null && node.val >= high) return false; return validate(node.left, low, (long) node.val) && validate(node.right, (long) node.val, high); } }

CODE
Copy

```java
// Approach 2: Iterative in-order traversal must be strictly increasing
class Solution {
    public boolean isValidBST(TreeNode root) {
        java.util.Deque<TreeNode> stack = new java.util.ArrayDeque<>();
        Integer prev = null;                    // nullable => no sentinel bug
        TreeNode cur = root;
        while (cur != null || !stack.isEmpty()) {
            while (cur != null) { stack.push(cur); cur = cur.left; }
            cur = stack.pop();
            if (prev != null && cur.val <= prev) return false;  // not strictly increasing
            prev = cur.val;
            cur = cur.right;
        }
        return true;
    }
}
PYTHON
Copy
# Python: bounds recursion
class Solution:
    def isValidBST(self, root):
        def validate(node, low=float('-inf'), high=float('inf')):
            if not node:
                return True
            if not (low < node.val < high):
                return False
            return validate(node.left, low, node.val) and \
                   validate(node.right, node.val, high)
        return validate(root)
```

**Example input / output:**

```

Example A (INVALID — the classic trap):

CODE
Copy
Input tree:            5
                      / \
                     1   6
                        / \
                       4   7
Output: false

Dry run (bounds recursion): validate(5, -inf, +inf) ok -> validate right(6, 5, +inf) ok -> validate left of 6 = (4, low=5, high=6): 4 <= low(5) => FALSE. Correctly rejected because 4 is in 5's right subtree but not > 5.

Example B (VALID):

CODE
Copy
Input tree:            2
                      / \
                     1   3
Output: true

In-order sequence = [1, 2, 3] -> strictly increasing -> true.

Example C (edge):

CODE
Copy
Input: root = null   -> Output: true   (empty tree is a valid BST)
Input: root = [ -2147483648 ]  -> Output: true (single node, extreme value ok with nullable bounds)
```

**Flow:**

1. yes
2. no
3. yes
4. no
5. validate(node, low, high)
6. node == null?
7. return true
8. node.val <= lowor node.val >= high?
9. return false
10. validate(left, low, node.val)ANDvalidate(right, node.val, high)
11. return result

