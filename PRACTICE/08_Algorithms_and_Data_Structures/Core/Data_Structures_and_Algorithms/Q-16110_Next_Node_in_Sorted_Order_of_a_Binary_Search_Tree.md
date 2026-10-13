---
type: "practice"
difficulty: "Other"
---

# Next Node in Sorted Order of a Binary Search Tree

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Tree](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Tree.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given the root of a binary search tree (BST) together with a pointer to one specific node target that is known to already exist inside that tree. Return the node that comes immediately after target when the tree is traversed in ascending (in-order) order.

Equivalently, return the node whose value is the smallest value strictly greater than target.val. If target holds the largest value in the whole tree, there is no such node, so return null (or the language's null/None equivalent).

Recall the BST invariant: for every node, all values in its left subtree are smaller than the node's value, and all values in its right subtree are larger. Every value in the tree is unique. You must return the actual node object, not just its value.

Input: the root node of a BST and a reference target to an existing node in it. Each node stores an integer value and pointers to its left and right child (either may be absent).

Output: the in-order successor node of target, or null if none exists.

#### Cons
traints:

The number of nodes in the tree is in the range [1, 10^4].
Each node value v satisfies -10^5 <= v <= 10^5.
All node values are unique.
target is guaranteed to be a node that is present in the tree.

#### Example
s & test cases

#### Cons
ider the BST:

CODE
        20
       /  \
      10   30
     /  \    \
    5   15    40

Its in-order (sorted) sequence is 5, 10, 15, 20, 30, 40.

Input: target = node(10) -> Output: node(15) Why: in sorted order 15 directly follows 10 (the leftmost node of 10's right subtree).

Input: target = node(15) -> Output: node(20) Why: 15 is a leaf with no right subtree, so the successor is the lowest ancestor for which 15 lies in the left subtree, namely 20.

Input: target = node(40) -> Output: null Why: 40 is the maximum value in the tree, so it has no in-order successor.

Input: target = node(20) -> Output: node(30) Why: the root's successor is the minimum of its right subtree, which is 30.

Input: single-node tree with root = node(7), target = node(7) -> Output: null Why: the only node is also the maximum, so no successor exists.

Input: target = node(5) -> Output: node(10) Why: 5 has no right child, so we climb to its parent 10, the first ancestor greater than 5.

#### Approach

Exploit the BST ordering instead of doing a full in-order traversal. Walk down from the root: whenever the current node's value is greater than target.val, that node is a possible successor, so record it and move left to see whether an even smaller (but still larger than target) value exists. Whenever the current value is less than or equal to target.val, the successor must be larger, so move right and discard the current node. When the walk falls off the tree, the last recorded candidate is the tightest upper bound on target.val, i.e. the in-order successor; if nothing was ever recorded, target was the maximum and the answer is null. This is a classic "predecessor/successor via binary search on the tree" pattern and needs no parent pointers or extra memory.

Complexity

Time: O(h) where h is the height of the tree — we descend a single root-to-node path, so O(log n) for a balanced BST and O(n) for a fully skewed one. Space: O(1) — the walk is iterative and keeps only two pointers, using no recursion stack or auxiliary structure.

Python
PYTHON
from typing import Optional

class TreeNode:
    def __init__(self, val=0, left=None, right=None):
        self.val = val
        self.left = left
        self.right = right

class Solution:
    def inorderSuccessor(self, root: Optional[TreeNode],
                         target: TreeNode) -> Optional[TreeNode]:
        successor = None
        cur = root
        while cur:
            if cur.val > target.val:
                successor = cur      # candidate; a smaller-yet-larger one may lie left
                cur = cur.left
            else:
                cur = cur.right
        return successor
JavaScript
JAVASCRIPT
function TreeNode(val, left = null, right = null) {
    this.val = val;
    this.left = left;
    this.right = right;
}

function inorderSuccessor(root, target) {
    let successor = null;
    let cur = root;
    while (cur !== null) {
        if (cur.val > target.val) {
            successor = cur;      // remember, then try to find a closer value on the left
            cur = cur.left;
        } else {
            cur = cur.right;
        }
    }
    return successor;
}
Java
JAVA
class TreeNode {
    int val;
    TreeNode left;
    TreeNode right;
    TreeNode(int x) { val = x; }
}

class Solution {
    public TreeNode inorderSuccessor(TreeNode root, TreeNode target) {
        TreeNode successor = null;
        TreeNode cur = root;
        while (cur != null) {
            if (cur.val > target.val) {
                successor = cur;      // candidate; keep tightening from the left
                cur = cur.left;
            } else {
                cur = cur.right;
            }
        }
        return successor;
    }
}
C++
CPP
struct TreeNode {
    int val;
    TreeNode* left;
    TreeNode* right;
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
};

class Solution {
public:
    TreeNode* inorderSuccessor(TreeNode* root, TreeNode* target) {
        TreeNode* successor = nullptr;
        for (TreeNode* cur = root; cur != nullptr; ) {
            if (cur->val > target->val) {
                successor = cur;      // remember, then look for a tighter bound
                cur = cur->left;
            } else {
                cur = cur->right;
            }
        }
        return successor;
    }
};
C
C
#include <stddef.h>

struct TreeNode {
    int val;
    struct TreeNode *left;
    struct TreeNode *right;
};

struct TreeNode* inorderSuccessor(struct TreeNode* root, struct TreeNode* target) {
    struct TreeNode* successor = NULL;
    struct TreeNode* cur = root;
    while (cur != NULL) {
        if (cur->val > target->val) {
            successor = cur;      /* candidate; a smaller one may still exist to the left */
            cur = cur->left;
        } else {
            cur = cur->right;     /* need something larger than target */
        }
    }
    return successor;
}

#### Follow-up
questions
Find the in-order predecessor instead — the largest value strictly smaller than target.val.
Suppose every node also stores a pointer to its parent — solve it starting from target alone without access to the root.
What changes if the tree is a general binary tree (not a BST)? Then you must actually perform an in-order traversal, giving O(n).
Return the k-th successor (the node k positions after target in sorted order).

