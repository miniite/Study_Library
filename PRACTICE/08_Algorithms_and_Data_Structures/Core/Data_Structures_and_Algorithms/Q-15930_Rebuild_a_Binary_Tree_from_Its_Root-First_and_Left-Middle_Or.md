---
type: "practice"
difficulty: "Other"
---

# Rebuild a Binary Tree from Its Root-First and Left-Middle Orders

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Divide and Conquer](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Divide_and_Conquer.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

A binary tree was walked in two different orders and each walk produced a list of the node values in the order they were visited.

The root-first walk visits the current node, then everything in its left subtree, then everything in its right subtree (i.e. node, then left, then right).
The left-middle walk visits everything in the left subtree first, then the current node, then everything in the right subtree (i.e. left, then node, then right).

You are given two integer arrays, rootFirst and leftMiddle, that are the two walks of the same binary tree. Every value in the tree is distinct. Reconstruct the tree and return its root.

The two arrays uniquely determine the tree because:

The first element of rootFirst is always the tree's root.
Locating that root value inside leftMiddle splits leftMiddle into the values that lie in the left subtree (everything before it) and the values that lie in the right subtree (everything after it). The count of left-subtree values then tells you exactly how many of the remaining rootFirst values belong to the left subtree, and the rest belong to the right subtree. Recurse on each side.

Return the reconstructed tree (its root node). If you serialize it level by level, use null for missing children.

#### Cons
traints

1 <= rootFirst.length <= 3000
leftMiddle.length == rootFirst.length
-3000 <= rootFirst[i], leftMiddle[i] <= 3000
All values are unique.
Every value in leftMiddle also appears in rootFirst.
rootFirst is guaranteed to be a valid root-first (preorder) walk of some tree, and leftMiddle the left-middle (inorder) walk of that same tree.

#### Example
s & test cases
TEXT
Input: rootFirst = [7,2,1,5,9,4], leftMiddle = [1,2,5,7,9,4] -> Output (level order) = [7,2,9,1,5,null,4]
Why: root 7 splits leftMiddle into left {1,2,5} and right {9,4}; recursing rebuilds a mixed-shape tree.

Input: rootFirst = [42], leftMiddle = [42] -> Output = [42]
Why: minimum size — a single node is its own root with no children.

Input: rootFirst = [5,4,3], leftMiddle = [3,4,5] -> Output = [5,4,null,3]
Why: every node has only a left child (a left-leaning chain); the right side is always empty.

Input: rootFirst = [1,2,3], leftMiddle = [1,2,3] -> Output = [1,null,2,null,3]
Why: every node has only a right child (a right-leaning chain); the left side is always empty.

Input: rootFirst = [-1,-2], leftMiddle = [-2,-1] -> Output = [-1,-2]
Why: negative values and a two-node tree where -2 is the left child of -1.

#### Approach

Divide and conquer. The first unused value of the root-first walk is always the root of the current subtree. Find that value's position in the left-middle walk: everything to its left is the left subtree, everything to its right is the right subtree. Advancing a single shared cursor through the root-first walk (root, then all of left, then all of right) means each recursive call just needs the left-middle index range it is responsible for; the cursor naturally hands out the correct root to each subtree. A precomputed value-to-index map on the left-middle walk turns each split into an O(1) operation, giving overall linear time.

Complexity

Time: O(n) — each of the n nodes is created exactly once, and a hash map gives O(1) lookup of a root's split point in the left-middle walk (avoiding a linear scan per node, which would be O(n^2)). Space: O(n) for the index map plus O(h) recursion stack (h = tree height, up to O(n) for a skewed/chain-shaped tree).

Python
PYTHON
from typing import List, Optional

class TreeNode:
    def __init__(self, val=0, left=None, right=None):
        self.val = val
        self.left = left
        self.right = right

class Solution:
    def buildTree(self, rootFirst: List[int], leftMiddle: List[int]) -> Optional[TreeNode]:
        pos = {v: i for i, v in enumerate(leftMiddle)}
        self.pre_idx = 0

        def build(in_left: int, in_right: int) -> Optional[TreeNode]:
            if in_left > in_right:
                return None
            root_val = rootFirst[self.pre_idx]
            self.pre_idx += 1
            root = TreeNode(root_val)
            mid = pos[root_val]
            root.left = build(in_left, mid - 1)
            root.right = build(mid + 1, in_right)
            return root

        return build(0, len(leftMiddle) - 1)
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

function buildTree(rootFirst, leftMiddle) {

#### cons
t pos = new Map();
    leftMiddle.forEach((v, i) => pos.set(v, i));
    let preIdx = 0;

    function build(inLeft, inRight) {
        if (inLeft > inRight) return null;

#### cons
t rootVal = rootFirst[preIdx++];

#### cons
t root = new TreeNode(rootVal);

#### cons
t mid = pos.get(rootVal);
        root.left = build(inLeft, mid - 1);
        root.right = build(mid + 1, inRight);
        return root;
    }

    return build(0, leftMiddle.length - 1);
}
Java
JAVA
import java.util.HashMap;
import java.util.Map;

class TreeNode {
    int val;
    TreeNode left;
    TreeNode right;
    TreeNode(int x) { val = x; }
}

class Solution {
    private int preIdx = 0;
    private final Map<Integer, Integer> pos = new HashMap<>();

    public TreeNode buildTree(int[] rootFirst, int[] leftMiddle) {
        for (int i = 0; i < leftMiddle.length; i++)
            pos.put(leftMiddle[i], i);
        preIdx = 0;
        return build(rootFirst, 0, leftMiddle.length - 1);
    }

    private TreeNode build(int[] rootFirst, int inLeft, int inRight) {
        if (inLeft > inRight)
            return null;
        int rootVal = rootFirst[preIdx++];
        TreeNode root = new TreeNode(rootVal);
        int mid = pos.get(rootVal);
        root.left  = build(rootFirst, inLeft, mid - 1);
        root.right = build(rootFirst, mid + 1, inRight);
        return root;
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
    TreeNode* buildTree(vector<int>& rootFirst, vector<int>& leftMiddle) {
        for (int i = 0; i < (int)leftMiddle.size(); ++i)
            pos[leftMiddle[i]] = i;
        preIdx = 0;
        return build(rootFirst, 0, (int)leftMiddle.size() - 1);
    }

private:
    unordered_map<int, int> pos;
    int preIdx = 0;

    TreeNode* build(vector<int>& rootFirst, int inLeft, int inRight) {
        if (inLeft > inRight)
            return nullptr;
        int rootVal = rootFirst[preIdx++];
        TreeNode* root = new TreeNode(rootVal);
        int mid = pos[rootVal];
        root->left  = build(rootFirst, inLeft, mid - 1);
        root->right = build(rootFirst, mid + 1, inRight);
        return root;
    }
};
C
C
#include <stdlib.h>

struct TreeNode {
    int val;
    struct TreeNode *left;
    struct TreeNode *right;
};

/* preIdx walks rootFirst left-to-right; pos maps a value to its index in leftMiddle
   (offset by 3000 so negative values index safely). */
static struct TreeNode* build(const int* rootFirst, int inLeft, int inRight,
                              int* preIdx, const int* pos) {
    if (inLeft > inRight)
        return NULL;

    int rootVal = rootFirst[*preIdx];
    (*preIdx)++;

    struct TreeNode* root = (struct TreeNode*)malloc(sizeof(struct TreeNode));
    root->val = rootVal;

    int mid = pos[rootVal + 3000];
    root->left  = build(rootFirst, inLeft, mid - 1, preIdx, pos);
    root->right = build(rootFirst, mid + 1, inRight, preIdx, pos);
    return root;
}

struct TreeNode* buildTree(int* rootFirst, int rootFirstSize,
                           int* leftMiddle, int leftMiddleSize) {
    int* pos = (int*)malloc(sizeof(int) * 6001);
    for (int i = 0; i < leftMiddleSize; ++i)
        pos[leftMiddle[i] + 3000] = i;

    int preIdx = 0;
    struct TreeNode* root = build(rootFirst, 0, leftMiddleSize - 1, &preIdx, pos);
    free(pos);
    return root;
}

#### Follow-up
questions
Can you reconstruct the tree from the root-first and root-last (postorder) walks instead? Explain why that pair is ambiguous unless the tree is full.
What breaks if the values are not unique, and how would you disambiguate the split?
Rewrite the construction iteratively with an explicit stack instead of recursion.
After building, return the tree's height or its level-order serialization without storing the whole tree first.

