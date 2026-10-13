---
type: "practice"
difficulty: "Basic"
---

# Explain what a binary search tree is and the property that governs its node placement.

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Binary Search Trees](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Binary_Search_Trees.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

What the question is really asking: Define a binary search tree (BST) and state the ordering invariant — the rule about where a node's value sits relative to its left and right subtrees. The interviewer wants a precise definition plus the property that makes it a search tree, not just any binary tree.

What the interviewer is probing for: Do you know the exact BST property (and that it applies to the entire left/right subtree, not just the immediate children)? Do you understand why that property matters — that it enables O(log n) search on a balanced tree via a decision at each node? A weak answer says "left is smaller, right is bigger" and stops. A strong answer states the invariant precisely, connects it to in-order traversal producing sorted order, and mentions balance affecting complexity.

#### Assumptions
/ clarifications I'd state: I'll assume distinct keys unless told otherwise, and I'll state how duplicates are usually handled. I'll describe the classic pointer-based binary tree where each node has a key, a left child, and a right child.

From first principles to the answer

1. What a binary tree is. A binary tree is a hierarchical structure where each node has at most two children, conventionally called left and right. There is one root at the top; nodes with no children are leaves. Nothing about a plain binary tree constrains where values go — it's just shape.

2. What makes it a search tree — the BST property. A binary search tree adds an ordering invariant that must hold at every node:

For any node x, every key in x's left subtree is less than x.key, and every key in x's right subtree is greater than x.key.

The critical word is subtree, not "child." It's not enough that the left child is smaller and the right child is larger — all descendants on the left must be smaller and all descendants on the right must be larger. This must be true recursively for every node in the tree.

3. Why this property is powerful. Because of the invariant, searching for a key t becomes a series of decisions instead of a scan:

Compare t with the current node's key.
If equal → found.
If t < node key → the answer, if present, can only be in the left subtree (everything on the right is larger), so go left.
If t > node key → go right.

Each comparison discards an entire subtree, so on a balanced BST you halve the remaining nodes each step → O(log n) search, insert, and delete. This is the payoff of the ordering rule.

4. In-order traversal gives sorted output. A direct consequence of the invariant: an in-order traversal (left → node → right) visits keys in ascending sorted order. This is a great sanity check — if in-order doesn't come out sorted, the tree isn't a valid BST. It's also a common interview follow-up ("how would you verify a tree is a BST?").

5. Duplicates. The strict "< left, > right" definition assumes distinct keys. Real implementations pick a convention: disallow duplicates, or consistently put equal keys on one side (e.g., <= goes right), or store a count per node. State whichever you assume.

Nuances a strong candidate adds
Balance determines complexity, not the BST property alone. The invariant guarantees correctness of search, but not speed. If you insert sorted data (1, 2, 3, 4, …) into a plain BST, it degenerates into a linked list of height n, and operations become O(n). Self-balancing variants (AVL, Red-Black) add rotations to keep height O(log n), guaranteeing O(log n) worst case.
Range and successor queries are natural. The ordering lets you efficiently find min (leftmost node), max (rightmost node), floor/ceiling, and in-order successor/predecessor — things a hash table can't do.
BST vs. hash table. A hash map gives O(1) average lookup but no ordering; a BST gives O(log n) lookup and preserves sorted order / range queries. Choose based on whether you need ordering.
Worked example

Insert keys [50, 30, 70, 20, 40, 60, 80] in that order:

CODE
          50
        /    \
      30      70
     /  \    /  \
   20   40  60   80

Check the invariant at the root 50: left subtree {30,20,40} are all < 50; right subtree {70,60,80} are all > 50. It holds at every node. In-order traversal yields 20 30 40 50 60 70 80 — sorted, confirming validity. Searching for 60: 60 > 50 → go right to 70; 60 < 70 → go left to 60 → found in 3 comparisons instead of scanning 7 nodes.

Summary — what a strong answer emphasizes

A binary search tree is a binary tree that maintains an ordering invariant at every node: all keys in the left subtree are less than the node's key, and all keys in the right subtree are greater (subtree, not just child — recursively). That invariant turns search into a halving decision at each node, giving O(log n) search/insert/delete on a balanced tree, and makes in-order traversal produce sorted order. A strong answer states the invariant precisely, stresses "subtree," explains why it yields logarithmic search, notes that balance (AVL/Red-Black) is what guarantees the log-n worst case, and mentions duplicate handling.

#### Common mistakes
/ follow-ups: saying only "left child smaller, right child larger" (misses the subtree requirement); claiming O(log n) unconditionally (it's O(n) when unbalanced/skewed). Likely follow-ups: "How do you validate a BST?" (check in-order is sorted, or recurse with min/max bounds), and "What breaks when you insert sorted data?" (it degenerates to a linked list — motivates self-balancing trees).

**Comparison:**

```
Aspect	Plain (unbalanced) BST	Balanced BST (AVL/Red-Black)
Search / Insert / Delete (avg)	O(log n)	O(log n)
Search / Insert / Delete (worst)	O(n) (skewed to a chain)	O(log n) guaranteed
In-order traversal	O(n), yields sorted keys	O(n), yields sorted keys
Ordering / range queries	Yes	Yes
BST vs. Hash Table	BST	Hash Table
Lookup	O(log n)	O(1) average
Keeps sorted order	Yes	No
Range / min / max / successor	Efficient	Not supported directly
```

**Code:**

```java
class Node {
    int key;
    Node left, right;
    Node(int k) { key = k; }
}

// Insert respecting the BST invariant.
Node insert(Node root, int key) {
    if (root == null) return new Node(key);
    if (key < root.key)      root.left  = insert(root.left, key);
    else if (key > root.key) root.right = insert(root.right, key);
    // key == root.key: duplicate policy (here: ignore)
    return root;
}

// Search leverages the invariant: O(h), O(log n) if balanced.
boolean search(Node root, int target) {
    while (root != null) {
        if (target == root.key) return true;
        root = (target < root.key) ? root.left : root.right;
    }
    return false;
}

// Validate the BST property using min/max bounds.
boolean isBST(Node n, long min, long max) {
    if (n == null) return true;
    if (n.key <= min || n.key >= max) return false;
    return isBST(n.left, min, n.key) && isBST(n.right, n.key, max);
}
// call: isBST(root, Long.MIN_VALUE, Long.MAX_VALUE)
```

**Example input / output:**

```text
Insert order: 50, 30, 70, 20, 40, 60, 80

Tree:
          50
        /    \
      30      70
     /  \    /  \
   20   40  60   80

In-order traversal -> 20 30 40 50 60 70 80   (sorted => valid BST)

search(60):
  at 50: 60 > 50 -> go right
  at 70: 60 < 70 -> go left
  at 60: 60 == 60 -> FOUND (3 comparisons)
```

**Flow:**

1. yes
2. no
3. yes
4. no
5. yes
6. no
7. search key t at node N
8. N is null?
9. Not found
10. t == N.key?
11. Found
12. t < N.key?
13. Go to N.leftright subtree ruled out
14. Go to N.rightleft subtree ruled out

