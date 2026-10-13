---
type: "practice"
difficulty: "Other"
---

# The K-th Smallest Key in a Search Tree

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Tree](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Tree.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given the root of a binary search tree (BST) and a positive integer k. Return the value of the node that holds the k-th smallest key among all nodes, using 1-indexed ordering (so k = 1 asks for the overall minimum).

Recall that in a binary search tree, for every node all keys in its left subtree are smaller than the node's key and all keys in its right subtree are larger. It is guaranteed that 1 <= k <= n, where n is the number of nodes, so a valid answer always exists.

Input: the root of a BST and an integer k. Each node stores an integer key and pointers to a left and right child, either of which may be absent.

Output: a single integer — the k-th smallest key in the tree.

#### Cons
traints:

The number of nodes n satisfies 1 <= k <= n <= 10^4.
0 <= Node.key <= 10^4.

Follow up: If the tree is modified frequently (nodes inserted and deleted) and you must answer many k-th-smallest queries, how would you augment the structure to make each query fast?

#### Example
s & test cases
TEXT
Input: root = [4, 2, 6, 1, 3, 5, 7], k = 3 -> Output: 3
Why: in-order (sorted) sequence is 1,2,3,4,5,6,7; the 3rd smallest is 3.

Input: root = [8], k = 1 -> Output: 8
Why: a single-node tree; its only key is also the smallest.

Input: root = [20, 10, 30], k = 1 -> Output: 10
Why: sorted order is 10,20,30; the smallest key is the leftmost node 10.

Input: root = [20, 10, 30], k = 3 -> Output: 30
Why: k equals n, so the answer is the maximum key, 30.

Input: root = [5, 4, null, 3, null, 2, null, 1], k = 2 -> Output: 2
Why: a fully left-skewed chain 5->4->3->2->1 sorts to 1,2,3,4,5; the 2nd smallest is 2.

Input: root = [15, 9, 25, null, 12], k = 4 -> Output: 25
Why: sorted order is 9,12,15,25; the 4th (and largest here) key is 25 (k = n edge case).

#### Approach

An in-order traversal of a BST visits keys in strictly increasing order, so the k-th node popped during in-order traversal is exactly the k-th smallest key. Rather than traversing the whole tree, we run the traversal iteratively with an explicit stack and stop as soon as we have popped k nodes: push the entire left spine, pop one node (the next-smallest key), decrement k, and if k hits zero return that node's value — otherwise move into its right subtree and repeat. This early exit makes the cost O(H + k) instead of O(n). For the follow-up with frequent inserts/deletes, augment each node with the size of its subtree; then each k-th query is answered in O(H) by comparing k against the left subtree size and descending left or right accordingly, and a balanced BST makes both updates and queries O(log n).

Complexity

Time: O(H + k), where H is the tree height — we descend to the leftmost node (O(H)) and then pop exactly k nodes in sorted order before stopping. Space: O(H) for the explicit stack; O(log n) for a balanced tree and O(n) worst case for a fully skewed one.

Python
PYTHON
from typing import Optional

class TreeNode:
    def __init__(self, val=0, left=None, right=None):
        self.val = val
        self.left = left
        self.right = right

class Solution:
    def kthSmallest(self, root: Optional[TreeNode], k: int) -> int:
        stack = []
        node = root
        while stack or node:
            while node:
                stack.append(node)
                node = node.left
            node = stack.pop()
            k -= 1
            if k == 0:
                return node.val
            node = node.right
        return -1  # unreachable given 1 <= k <= n
JavaScript
JAVASCRIPT
function TreeNode(val, left = null, right = null) {
    this.val = val;
    this.left = left;
    this.right = right;
}

function kthSmallest(root, k) {

#### cons
t stack = [];
    let node = root;
    while (stack.length > 0 || node !== null) {
        while (node !== null) {
            stack.push(node);
            node = node.left;
        }
        node = stack.pop();
        if (--k === 0)
            return node.val;
        node = node.right;
    }
    return -1; // unreachable given 1 <= k <= n
}
Java
JAVA
import java.util.ArrayDeque;
import java.util.Deque;

class TreeNode {
    int val;
    TreeNode left;
    TreeNode right;
    TreeNode(int x) { val = x; }
}

class Solution {
    public int kthSmallest(TreeNode root, int k) {
        Deque<TreeNode> stack = new ArrayDeque<>();
        TreeNode node = root;
        while (!stack.isEmpty() || node != null) {
            while (node != null) {
                stack.push(node);
                node = node.left;
            }
            node = stack.pop();
            if (--k == 0)
                return node.val;
            node = node.right;
        }
        return -1; // unreachable given 1 <= k <= n
    }
}
C++
CPP
#include <stack>
using namespace std;

struct TreeNode {
    int val;
    TreeNode* left;
    TreeNode* right;
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
};

class Solution {
public:
    int kthSmallest(TreeNode* root, int k) {
        stack<TreeNode*> stk;
        TreeNode* node = root;
        while (!stk.empty() || node) {
            while (node) {
                stk.push(node);
                node = node->left;
            }
            node = stk.top();
            stk.pop();
            if (--k == 0)
                return node->val;
            node = node->right;
        }
        return -1;  // unreachable given 1 <= k <= n
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

int kthSmallest(struct TreeNode* root, int k) {
    struct TreeNode* stack[10001];   // height <= n <= 10^4
    int top = 0;
    struct TreeNode* node = root;
    while (top > 0 || node != NULL) {
        while (node != NULL) {
            stack[top++] = node;
            node = node->left;
        }
        node = stack[--top];
        if (--k == 0)
            return node->val;
        node = node->right;
    }
    return -1;   // unreachable given 1 <= k <= n
}

#### Follow-up
questions
Augment nodes with subtree sizes so repeated k-th-smallest queries under insert/delete each run in O(H).
Return the k-th largest key instead — mirror the traversal (right spine first).
Solve it recursively while still stopping early once k nodes have been counted.
Extend to reporting all keys whose rank falls within an inclusive range [k1, k2].

