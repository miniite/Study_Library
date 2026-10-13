---
type: "practice"
difficulty: "Other"
---

# Locate a Node by Key in a Binary Search Tree

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Tree](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Tree.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given the root of a binary search tree (BST) and an integer target.

Recall that in a binary search tree, for every node the entire left subtree contains values strictly smaller than the node's value, and the entire right subtree contains values strictly larger. This ordering holds recursively at every node.

Find the node whose value equals target and return the whole subtree rooted at that node (that is, the node together with all of its descendants). If no node in the tree holds the value target, return an empty/null result.

Because the tree obeys the BST ordering, you should not need to inspect every node — the ordering lets you decide at each step whether to descend left or right.

Input

root: the root of a binary search tree, given in level-order form where null marks an absent child.
target: the integer value to look for.

Output

The subtree (its root node) whose value is target, or null/empty if the value is absent. When printed in level-order, the result is that node and everything beneath it.

#### Cons
traints

The number of nodes in the tree is in the range [1, 5000].
1 <= node value <= 10^7.
1 <= target <= 10^7.
All node values are distinct and the input tree is guaranteed to satisfy the BST property.

#### Example
s & test cases
TEXT
Input: root = [8,3,10,1,6], target = 3 -> Output: [3,1,6]
  Why: 3 sits in the tree; the returned subtree is that node plus its children 1 and 6.

Input: root = [8,3,10,1,6], target = 10 -> Output: [10]
  Why: 10 is a leaf-like node on the right; its subtree is just itself.

Input: root = [8,3,10,1,6], target = 5 -> Output: []
  Why: 5 is not present anywhere, so the search returns null / empty.

Input: root = [8,3,10,1,6], target = 8 -> Output: [8,3,10,1,6]
  Why: matching the root returns the entire tree.

Input: root = [42], target = 42 -> Output: [42]
  Why: single-node tree where the only node is the match (minimum-size case).

Input: root = [42], target = 7 -> Output: []
  Why: single-node tree with no match (edge case, target smaller than root).

#### Approach

Exploit the BST ordering to prune half the tree at every step instead of scanning all nodes. Start at the root and repeat: if the current node's value equals target, you have found it and return that node (its subtree comes for free since the node still links to its descendants); if target is smaller, the answer can only be in the left subtree, so move left; otherwise move right. Stop when you either match or fall off the tree (null). This is binary search applied to a tree structure. The iterative form avoids recursion overhead and keeps auxiliary space at O(1).

Complexity

Time: O(h), where h is the height of the tree — the search follows a single root-to-leaf path, so it is O(log n) on a balanced BST and O(n) on a degenerate (chain-like) BST. Space: O(1), because the iterative walk keeps only a single moving pointer and uses no recursion stack.

Python
PYTHON
class TreeNode:
    def __init__(self, val=0, left=None, right=None):
        self.val = val
        self.left = left
        self.right = right

class Solution:
    def searchBST(self, root: 'TreeNode', target: int) -> 'TreeNode':
        while root is not None and root.val != target:
            root = root.left if target < root.val else root.right
        return root
JavaScript
JAVASCRIPT
class TreeNode {

#### cons
tructor(val = 0, left = null, right = null) {
        this.val = val;
        this.left = left;
        this.right = right;
    }
}

function searchBST(root, target) {
    while (root !== null && root.val !== target) {
        root = target < root.val ? root.left : root.right;
    }
    return root;
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
    public TreeNode searchBST(TreeNode root, int target) {
        while (root != null && root.val != target) {
            root = (target < root.val) ? root.left : root.right;
        }
        return root;
    }
}
C++
CPP
struct TreeNode {
    int val;
    TreeNode *left;
    TreeNode *right;
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
};

class Solution {
public:
    TreeNode* searchBST(TreeNode* root, int target) {
        while (root != nullptr && root->val != target) {
            root = (target < root->val) ? root->left : root->right;
        }
        return root;
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

struct TreeNode* searchBST(struct TreeNode* root, int target) {
    while (root != NULL && root->val != target) {
        if (target < root->val)
            root = root->left;
        else
            root = root->right;
    }
    return root;
}

#### Follow-up
questions
What changes if the tree is NOT a BST (arbitrary binary tree)? (Answer: you lose the ordering shortcut and must do a full DFS/BFS, O(n).)
Can you write it recursively, and what is the recursion's space cost? (O(h) stack space.)
How would you return only whether the key exists (boolean) versus the subtree, and does that change complexity?
If node values could repeat, how would "find the node" be defined and handled?

