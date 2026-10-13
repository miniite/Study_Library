---
type: "practice"
difficulty: "Other"
---

# Count Downward Paths Summing to a Target

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Depth-First Search](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Depth-First_Search.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given the root of a binary tree whose nodes hold integer values (any of which may be negative), together with an integer target. Count how many downward paths in the tree have node values that add up to exactly target.

A downward path is any sequence of one or more nodes in which every node after the first is a direct child of the node before it. Such a path may begin at any node and end at any node strictly below it — it does not have to start at the root and does not have to end at a leaf. Because it only ever moves from a parent to one of its children, it never travels upward or sideways.

Two paths are counted as different whenever they start or end at different nodes, even if the values they cover happen to be identical. A single node on its own is a valid path (of length one) and counts if its value equals target.

Input: the root of the tree. Each node stores an integer value and references to a left and a right child, either of which may be absent. The tree may be empty. You are also given the integer target.

Output: a single integer — the number of distinct downward paths whose values sum to target.

#### Cons
traints:

The number of nodes is in the range [0, 1000].
Each node value v satisfies -10^9 <= v <= 10^9.
-1000 <= target <= 1000.
Because a path may cover up to 1000 large values, a running sum can exceed the 32-bit range — accumulate sums with 64-bit integers.

#### Example
s & test cases
TEXT
Input: root = [1, 2, 3, 1, null, 2, -1], target = 3 -> Output: 4
Why: the qualifying paths are the single node {3}, {1->2} down the left, {1->3->-1} down the right, and {2->1} starting below the root — four in all.

Input: root = [], target = 5 -> Output: 0
Why: an empty tree contains no nodes, so no path can sum to anything.

Input: root = [7], target = 7 -> Output: 1
Why: the lone node forms a one-node path whose value equals the target.

Input: root = [7], target = 3 -> Output: 0
Why: the only path is the single node 7, which does not equal 3.

Input: root = [5, 2, -1, null, 4], target = 4 -> Output: 2
Why: the single node {4} sums to 4, and the path {5->-1} that starts at the root also sums to 4; negatives and a non-leaf endpoint both matter.

Input: root = [-2, -3, -3], target = -5 -> Output: 2
Why: both {-2->-3} on the left and {-2->-3} on the right sum to -5, and identical-valued paths at different positions are counted separately.

#### Approach

Turn the tree into a running "prefix sum" problem. As a DFS walks from the root to the current node, keep running, the sum of every value on that root-to-node chain. A downward path that ends at the current node and sums to target corresponds exactly to an earlier node on the chain whose prefix sum was running - target; the number of such paths is the count of that earlier prefix value. Maintain a hashmap of prefix_sum -> how many times it appears on the current chain, seeded with {0: 1} so that paths beginning at the root are counted. Before recursing into children, add running to the map; after both children return, decrement it (backtracking) so sibling branches never see prefixes from a chain they don't belong to. This collapses the naive O(n^2) "start a fresh sum from every node" idea into a single O(n) pass — the same prefix-sum-with-hashmap pattern used for counting subarrays with a given sum, lifted onto a tree.

Complexity

Time: O(n) — each node is visited once and does O(1) expected hashmap work. Space: O(n) for the prefix-sum hashmap in the worst case (a skewed tree keeps a full root-to-leaf chain of prefixes), plus O(h) recursion stack where h is the tree height.

Python
PYTHON
from typing import Optional
from collections import defaultdict

class TreeNode:
    def __init__(self, val=0, left=None, right=None):
        self.val = val
        self.left = left
        self.right = right

class Solution:
    def pathSum(self, root: Optional[TreeNode], targetSum: int) -> int:
        prefix = defaultdict(int)
        prefix[0] = 1  # empty prefix seeds paths that start at the root

        def dfs(node: Optional[TreeNode], running: int) -> int:
            if not node:
                return 0
            running += node.val
            found = prefix.get(running - targetSum, 0)
            prefix[running] += 1            # add current prefix before descending
            found += dfs(node.left, running)
            found += dfs(node.right, running)
            prefix[running] -= 1            # backtrack when leaving this branch
            return found

        return dfs(root, 0)
JavaScript
JAVASCRIPT
function TreeNode(val, left = null, right = null) {
    this.val = val;
    this.left = left;
    this.right = right;
}

function pathSum(root, targetSum) {

#### cons
t prefix = new Map();
    prefix.set(0, 1);                        // empty prefix seeds root-started paths

    function dfs(node, running) {
        if (node === null) return 0;
        running += node.val;
        let found = prefix.get(running - targetSum) || 0;
        prefix.set(running, (prefix.get(running) || 0) + 1);   // add prefix before descent
        found += dfs(node.left, running);
        found += dfs(node.right, running);
        prefix.set(running, prefix.get(running) - 1);          // backtrack on the way up
        return found;
    }

    return dfs(root, 0);
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
    public int pathSum(TreeNode root, int targetSum) {
        Map<Long, Integer> prefix = new HashMap<>();
        prefix.put(0L, 1);                   // empty prefix seeds root-started paths
        return dfs(root, 0L, (long) targetSum, prefix);
    }

    private int dfs(TreeNode node, long running, long target, Map<Long, Integer> prefix) {
        if (node == null) return 0;
        running += node.val;
        int found = prefix.getOrDefault(running - target, 0);
        prefix.merge(running, 1, Integer::sum);   // add current prefix before descending
        found += dfs(node.left, running, target, prefix);
        found += dfs(node.right, running, target, prefix);
        prefix.merge(running, -1, Integer::sum);  // backtrack when leaving this branch
        return found;
    }
}
C++
CPP
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
    int pathSum(TreeNode* root, int targetSum) {
        unordered_map<long long, int> prefix;
        prefix[0] = 1;                       // empty prefix seeds root-started paths
        return dfs(root, 0LL, (long long)targetSum, prefix);
    }

private:
    int dfs(TreeNode* node, long long running, long long target,
            unordered_map<long long, int>& prefix) {
        if (!node) return 0;
        running += node->val;
        int found = prefix.count(running - target) ? prefix[running - target] : 0;
        prefix[running]++;                   // add current prefix before descending
        found += dfs(node->left, running, target, prefix);
        found += dfs(node->right, running, target, prefix);
        prefix[running]--;                   // backtrack when leaving this branch
        return found;
    }
};
C
C
#include <stdlib.h>
#include <stddef.h>

