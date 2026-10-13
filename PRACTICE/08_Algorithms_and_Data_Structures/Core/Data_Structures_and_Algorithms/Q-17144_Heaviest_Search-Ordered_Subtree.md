---
type: "practice"
difficulty: "Other"
---

# Heaviest Search-Ordered Subtree

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Depth-First Search](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Depth-First_Search.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given the root of a binary tree in which every node stores an integer key. A subtree is any node together with all of its descendants (the node becomes the root of that subtree).

Among all subtrees that themselves obey the binary-search-tree (BST) ordering property, return the largest possible sum of keys.

A subtree obeys the BST ordering property when all of the following hold:

Every key in its left subtree is strictly less than the subtree root's key.
Every key in its right subtree is strictly greater than the subtree root's key.
Both the left and the right subtrees are themselves valid BSTs.

Additional rules:

A single leaf node is always a valid BST.
The empty subtree is considered a valid BST whose key sum is 0. Because of this, the answer is never negative — if every non-empty valid BST has a negative sum, the best choice is the empty subtree and the answer is 0.

Return one integer: the maximum key sum over all valid BST subtrees (including the empty one).

Input: the root of a binary tree (may be described by a level-order serialization where null marks an absent child).

Output: a single integer, the maximum achievable sum.

#### Cons
traints:

The number of nodes is in the range [1, 4 * 10^4].
-4 * 10^4 <= key <= 4 * 10^4.

#### Example
s & test cases
TEXT
Input: root = [7] -> Output: 7
Why: a single positive node is a valid BST; its sum 7 beats the empty subtree (0).

Input: root = [-3] -> Output: 0
Why: the only non-empty BST has sum -3, so the empty subtree (sum 0) is optimal.

Input: root = [-2,-5,-1] -> Output: 0
Why: the whole tree is a valid BST but sums to -8; all keys are negative, so the empty subtree wins.

Input: root = [3,1,5] -> Output: 9
Why: 1 < 3 < 5, so the entire tree is a valid BST and its sum 1+3+5 = 9 is the best.

Input: root = [5,4,8,null,null,6,9,2] -> Output: 25
Why: the root fails the BST rule (a key of 2 sits in its right subtree), but the subtree rooted at 8 holds {2,6,8,9}, which is a valid BST summing to 25 — the heaviest valid subtree.

Input: root = [4,3,null,1,2] -> Output: 2
Why: the parent chains break ordering everywhere except the single leaf 2, whose sum 2 is the largest valid BST sum.

#### Approach

This is tree dynamic programming solved with a single bottom-up (post-order) DFS. A subtree can only be validated once its children are known, so each call returns four aggregated facts about the subtree it roots: whether it is a valid BST, the sum of its keys, and its minimum and maximum keys. An empty child returns a neutral sentinel (valid, sum 0, min = +infinity, max = -infinity) so that a leaf trivially becomes a valid BST. A node forms a valid BST exactly when both children are valid BSTs and the node's key is strictly greater than the maximum key on the left and strictly less than the minimum key on the right. When valid, we compute its total sum, update a global best, and propagate the merged min/max upward; when invalid, we return the invalid flag so no ancestor can be a BST through this branch. Tracking the best sum with a variable initialized to 0 automatically accounts for the empty-subtree option, keeping the answer non-negative.

Complexity

Time: O(n) — each node is visited exactly once during the single post-order pass. Space: O(h) — recursion stack proportional to tree height h (O(n) worst case for a skewed tree, O(log n) if balanced); each call returns O(1) aggregated data.

Python
PYTHON
class TreeNode:
    def __init__(self, val=0, left=None, right=None):
        self.val = val
        self.left = left
        self.right = right

class Solution:
    def maxSumBST(self, root: 'TreeNode') -> int:
        self.best = 0

        def dfs(node):
            # returns (is_bst, sum, min_key, max_key)
            if node is None:
                return (True, 0, float('inf'), float('-inf'))
            l_ok, l_sum, l_min, l_max = dfs(node.left)
            r_ok, r_sum, r_min, r_max = dfs(node.right)

            if l_ok and r_ok and l_max < node.val < r_min:
                total = node.val + l_sum + r_sum
                self.best = max(self.best, total)
                return (True, total,
                        min(node.val, l_min),
                        max(node.val, r_max))
            return (False, 0, 0, 0)

        dfs(root)
        return self.best
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

function maxSumBST(root) {
    let best = 0;

    function dfs(node) {
        // returns [isBST, sum, minKey, maxKey]
        if (node === null) return [true, 0, Infinity, -Infinity];

#### cons
t [lOk, lSum, lMin, lMax] = dfs(node.left);

#### cons
t [rOk, rSum, rMin, rMax] = dfs(node.right);

        if (lOk && rOk && lMax < node.val && node.val < rMin) {

#### cons
t total = node.val + lSum + rSum;
            best = Math.max(best, total);
            return [true, total,
                Math.min(node.val, lMin),
                Math.max(node.val, rMax)];
        }
        return [false, 0, 0, 0];
    }

    dfs(root);
    return best;
}
Java
JAVA
class TreeNode {
    int val;
    TreeNode left;
    TreeNode right;
    TreeNode() {}
    TreeNode(int val) { this.val = val; }
    TreeNode(int val, TreeNode left, TreeNode right) {
        this.val = val; this.left = left; this.right = right;
    }
}

class Solution {
    private int best = 0;

    public int maxSumBST(TreeNode root) {
        best = 0;
        dfs(root);
        return best;
    }

    // returns long[]{isBST, sum, minKey, maxKey}
    private long[] dfs(TreeNode node) {
        if (node == null) {
            return new long[]{1, 0, Long.MAX_VALUE, Long.MIN_VALUE};
        }
        long[] l = dfs(node.left);
        long[] r = dfs(node.right);

        if (l[0] == 1 && r[0] == 1 && node.val > l[3] && node.val < r[2]) {
            long s = (long) node.val + l[1] + r[1];
            best = Math.max(best, (int) s);
            return new long[]{1, s,
                Math.min(node.val, l[2]),
                Math.max(node.val, r[3])};
        }
        return new long[]{0, 0, 0, 0};
    }
}
C++
CPP
#include <array>
#include <algorithm>
#include <climits>
using namespace std;

struct TreeNode {
    int val;
    TreeNode *left;
    TreeNode *right;
    TreeNode() : val(0), left(nullptr), right(nullptr) {}
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
    TreeNode(int x, TreeNode *l, TreeNode *r) : val(x), left(l), right(r) {}
};

class Solution {
public:
    int maxSumBST(TreeNode* root) {
        int best = 0;
        dfs(root, best);
        return best;
    }
private:
    // returns {isBST, sum, minKey, maxKey}
    array<long, 4> dfs(TreeNode* node, int& best) {
        if (!node) return {1, 0, LONG_MAX, LONG_MIN};
        array<long, 4> l = dfs(node->left, best);
        array<long, 4> r = dfs(node->right, best);

        if (l[0] && r[0] && node->val > l[3] && node->val < r[2]) {
            long s = (long)node->val + l[1] + r[1];
            best = max(best, (int)s);
            return {1, s, min((long)node->val, l[2]), max((long)node->val, r[3])};
        }
        return {0, 0, 0, 0};
    }
};
C
C
#include <limits.h>

struct TreeNode {
    int val;
    struct TreeNode *left;
    struct TreeNode *right;
};

/* info carried up: isBst flag, subtree sum, min key, max key */
typedef struct {
    int isBst;
    long sum;
    int lo;
    int hi;
} Info;

static Info dfs(struct TreeNode* node, int* best) {
    if (node == NULL) {
        Info empty = {1, 0, INT_MAX, INT_MIN};
        return empty;
    }
    Info l = dfs(node->left, best);
    Info r = dfs(node->right, best);

    if (l.isBst && r.isBst && node->val > l.hi && node->val < r.lo) {
        long s = (long)node->val + l.sum + r.sum;
        if (s > *best) *best = (int)s;
        Info cur;
        cur.isBst = 1;
        cur.sum = s;
        cur.lo = node->val < l.lo ? node->val : l.lo;
        cur.hi = node->val > r.hi ? node->val : r.hi;
        return cur;
    }
    Info bad = {0, 0, 0, 0};
    return bad;
}

int maxSumBST(struct TreeNode* root) {
    int best = 0;
    dfs(root, &best);
    return best;
}

#### Follow-up
questions
How does the validity check change if the BST is allowed to contain duplicate keys (using <= / >= instead of strict comparisons)?
Instead of the sum, return the root (or size) of the heaviest valid BST subtree.
Also report the node count of that heaviest BST alongside its sum.
For an extremely deep, skewed tree that would overflow the recursion stack, convert the post-order traversal to an explicit iterative stack-based version.

