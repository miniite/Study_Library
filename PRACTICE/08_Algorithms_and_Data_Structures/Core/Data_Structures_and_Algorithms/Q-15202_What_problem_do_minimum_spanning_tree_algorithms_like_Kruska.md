---
type: "practice"
difficulty: "Deep"
---

# What problem do minimum spanning tree algorithms like Kruskal and Prim solve, and how do they differ?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Graph Connectivity](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Graph_Connectivity.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Minimum Spanning Tree (MST) algorithms like Kruskal's and Prim's are used to find a subset of edges in an undirected graph that connects all vertices together with the minimum possible total edge weight. In essence, they ensure we have a tree that spans all nodes without cycles, minimizing the overall cost.

#### Triaging the Question

The interviewer is probing your understanding of fundamental concepts in graph theory and algorithm design. They are interested in how you articulate these algorithms' goals and differences from a first-principles perspective. Assumptions include an undirected weighted graph where each edge has a positive weight, and no cycles exist within the MST.

#### Solution
from First Principles
Problem Statement: Given a connected, undirected graph with weighted edges, find a subset of these edges that connects all vertices together while minimizing the total sum of their weights.
Algorithms Overview:
Kruskal's Algorithm: This algorithm starts by sorting all the edges in non-decreasing order of their weight and then iteratively adds the next lightest edge to the MST, ensuring no cycles are formed. It uses a Disjoint Set (Union-Find) data structure for efficiently managing connected components.
Prim's Algorithm: This algorithm builds the MST starting from an arbitrary vertex and grows the tree by adding the nearest vertex not yet in the tree at each step until all vertices are included.
Kruskal’s Algorithm
Step 1: Sort all edges by weight.
Step 2: Initialize a disjoint set (forest) for every node.
Step 3: Iterate through sorted edges; if adding an edge doesn't form a cycle, include it in the MST. Use Union-Find to detect cycles.
Prim’s Algorithm
Step 1: Start with any arbitrary vertex and initialize its key value as 0 (smallest) and all other keys as infinity.
Step 2: Keep adding new nodes to the spanning tree until all vertices are included. Always add a node from the set of not yet included in MST whose key is minimum.

#### Closing Summary

Both Kruskal's and Prim's algorithms aim to find an MST but differ in their approach. Kruskal’s algorithm sorts edges first, while Prim’s starts with a single vertex and grows outward. Kruskal’s use of Union-Find simplifies cycle detection but requires sorted edges, whereas Prim’s uses a priority queue for selecting the next minimum-weight edge.

#### Common Mistakes/Follow-Ups
Common Mistake: Not properly managing cycles or understanding how to detect them using Union-Find in Kruskal's.

#### Follow-Up
Discussing time and space complexities of both algorithms, their trade-offs, and scenarios where one might be preferred over the other.

#### Conclusion

Understanding MST algorithms is crucial for solving network design problems efficiently. Both Kruskal’s and Prim’s have unique strengths that make them suitable in different contexts. Mastery of these concepts highlights your problem-solving skills and algorithmic depth.

**Comparison:**

```
Vertex	Key	MSet
A	∞
B	∞
C	∞
D	∞
E	0	{A}
Edge Added	Key	MSet
(B, D)	9	{A, B, C, D, E}
Vertex	Key	MSet
A	∞
B	6	{A, B}
C	5	{A, B, C}
D	9	{A, B, C, D, E}
Edge Added	Key	MSet
(C, A)	5	{A, B, C}
(D, B)	6	{A, B, C, D}
(E, C)	8	{A, B, C, D, E}
Vertex	Key	MSet
A	∞
B	5	{A, B, C, D, E}
C	8	{A, B, C, D, E}
D	6	{A, B, C, D, E}
E	7	{A, B, C, D, E}
Edge Added	Key	MSet
(E, A)	5	{A, B, C, D, E}
```

**Code:**

```python
# Example implementation of Kruskal's Algorithm using Python

def find(parent, i):
    if parent[i] == -1:
        return i
    return find(parent, parent[i])

def union(parent, x, y):
    rootX = find(parent, x)
    rootY = find(parent, y)
    parent[rootY] = rootX

def kruskal(graph, V, E):
    result = []
    i, e = 0, 0
    graph = sorted(graph, key=lambda item: item[2])

    parent = [-1 for _ in range(V)]

    while e < V - 1:
        u, v, w = graph[i]
        i += 1
        x = find(parent, u)
        y = find(parent, v)

        if x != y:
            e += 1
            result.append([u, v, w])
            union(parent, x, y)

    return result

# Example Graph Input
graph = [
    [0, 1, 6],
    [0, 2, 5],
    [0, 3, 7],
    [1, 4, 9],
    [2, 3, 8]
]

V = 5
E = len(graph)
result = kruskal(graph, V, E)

# Expected Output: [[0, 2, 5], [0, 1, 6], [2, 3, 8]]
print(result)
```

**Example input / output:**

```text
Input:
Graph = [[0, 1, 6], [0, 2, 5], [0, 3, 7], [1, 4, 9], [2, 3, 8]]
Vertices (V) = 5
Edges (E) = 5

Output:
[[0, 2, 5], [0, 1, 6], [2, 3, 8]]
```

**Data:**

```json
{
    "graph": [
        {"u": 0, "v": 1, "w": 6},
        {"u": 0, "v": 2, "w": 5},
        {"u": 0, "v": 3, "w": 7},
        {"u": 1, "v": 4, "w": 9},
        {"u": 2, "v": 3, "w": 8}
    ]
}
```

**Flow:**

1. Start
2. Is there an edge?
3. Add the next lightest edge
4. Union-Find to check for cycles
5. Continue adding edges?
6. End, MST formed
7. Yes, add and union-find
8. Start
9. Is the key of vertex smallerthan current minimum?
10. Set the key to the edge weight
11. Mark the new minimum asadded
12. Continue adding vertices?
13. End, MST formed
14. No, check next vertex

