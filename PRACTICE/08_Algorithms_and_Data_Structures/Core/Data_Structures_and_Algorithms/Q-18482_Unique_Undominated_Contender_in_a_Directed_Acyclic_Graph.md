---
type: "practice"
difficulty: "Other"
---

# Unique Undominated Contender in a Directed Acyclic Graph

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Graph](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Graph.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

A knockout-style strength ranking is held among k contenders labeled from 0 to k - 1. Their pairwise strength is described by a directed acyclic graph (DAG) in which each contender is a node.

You are given the integer k and a list links of m directed edges, where each links[i] = [x, y] means there is a directed edge from contender x to contender y. Such an edge means contender x is strictly stronger than contender y (equivalently, contender y is weaker than contender x).

The strength relation behaves consistently:

It is antisymmetric: if x is stronger than y, then y is never stronger than x.
It is transitive: if x is stronger than y and y is stronger than z, then x is stronger than z.
The graph contains no cycle (it is a DAG).

A contender is the supreme champion if there is no other contender that is stronger than it.

Return the label of the supreme champion if it is unique; otherwise return -1.

A key observation: a contender has someone stronger than it exactly when it has at least one incoming edge (directly or, by transitivity, along some path — and any such path ends with an incoming edge at that node). So the supreme champion is a node with in-degree 0. If exactly one node has in-degree 0 it is the unique champion; if two or more nodes have in-degree 0, none of them is stronger than the others, so the champion is ambiguous.

#### Cons
traints

1 <= k <= 100
0 <= m <= k * (k - 1) / 2
links[i].length == 2
0 <= links[i][0], links[i][1] <= k - 1
links[i][0] != links[i][1]
The input is a valid DAG whose relation is antisymmetric and transitive.

#### Example
s & test cases
TEXT
Input: k = 3, links = [[0,1],[1,2]] -> Output: 0
Why: only contender 0 has no incoming edge (0 > 1 > 2), so it is the unique champion.

Input: k = 4, links = [[0,2],[1,3],[1,2]] -> Output: -1
Why: contenders 0 and 1 both have in-degree 0; neither dominates the other, so the champion is ambiguous.

Input: k = 1, links = [] -> Output: 0
Why: a single contender is trivially undominated and unique.

Input: k = 2, links = [] -> Output: -1
Why: two independent contenders each have in-degree 0, so there is no unique champion.

Input: k = 5, links = [[3,0],[3,1],[3,2],[3,4]] -> Output: 3
Why: contender 3 is stronger than everyone (in-degree 0) and is the only such node.

Input: k = 2, links = [[1,0]] -> Output: 1
Why: contender 1 is stronger than 0; 1 has in-degree 0 and is unique.

#### Approach

The champion is a node that no one is stronger than. Because the relation is transitive and the graph is a DAG, "someone is stronger than node v" is equivalent to "v has an incoming edge" (any dominating path ends at v with an incoming edge). So compute the in-degree of every node from the edge list; the candidate champions are exactly the nodes with in-degree 0 (the graph's sources). If there is exactly one source, it is the unique supreme champion; if there are two or more sources, none dominates the others and the answer is -1. This is the source-counting / in-degree idea underlying topological ordering.

Complexity

Time: O(k + m) — one pass over the m edges to build in-degrees plus one pass over the k nodes. Space: O(k) — the in-degree array. (Justification: each edge and each node is touched a constant number of times.)

Python
PYTHON
from typing import List

def find_supreme_champion(k: int, links: List[List[int]]) -> int:
    indeg = [0] * k
    for _, y in links:            # x -> y means y has someone stronger
        indeg[y] += 1
    champ, zero_count = -1, 0
    for i in range(k):
        if indeg[i] == 0:
            champ, zero_count = i, zero_count + 1
    return champ if zero_count == 1 else -1
JavaScript
JAVASCRIPT
function findSupremeChampion(k, links) {

#### cons
t indeg = new Array(k).fill(0);
    for (const [, y] of links) {
        indeg[y]++;                  // y has an incoming edge -> not a champion
    }
    let champ = -1, zeroCount = 0;
    for (let i = 0; i < k; i++) {
        if (indeg[i] === 0) {
            champ = i;
            zeroCount++;
        }
    }
    return zeroCount === 1 ? champ : -1;
}
Java
JAVA
class Solution {
    public int findSupremeChampion(int k, int[][] links) {
        int[] indeg = new int[k];
        for (int[] e : links) {
            indeg[e[1]]++;               // e[1] has an incoming edge -> not a champion
        }
        int champ = -1, zeroCount = 0;
        for (int i = 0; i < k; i++) {
            if (indeg[i] == 0) {
                champ = i;
                zeroCount++;
            }
        }
        return zeroCount == 1 ? champ : -1;
    }
}
C++
CPP
#include <vector>
using namespace std;

class Solution {
public:
    int findSupremeChampion(int k, vector<vector<int>>& links) {
        vector<int> indeg(k, 0);
        for (auto& e : links) {
            indeg[e[1]]++;               // e[1] has someone stronger
        }
        int champ = -1, zeroCount = 0;
        for (int i = 0; i < k; i++) {
            if (indeg[i] == 0) {
                champ = i;
                zeroCount++;
            }
        }
        return zeroCount == 1 ? champ : -1;
    }
};
C
C
#include <stdlib.h>

// links is an array of linksSize edges, each edge {x, y} meaning x is stronger than y.
int findSupremeChampion(int k, int links[][2], int linksSize) {
    int* indeg = (int*)calloc(k, sizeof(int));
    for (int i = 0; i < linksSize; i++) {
        indeg[links[i][1]]++;        // y gets an incoming edge -> someone is stronger than y
    }
    int champ = -1, zeroCount = 0;
    for (int i = 0; i < k; i++) {
        if (indeg[i] == 0) {
            champ = i;
            zeroCount++;
        }
    }
    free(indeg);
    return zeroCount == 1 ? champ : -1;
}

#### Follow-up
questions
What if the graph is not guaranteed acyclic — how would you first detect a cycle and report that no valid champion exists?
The graph may not be the full transitive closure (only some dominance edges are given). Argue why the in-degree-0 test is still correct.
Instead of requiring uniqueness, return the list of all nodes that could be champions.
Extend the solution to output a full ranking (a topological order) of all contenders.

