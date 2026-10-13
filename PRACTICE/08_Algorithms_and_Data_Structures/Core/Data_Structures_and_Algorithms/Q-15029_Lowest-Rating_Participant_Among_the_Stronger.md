---
type: "practice"
difficulty: "Other"
---

# Lowest-Rating Participant Among the Stronger

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Depth-First Search](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Depth-First_Search.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

There are n participants labeled 0 to n - 1. Every participant has a distinct amount of influence and a distinct focus rating.

You are given:

a list strongerPairs, where each strongerPairs[i] = [a, b] states that participant a has strictly more influence than participant b, and
an integer array focus, where focus[i] is the focus rating of participant i (all values are distinct).

The influence relation is guaranteed to be logically consistent — it never implies that some participant has both more and less influence than another.

For each participant x, consider the set of all participants who definitely have influence greater than or equal to x's (this set always includes x itself, plus everyone reachable from x through the stronger-than relation). Among that set, find the participant with the smallest focus rating. Return an array result where result[x] is the label of that participant.

Input: strongerPairs, a list of ordered pairs of participant labels; and focus, an array of n distinct integers.

Output: an integer array result of length n, where result[x] is the label of the lowest-focus participant guaranteed to be at least as strong as x.

#### Cons
traints:

n == focus.length
1 <= n <= 500
0 <= focus[i] < n, and all values of focus are unique.
0 <= strongerPairs.length <= n * (n - 1) / 2
0 <= a, b < n and a != b for every pair.
All pairs are unique and the relation is acyclic (logically consistent).

#### Example
s & test cases
TEXT
Input: strongerPairs = [[1,0],[2,1],[3,1],[3,7],[4,3],[5,3],[6,3]], focus = [3,2,5,4,6,1,7,0] -> Output: [5,5,2,5,4,5,6,7]
Why: for participant 0 the chain 5 -> 3 -> 1 -> 0 makes 5 reachable, and 5 has the lowest focus (1); participant 2 has no stronger participant with lower focus, so it stays itself.

Input: strongerPairs = [], focus = [0] -> Output: [0]
Why: a single participant with no relations is trivially the lowest-focus person at least as strong as itself.

Input: strongerPairs = [], focus = [2,0,1] -> Output: [0,1,2]
Why: with no influence relations, each participant only knows about itself, so result[x] = x.

Input: strongerPairs = [[0,1],[1,2]], focus = [0,1,2] -> Output: [0,0,0]
Why: 0 is stronger than 1 who is stronger than 2, so 0 is reachable from all three; 0 also holds the lowest focus (0), so it dominates every participant's answer.

Input: strongerPairs = [[2,0],[2,1]], focus = [1,2,0] -> Output: [2,2,2]
Why: participant 2 is stronger than both 0 and 1 and also holds the smallest focus rating (0), so every participant's answer is 2.

Input: strongerPairs = [[0,1]], focus = [3,1] -> Output: [0,1]
Why: participant 0 is stronger than 1, but 0's focus (3) is higher than 1's (1); for x=1 the reachable set {1,0} has its minimum focus at participant 1, so 1 stays itself.

#### Approach

Model the influence relation as a directed acyclic graph where an edge points from a participant to everyone directly known to be stronger. Then the "at least as strong" set of a node x is exactly the nodes reachable from x (including x). For each node we want the reachable node with the minimum focus rating. Compute this with a memoized depth-first search: dfs(u) returns u if none of its stronger neighbors yields a lower focus, otherwise the best-focus label found among u and the recursively computed answers of its neighbors. Memoization (the ans/memo array) guarantees each node is expanded only once, turning the exponential-looking reachability into a single linear pass — the standard DAG-DFS-with-caching / topological relaxation pattern.

Complexity

Let n be the number of participants and m the number of stronger-than pairs. Building the graph is O(n + m). The memoized DFS visits each node once and relaxes each edge once, so the whole computation is O(n + m). Time: O(n + m); Space: O(n + m) for the adjacency lists plus the memo array and recursion stack.

Python
PYTHON
from functools import lru_cache
from typing import List

class Solution:
    def lowestRatingStronger(self, stronger_pairs: List[List[int]], focus: List[int]) -> List[int]:
        n = len(focus)
        graph = [[] for _ in range(n)]
        for a, b in stronger_pairs:
            graph[b].append(a)                 # edge b -> a (a is stronger)

        @lru_cache(maxsize=None)
        def dfs(u: int) -> int:
            best = u
            for v in graph[u]:
                res = dfs(v)
                if focus[res] < focus[best]:
                    best = res
            return best

        return [dfs(i) for i in range(n)]
JavaScript
JAVASCRIPT
function lowestRatingStronger(strongerPairs, focus) {

#### cons
t n = focus.length;

#### cons
t graph = Array.from({ length: n }, () => []);
    for (const [a, b] of strongerPairs) graph[b].push(a);   // edge b -> a (a is stronger)

#### cons
t ans = new Array(n).fill(-1);
    function dfs(u) {
        if (ans[u] !== -1) return ans[u];
        ans[u] = u;
        for (const v of graph[u]) {

#### cons
t res = dfs(v);
            if (focus[res] < focus[ans[u]]) ans[u] = res;
        }
        return ans[u];
    }

    for (let i = 0; i < n; i++) dfs(i);
    return ans;
}
Java
JAVA
import java.util.*;

class Solution {
    private List<Integer>[] g;
    private int[] focus;
    private int[] ans;

    public int[] lowestRatingStronger(int[][] strongerPairs, int[] focus) {
        int n = focus.length;
        this.focus = focus;
        g = new List[n];
        for (int i = 0; i < n; i++) g[i] = new ArrayList<>();
        for (int[] p : strongerPairs) g[p[1]].add(p[0]);   // edge b -> a (a is stronger)

        ans = new int[n];
        Arrays.fill(ans, -1);
        for (int i = 0; i < n; i++) dfs(i);
        return ans;
    }

    private int dfs(int u) {
        if (ans[u] != -1) return ans[u];
        ans[u] = u;
        for (int v : g[u]) {
            int res = dfs(v);
            if (focus[res] < focus[ans[u]]) ans[u] = res;
        }
        return ans[u];
    }
}
C++
CPP
#include <vector>
#include <functional>
using namespace std;

class Solution {
public:
    vector<int> lowestRatingStronger(vector<vector<int>>& strongerPairs, vector<int>& focus) {
        int n = focus.size();
        vector<vector<int>> g(n);
        for (const auto& p : strongerPairs)
            g[p[1]].push_back(p[0]);            // edge b -> a (a is stronger)

        vector<int> ans(n, -1);
        function<int(int)> dfs = [&] (int u) -> int {
            if (ans[u] != -1) return ans[u];
            ans[u] = u;
            for (int v : g[u]) {
                int res = dfs(v);
                if (focus[res] < focus[ans[u]]) ans[u] = res;
            }
            return ans[u];
        };

        for (int i = 0; i < n; ++i) dfs(i);
        return ans;
    }
};
C
C
#include <stdlib.h>

static int** g;        /* adjacency: g[u] = participants at least as strong as u */
static int*  gsz;
static int*  memo;     /* memoized answer per node, -1 = unvisited */
static int*  focusArr;

static int dfs(int u) {
    if (memo[u] != -1) return memo[u];
    memo[u] = u;
    for (int k = 0; k < gsz[u]; ++k) {
        int res = dfs(g[u][k]);
        if (focusArr[res] < focusArr[memo[u]]) memo[u] = res;
    }
    return memo[u];
}

int* lowestRatingStronger(int** strongerPairs, int strongerPairsSize, int* strongerPairsColSize,
                          int* focus, int focusSize, int* returnSize) {
    int n = focusSize;
    int* deg = calloc(n, sizeof(int));
    for (int i = 0; i < strongerPairsSize; ++i) deg[strongerPairs[i][1]]++;

    g   = malloc(sizeof(int*) * n);
    gsz = calloc(n, sizeof(int));
    for (int i = 0; i < n; ++i) g[i] = malloc(sizeof(int) * (deg[i] > 0 ? deg[i] : 1));
    for (int i = 0; i < strongerPairsSize; ++i) {
        int a = strongerPairs[i][0], b = strongerPairs[i][1];
        g[b][gsz[b]++] = a;                 /* edge b -> a (a is stronger) */
    }

    memo = malloc(sizeof(int) * n);
    for (int i = 0; i < n; ++i) memo[i] = -1;
    focusArr = focus;

    for (int i = 0; i < n; ++i) dfs(i);

    for (int i = 0; i < n; ++i) free(g[i]);
    free(g); free(gsz); free(deg);
    *returnSize = n;
    return memo;                            /* caller owns this array */
}

#### Follow-up
questions
Solve it iteratively using Kahn's topological sort instead of recursive DFS, to avoid deep recursion on large chains.
Instead of the minimum focus, return the count of distinct participants guaranteed to be at least as strong as each x.
Handle updates: pairs are added over time and you must re-answer efficiently after each insertion.
What changes if the relation may contain cycles (equal influence groups)? Discuss condensing strongly connected components first.

