---
type: "practice"
difficulty: "Other"
---

# Build a Height-Balanced Search Tree from a Sorted List

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Divide and Conquer](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Divide_and_Conquer.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given an array of integers that is already sorted in strictly increasing order. Construct a binary search tree (BST) from these values such that the resulting tree is height-balanced.

A binary search tree is a binary tree in which, for every node, all values in its left subtree are smaller than the node's value and all values in its right subtree are larger. A tree is height-balanced when, for every node, the heights of its left and right subtrees differ by at most one.

Because more than one valid height-balanced BST can usually be built from the same input, any correct height-balanced BST whose in-order traversal reproduces the given sorted array is an acceptable answer.

Input: a sorted array vals of distinct integers. Output: the root of a height-balanced binary search tree built from those values.

#### Cons
traints:

1 <= vals.length <= 10^4.
-10^4 <= vals[i] <= 10^4.
vals is sorted in strictly increasing order (all elements are distinct).

#### Example
s & test cases
TEXT
- Input: vals = [-8,-2,1,6,11] -> Output: [1,-8,6,null,-2,null,11]
  Why: picking the middle element (1) as root and recursing on each half yields a balanced tree of height 2; its in-order walk is the original array.
- Input: vals = [-4,-1,0,3,8,12] -> Output: [0,-4,8,null,-1,3,12]
  Why: even length; the lower-middle index (value 0) becomes the root, giving a valid balanced BST.
- Input: vals = [2,5] -> Output: [2,null,5]
  Why: two elements; the root is the lower-middle (2) and 5 becomes its right child — heights differ by exactly one, which is allowed.
- Input: vals = [7] -> Output: [7]
  Why: a single element is itself a balanced BST with just a root.
- Input: vals = [-3,0,3,9] -> Output: [0,-3,3,null,null,null,9]
  Why: root is value 0 (index 1); left subtree is the single node -3, right subtree is [3,9] with 3 as root and 9 as its right child.

#### Approach

Because the array is sorted, choosing its middle element as the root guarantees an equal (±1) split of the remaining values into the left and right subtrees, which is exactly the condition for height balance. Recursively apply the same rule to the left half to build the left subtree and to the right half to build the right subtree; a range where the low index passes the high index yields an empty (null) subtree. The middle element being the root also preserves the BST ordering property, since everything to its left is smaller and everything to its right is larger. This is a classic divide-and-conquer construction over index ranges — no data is copied, only the [lo, hi] bounds are passed down.

Complexity

Time: O(n) — each of the n array elements becomes exactly one tree node, created in constant work. Space: O(log n) for the recursion stack (the tree is balanced, so its height is O(log n)), excluding the O(n) output tree itself.

Python
PYTHON
from typing import Optional, List

class TreeNode:
    def __init__(self, val=0, left=None, right=None):
        self.val = val
        self.left = left
        self.right = right

class Solution:
    def sortedArrayToBST(self, vals: List[int]) -> Optional[TreeNode]:
        def build(lo: int, hi: int) -> Optional[TreeNode]:
            if lo > hi:
                return None
            mid = (lo + hi) // 2
            return TreeNode(vals[mid],
                            build(lo, mid - 1),
                            build(mid + 1, hi))

        return build(0, len(vals) - 1)
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

function sortedArrayToBST(vals) {
    function build(lo, hi) {
        if (lo > hi) return null;

#### cons
t mid = lo + Math.floor((hi - lo) / 2);

#### cons
t node = new TreeNode(vals[mid]);
        node.left  = build(lo, mid - 1);
        node.right = build(mid + 1, hi);
        return node;
    }

    return build(0, vals.length - 1);
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
    public TreeNode sortedArrayToBST(int[] vals) {
        return build(vals, 0, vals.length - 1);
    }

    private TreeNode build(int[] vals, int lo, int hi) {
        if (lo > hi) return null;

        int mid = lo + (hi - lo) / 2;
        TreeNode node = new TreeNode(vals[mid]);
        node.left  = build(vals, lo, mid - 1);
        node.right = build(vals, mid + 1, hi);
        return node;
    }
}
C++
CPP
#include <vector>
using namespace std;

struct TreeNode {
    int val;
    TreeNode* left;
    TreeNode* right;
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
};

class Solution {
public:
    TreeNode* sortedArrayToBST(vector<int>& vals) {
        return build(vals, 0, (int)vals.size() - 1);
    }

private:
    TreeNode* build(const vector<int>& vals, int lo, int hi) {
        if (lo > hi) return nullptr;

        int mid = lo + (hi - lo) / 2;
        TreeNode* node = new TreeNode(vals[mid]);
        node->left  = build(vals, lo, mid - 1);
        node->right = build(vals, mid + 1, hi);
        return node;
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

static struct TreeNode* build(int* vals, int lo, int hi) {
    if (lo > hi) return NULL;

    int mid = lo + (hi - lo) / 2;
    struct TreeNode* node = malloc(sizeof(struct TreeNode));
    node->val   = vals[mid];
    node->left  = build(vals, lo, mid - 1);
    node->right = build(vals, mid + 1, hi);
    return node;
}

struct TreeNode* sortedArrayToBST(int* vals, int valsSize) {
    return build(vals, 0, valsSize - 1);
}

#### Follow-up
questions
If the input were a sorted singly linked list instead of an array, how would you build the balanced BST (in-order simulation in O(n) without random access)?
Prove that always choosing the middle element produces a tree whose height is O(log n).
Choose the upper-middle element instead of the lower-middle for even-length ranges — is the result still valid and balanced?
Extend it to keep the tree balanced as new sorted values are appended over time.

