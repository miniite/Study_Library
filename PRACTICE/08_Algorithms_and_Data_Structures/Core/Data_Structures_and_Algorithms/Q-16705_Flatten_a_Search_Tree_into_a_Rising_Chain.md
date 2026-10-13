---
type: "practice"
difficulty: "Other"
---

# Flatten a Search Tree into a Rising Chain

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Tree](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Tree.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given the root of a binary search tree. Restructure the tree in place so that it becomes a single chain that only leans to the right, with the values increasing as you follow the chain.

Concretely, after the transformation:

The node holding the smallest value becomes the new root of the tree.
Every node has no left child (its left pointer is null).
Every node has at most one right child, and following the right pointers visits the values in strictly ascending order — exactly the order an in-order traversal of the original search tree would produce.

Because the input is a binary search tree, the in-order sequence is already sorted; your job is to relink the existing nodes into that sorted right-going chain without allocating new nodes. Return the new root (the node with the minimum value).

Input: the root of a binary search tree. Each node stores an integer value and left/right child pointers (either may be absent).

Output: the root of the rearranged tree — a right-leaning chain in which every left pointer is null and the values ascend along the right pointers.

#### Cons
traints:

The number of nodes is in the range [1, 100].
0 <= Node.val <= 1000
The input is a valid binary search tree (values are unique and BST-ordered).

#### Example
s & test cases
TEXT
Input: root = [4,2,6,1,3,5,7] -> Output: [1,null,2,null,3,null,4,null,5,null,6,null,7]
Why: in-order of the tree is 1,2,3,4,5,6,7, which becomes a pure right chain with no left children.

Input: root = [3,1,5] -> Output: [1,null,3,null,5]
Why: the leftmost node 1 becomes the root, then 3, then 5 along right pointers.

Input: root = [1] -> Output: [1]
Why: a single node is already a valid one-element rising chain.

Input: root = [2,1] -> Output: [1,null,2]
Why: the left child 1 becomes the root and 2 hangs off it as the right child.

Input: root = [1,null,2] -> Output: [1,null,2]
Why: the tree is already a right-leaning chain, so it is returned unchanged.

Input: root = [5,3,8,2,4,null,9,1] -> Output: [1,null,2,null,3,null,4,null,5,null,8,null,9]
Why: in-order traversal yields 1,2,3,4,5,8,9, relinked as a single ascending right chain.

#### Approach

Perform an in-order traversal, which for a binary search tree yields values in sorted order, and relink nodes as you go. The key trick is passing down a tail argument — the node that must come right after the current subtree in the final ascending chain. For a node: recursively process its left subtree (whose successor is the node itself, so pass root as the tail) to obtain the head of the left chain; sever the node's left pointer; then set the node's right pointer to the result of processing the right subtree with the inherited tail. Return the head of the left chain (or the node itself if there is no left subtree). This threads all nodes into one right-leaning list in a single pass with no auxiliary array. The underlying pattern is in-order DFS with a successor accumulator (a Morris-style relink is also possible for O(1) extra space).

Complexity

Time: O(n) — each of the n nodes is visited exactly once and relinked in O(1). Space: O(h) for the recursion stack, where h is the tree height (O(n) worst case for a degenerate tree, O(log n) for a balanced one); no extra nodes are allocated.

Python
PYTHON
class TreeNode:
    def __init__(self, val=0, left=None, right=None):
        self.val = val
        self.left = left
        self.right = right

class Solution:
    def risingChain(self, root: 'TreeNode', tail: 'TreeNode' = None) -> 'TreeNode':
        if root is None:
            return tail
        head = self.risingChain(root.left, root)  # smallest node in this subtree
        root.left = None
        root.right = self.risingChain(root.right, tail)
        return head
JavaScript
JAVASCRIPT
function TreeNode(val, left = null, right = null) {
    this.val = val;
    this.left = left;
    this.right = right;
}

// 'tail' is the node that should follow this subtree in ascending order.
function risingChain(root, tail = null) {
    if (root === null) return tail;

#### cons
t head = risingChain(root.left, root);
    root.left = null;
    root.right = risingChain(root.right, tail);
    return head;
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
    public TreeNode risingChain(TreeNode root) {
        return build(root, null);
    }

    // 'tail' is the node that should follow this subtree in ascending order.
    private TreeNode build(TreeNode root, TreeNode tail) {
        if (root == null)
            return tail;
        TreeNode head = build(root.left, root);
        root.left = null;
        root.right = build(root.right, tail);
        return head;
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
    // 'tail' is the node that should follow this subtree in ascending order.
    TreeNode* risingChain(TreeNode* root, TreeNode* tail = nullptr) {
        if (root == nullptr)
            return tail;
        TreeNode* head = risingChain(root->left, root);
        root->left = nullptr;
        root->right = risingChain(root->right, tail);
        return head;
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

/* In-order relink: 'tail' is the node that should follow this subtree. */
static struct TreeNode* build(struct TreeNode* root, struct TreeNode* tail) {
    if (root == NULL)
        return tail;
    struct TreeNode* head = build(root->left, root);
    root->left = NULL;
    root->right = build(root->right, tail);
    return head;
}

struct TreeNode* risingChain(struct TreeNode* root) {
    return build(root, NULL);
}

#### Follow-up
questions
Rebuild the chain iteratively using an explicit stack instead of recursion.
Achieve O(1) extra space with a Morris in-order threading traversal.
Produce a left-leaning descending chain (largest value as root) instead.
If the input were an arbitrary binary tree (not a BST), how would you produce a chain sorted by value?

