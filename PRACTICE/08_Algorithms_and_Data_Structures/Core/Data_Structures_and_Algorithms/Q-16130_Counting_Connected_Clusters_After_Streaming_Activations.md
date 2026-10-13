---
type: "practice"
difficulty: "Other"
---

# Counting Connected Clusters After Streaming Activations

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Union Find](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Union_Find.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You control a rectangular panel of lamps arranged in R rows and C columns. Every lamp starts switched off. Lamps are switched on one at a time, in the order given by a stream of activation requests.

You are given the panel dimensions R and C, and a list activations, where activations[t] = [rt, ct] means the t-th request switches on the lamp located at row rt, column ct. If a request targets a lamp that is already on, the panel state does not change.

At every point in time, a cluster is a maximal group of switched-on lamps that are connected to one another. Two on-lamps are directly connected only if they are horizontally or vertically adjacent (up, down, left, right) — diagonal neighbours do not connect. All four borders of the panel are surrounded by off cells.

Process the activation requests in order. After applying each request, record how many clusters currently exist on the panel.

Return an integer array answer of the same length as activations, where answer[t] is the number of clusters present immediately after the t-th request has been applied.

Input

R, C: positive integers giving the number of rows and columns.
activations: a list of [row, col] pairs, processed in the given order.

Output

An integer array answer, where answer[t] is the cluster count after request t.

#### Cons
traints

1 <= R, C
1 <= R * C <= 10^4
1 <= activations.length <= 10^4
activations[t].length == 2
0 <= rt < R
0 <= ct < C
A request may repeat a cell that is already switched on; such a request leaves the cluster count unchanged.

#### Example
s & test cases
TEXT
Input: R = 3, C = 3, activations = [[0,0],[0,1],[1,2],[2,1]] -> Output: [1,1,2,3]
Why: (0,0) makes 1 cluster; (0,1) touches (0,0) so still 1; (1,2) is isolated -> 2; (2,1) is isolated from both -> 3. Standard growth case.

Input: R = 3, C = 4, activations = [[0,0],[0,1],[1,3],[2,3],[0,1]] -> Output: [1,1,2,2,2]
Why: (0,0)->1; (0,1) merges with (0,0)->1; (1,3) isolated->2; (2,3) touches (1,3)->2; the final (0,1) is already on, so the count is unchanged->2. Exercises the duplicate-activation edge case.

Input: R = 1, C = 5, activations = [[0,0],[0,2],[0,4],[0,1],[0,3]] -> Output: [1,2,3,2,1]
Why: three isolated lamps give counts 1,2,3; activating (0,1) bridges (0,0) and (0,2) into one cluster (3->2); activating (0,3) bridges the remaining two clusters into a single row (2->1). A single new lamp can merge two clusters at once.

Input: R = 1, C = 1, activations = [[0,0]] -> Output: [1]
Why: Minimum-size panel; the only lamp forms exactly one cluster.

Input: R = 2, C = 2, activations = [[0,0],[1,1],[0,1],[1,0]] -> Output: [1,2,1,1]
Why: (0,0)->1; diagonal (1,1) does NOT connect -> 2; (0,1) connects both (0,0) and (1,1) -> 1; (1,0) is already surrounded by on-cells and joins the same cluster -> 1. Confirms diagonals are not adjacent.

#### Approach

This is an incremental / dynamic-connectivity problem: cells are added over time and after each addition we must report the number of connected components. A disjoint-set union (union–find) structure is ideal because it supports fast "merge" and "same-set?" queries. Keep a running count of components. When a new cell is switched on, first increment count (it starts as its own component); then look at its four orthogonal neighbours, and for every neighbour that is already on and lies in a different set, union the two sets and decrement count. A single new cell can bridge up to four previously separate components, so up to four decrements can happen in one step. Cells that are switched on again are detected by the parent array still holding a non-off value and are skipped without changing the count. Encoding each (row, col) as the flat index row * C + col lets the whole grid live in one array.

Complexity

Time: O(k · α(R·C)) where k is the number of activations and α is the inverse-Ackermann function from union–find with path compression — effectively O(k), or O(k · log(R·C)) with only path-halving and no union-by-rank. Space: O(R·C) for the parent array plus O(k) for the output. Each activation does at most four constant neighbour checks, each amortized near-constant.

Python
PYTHON
from typing import List

class Solution:
    def countClusters(self, R: int, C: int,
                      activations: List[List[int]]) -> List[int]:
        parent = [-1] * (R * C)  # -1 => off

        def find(x: int) -> int:
            while parent[x] != x:
                parent[x] = parent[parent[x]]  # path halving
                x = parent[x]
            return x

        ans = []
        dirs = ((0, 1), (0, -1), (1, 0), (-1, 0))
        count = 0

        for r, c in activations:
            idx = r * C + c
            if parent[idx] != -1:          # already on
                ans.append(count)
                continue
            parent[idx] = idx
            count += 1
            for dr, dc in dirs:
                nr, nc = r + dr, c + dc
                if nr < 0 or nr >= R or nc < 0 or nc >= C:
                    continue
                nid = nr * C + nc
                if parent[nid] == -1:      # neighbour off
                    continue
                ra, rb = find(idx), find(nid)
                if ra != rb:
                    parent[ra] = rb
                    count -= 1
            ans.append(count)

        return ans
JavaScript
JAVASCRIPT
function countClusters(R, C, activations) {

#### cons
t parent = new Array(R * C).fill(-1); // -1 => off

#### cons
t find = (x) => {
        while (parent[x] !== x) {
            parent[x] = parent[parent[x]]; // path halving
            x = parent[x];
        }
        return x;
    };

#### cons
t dirs = [[0, 1], [0, -1], [1, 0], [-1, 0]];

#### cons
t ans = [];
    let count = 0;

    for (const [r, c] of activations) {

#### cons
t id = r * C + c;
        if (parent[id] !== -1) { ans.push(count); continue; }
        parent[id] = id;
        count++;
        for (const [dr, dc] of dirs) {

#### cons
t nr = r + dr, nc = c + dc;
            if (nr < 0 || nr >= R || nc < 0 || nc >= C) continue;

#### cons
t nid = nr * C + nc;
            if (parent[nid] === -1) continue;

#### cons
t ra = find(id), rb = find(nid);
            if (ra !== rb) { parent[ra] = rb; count--; }
        }
        ans.push(count);
    }
    return ans;
}
Java
JAVA
import java.util.*;

class Solution {
    private int[] parent; // -1 => off

    private int find(int x) {
        while (parent[x] != x) {
            parent[x] = parent[parent[x]];
            x = parent[x];
        }
        return x;
    }

    public List<Integer> countClusters(int R, int C, int[][] activations) {
        parent = new int[R * C];
        Arrays.fill(parent, -1);
        List<Integer> ans = new ArrayList<>();
        int[] dr = {0, 0, 1, -1};
        int[] dc = {1, -1, 0, 0};
        int count = 0;

        for (int[] a : activations) {
            int r = a[0], c = a[1], id = r * C + c;
            if (parent[id] != -1) { ans.add(count); continue; }
            parent[id] = id;
            ++count;
            for (int k = 0; k < 4; ++k) {
                int nr = r + dr[k], nc = c + dc[k];
                if (nr < 0 || nr >= R || nc < 0 || nc >= C) continue;
                int nid = nr * C + nc;
                if (parent[nid] == -1) continue;
                int ra = find(id), rb = find(nid);
                if (ra != rb) { parent[ra] = rb; --count; }
            }
            ans.add(count);
        }
        return ans;
    }
}
C++
CPP
#include <vector>
using namespace std;

class DSU {
 public:
  vector<int> parent; // -1 => off
  explicit DSU(int n) : parent(n, -1) {}
  int find(int x) {
    while (parent[x] != x) {
      parent[x] = parent[parent[x]];
      x = parent[x];
    }
    return x;
  }
};

class Solution {
 public:
  vector<int> countClusters(int R, int C, vector<vector<int>>& activations) {
    DSU dsu(R * C);
    vector<int> ans;
    ans.reserve(activations.size());
    static const int dr[4] = {0, 0, 1, -1};
    static const int dc[4] = {1, -1, 0, 0};
    int count = 0;

    for (const auto& a : activations) {
      int r = a[0], c = a[1], id = r * C + c;
      if (dsu.parent[id] != -1) { ans.push_back(count); continue; }
      dsu.parent[id] = id;
      ++count;
      for (int k = 0; k < 4; ++k) {
        int nr = r + dr[k], nc = c + dc[k];
        if (nr < 0 || nr >= R || nc < 0 || nc >= C) continue;
        int nid = nr * C + nc;
        if (dsu.parent[nid] == -1) continue;
        int ra = dsu.find(id), rb = dsu.find(nid);
        if (ra != rb) { dsu.parent[ra] = rb; --count; }
      }
      ans.push_back(count);
    }
    return ans;
  }
};
C
C
#include <stdlib.h>

static int find(int* parent, int x) {
    while (parent[x] != x) {
        parent[x] = parent[parent[x]]; /* path halving */
        x = parent[x];
    }
    return x;
}

/* Returns a malloc'ed array of length activationsSize via returnSize. */
int* countClusters(int R, int C, int** activations, int activationsSize,
                   int* activationsColSize, int* returnSize) {
    int total = R * C;
    int* parent = (int*)malloc(sizeof(int) * total);
    for (int i = 0; i < total; ++i) parent[i] = -1; /* -1 means "off" */

    int* ans = (int*)malloc(sizeof(int) * activationsSize);
    *returnSize = activationsSize;

    static const int dr[4] = {0, 0, 1, -1};
    static const int dc[4] = {1, -1, 0, 0};
    int count = 0;

    for (int t = 0; t < activationsSize; ++t) {
        int r = activations[t][0];
        int c = activations[t][1];
        int id = r * C + c;
        if (parent[id] != -1) { ans[t] = count; continue; } /* already on */
        parent[id] = id;
        ++count;
        for (int k = 0; k < 4; ++k) {
            int nr = r + dr[k], nc = c + dc[k];
            if (nr < 0 || nr >= R || nc < 0 || nc >= C) continue;
            int nid = nr * C + nc;
            if (parent[nid] == -1) continue; /* neighbour off */
            int ra = find(parent, id), rb = find(parent, nid);
            if (ra != rb) { parent[ra] = rb; --count; }
        }
        ans[t] = count;
    }

    free(parent);
    return ans;
}

#### Follow-up
questions
Prove the O(k · log(R·C)) bound and improve it to near-linear by adding union-by-rank/size alongside path compression.
What changes if diagonal adjacency also connects lamps (8-directional)? (Just widen the neighbour set to eight offsets.)
Support switching lamps back off as well — why is decremental/dynamic connectivity much harder, and how would an offline reverse-time trick help?
The panel can be huge but sparse (R·C up to 10^4 but few activations). Replace the full parent array with a hash map keyed by cell id to bound memory by the number of activated cells.

