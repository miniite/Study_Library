---
type: "practice"
difficulty: "Other"
---

# Deepest Shared Ancestor in an Ordered Binary Tree

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Depth-First Search](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Depth-First_Search.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given the root of an ordered binary tree (a binary search tree). In such a tree, for every node n, all keys stored in the left subtree of n are strictly smaller than n.key, and all keys stored in the right subtree of n are strictly larger than n.key. All keys in the tree are distinct integers.

You are also given references to two distinct nodes in this tree, a and b. Your task is to return the deepest shared ancestor of a and b — that is, the node that is farthest from the root while still having both a and b in its subtree.

An important detail: a node is considered to be an ancestor of itself. So if a lies on the path from the root down to b, then a itself is the deepest shared ancestor (and vice versa).

Return the actual node (not just its key). Both a and b are guaranteed to be present in the tree.

Input

root: the root node of the ordered binary tree.
a, b: two distinct nodes that both exist in the tree.

Output

The node that is the deepest common ancestor of a and b.

#### Cons
traints

The number of nodes is in the range [2, 100000].
-10^9 <= node.key <= 10^9.
All node keys are unique.
a and b refer to two different nodes, and both exist in the tree.

#### Example
s & test cases
TEXT
Assume the tree is built from the level-order layout `[key, ...]` where `null` marks a missing child.

- Input: tree = `[20, 10, 30, 5, 15, 25, 35, null, null, 12, 18]`, a = node(10), b = node(30) -> Output: node(20)
  Why: 10 sits in the left subtree of 20 and 30 in the right subtree, so 20 is the split point where their paths diverge.

- Input: tree = `[20, 10, 30, 5, 15, 25, 35, null, null, 12, 18]`, a = node(10), b = node(15) -> Output: node(10)
  Why: 15 is a descendant of 10 (in 10's right subtree), and a node is an ancestor of itself, so the answer is 10.

- Input: tree = `[20, 10, 30, 5, 15, 25, 35, null, null, 12, 18]`, a = node(12), b = node(18) -> Output: node(15)
  Why: both keys are on the same side of 20 and of 10, but diverge at 15 (12 goes left, 18 goes right).

- Input: tree = `[8, 3]`, a = node(8), b = node(3) -> Output: node(8)
  Why: minimal tree; 3 is the left child of 8, and 8 is the root ancestor of both.

- Input: tree = `[20, 10, 30, 5, 15, 25, 35]`, a = node(5), b = node(35) -> Output: node(20)
  Why: the two extreme leaves split immediately at the root; 20 is their only shared ancestor.

#### Approach

Exploit the ordering invariant of a binary search tree. Starting at the root, compare both target keys against the current node's key. If both targets are smaller, their common ancestor must lie entirely in the left subtree, so move left; if both are larger, move right. The first node at which the two targets fall on opposite sides (one smaller-or-equal, one larger-or-equal, i.e. the current key sits between them, or equals one of them) is exactly the point where their root-to-node paths diverge — that node is the deepest common ancestor. Because the BST property lets us decide the direction in O(1) per step, we never need to explore both subtrees; this is a "binary-search descent" pattern rather than a full tree traversal.

Complexity

Time: O(h), where h is the height of the tree (O(log n) if balanced, O(n) worst case for a skewed tree) — we descend one level per iteration. Space: O(1) — the iterative walk uses only a constant number of pointers.

Python
PYTHON
class TreeNode:
    def __init__(self, key=0, left=None, right=None):
        self.key = key
        self.left = left
        self.right = right

class Solution:
    def deepestSharedAncestor(self, root, a, b):
        cur = root
        while cur:
            if a.key < cur.key and b.key < cur.key:
                cur = cur.left           # both keys smaller -> go left
            elif a.key > cur.key and b.key > cur.key:
                cur = cur.right          # both keys larger  -> go right
            else:
                return cur               # keys straddle cur -> LCA found
        return None
JavaScript
JAVASCRIPT
class TreeNode {

#### cons
tructor(key, left = null, right = null) {
        this.key = key;
        this.left = left;
        this.right = right;
    }
}

function deepestSharedAncestor(root, a, b) {
    let cur = root;
    while (cur) {
        if (a.key < cur.key && b.key < cur.key) {
            cur = cur.left;              // both keys smaller -> go left
        } else if (a.key > cur.key && b.key > cur.key) {
            cur = cur.right;             // both keys larger  -> go right
        } else {
            return cur;                  // keys split here -> this is the LCA
        }
    }
    return null;
}
Java
JAVA
class TreeNode {
    int key;
    TreeNode left, right;
    TreeNode(int k) { key = k; }
}

class Solution {
    public TreeNode deepestSharedAncestor(TreeNode root, TreeNode a, TreeNode b) {
        TreeNode cur = root;
        while (cur != null) {
            if (a.key < cur.key && b.key < cur.key)
                cur = cur.left;          // both keys smaller -> go left
            else if (a.key > cur.key && b.key > cur.key)
                cur = cur.right;         // both keys larger  -> go right
            else
                return cur;              // keys split here -> this is the LCA
        }
        return null;
    }
}
C++
CPP
struct TreeNode {
    int key;
    TreeNode* left;
    TreeNode* right;
    TreeNode(int k) : key(k), left(nullptr), right(nullptr) {}
};

class Solution {
public:
    TreeNode* deepestSharedAncestor(TreeNode* root, TreeNode* a, TreeNode* b) {
        TreeNode* cur = root;
        while (cur) {
            if (a->key < cur->key && b->key < cur->key)
                cur = cur->left;          // both keys smaller -> go left
            else if (a->key > cur->key && b->key > cur->key)
                cur = cur->right;         // both keys larger  -> go right
            else
                return cur;               // keys straddle cur -> LCA found
        }
        return nullptr;
    }
};
C
C
#include <stdlib.h>

struct TreeNode {
    int key;
    struct TreeNode *left;
    struct TreeNode *right;
};

struct TreeNode* deepestSharedAncestor(struct TreeNode* root,
                                       struct TreeNode* a,
                                       struct TreeNode* b) {
    struct TreeNode* cur = root;
    while (cur) {
        if (a->key < cur->key && b->key < cur->key) {
            cur = cur->left;          /* both keys smaller -> go left */
        } else if (a->key > cur->key && b->key > cur->key) {
            cur = cur->right;         /* both keys larger  -> go right */
        } else {
            return cur;               /* keys split here -> this is the LCA */
        }
    }
    return NULL;
}

#### Follow-up
questions
How does the answer change if the tree is a general binary tree with no ordering guarantee? (You lose the O(1) direction choice and must do a full DFS.)
What if one of the two nodes might NOT exist in the tree — how would you detect that and return null safely?
Can you solve it recursively instead of iteratively, and what is the space trade-off of the recursion stack?
Extend it to return the distance (number of edges) between the two nodes using the LCA.

