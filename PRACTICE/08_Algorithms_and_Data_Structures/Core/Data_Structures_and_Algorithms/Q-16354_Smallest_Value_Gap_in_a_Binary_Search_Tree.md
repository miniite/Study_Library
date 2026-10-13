---
type: "practice"
difficulty: "Other"
---

# Smallest Value Gap in a Binary Search Tree

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Tree](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Tree.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given the root of a Binary Search Tree (BST). In a BST, every node's value is greater than all values in its left subtree and less than or equal to the values that appear later in sorted order, so an in-order walk of the tree visits the values in non-decreasing order.

Return the minimum absolute difference between the values of any two distinct nodes in the tree. Two nodes are distinct if they occupy different positions in the tree (even if they store equal values), and the difference is |a - b| for their stored values a and b.

Because the structure is a BST, the answer depends only on the multiset of node values: the smallest gap across all pairs equals the smallest gap between values that are adjacent when the values are sorted.

Input: the root node of a BST (each node has an integer value and up to two children).

Output: a single integer — the minimum absolute difference over all pairs of distinct nodes.

#### Cons
traints:

The number of nodes in the tree is in the range [2, 10^4].
0 <= node value <= 10^5.
The tree is a valid binary search tree.

#### Example
s & test cases
TEXT
- Input: BST with node values {4, 8, 12, 20, 22} -> Output: 2
  Why: sorted values are 4, 8, 12, 20, 22; adjacent gaps are 4, 4, 8, 2 and the smallest is 2 (between 20 and 22).
- Input: BST with node values {2, 7, 9, 10} -> Output: 1
  Why: adjacent gaps are 5, 2, 1; the closest pair is 9 and 10.
- Input: BST with node values {5, 8} -> Output: 3
  Why: smallest possible tree (2 nodes); only one pair, gap 3.
- Input: BST with node values {6, 6, 14} -> Output: 0
  Why: two distinct nodes share the value 6, so the minimum absolute difference is 0 (duplicate-value edge case).
- Input: BST with node values {0, 100000} -> Output: 100000
  Why: extreme bounds; the two values are as far apart as the constraints allow.

#### Approach

Key insight: an in-order traversal of a BST yields the values in sorted order. The minimum absolute difference between any two nodes must occur between two values that are adjacent in that sorted sequence — if two values are not adjacent, any value between them would give a smaller gap. So we perform an in-order walk, keep the previously visited value prev, and each time we visit a node we update the answer with cur.val - prev (guaranteed non-negative because the sequence is non-decreasing). This is the classic "BST in-order gives sorted order, compare consecutive elements" technique, a specialization of finding the minimum gap in a sorted array without an explicit sort.

Complexity

Time: O(n) — an in-order traversal visits each of the n nodes exactly once. Space: O(h) where h is the tree height, for the explicit stack (or recursion in the C version): O(log n) for a balanced tree, O(n) worst case for a skewed tree. (Morris traversal would reduce this to O(1) extra space.)

Python
PYTHON
class TreeNode:
    def __init__(self, val=0, left=None, right=None):
        self.val = val
        self.left = left
        self.right = right

class Solution:
    def minValueGap(self, root: 'TreeNode') -> int:
        best = float('inf')
        prev = None
        stack = []
        cur = root
        while cur or stack:
            while cur:
                stack.append(cur)
                cur = cur.left
            cur = stack.pop()
            if prev is not None:
                best = min(best, cur.val - prev)   # in-order => non-negative
            prev = cur.val
            cur = cur.right
        return best
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

function minValueGap(root) {
    let best = Infinity;
    let prev = null;               // no previous value yet

#### cons
t stack = [];
    let cur = root;
    while (cur || stack.length) {
        while (cur) { stack.push(cur); cur = cur.left; }
        cur = stack.pop();
        if (prev !== null) best = Math.min(best, cur.val - prev);
        prev = cur.val;
        cur = cur.right;
    }
    return best;
}
Java
JAVA
import java.util.ArrayDeque;
import java.util.Deque;

class TreeNode {
    int val;
    TreeNode left, right;
    TreeNode(int x) { val = x; }
}

class Solution {
    public int minValueGap(TreeNode root) {
        int best = Integer.MAX_VALUE;
        long prev = Long.MIN_VALUE;         // sentinel for "no previous value"
        Deque<TreeNode> stack = new ArrayDeque<>();
        TreeNode cur = root;
        while (cur != null || !stack.isEmpty()) {
            while (cur != null) { stack.push(cur); cur = cur.left; }
            cur = stack.pop();
            if (prev != Long.MIN_VALUE)
                best = Math.min(best, (int)(cur.val - prev));
            prev = cur.val;
            cur = cur.right;
        }
        return best;
    }
}
C++
CPP
#include <climits>
#include <stack>
#include <algorithm>

struct TreeNode {
    int val;
    TreeNode* left;
    TreeNode* right;
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
};

class Solution {
public:
    int minValueGap(TreeNode* root) {
        int best = INT_MAX;
        long prev = LONG_MIN;               // sentinel for "no previous value"
        std::stack<TreeNode*> st;
        TreeNode* cur = root;
        while (cur || !st.empty()) {
            while (cur) { st.push(cur); cur = cur->left; }
            cur = st.top(); st.pop();
            if (prev != LONG_MIN)
                best = std::min(best, (int)(cur->val - prev));
            prev = cur->val;
            cur = cur->right;
        }
        return best;
    }
};
C
C
#include <limits.h>
#include <stdlib.h>

struct TreeNode {
    int val;
    struct TreeNode *left;
    struct TreeNode *right;
};

static void inorder(struct TreeNode* node, long* prev, int* best) {
    if (!node) return;
    inorder(node->left, prev, best);
    if (*prev != LONG_MIN) {
        long diff = (long)node->val - *prev;   /* sorted order => always >= 0 */
        if (diff < *best) *best = (int)diff;
    }
    *prev = node->val;
    inorder(node->right, prev, best);
}

int minValueGap(struct TreeNode* root) {
    int best = INT_MAX;
    long prev = LONG_MIN;   /* sentinel: no previous value yet */
    inorder(root, &prev, &best);
    return best;
}

#### Follow-up
questions
What if the input is a general binary tree, not a BST? (Collect all values, sort them, then take the minimum adjacent gap in O(n log n).)
Return the actual pair of nodes that achieves the minimum difference, not just the value.
Reduce the extra space to O(1) using Morris in-order traversal.
How does the answer change if duplicate values are allowed, or if values may be negative?

