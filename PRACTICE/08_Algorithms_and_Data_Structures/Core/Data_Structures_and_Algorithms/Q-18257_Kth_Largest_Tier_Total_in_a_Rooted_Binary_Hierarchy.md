---
type: "practice"
difficulty: "Other"
---

# Kth Largest Tier Total in a Rooted Binary Hierarchy

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Breadth-First Search](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Breadth-First_Search.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given the root of a binary tree together with a positive integer k. Every node stores a positive integer weight.

Group the nodes by their tier (also called depth or level): the root sits on tier 1, its children on tier 2, their children on tier 3, and so on. Two nodes belong to the same tier exactly when they are the same number of edges away from the root.

For each tier, define its tier total as the sum of the weights of all nodes on that tier. Return the k-th largest tier total across the whole tree. The tier totals are not required to be distinct, so ties count as separate values when ranking (for example, if the tier totals are 9, 7, 7, 4 then the 2nd largest is 7 and the 3rd largest is also 7).

If the tree has fewer than k tiers, return -1.

Input:

root: the root node of a binary tree (each node has an integer val, a left child, and a right child).
k: a positive integer.

Output:

A single integer: the k-th largest tier total, or -1 when there are fewer than k tiers.

#### Cons
traints:

Let n be the number of nodes in the tree.
2 <= n <= 10^5
1 <= node weight <= 10^6
1 <= k <= n
A tier total can be as large as 10^5 * 10^6 = 10^11, so it must be held in a 64-bit integer.

#### Example
s & test cases
TEXT
1. Input: root = [10,3,8,1,4,6,2,5,7], k = 2 -> Output: 12
   Why: tier totals are 10 (tier 1), 3+8=11 (tier 2), 1+4+6+2=13 (tier 3), 5+7=12 (tier 4); sorted descending they are 13,12,11,10, so the 2nd largest is 12.

2. Input: root = [10,3,8,1,4,6,2,5,7], k = 1 -> Output: 13
   Why: the single largest tier total is tier 3 with 13. Normal "top-1" case.

3. Input: root = [10,3,8,1,4,6,2,5,7], k = 4 -> Output: 10
   Why: with 4 tiers and k equal to the tier count, the answer is the smallest tier total, which is the root tier (10).

4. Input: root = [7,2,9], k = 5 -> Output: -1
   Why: the tree has only 2 tiers (7, then 2+9=11); k = 5 exceeds the number of tiers, so return -1. Edge: k larger than tier count.

5. Input: root = [4,6], k = 2 -> Output: 4
   Why: minimum-size tree (n = 2) has tiers 4 and 6; sorted descending they are 6,4, so the 2nd largest is 4. Edge: smallest allowed tree.

6. Input: root = [5,2,3], k = 2 -> Output: 5
   Why: tier totals are 5 (root) and 2+3=5; they tie, and since ties are ranked separately the 2nd largest is also 5. Edge: duplicate tier totals.

#### Approach

Do a breadth-first (level-order) traversal. Process the tree one tier at a time by recording the current queue size, popping exactly that many nodes, summing their weights into a tier total, and enqueueing their children for the next tier. Append each tier total to a list. After the sweep, if the list has fewer than k entries return -1; otherwise sort the list in descending order and return the element at index k-1. The core pattern is level-by-level BFS combined with a "k-th largest via sorting" selection. (For very large trees one could replace the full sort with a size-k min-heap to get O(L log k) selection, but sorting is simplest and well within limits here.)

Complexity

Time: O(n + L log L), where n is the number of nodes and L is the number of tiers. Every node is visited exactly once during the level-order sweep (O(n)); the L tier totals are then sorted (O(L log L), and L <= n). Space: O(n) for the queue in the worst case (a full bottom tier holds up to n/2 nodes) plus O(L) for the tier-total list.

Python
PYTHON
from collections import deque

class TreeNode:
    def __init__(self, val=0, left=None, right=None):
        self.val = val
        self.left = left
        self.right = right

class Solution:
    def kthLargestTierTotal(self, root: 'TreeNode', k: int) -> int:
        if not root:
            return -1
        sums = []
        q = deque([root])
        while q:
            s = 0
            for _ in range(len(q)):
                node = q.popleft()
                s += node.val
                if node.left:
                    q.append(node.left)
                if node.right:
                    q.append(node.right)
            sums.append(s)
        if k > len(sums):
            return -1
        sums.sort(reverse=True)
        return sums[k - 1]
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

function kthLargestTierTotal(root, k) {
    if (!root) return -1;

#### cons
t sums = [];
    let level = [root];
    while (level.length > 0) {
        let sum = 0;

#### cons
t next = [];
        for (const node of level) {
            sum += node.val;
            if (node.left) next.push(node.left);
            if (node.right) next.push(node.right);
        }
        sums.push(sum);
        level = next;
    }
    if (k > sums.length) return -1;
    sums.sort((a, b) => b - a);
    return sums[k - 1];
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
    public long kthLargestTierTotal(TreeNode root, int k) {
        if (root == null) return -1;
        List<Long> sums = new ArrayList<>();
        Queue<TreeNode> q = new LinkedList<>();
        q.offer(root);
        while (!q.isEmpty()) {
            long s = 0;
            int sz = q.size();
            for (int i = 0; i < sz; i++) {
                TreeNode node = q.poll();
                s += node.val;
                if (node.left != null)  q.offer(node.left);
                if (node.right != null) q.offer(node.right);
            }
            sums.add(s);
        }
        if (sums.size() < k) return -1;
        sums.sort(Collections.reverseOrder());
        return sums.get(k - 1);
    }
}
C++
CPP
#include <vector>
#include <queue>
#include <algorithm>
using namespace std;

struct TreeNode {
    int val;
    TreeNode *left;
    TreeNode *right;
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
};

class Solution {
public:
    long long kthLargestTierTotal(TreeNode *root, int k) {
        if (!root) return -1;
        vector<long long> sums;
        queue<TreeNode *> q;
        q.push(root);
        while (!q.empty()) {
            long long s = 0;
            int sz = q.size();
            for (int i = 0; i < sz; i++) {
                TreeNode *node = q.front();
                q.pop();
                s += node->val;
                if (node->left)  q.push(node->left);
                if (node->right) q.push(node->right);
            }
            sums.push_back(s);
        }
        if ((int)sums.size() < k) return -1;
        sort(sums.begin(), sums.end(), greater<long long>());
        return sums[k - 1];
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

static int cmpDesc(const void *a, const void *b) {
    long long x = *(const long long *)a;
    long long y = *(const long long *)b;
    if (x < y) return 1;
    if (x > y) return -1;
    return 0;
}

long long kthLargestTierTotal(struct TreeNode *root, int k) {
    if (!root) return -1;

    long long *sums = NULL;
    int sumsCap = 0, sumsLen = 0;

    int qcap = 16, head = 0, tail = 0;
    struct TreeNode **q = malloc(sizeof(struct TreeNode *) * qcap);
    q[tail++] = root;

    while (tail - head > 0) {
        int levelSize = tail - head;
        long long s = 0;
        for (int i = 0; i < levelSize; i++) {
            struct TreeNode *node = q[head++];
            s += node->val;
            if (tail + 2 > qcap) {
                qcap *= 2;
                q = realloc(q, sizeof(struct TreeNode *) * qcap);
            }
            if (node->left)  q[tail++] = node->left;
            if (node->right) q[tail++] = node->right;
        }
        if (sumsLen + 1 > sumsCap) {
            sumsCap = sumsCap ? sumsCap * 2 : 8;
            sums = realloc(sums, sizeof(long long) * sumsCap);
        }
        sums[sumsLen++] = s;
    }
    free(q);

    long long ans;
    if (k > sumsLen) {
        ans = -1;
    } else {
        qsort(sums, sumsLen, sizeof(long long), cmpDesc);
        ans = sums[k - 1];
    }
    free(sums);
    return ans;
}

#### Follow-up
questions
Return the k-th smallest tier total instead of the largest.
Handle streaming input where the tree is enormous: select the k-th largest using a bounded size-k min-heap so you never store all tier totals or fully sort them.
Return the depth (tier index) that produces the k-th largest total, breaking ties by the shallower tier.
Extend to an n-ary tree (each node has an arbitrary list of children) rather than a strict binary tree.

