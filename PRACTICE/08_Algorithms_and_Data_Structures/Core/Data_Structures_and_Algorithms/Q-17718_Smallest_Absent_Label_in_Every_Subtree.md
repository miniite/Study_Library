---
type: "practice"
difficulty: "Other"
---

# Smallest Absent Label in Every Subtree

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Union Find](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Union_Find.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given a rooted tree with n nodes labelled 0 to n - 1. Node 0 is the root. The tree is described by an integer array parent of length n, where parent[i] is the index of the parent of node i. Because node 0 is the root, parent[0] = -1; for every other node, parent[i] is a valid node index.

Every node also stores a distinct positive integer called its code, given in an integer array code of length n, where code[i] is the code held at node i. No two nodes share the same code.

The subtree of a node x consists of x together with all of its descendants (a node is considered part of its own subtree).

For every node i, find the smallest positive integer that does not appear as the code of any node inside the subtree rooted at i. Return these values in an array ans of length n, where ans[i] is that smallest absent positive integer for the subtree of node i.

Input
parent: integer array of length n; parent[0] = -1, and every other entry is the parent node's index.
code: integer array of length n of pairwise-distinct positive integers.
Output
ans: integer array of length n; ans[i] is the smallest positive integer missing from the multiset of codes in the subtree rooted at i.

#### Cons
traints
n == parent.length == code.length
2 <= n <= 10^5
parent[0] == -1
0 <= parent[i] <= n - 1 for every i != 0
parent describes a valid rooted tree (connected, no cycles).
1 <= code[i] <= 10^5
All values in code are distinct.

#### Example
s & test cases
TEXT
- Input: `parent = [-1,0,0,1]`, `code = [4,1,3,2]` -> Output: `[5,3,1,1]`
  Why: subtree of 0 holds codes {1,2,3,4} so 5 is absent; subtree of 1 holds {1,2} so 3 is absent; nodes 2 and 3 individually miss 1. Normal branching case.
- Input: `parent = [-1,0,1,2]`, `code = [2,3,1,5]` -> Output: `[4,2,2,1]`
  Why: a straight chain 0->1->2->3; walking up from the node holding code 1 the answer grows as consecutive codes accumulate.
- Input: `parent = [-1,0,0,1,1]`, `code = [3,1,7,2,4]` -> Output: `[5,3,1,1,1]`
  Why: only the ancestors of the node that holds code 1 (nodes 1 and 0) get answers above 1; every other subtree simply misses 1.
- Input: `parent = [-1,0,1,0]`, `code = [3,4,5,6]` -> Output: `[1,1,1,1]`
  Why: code 1 exists nowhere, so 1 is absent from every subtree. Edge case: value 1 entirely absent.
- Input: `parent = [-1,0]`, `code = [1,2]` -> Output: `[3,1]`
  Why: minimum tree size; the root's subtree has {1,2} missing 3, the single child misses 1.

#### Approach

Key insight: the answer for a subtree is 1 unless that subtree actually contains the code 1. Only the nodes on the path from the (unique) node holding code 1 up to the root can possibly have a subtree containing 1; for every other node the answer is trivially 1. So we prefill all answers with 1, locate the node whose code is 1, and walk upward along its ancestor chain. Along that chain we maintain a boolean seen bitmap of which codes have appeared and a running miss = the smallest positive integer not yet seen. At each ancestor we mark the codes of the newly included nodes — the ancestor's whole subtree except the child branch we just came from, which is already marked — then advance miss while seen[miss] is true. Because each node is marked once and miss is monotonic, the walk is linear. The pattern is DFS subtree accumulation combined with an incrementally-maintained "smallest missing value" (mex) along a root path; a union-find "merge child into parent" formulation solves it with the same idea.

Complexity

Time: O(n + M) where M is the code ceiling (10^5). Every node is marked into seen exactly once across the whole ancestor walk (each iteration only visits the newly added child subtree, skipping the branch already processed), and miss only ever moves forward, so its total advance is bounded by M. Space: O(n + M) for the children lists, the seen bitmap, and the explicit stack.

Python
PYTHON
from typing import List

class Solution:
    def smallestMissing(self, parent: List[int], code: List[int]) -> List[int]:
        n = len(parent)
        ans = [1] * n                                 # default: subtree misses 1
        start = -1
        for i, c in enumerate(code):
            if c == 1:
                start = i
                break
        if start == -1:
            return ans                                # no code 1 -> all answers 1

        children = [[] for _ in range(n)]
        for i in range(1, n):
            children[parent[i]].append(i)

        MAXV = 100002
        seen = bytearray(MAXV)
        miss, prev, cur = 1, -1, start
        while cur != -1:
            stack = [cur]                             # mark subtree(cur), skip prev
            while stack:
                x = stack.pop()
                if code[x] < MAXV:
                    seen[code[x]] = 1
                for ch in children[x]:
                    if ch != prev:
                        stack.append(ch)
            while miss < MAXV and seen[miss]:
                miss += 1
            ans[cur] = miss
            prev, cur = cur, parent[cur]
        return ans
JavaScript
JAVASCRIPT
function smallestMissing(parent, code) {

#### cons
t n = parent.length;

#### cons
t ans = new Array(n).fill(1);                 // default: subtree misses 1
    let start = -1;
    for (let i = 0; i < n; i++) if (code[i] === 1) { start = i; break; }
    if (start === -1) return ans;                     // no code 1 -> all answers 1

#### cons
t children = Array.from({ length: n }, () => []);
    for (let i = 1; i < n; i++) children[parent[i]].push(i);

#### cons
t MAXV = 100002;

#### cons
t seen = new Uint8Array(MAXV);
    let miss = 1, prev = -1, cur = start;
    while (cur !== -1) {

#### cons
t stack = [cur];                          // mark subtree(cur), skip prev
        while (stack.length) {

#### cons
t x = stack.pop();
            if (code[x] < MAXV) seen[code[x]] = 1;
            for (const ch of children[x]) if (ch !== prev) stack.push(ch);
        }
        while (miss < MAXV && seen[miss]) miss++;
        ans[cur] = miss;
        prev = cur;
        cur = parent[cur];
    }
    return ans;
}
Java
JAVA
import java.util.*;

class Solution {
    public int[] smallestMissing(int[] parent, int[] code) {
        int n = parent.length;
        int[] ans = new int[n];
        Arrays.fill(ans, 1);                          // default: subtree misses 1
        int start = -1;
        for (int i = 0; i < n; i++) if (code[i] == 1) { start = i; break; }
        if (start == -1) return ans;                  // no code 1 -> all answers 1

        List<List<Integer>> children = new ArrayList<>();
        for (int i = 0; i < n; i++) children.add(new ArrayList<>());
        for (int i = 1; i < n; i++) children.get(parent[i]).add(i);

        final int MAXV = 100002;
        boolean[] seen = new boolean[MAXV];
        Deque<Integer> stack = new ArrayDeque<>();
        int miss = 1, prev = -1, cur = start;
        while (cur != -1) {
            stack.push(cur);                          // mark subtree(cur), skip prev
            while (!stack.isEmpty()) {
                int x = stack.pop();
                if (code[x] < MAXV) seen[code[x]] = true;
                for (int ch : children.get(x)) if (ch != prev) stack.push(ch);
            }
            while (miss < MAXV && seen[miss]) miss++;
            ans[cur] = miss;
            prev = cur;
            cur = parent[cur];
        }
        return ans;
    }
}
C++
CPP
#include <vector>
using namespace std;

class Solution {
public:
    vector<int> smallestMissing(vector<int>& parent, vector<int>& code) {
        int n = (int)parent.size();
        vector<int> ans(n, 1);                       // default: subtree misses 1
        int start = -1;
        for (int i = 0; i < n; i++) if (code[i] == 1) { start = i; break; }
        if (start == -1) return ans;                 // no code 1 -> all answers 1

        vector<vector<int>> children(n);
        for (int i = 1; i < n; i++) children[parent[i]].push_back(i);

#### cons
t int MAXV = 100002;
        vector<char> seen(MAXV, 0);
        vector<int> stack;
        int miss = 1, prev = -1, cur = start;
        while (cur != -1) {
            stack.clear();
            stack.push_back(cur);                    // mark subtree(cur), skip prev
            while (!stack.empty()) {
                int x = stack.back(); stack.pop_back();
                if (code[x] < MAXV) seen[code[x]] = 1;
                for (int ch : children[x]) if (ch != prev) stack.push_back(ch);
            }
            while (miss < MAXV && seen[miss]) miss++;
            ans[cur] = miss;
            prev = cur;
            cur = parent[cur];
        }
        return ans;
    }
};
C
C
#include <stdlib.h>

/* Returns a malloc'd array of length parentSize; caller frees. */
int* smallestMissing(int* parent, int parentSize, int* code, int codeSize, int* returnSize) {
    int n = parentSize;
    *returnSize = n;
    int* ans = (int*)malloc(sizeof(int) * n);
    for (int i = 0; i < n; i++) ans[i] = 1;      /* default: subtree misses 1 */

    /* locate the node whose code equals 1 */
    int start = -1;
    for (int i = 0; i < n; i++) if (code[i] == 1) { start = i; break; }
    if (start == -1) return ans;                 /* no code 1 -> all answers 1 */

    /* children in CSR form */
    int* head = (int*)calloc(n + 1, sizeof(int));
    for (int i = 1; i < n; i++) head[parent[i] + 1]++;
    for (int i = 0; i < n; i++) head[i + 1] += head[i];
    int* childArr = (int*)malloc(sizeof(int) * (n > 0 ? n : 1));
    int* cursor = (int*)malloc(sizeof(int) * n);
    for (int i = 0; i < n; i++) cursor[i] = head[i];
    for (int i = 1; i < n; i++) { int p = parent[i]; childArr[cursor[p]++] = i; }

#### cons
t int MAXV = 100002;
    char* seen = (char*)calloc(MAXV, sizeof(char));
    int* stack = (int*)malloc(sizeof(int) * n);

    int miss = 1, prev = -1, cur = start;
    while (cur != -1) {
        int sp = 0;
        stack[sp++] = cur;                       /* mark subtree(cur), skip prev */
        while (sp > 0) {
            int x = stack[--sp];
            int c = code[x];
            if (c < MAXV) seen[c] = 1;
            for (int e = head[x]; e < head[x + 1]; e++) {
                int ch = childArr[e];
                if (ch != prev) stack[sp++] = ch;
            }
        }
        while (miss < MAXV && seen[miss]) miss++;
        ans[cur] = miss;
        prev = cur;
        cur = parent[cur];
    }

    free(head); free(childArr); free(cursor); free(seen); free(stack);
    return ans;
}

#### Follow-up
questions
How does the argument change if codes were not required to be distinct (a code could repeat across nodes)?
Answer the same query but for the smallest missing value that is >= L for a given lower bound L.
Give a Union-Find solution that merges each processed child's subtree into its parent instead of re-walking, and argue it is also linear.
Support the tree changing over time (nodes added as leaves) and re-answering efficiently.

