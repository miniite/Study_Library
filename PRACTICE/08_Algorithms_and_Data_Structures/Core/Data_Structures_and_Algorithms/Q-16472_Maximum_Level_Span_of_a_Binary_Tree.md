---
type: "practice"
difficulty: "Other"
---

# Maximum Level Span of a Binary Tree

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Breadth-First Search](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Breadth-First_Search.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given the root of a binary tree. Imagine that the tree is embedded inside a perfect (fully complete) binary tree of the same height: every node is assigned the position it would occupy in that perfect tree, so a node at position p has its left child at position 2*p and its right child at position 2*p + 1.

For any single level (depth) of the tree, its span is the number of positions between the leftmost present node and the rightmost present node on that level, inclusive. Crucially, the empty positions that fall between those two end nodes — the gaps that would be filled in a perfect tree — are counted toward the span. Positions to the left of the leftmost node and to the right of the rightmost node are not counted.

Return the maximum span taken over all levels of the tree.

It is guaranteed that the answer fits in a signed 32-bit integer.

Input:

root: the root of a binary tree, given in level-order form where null marks a missing child. Each node holds an integer value; the values do not affect the answer.

Output:

A single integer: the largest level span in the tree.

#### Cons
traints:

The number of nodes is in the range [1, 3000].
Each node value is in the range [-100, 100].
The tree always has at least one node, so the answer is at least 1.

#### Example
s & test cases
TEXT
Input: root = [7,4,9,2,6,null,1] -> Output: 4
Why: On the deepest level the present nodes are 2 (position 0), 6 (position 1), a gap (position 2), and 1 (position 3), spanning 4 positions from 0 to 3.

Input: root = [7] -> Output: 1
Why: A single-node tree has one level containing only the root, so the maximum span is 1 (minimum edge case).

Input: root = [7,4,9,2] -> Output: 2
Why: The widest level is the second one, holding 4 and 9 at adjacent positions with no interior gaps, giving span 2.

Input: root = [7,4,9,2,null,null,1,8,null,null,null,null,null,null,5] -> Output: 8
Why: Deep-gap case. The bottom level has 8 at position 0 (left child of 2) and 5 at position 7 (right child of 1); the six missing positions in between are counted, so the span is 8.

Input: root = [7,4,null,2,null,8] -> Output: 1
Why: A fully left-skewed chain — every level has exactly one node, so every level's span is 1 and the maximum stays 1.

#### Approach

Model each node's position with a heap-style index: root = 0, and a node at index p gives its children indices 2p and 2p + 1. Do a level-order (BFS) traversal, and for every level the span is (rightmost index) - (leftmost index) + 1, which automatically counts the missing interior positions. The one trap is that raw indices double each level and overflow for deep trees; the fix is to normalize each level by subtracting the level's leftmost index before computing children, which preserves all index differences (hence the spans) while keeping the numbers bounded by the tree size. The underlying pattern is BFS with implicit array indexing of a binary tree, plus per-level offset normalization.

Complexity

Time: O(n), where n is the number of nodes — every node is enqueued and processed exactly once. Space: O(n) — in the worst case (a nearly complete last level) the queue/level array holds up to about n/2 nodes at once.

Python
PYTHON
from collections import deque
from typing import Optional

class TreeNode:
    def __init__(self, val=0, left=None, right=None):
        self.val = val
        self.left = left
        self.right = right

class Solution:
    def maxLevelSpan(self, root: Optional[TreeNode]) -> int:
        if not root:
            return 0

        ans = 0
        q = deque([(root, 0)])
        while q:
            first = q[0][1]          # leftmost index on this level
            last = first
            for _ in range(len(q)):
                node, idx = q.popleft()
                last = idx
                norm = idx - first    # normalize so indices stay small
                if node.left:
                    q.append((node.left, norm * 2))
                if node.right:
                    q.append((node.right, norm * 2 + 1))
            ans = max(ans, last - first + 1)
        return ans
JavaScript
JAVASCRIPT
function TreeNode(val, left, right) {
    this.val = val === undefined ? 0 : val;
    this.left = left === undefined ? null : left;
    this.right = right === undefined ? null : right;
}

function maxLevelSpan(root) {
    if (!root) return 0;

    let ans = 0;
    let level = [[root, 0]];
    while (level.length) {

#### cons
t first = level[0][1];   // leftmost index at this level
        let last = first;

#### cons
t next = [];
        for (const [node, idx] of level) {
            last = idx;

#### cons
t norm = idx - first; // normalize to keep indices small
            if (node.left)  next.push([node.left,  norm * 2]);
            if (node.right) next.push([node.right, norm * 2 + 1]);
        }
        ans = Math.max(ans, last - first + 1);
        level = next;
    }
    return ans;
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
    public int maxLevelSpan(TreeNode root) {
        if (root == null) return 0;

        int ans = 0;
        Deque<TreeNode> nodes = new ArrayDeque<>();
        Deque<Long> idxs = new ArrayDeque<>();
        nodes.offer(root);
        idxs.offer(0L);

        while (!nodes.isEmpty()) {
            int sz = nodes.size();
            long first = idxs.peekFirst(); // leftmost index at this level
            long last = first;
            for (int i = 0; i < sz; i++) {
                TreeNode node = nodes.pollFirst();
                long idx = idxs.pollFirst();
                last = idx;
                long norm = idx - first; // normalize to avoid overflow
                if (node.left != null)  { nodes.offer(node.left);  idxs.offer(norm * 2); }
                if (node.right != null) { nodes.offer(node.right); idxs.offer(norm * 2 + 1); }
            }
            ans = Math.max(ans, (int)(last - first + 1));
        }
        return ans;
    }
}
C++
CPP
#include <queue>
#include <utility>
#include <algorithm>
using namespace std;

struct TreeNode {
    int val;
    TreeNode* left;
    TreeNode* right;
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
};

class Solution {
public:
    int maxLevelSpan(TreeNode* root) {
        if (!root) return 0;

        int ans = 0;
        queue<pair<TreeNode*, unsigned long long>> q;
        q.push({root, 0ULL});

        while (!q.empty()) {
            int sz = q.size();
            unsigned long long first = q.front().second; // leftmost index
            unsigned long long last = first;
            for (int i = 0; i < sz; ++i) {
                auto [node, idx] = q.front();
                q.pop();
                last = idx;
                unsigned long long norm = idx - first; // keep indices bounded
                if (node->left)  q.push({node->left,  norm * 2ULL});
                if (node->right) q.push({node->right, norm * 2ULL + 1ULL});
            }
            ans = max(ans, (int)(last - first + 1ULL));
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

int maxLevelSpan(struct TreeNode* root) {
    if (!root) return 0;

    int ans = 0;
    int curCount = 1;
    struct TreeNode** curNodes =
        (struct TreeNode**)malloc(sizeof(struct TreeNode*));
    unsigned long long* curIdx =
        (unsigned long long*)malloc(sizeof(unsigned long long));
    curNodes[0] = root;
    curIdx[0] = 0ULL;

    while (curCount > 0) {
        unsigned long long span = curIdx[curCount - 1] - curIdx[0] + 1ULL;
        if ((int)span > ans) ans = (int)span;

        struct TreeNode** nextNodes = (struct TreeNode**)malloc(
            sizeof(struct TreeNode*) * (size_t)curCount * 2);
        unsigned long long* nextIdx = (unsigned long long*)malloc(
            sizeof(unsigned long long) * (size_t)curCount * 2);
        int nextCount = 0;

        /* normalize this level so indices stay small and never overflow */
        unsigned long long offset = curIdx[0];
        for (int i = 0; i < curCount; i++) {
            unsigned long long norm = curIdx[i] - offset;
            if (curNodes[i]->left) {
                nextNodes[nextCount] = curNodes[i]->left;
                nextIdx[nextCount] = norm * 2ULL;
                nextCount++;
            }
            if (curNodes[i]->right) {
                nextNodes[nextCount] = curNodes[i]->right;
                nextIdx[nextCount] = norm * 2ULL + 1ULL;
                nextCount++;
            }
        }

        free(curNodes);
        free(curIdx);
        curNodes = nextNodes;
        curIdx = nextIdx;
        curCount = nextCount;
    }

    free(curNodes);
    free(curIdx);
    return ans;
}

#### Follow-up
questions
How do you keep the indices from overflowing when the tree is deep and positions double every level? (normalize per level or use a wider unsigned type)
Re-solve it with DFS instead of BFS, recording the first index seen at each depth and comparing later nodes against it.
Return the level number that achieves the maximum span, not just the span value.
What degenerates first if the tree is highly skewed versus nearly complete, and how does that affect time and space?

