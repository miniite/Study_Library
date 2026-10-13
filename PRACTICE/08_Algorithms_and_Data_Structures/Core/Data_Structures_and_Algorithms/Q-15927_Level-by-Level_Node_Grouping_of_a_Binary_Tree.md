---
type: "practice"
difficulty: "Other"
---

# Level-by-Level Node Grouping of a Binary Tree

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Breadth-First Search](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Breadth-First_Search.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given the root of a binary tree. Return its level-order grouping: a list of lists where the k-th inner list contains the values of all nodes at depth k, read left to right.

The root sits at depth 0 and forms the first group. Its children form the second group, their children the third, and so on. Within each group, values must appear in left-to-right order as the nodes sit in the tree.

If the tree is empty, return an empty list.

Input: the root node of a binary tree. Each node stores an integer value and pointers to a left child and a right child (either child may be absent).

Output: a list of integer lists — one list per level, top to bottom, each ordered left to right.

#### Cons
traints:

The number of nodes in the tree is in the range [0, 2000].
Each node value v satisfies -1000 <= v <= 1000.

#### Example
s & test cases
TEXT
Input: root = [6, 4, 15, null, null, 11, 20] -> Output: [[6], [4, 15], [11, 20]]
Why: depth 0 is [6], depth 1 is [4, 15], depth 2 gathers the children of 15 as [11, 20].

Input: root = [42] -> Output: [[42]]
Why: a single node produces one level with one value.

Input: root = [] -> Output: []
Why: an empty tree has no levels.

Input: root = [1, 2, 3, 4, null, null, 5] -> Output: [[1], [2, 3], [4, 5]]
Why: left-to-right order is preserved within each level even when some children are missing.

Input: root = [0, -1, -2, -3] -> Output: [[0], [-1, -2], [-3]]
Why: a left-leaning tree still lists each level top-to-bottom, and negatives are handled normally.

Input: root = [7, 7, 7, 7, 7] -> Output: [[7], [7, 7], [7, 7]]
Why: duplicate values are all kept; grouping depends on position, not uniqueness.

#### Approach

Breadth-first search with explicit level boundaries. Seed a queue with the root. On each outer iteration, snapshot the current queue size — that count is exactly the number of nodes on the current level. Pop that many nodes, collect their values into one group, and enqueue each node's present children (left before right, preserving order). Append the group to the answer and repeat until the queue empties. Capturing the size before the inner loop is the key trick that separates one level from the next. The JS version uses a "current level / next level" array swap, which is an equivalent BFS formulation.

Complexity

Time: O(n) — each of the n nodes is enqueued and dequeued exactly once and does O(1) work. Space: O(n) — the output stores every value, and the queue holds at most one full level, which is O(n) in the worst case (a complete tree's last level has ~n/2 nodes).

Python
PYTHON
from collections import deque
from typing import List, Optional

class TreeNode:
    def __init__(self, val=0, left=None, right=None):
        self.val = val
        self.left = left
        self.right = right

class Solution:
    def levelOrder(self, root: Optional['TreeNode']) -> List[List[int]]:
        if not root:
            return []

        ans: List[List[int]] = []
        q = deque([root])
        while q:
            level = []
            for _ in range(len(q)):
                node = q.popleft()
                level.append(node.val)
                if node.left:
                    q.append(node.left)
                if node.right:
                    q.append(node.right)
            ans.append(level)
        return ans
JavaScript
JAVASCRIPT
function TreeNode(val, left = null, right = null) {
    this.val = val;
    this.left = left;
    this.right = right;
}

function levelOrder(root) {
    if (!root) return [];

#### cons
t ans = [];
    let queue = [root];
    while (queue.length) {

#### cons
t level = [];

#### cons
t next = [];
        for (const node of queue) {
            level.push(node.val);
            if (node.left)  next.push(node.left);
            if (node.right) next.push(node.right);
        }
        ans.push(level);
        queue = next;
    }
    return ans;
}
Java
JAVA
import java.util.*;

class TreeNode {
    int val;
    TreeNode left;
    TreeNode right;
    TreeNode(int x) { val = x; }
}

class Solution {
    public List<List<Integer>> levelOrder(TreeNode root) {
        List<List<Integer>> ans = new ArrayList<>();
        if (root == null) return ans;

        Queue<TreeNode> q = new ArrayDeque<>();
        q.offer(root);
        while (!q.isEmpty()) {
            List<Integer> level = new ArrayList<>();
            for (int sz = q.size(); sz > 0; --sz) {
                TreeNode node = q.poll();
                level.add(node.val);
                if (node.left != null)  q.offer(node.left);
                if (node.right != null) q.offer(node.right);
            }
            ans.add(level);
        }
        return ans;
    }
}
C++
CPP
#include <vector>
#include <queue>
using namespace std;

struct TreeNode {
    int val;
    TreeNode* left;
    TreeNode* right;
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
};

class Solution {
public:
    vector<vector<int>> levelOrder(TreeNode* root) {
        vector<vector<int>> ans;
        if (!root) return ans;

        queue<TreeNode*> q;
        q.push(root);
        while (!q.empty()) {
            vector<int> level;
            for (int sz = q.size(); sz > 0; --sz) {
                TreeNode* node = q.front();
                q.pop();
                level.push_back(node->val);
                if (node->left)  q.push(node->left);
                if (node->right) q.push(node->right);
            }
            ans.push_back(move(level));
        }
        return ans;
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

/**
 * Returns an array of levels. *returnSize gets the number of levels and
 * (*returnColumnSizes)[i] gets the count of nodes on level i.
 */
int** levelOrder(struct TreeNode* root, int* returnSize, int** returnColumnSizes) {
    *returnSize = 0;
    *returnColumnSizes = NULL;
    if (root == NULL) return NULL;

    /* Up to 2000 nodes => at most 2000 levels in a degenerate tree. */
    int cap = 2000;
    int** ans = (int**)malloc(sizeof(int*) * cap);
    *returnColumnSizes = (int*)malloc(sizeof(int) * cap);

    struct TreeNode** queue = (struct TreeNode**)malloc(sizeof(struct TreeNode*) * 2001);
    int head = 0, tail = 0;
    queue[tail++] = root;

    while (head < tail) {
        int levelSize = tail - head;
        int* level = (int*)malloc(sizeof(int) * levelSize);
        for (int i = 0; i < levelSize; ++i) {
            struct TreeNode* node = queue[head++];
            level[i] = node->val;
            if (node->left)  queue[tail++] = node->left;
            if (node->right) queue[tail++] = node->right;
        }
        (*returnColumnSizes)[*returnSize] = levelSize;
        ans[(*returnSize)++] = level;
    }

    free(queue);
    return ans;
}

#### Follow-up
questions
Return the levels bottom-up (deepest level first).
Traverse in zig-zag order: left-to-right on one level, right-to-left on the next.
Return only the rightmost node of each level (the right-side view).
Compute the average value of each level instead of listing all values.

