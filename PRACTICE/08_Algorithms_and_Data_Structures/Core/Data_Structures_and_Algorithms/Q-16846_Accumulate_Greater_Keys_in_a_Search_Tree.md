---
type: "practice"
difficulty: "Other"
---

# Accumulate Greater Keys in a Search Tree

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Tree](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Tree.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given the root of a binary search tree (BST) whose nodes hold integer keys. Recall that in a BST, for every node, all keys in its left subtree are strictly smaller than the node's key and all keys in its right subtree are strictly larger, and this holds recursively for every subtree.

Transform the tree in place so that each node's key is replaced by its original key plus the sum of all keys in the tree that are strictly greater than that original key. Return the root of the modified tree.

In other words, if you list the keys in increasing order as k[0] < k[1] < ... < k[m-1], then after the transform the node that originally held k[i] must hold k[i] + k[i+1] + ... + k[m-1] (the suffix sum from that key onward). The largest key keeps its own value, and every smaller key grows by everything above it.

Input: the root of a BST (a set of linked nodes, each with an integer val, a left child pointer, and a right child pointer). All keys are distinct.

Output: the root of the same tree, with every node's val overwritten by its greater-keys-inclusive sum.

#### Cons
traints:

The number of nodes is in the range [1, 100].
0 <= Node.val <= 100.
All keys in the tree are unique.

#### Example
s & test cases
TEXT
Trees are shown in level order (breadth-first), with `null` marking an absent child.

Input: root = [5,2,8] -> Output: [13,15,8]
Why: sorted keys are 2 < 5 < 8. The 8 has nothing greater (stays 8); 5 becomes 5+8=13; 2 becomes 2+5+8=15.

Input: root = [10] -> Output: [10]
Why: a single node has no greater key, so its value is unchanged.

Input: root = [3,null,7] -> Output: [10,null,7]
Why: 7 is the largest (stays 7); 3 becomes 3+7=10. Edge case: a right-only chain.

Input: root = [6,3,9,1,4,null,12] -> Output: [27,34,21,35,31,null,12]
Why: sorted keys 1,3,4,6,9,12 with suffix sums 12, 21, 27, 31, 34, 35; e.g. 6 -> 6+9+12=27, 1 -> 1+3+4+6+9+12=35.

Input: root = [0,null,1,null,2] -> Output: [3,null,3,null,2]
Why: keys 0,1,2 (a zero-valued key included). 2 stays 2; 1 becomes 1+2=3; 0 becomes 0+1+2=3. Two different nodes end up equal, which is allowed post-transform.

#### Approach

The key insight is that an in-order traversal of a BST yields keys in ascending order, so a reverse in-order traversal (right subtree, node, left subtree) yields keys in descending order. If you walk the nodes largest-first and keep a running sum of everything visited so far, then at the moment you reach any node that running sum equals exactly the total of all keys greater than it. So you add the running sum to the node, then fold the node's (new or original) value into the running sum before descending left. This turns the problem into a single linear pass instead of, for each node, separately searching for and summing all greater keys (which would be O(n^2)). The underlying pattern is a suffix-sum accumulation carried through an ordered traversal.

Complexity

Time: O(n) where n is the number of nodes — each node is visited exactly once during the reverse in-order walk with O(1) work per node. Space: O(h) where h is the tree height for the recursion stack (O(n) worst case for a skewed tree, O(log n) for a balanced one); no other auxiliary storage.

Python
PYTHON
from typing import Optional

class TreeNode:
    def __init__(self, val=0, left=None, right=None):
        self.val = val
        self.left = left
        self.right = right

class Solution:
    def transformTree(self, root: Optional[TreeNode]) -> Optional[TreeNode]:
        self.prefix = 0

        def reverse_inorder(node: Optional[TreeNode]) -> None:
            if node is None:
                return
            reverse_inorder(node.right)     # larger keys first
            node.val += self.prefix         # add sum of all greater keys
            self.prefix = node.val          # fold this key into the total
            reverse_inorder(node.left)      # then smaller keys

        reverse_inorder(root)
        return root
JavaScript
JAVASCRIPT
function TreeNode(val, left, right) {
    this.val = (val === undefined ? 0 : val);
    this.left = (left === undefined ? null : left);
    this.right = (right === undefined ? null : right);
}

function transformTree(root) {
    let prefix = 0;

    // Reverse in-order: visit larger keys first so `prefix` is their sum.
    function reverseInorder(node) {
        if (node === null) return;
        reverseInorder(node.right);
        node.val += prefix;   // add sum of all greater keys
        prefix = node.val;    // include this key in the running total
        reverseInorder(node.left);
    }

    reverseInorder(root);
    return root;
}
Java
JAVA
class TreeNode {
    int val;
    TreeNode left;
    TreeNode right;
    TreeNode() {}
    TreeNode(int x) { val = x; }
}

class Solution {
    private int prefix = 0;

    public TreeNode transformTree(TreeNode root) {
        prefix = 0;
        reverseInorder(root);
        return root;
    }

    // Visit right subtree (larger keys) first, so prefix holds their sum.
    private void reverseInorder(TreeNode node) {
        if (node == null) return;
        reverseInorder(node.right);
        node.val += prefix;   // add sum of all greater keys
        prefix = node.val;    // include this key in the running total
        reverseInorder(node.left);
    }
}
C++
CPP
struct TreeNode {
    int val;
    TreeNode* left;
    TreeNode* right;
    TreeNode() : val(0), left(nullptr), right(nullptr) {}
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
};

class Solution {
public:
    TreeNode* transformTree(TreeNode* root) {
        int prefix = 0;
        reverseInorder(root, prefix);
        return root;
    }
private:
    // Reverse in-order visit: right subtree (larger keys) before the node.
    void reverseInorder(TreeNode* node, int& prefix) {
        if (node == nullptr) return;
        reverseInorder(node->right, prefix);
        node->val += prefix;   // add running sum of all greater keys
        prefix = node->val;    // fold this key into the running sum
        reverseInorder(node->left, prefix);
    }
};
C
C
struct TreeNode {
    int val;
    struct TreeNode *left;
    struct TreeNode *right;
};

/* Visit nodes largest-first (reverse in-order); keep a running total of all
   keys already seen, which are exactly the keys greater than the current one. */
static void reverseInorder(struct TreeNode* node, int* prefix) {
    if (node == NULL) return;
    reverseInorder(node->right, prefix);   /* larger keys first */
    node->val += *prefix;                  /* add sum of all greater keys */
    *prefix = node->val;                   /* this node is now part of the total */
    reverseInorder(node->left, prefix);    /* then the smaller keys */
}

struct TreeNode* transformTree(struct TreeNode* root) {
    int prefix = 0;
    reverseInorder(root, &prefix);
    return root;
}

#### Follow-up
questions
Do the transform without recursion and in O(1) extra space using a Morris-style threaded reverse in-order traversal.
Change the requirement so each node holds the sum of all keys strictly less than it instead — how does the traversal order change?
What if the input is an arbitrary binary tree rather than a BST? (You would need to extract and sort the values first, then map back.)
Return the transformed values as an array in ascending key order rather than mutating the tree in place.