struct TreeNode {
    int val;
    struct TreeNode *left;
    struct TreeNode *right;
};

typedef struct Entry {
    long long key;
    int count;
    struct Entry *next;
} Entry;

#define NBUCKETS 4096

typedef struct {
    Entry *buckets[NBUCKETS];
} HashMap;

static unsigned long hashKey(long long key) {
    unsigned long long x = (unsigned long long)key;
    x ^= x >> 33;
    x *= 0xff51afd7ed558ccdULL;
    x ^= x >> 33;
    return (unsigned long)(x & (NBUCKETS - 1));
}

static int mapGet(HashMap *m, long long key) {
    Entry *e = m->buckets[hashKey(key)];
    while (e) {
        if (e->key == key) return e->count;
        e = e->next;
    }
    return 0;
}

static void mapAdd(HashMap *m, long long key, int delta) {
    unsigned long b = hashKey(key);
    Entry *e = m->buckets[b];
    while (e) {
        if (e->key == key) { e->count += delta; return; }
        e = e->next;
    }
    e = (Entry *)malloc(sizeof(Entry));
    e->key = key;
    e->count = delta;
    e->next = m->buckets[b];
    m->buckets[b] = e;
}

static int dfs(struct TreeNode *node, long long running, long long target, HashMap *m) {
    if (!node) return 0;
    running += node->val;
    int found = mapGet(m, running - target);   /* paths ending here summing to target */
    mapAdd(m, running, 1);                      /* record this prefix on the way down  */
    found += dfs(node->left, running, target, m);
    found += dfs(node->right, running, target, m);
    mapAdd(m, running, -1);                     /* backtrack: leave this branch         */
    return found;
}

int pathSum(struct TreeNode* root, int targetSum) {
    HashMap m;
    for (int i = 0; i < NBUCKETS; i++) m.buckets[i] = NULL;
    mapAdd(&m, 0, 1);                           /* empty prefix, so root-started paths count */
    int result = dfs(root, 0, (long long)targetSum, &m);
    for (int i = 0; i < NBUCKETS; i++) {
        Entry *e = m.buckets[i];
        while (e) { Entry *nx = e->next; free(e); e = nx; }
    }
    return result;
}

#### Follow-up
questions
Explain precisely why seeding the prefix map with the single entry {0: 1} is exactly what makes paths that begin at the root count correctly.
What goes wrong if you forget to decrement the prefix count when backtracking out of a subtree? Construct a tree that would then be over-counted.
Return the actual node lists of every qualifying path instead of just the count.
Generalize to a tree where each node has an arbitrary number of children, or restrict to paths of at most k nodes.

