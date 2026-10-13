---
type: "practice"
difficulty: "Other"
---

# Count One-Valued Nodes After Subtree Toggles

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Depth-First Search](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Depth-First_Search.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Cons
ider an implicit rooted binary tree with n nodes whose labels run from 1 to n. The structure is defined purely by the labels:

The root is the node labeled 1.
For any node labeled v with v >= 2, its parent is the node labeled floor(v / 2).
Equivalently, a node labeled v has left child 2*v and right child 2*v + 1, whenever those labels are <= n.

For example, if n = 7, the node labeled 3 has parent floor(3/2) = 1, and the node labeled 7 has parent floor(7/2) = 3.

Every node initially stores the value 0. You are given an integer array ops. For each operation ops[i], you must toggle the value of every node in the subtree rooted at the node labeled ops[i] (a node in that subtree keeps its own subtree; toggling means 0 becomes 1 and 1 becomes 0).

After all operations have been applied in order, return the total number of nodes whose value is 1.

#### Cons
traints:

1 <= n <= 10^5
1 <= ops.length <= 10^5
1 <= ops[i] <= n

#### Example
s & test cases
TEXT
Input: n = 6, ops = [1,3,6] -> Output: 5
Why: Final values by label are 1:1, 2:1, 3:0, 4:1, 5:1, 6:1. Node 3 is toggled by ops on 1 and on 3 (twice) so it returns to 0; the other five end at 1.

Input: n = 7, ops = [2,5,5,7] -> Output: 4
Why: Node 5 is toggled by the op on 2 (its ancestor) and twice by the ops on 5, netting one toggle -> value 1. Final ones are nodes 2, 4, 5, 7. Shows a double toggle cancelling.

Input: n = 1, ops = [1] -> Output: 1
Why: Only the root exists; toggling its subtree flips it once from 0 to 1. Edge case: minimum tree size.

Input: n = 3, ops = [1,1] -> Output: 0
Why: The whole tree is toggled twice, so every node returns to 0. Edge case: even number of full-tree toggles cancels out.

Input: n = 5, ops = [1] -> Output: 5
Why: The subtree of the root is the entire tree, so all five nodes flip from 0 to 1. Edge case: single op covering everything.

#### Approach

The crucial observation is that toggling the subtree rooted at q flips a node x exactly when q lies on the path from the root down to x (that is, q is x itself or one of its ancestors). Therefore the final parity of node x equals the number of operations that targeted an ancestor-or-self of x, taken modulo 2. First tally, for each node, how many operations directly targeted it. Then propagate these counts downward: because the label rule guarantees a node's parent index i/2 is always smaller than i, a single ascending sweep from 1 to n lets each node add its already-finalized parent's cumulative count. A node ends at value 1 iff its accumulated count is odd. This is a root-to-leaf prefix-sum / difference-propagation over an implicit heap-indexed tree, using parity to collapse repeated toggles.

Complexity

Time: O(n + m) where m = ops.length — one tally pass over the operations plus one linear pass over all nodes. Space: O(n) for the per-node toggle-count array.

Python
PYTHON
def count_one_nodes(n, ops):
    """Count nodes holding value 1 after all subtree toggles."""
    flips = [0] * (n + 1)
    for x in ops:
        flips[x] += 1                 # direct toggles landing on each node

    count = 0
    for i in range(1, n + 1):
        if i >= 2:
            flips[i] += flips[i // 2]  # add cumulative ancestor toggles
        if flips[i] & 1:               # odd total => node value is 1
            count += 1
    return count
JavaScript
JAVASCRIPT
function countOneNodes(n, ops) {

#### cons
t flips = new Array(n + 1).fill(0);
    for (const x of ops) flips[x]++;                 // direct toggles per node

    let count = 0;
    for (let i = 1; i <= n; i++) {
        if (i >= 2) flips[i] += flips[Math.floor(i / 2)]; // inherit ancestor toggles
        if (flips[i] & 1) count++;                   // odd total => value 1
    }
    return count;
}
Java
JAVA
class Solution {
    // Count nodes with value 1 after applying every subtree toggle.
    public int countOneNodes(int n, int[] ops) {
        int[] flips = new int[n + 1];
        for (int x : ops) flips[x]++;             // direct toggles per node

        int count = 0;
        for (int i = 1; i <= n; i++) {
            if (i >= 2) flips[i] += flips[i / 2]; // inherit ancestor toggle count
            if ((flips[i] & 1) == 1) count++;     // odd total => value 1
        }
        return count;
    }
}
C++
CPP
#include <vector>
using namespace std;

class Solution {
public:
    // Count nodes with value 1 after applying every subtree toggle.
    int countOneNodes(int n, const vector<int>& ops) {
        vector<int> flips(n + 1, 0);
        for (int x : ops) flips[x]++;            // direct toggles landing on each node

        int count = 0;
        for (int i = 1; i <= n; i++) {
            if (i >= 2) flips[i] += flips[i / 2]; // add cumulative ancestor toggles
            if (flips[i] & 1) count++;            // odd total => node holds 1
        }
        return count;
    }
};
C
C
#include <stdio.h>
#include <stdlib.h>

/* Count nodes left holding value 1 after all subtree toggles. */
int countOneNodes(int n, const int *ops, int opsSize) {
    int *flips = (int *)calloc((size_t)n + 1, sizeof(int));
    for (int i = 0; i < opsSize; i++) flips[ops[i]]++;   /* direct toggle count per node */

    int count = 0;
    for (int i = 1; i <= n; i++) {
        if (i >= 2) flips[i] += flips[i / 2];            /* inherit ancestor toggles */
        if (flips[i] & 1) count++;                       /* odd total => value 1 */
    }
    free(flips);
    return count;
}

int main(void) {
    int ops[] = {1, 3, 6};
    printf("%d\n", countOneNodes(6, ops, 3)); /* 5 */
    return 0;
}

#### Follow-up
questions
How would your solution change if the tree were given by an explicit parent array instead of the implicit label rule?
Can you maintain the running count of one-valued nodes after each operation, online, rather than only at the end?
What if each operation toggled only the nodes at even depth within the chosen subtree?
How would you handle the tree being a general (non-binary) rooted tree given by an adjacency list?

