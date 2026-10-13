---
type: "practice"
difficulty: "Other"
---

# Rebuild a Binary Tree From Its Pre-order and Post-order Sequences

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Divide and Conquer](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Divide_and_Conquer.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given two integer arrays, preorder and postorder, that describe the same binary tree whose node values are all distinct. preorder lists the node values in the order visited by a pre-order traversal (visit the node, then its left subtree, then its right subtree). postorder lists the node values in the order visited by a post-order traversal (visit the left subtree, then the right subtree, then the node).

Reconstruct a binary tree that produces exactly these two traversal sequences and return its root. In general a pre-order/post-order pair does not uniquely identify a binary tree (a node with a single child is ambiguous), so if more than one tree is consistent with the input, you may return any one of them.

The output is described here by the tree's level-order (breadth-first) listing, using null for missing children, so the examples can be written compactly.

#### Cons
traints

1 <= preorder.length <= 30
postorder.length == preorder.length
All values in preorder are distinct, and postorder is a permutation of the same values.
Every value lies in the range 1 to preorder.length inclusive.
The input is guaranteed to be a valid pre-order and post-order pair of one binary tree.

#### Example
s & test cases
TEXT
Input: preorder = [8,4,2,6,9,3,7], postorder = [2,6,4,3,7,9,8] -> Output (level order): [8,4,9,2,6,3,7]
Why: a full tree where every internal node has two children; root 8 has left subtree rooted at 4 (children 2, 6) and right subtree rooted at 9 (children 3, 7).

Input: preorder = [5], postorder = [5] -> Output (level order): [5]
Why: single-node edge case; the only node is the root.

Input: preorder = [1,2], postorder = [2,1] -> Output (level order): [1,2]
Why: root 1 has exactly one child (value 2). The pair is ambiguous (2 could be a left or right child); the reconstruction attaches it as the left child.

Input: preorder = [10,20,30], postorder = [30,20,10] -> Output (level order): [10,20,null,30]
Why: a left-leaning chain 10 -> 20 -> 30, each node having a single child; illustrates that the pair does not uniquely determine the tree.

#### Approach

Divide and conquer on index ranges. The first element of the current pre-order slice is the subtree's root, and the last element of the current post-order slice is that same root. The element immediately after the root in pre-order (when the slice has more than one node) is the root of the left subtree; locate that value in post-order to learn where the left subtree ends there, which gives the left subtree's size. That size lets us split both the pre-order and post-order slices into left and right portions, and we recurse on each. A hash map from value to its post-order index turns the "find the left root" step into O(1), giving overall O(n). Because a single-child node is indistinguishable as left or right in a pre/post pair, we always treat the next pre-order element as the left child, yielding one valid tree among possibly many.

Complexity

Time: O(n) — a value-to-index map over the post-order array is built once, and each node is created by exactly one constant-work recursive call. Space: O(n) for the map plus O(h) recursion stack (O(n) worst case for a skewed tree).

Python
PYTHON
from typing import List, Optional

class TreeNode:
    def __init__(self, val=0, left=None, right=None):
        self.val = val
        self.left = left
        self.right = right

def build_tree(preorder: List[int], postorder: List[int]) -> Optional[TreeNode]:
    post_index = {value: i for i, value in enumerate(postorder)}

    def build(pre_lo: int, pre_hi: int, post_lo: int, post_hi: int) -> Optional[TreeNode]:
        if pre_lo > pre_hi:
            return None
        node = TreeNode(preorder[pre_lo])
        if pre_lo == pre_hi:
            return node

        left_root = preorder[pre_lo + 1]
        left_post = post_index[left_root]
        left_size = left_post - post_lo + 1

        node.left = build(pre_lo + 1, pre_lo + left_size, post_lo, left_post)
        node.right = build(pre_lo + left_size + 1, pre_hi, left_post + 1, post_hi - 1)
        return node

    return build(0, len(preorder) - 1, 0, len(postorder) - 1)
JavaScript
JAVASCRIPT
function TreeNode(val) {
    this.val = val;
    this.left = null;
    this.right = null;
}

function buildTree(preorder, postorder) {

#### cons
t postIndex = new Map();
    postorder.forEach((v, i) => postIndex.set(v, i));

    function build(preLo, preHi, postLo, postHi) {
        if (preLo > preHi) return null;

#### cons
t node = new TreeNode(preorder[preLo]);
        if (preLo === preHi) return node;

#### cons
t leftRoot = preorder[preLo + 1];

#### cons
t leftPost = postIndex.get(leftRoot);

#### cons
t leftSize = leftPost - postLo + 1;

        node.left  = build(preLo + 1, preLo + leftSize, postLo, leftPost);
        node.right = build(preLo + leftSize + 1, preHi, leftPost + 1, postHi - 1);
        return node;
    }

    return build(0, preorder.length - 1, 0, postorder.length - 1);
}
Java
JAVA
import java.util.HashMap;
import java.util.Map;

class TreeNode {
    int val;
    TreeNode left, right;
    TreeNode(int val) { this.val = val; }
}

class Solution {
    private int[] pre;
    private final Map<Integer, Integer> postIndex = new HashMap<>();

    public TreeNode buildTree(int[] preorder, int[] postorder) {
        pre = preorder;
        for (int i = 0; i < postorder.length; i++)
            postIndex.put(postorder[i], i);
        return build(0, preorder.length - 1, 0, postorder.length - 1);
    }

    private TreeNode build(int preLo, int preHi, int postLo, int postHi) {
        if (preLo > preHi) return null;
        TreeNode node = new TreeNode(pre[preLo]);
        if (preLo == preHi) return node;

        int leftRoot = pre[preLo + 1];
        int leftPost = postIndex.get(leftRoot);
        int leftSize = leftPost - postLo + 1;

        node.left  = build(preLo + 1, preLo + leftSize, postLo, leftPost);
        node.right = build(preLo + leftSize + 1, preHi, leftPost + 1, postHi - 1);
        return node;
    }
}
C++
CPP
#include <vector>
#include <unordered_map>
using namespace std;

struct TreeNode {
    int val;
    TreeNode* left;
    TreeNode* right;
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
};

class Solution {
public:
    TreeNode* buildTree(vector<int>& preorder, vector<int>& postorder) {
        pre = &preorder;
        for (int i = 0; i < (int)postorder.size(); ++i)
            postIndex[postorder[i]] = i;
        return build(0, (int)preorder.size() - 1, 0, (int)postorder.size() - 1);
    }

private:
    vector<int>* pre;
    unordered_map<int, int> postIndex;

    TreeNode* build(int preLo, int preHi, int postLo, int postHi) {
        if (preLo > preHi) return nullptr;
        TreeNode* node = new TreeNode((*pre)[preLo]);
        if (preLo == preHi) return node;

        int leftRoot = (*pre)[preLo + 1];
        int leftPost = postIndex[leftRoot];
        int leftSize = leftPost - postLo + 1;

        node->left  = build(preLo + 1, preLo + leftSize, postLo, leftPost);
        node->right = build(preLo + leftSize + 1, preHi, leftPost + 1, postHi - 1);
        return node;
    }
};
C
C
#include <stdlib.h>

struct TreeNode {
    int val;
    struct TreeNode* left;
    struct TreeNode* right;
};

static int* g_pre;
static int* g_postIndex;   /* value -> index in postorder (values are 1..n) */

static struct TreeNode* build(int preLo, int preHi, int postLo, int postHi) {
    if (preLo > preHi) return NULL;
    struct TreeNode* node = (struct TreeNode*)malloc(sizeof(struct TreeNode));
    node->val = g_pre[preLo];
    node->left = node->right = NULL;
    if (preLo == preHi) return node;

    int leftRoot  = g_pre[preLo + 1];
    int leftPost  = g_postIndex[leftRoot];
    int leftSize  = leftPost - postLo + 1;

    node->left  = build(preLo + 1, preLo + leftSize, postLo, leftPost);
    node->right = build(preLo + leftSize + 1, preHi, leftPost + 1, postHi - 1);
    return node;
}

struct TreeNode* buildTree(int* preorder, int preSize, int* postorder, int postSize) {
    g_pre = preorder;
    g_postIndex = (int*)malloc((postSize + 1) * sizeof(int));
    for (int i = 0; i < postSize; i++) g_postIndex[postorder[i]] = i;
    struct TreeNode* root = build(0, preSize - 1, 0, postSize - 1);
    free(g_postIndex);
    return root;
}

#### Follow-up
questions
Exactly why is the tree not unique from a pre/post pair, and how many distinct trees can match a given pair?
Reconstruct instead from pre-order plus in-order — why is that result unique?
Rewrite the reconstruction iteratively using an explicit stack instead of recursion.
Count the total number of binary trees consistent with the given pre-order and post-order arrays.

