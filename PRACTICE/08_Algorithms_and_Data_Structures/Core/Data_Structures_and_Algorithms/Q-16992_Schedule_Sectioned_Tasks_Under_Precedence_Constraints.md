---
type: "practice"
difficulty: "Other"
---

# Schedule Sectioned Tasks Under Precedence Constraints

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Topological Sort](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Topological_Sort.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You must produce a single linear ordering of n tasks, numbered 0 to n - 1.

Some tasks are filed under a section. You are given an array section of length n, where section[i] is the index of the section that task i belongs to (a value in 0 .. m - 1), or -1 if task i belongs to no section at all. There are m sections in total, numbered 0 to m - 1, and a section is allowed to end up with no tasks in it.

You are also given precedence rules as a list before, where before[i] is the list of tasks that must appear somewhere to the left of (earlier than) task i in the final ordering.

Return an ordering of all n tasks that satisfies BOTH conditions:

Contiguity — all tasks that belong to the same section must form one unbroken block. No task from a different section (and no unsectioned task) may sit between two tasks of the same section.
Precedence — for every task i, every task listed in before[i] must appear earlier in the ordering than task i itself.

A task with section = -1 belongs to no section; it may be placed anywhere and behaves as its own independent, self-contained block.

If more than one valid ordering exists, return any one of them. If it is impossible to satisfy all of the rules, return an empty list.

Input

n: the number of tasks.
m: the number of sections.
section: an array of length n; each entry is in [-1, m - 1].
before: an array of n lists; before[i] holds distinct task indices, none equal to i.

Output

A list of n distinct task indices forming a valid ordering, or an empty list if none exists.

#### Cons
traints

1 <= m <= n <= 3 * 10^4
section.length == before.length == n
-1 <= section[i] <= m - 1
0 <= before[i].length <= n - 1
0 <= before[i][j] <= n - 1
i is never contained in before[i], and before[i] has no duplicate entries.

#### Example
s & test cases
TEXT
- Input: `n = 6, m = 2, section = [-1,0,0,1,1,-1], before = [[],[],[1],[],[3],[2]]` -> Output: `[1,2,3,4,0,5]`
  Why: inside section 0, task 1 must precede task 2; inside section 1, task 3 must precede task 4; task 2 (section 0) must precede standalone task 5, so section 0's block comes before task 5. Each section stays contiguous. (Any ordering meeting the rules is acceptable.)
- Input: `n = 3, m = 1, section = [-1,-1,-1], before = [[],[0],[1]]` -> Output: `[0,1,2]`
  Why: no task is sectioned, so contiguity is trivial; the precedence chain 0 -> 1 -> 2 forces this single ordering (a plain topological chain).
- Input: `n = 4, m = 1, section = [0,0,-1,-1], before = [[1],[0],[],[]]` -> Output: `[]`
  Why: task 0 must come before task 1 AND task 1 before task 0 — a cycle inside one section, so no ordering is possible (impossibility case).
- Input: `n = 2, m = 2, section = [0,1], before = [[],[]]` -> Output: `[0,1]`
  Why: two singleton sections with no constraints; any order works and contiguity holds trivially (minimal multi-section case).
- Input: `n = 5, m = 2, section = [0,1,0,1,-1], before = [[],[],[0],[1],[]]` -> Output: `[0,2,1,3,4]`
  Why: section 0 = {0,2} with 0 before 2, section 1 = {1,3} with 1 before 3, task 4 standalone; both sections remain contiguous and both intra-section orders hold. (Any valid ordering accepted.)

#### Approach

This is a two-level (nested) topological sort. First, give every unsectioned task (section = -1) its own brand-new unique section, so that "each loner is its own block" is expressed uniformly and every task now belongs to exactly one section. Then split each precedence edge p -> i into one of two graphs: if p and i share a section, it is an intra-section (item-level) edge; if they are in different sections, it becomes an inter-section (group-level) edge between their two sections. Topologically sort the item-level graph to get a valid relative order of tasks within each section, and topologically sort the group-level graph to get a valid order of the sections themselves. If either sort fails to place all its nodes, a cycle exists and no ordering is possible, so return empty. Otherwise, bucket the item order by section (preserving the item-level order inside each bucket) and emit the buckets in the section order. Contiguity is guaranteed because each section is emitted as one whole bucket; precedence is guaranteed because intra-section edges are honored inside buckets and inter-section edges are honored by the section ordering. The underlying pattern is dependency resolution via Kahn's algorithm applied at two granularities.

Complexity

Time: O(n + E), where E is the total number of precedence pairs (the sum of all before[i] lengths). Building both graphs is O(n + E), and each topological sort (item-level and group-level) visits every node and edge once. Space: O(n + E) for the two adjacency structures, indegree arrays, and the per-section buckets. Justification: every task and every precedence edge is touched a constant number of times; the two Kahn passes and the bucketing are all linear.

Python
PYTHON
from collections import deque
from typing import List

class Solution:
    def sortTasks(self, n: int, m: int, section: List[int],
                  before: List[List[int]]) -> List[int]:
        sec = section[:]
        groups = m
        for i in range(n):                       # each unsectioned task -> its own section
            if sec[i] == -1:
                sec[i] = groups
                groups += 1

        item_g = [[] for _ in range(n)]
        grp_g = [[] for _ in range(groups)]
        item_in = [0] * n
        grp_in = [0] * groups
        for i in range(n):
            for p in before[i]:
                if sec[p] == sec[i]:             # same section -> item-level edge
                    item_g[p].append(i)
                    item_in[i] += 1
                else:                            # cross section -> group-level edge
                    grp_g[sec[p]].append(sec[i])
                    grp_in[sec[i]] += 1

        def topo(graph, indeg, nodes):
            order = []
            q = deque(i for i in range(nodes) if indeg[i] == 0)
            while q:
                u = q.popleft()
                order.append(u)
                for v in graph[u]:
                    indeg[v] -= 1
                    if indeg[v] == 0:
                        q.append(v)
            return order if len(order) == nodes else None

        item_order = topo(item_g, item_in, n)
        grp_order = topo(grp_g, grp_in, groups)
        if item_order is None or grp_order is None:
            return []

        bucket = [[] for _ in range(groups)]
        for it in item_order:
            bucket[sec[it]].append(it)
        res = []
        for g in grp_order:
            res.extend(bucket[g])
        return res
JavaScript
JAVASCRIPT
function sortTasks(n, m, section, before) {

#### cons
t sec = section.slice();
    let groups = m;
    for (let i = 0; i < n; i++) if (sec[i] === -1) sec[i] = groups++; // own section per loner

#### cons
t itemG = Array.from({ length: n }, () => []);

#### cons
t grpG = Array.from({ length: groups }, () => []);

#### cons
t itemIn = new Array(n).fill(0);

#### cons
t grpIn = new Array(groups).fill(0);
    for (let i = 0; i < n; i++) {
        for (const p of before[i]) {
            if (sec[p] === sec[i]) { itemG[p].push(i); itemIn[i]++; }
            else { grpG[sec[p]].push(sec[i]); grpIn[sec[i]]++; }
        }
    }

#### cons
t topo = (graph, indeg, nodes) => {

#### cons
t order = [];

#### cons
t q = [];
        for (let i = 0; i < nodes; i++) if (indeg[i] === 0) q.push(i);
        for (let h = 0; h < q.length; h++) {

#### cons
t u = q[h];
            order.push(u);
            for (const v of graph[u]) if (--indeg[v] === 0) q.push(v);
        }
        return order.length === nodes ? order : null;
    };

#### cons
t itemOrder = topo(itemG, itemIn, n);

#### cons
t grpOrder = topo(grpG, grpIn, groups);
    if (!itemOrder || !grpOrder) return [];

#### cons
t bucket = Array.from({ length: groups }, () => []);
    for (const it of itemOrder) bucket[sec[it]].push(it);

#### cons
t res = [];
    for (const g of grpOrder) for (const it of bucket[g]) res.push(it);
    return res;
}
Java
JAVA
import java.util.*;

class Solution {
    public int[] sortTasks(int n, int m, int[] section, List<List<Integer>> before) {
        int groups = m;
        int[] sec = section.clone();
        for (int i = 0; i < n; i++)
            if (sec[i] == -1) sec[i] = groups++;          // own section per unsectioned task

        List<List<Integer>> itemG = new ArrayList<>(), grpG = new ArrayList<>();
        for (int i = 0; i < n; i++) itemG.add(new ArrayList<>());
        for (int i = 0; i < groups; i++) grpG.add(new ArrayList<>());
        int[] itemIn = new int[n], grpIn = new int[groups];
        for (int i = 0; i < n; i++)
            for (int p : before.get(i)) {
                if (sec[p] == sec[i]) { itemG.get(p).add(i); itemIn[i]++; }
                else { grpG.get(sec[p]).add(sec[i]); grpIn[sec[i]]++; }
            }

        int[] itemOrder = topo(itemG, itemIn, n);
        int[] grpOrder  = topo(grpG, grpIn, groups);
        if (itemOrder == null || grpOrder == null) return new int[0];

        List<List<Integer>> bucket = new ArrayList<>();
        for (int i = 0; i < groups; i++) bucket.add(new ArrayList<>());
        for (int it : itemOrder) bucket.get(sec[it]).add(it);

        int[] res = new int[n];
        int idx = 0;
        for (int g : grpOrder)
            for (int it : bucket.get(g)) res[idx++] = it;
        return res;
    }

    // Kahn's algorithm; returns a full order, or null if a cycle prevents one.
    private int[] topo(List<List<Integer>> g, int[] indeg, int nodes) {
        int[] order = new int[nodes];
        int cnt = 0;
        Deque<Integer> q = new ArrayDeque<>();
        for (int i = 0; i < nodes; i++) if (indeg[i] == 0) q.add(i);
        while (!q.isEmpty()) {
            int u = q.poll();
            order[cnt++] = u;
            for (int v : g.get(u)) if (--indeg[v] == 0) q.add(v);
        }
        return cnt == nodes ? order : null;
    }
}
C++
CPP
#include <vector>
#include <queue>
using namespace std;

class Solution {
    // Kahn's algorithm; returns a full topological order, or {} if a cycle exists.
    vector<int> topo(const vector<vector<int>>& g, vector<int> indeg) {
        vector<int> order;
        queue<int> q;
        for (int i = 0; i < (int)indeg.size(); ++i)
            if (indeg[i] == 0) q.push(i);
        while (!q.empty()) {
            int u = q.front(); q.pop();
            order.push_back(u);
            for (int v : g[u])
                if (--indeg[v] == 0) q.push(v);
        }
        return order.size() == indeg.size() ? order : vector<int>{};
    }

 public:
    vector<int> sortTasks(int n, int m, vector<int> section,
                          vector<vector<int>>& before) {
        int groups = m;
        for (int i = 0; i < n; ++i)
            if (section[i] == -1) section[i] = groups++;   // own section per unsectioned task

        vector<vector<int>> itemG(n), grpG(groups);
        vector<int> itemIn(n, 0), grpIn(groups, 0);
        for (int i = 0; i < n; ++i)
            for (int p : before[i]) {
                if (section[p] == section[i]) { itemG[p].push_back(i); itemIn[i]++; }
                else { grpG[section[p]].push_back(section[i]); grpIn[section[i]]++; }
            }

        vector<int> itemOrder = topo(itemG, itemIn);
        vector<int> grpOrder  = topo(grpG, grpIn);
        if ((int)itemOrder.size() != n || (int)grpOrder.size() != groups)
            return {};

        vector<vector<int>> bucket(groups);
        for (int it : itemOrder) bucket[section[it]].push_back(it);
        vector<int> res;
        res.reserve(n);
        for (int g : grpOrder)
            for (int it : bucket[g]) res.push_back(it);
        return res;
    }
};
C
C
#include <stdlib.h>

/* Kahn's BFS topological sort. Fills order[] with a valid ordering and returns 1
   when every node is placed (graph acyclic), or 0 if a cycle blocks a full order.
   indeg[] is consumed (decremented) in place. */
static int topoSort(int nnodes, const int *head, const int *nxt,

#### cons
t int *to, int *indeg, int *order) {
    int *q = (int *)malloc(sizeof(int) * (nnodes > 0 ? nnodes : 1));
    int qh = 0, qt = 0, cnt = 0;
    for (int i = 0; i < nnodes; ++i)
        if (indeg[i] == 0) q[qt++] = i;
    while (qh < qt) {
        int u = q[qh++];
        order[cnt++] = u;
        for (int e = head[u]; e != -1; e = nxt[e]) {
            int v = to[e];
            if (--indeg[v] == 0) q[qt++] = v;
        }
    }
    free(q);
    return cnt == nnodes;
}

/* Returns a malloc'd ordering of the n tasks (*returnSize == n), or an empty
   array (*returnSize == 0) when the constraints are unsatisfiable. */
int *sortTasks(int n, int m, const int *section, int **before,

#### cons
t int *beforeColSize, int *returnSize) {
    int groups = m;
    int *sec = (int *)malloc(sizeof(int) * n);
    for (int i = 0; i < n; ++i) sec[i] = section[i];
    for (int i = 0; i < n; ++i)                 /* each unsectioned task gets its own section */
        if (sec[i] == -1) sec[i] = groups++;

    long edges = 0;
    for (int i = 0; i < n; ++i) edges += beforeColSize[i];
    long cap = edges > 0 ? edges : 1;

    int *itemHead = (int *)malloc(sizeof(int) * (n > 0 ? n : 1));
    int *grpHead  = (int *)malloc(sizeof(int) * (groups > 0 ? groups : 1));
    for (int i = 0; i < n; ++i)      itemHead[i] = -1;
    for (int i = 0; i < groups; ++i) grpHead[i]  = -1;

    int *itemNext = (int *)malloc(sizeof(int) * cap);
    int *itemTo   = (int *)malloc(sizeof(int) * cap);
    int *grpNext  = (int *)malloc(sizeof(int) * cap);
    int *grpTo    = (int *)malloc(sizeof(int) * cap);
    int *itemIndeg = (int *)calloc(n > 0 ? n : 1, sizeof(int));
    int *grpIndeg  = (int *)calloc(groups > 0 ? groups : 1, sizeof(int));

    int ie = 0, ge = 0;
    for (int i = 0; i < n; ++i) {
        for (int j = 0; j < beforeColSize[i]; ++j) {
            int p = before[i][j];
            if (sec[p] == sec[i]) {              /* same section -> item-level edge p -> i */
                itemTo[ie] = i; itemNext[ie] = itemHead[p]; itemHead[p] = ie++;
                itemIndeg[i]++;
            } else {                             /* cross section -> group-level edge */
                grpTo[ge] = sec[i]; grpNext[ge] = grpHead[sec[p]]; grpHead[sec[p]] = ge++;
                grpIndeg[sec[i]]++;
            }
        }
    }

    int *itemOrder = (int *)malloc(sizeof(int) * (n > 0 ? n : 1));
    int *grpOrder  = (int *)malloc(sizeof(int) * (groups > 0 ? groups : 1));
    int *res = (int *)malloc(sizeof(int) * (n > 0 ? n : 1));
    *returnSize = 0;

    if (topoSort(n, itemHead, itemNext, itemTo, itemIndeg, itemOrder) &&
        topoSort(groups, grpHead, grpNext, grpTo, grpIndeg, grpOrder)) {
        int *count = (int *)calloc(groups > 0 ? groups : 1, sizeof(int));
        int *pos   = (int *)malloc(sizeof(int) * (groups > 0 ? groups : 1));
        int *fill  = (int *)calloc(groups > 0 ? groups : 1, sizeof(int));
        for (int i = 0; i < n; ++i) count[sec[i]]++;
        int acc = 0;
        for (int k = 0; k < groups; ++k) { int g = grpOrder[k]; pos[g] = acc; acc += count[g]; }
        for (int i = 0; i < n; ++i) { int it = itemOrder[i]; int g = sec[it]; res[pos[g] + fill[g]++] = it; }
        *returnSize = n;
        free(count); free(pos); free(fill);
    }

    free(sec); free(itemHead); free(grpHead); free(itemNext); free(itemTo);
    free(grpNext); free(grpTo); free(itemIndeg); free(grpIndeg);
    free(itemOrder); free(grpOrder);
    if (*returnSize == 0) { free(res); res = (int *)malloc(0); }
    return res;
}

#### Follow-up
questions
Report which precedence rule (or which cycle of tasks) makes the ordering impossible, not just that it is impossible.
What changes if a task may belong to more than one section at once?
Produce the lexicographically smallest valid ordering instead of any valid one — how do the two Kahn passes change?
If independent tasks can run in parallel, what is the minimum number of sequential "rounds" needed to finish all tasks under the same constraints?

