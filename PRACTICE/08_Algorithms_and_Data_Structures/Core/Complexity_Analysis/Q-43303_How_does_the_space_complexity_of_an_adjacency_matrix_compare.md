---
type: "practice"
difficulty: "Intermediate"
---

# How does the space complexity of an adjacency matrix compare to an adjacency list for a sparse graph?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Set Operation Complexity](<../../../../08_Algorithms_and_Data_Structures/Core/Complexity_Analysis/Set_Operation_Complexity.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To understand how the space complexity of an adjacency matrix compares to an adjacency list for a sparse graph, we need to first define and analyze both data structures.

TL;DR
Adjacency Matrix: Uses (O(V^2)) space where (V) is the number of vertices. Suitable for dense graphs.
Adjacency List: Uses (O(V + E)) space where (E) is the number of edges, making it more efficient for sparse graphs.

#### Triage

The question asks us to compare the space complexities of two common graph representations: an adjacency matrix and an adjacency list. The context is a sparse graph, meaning that there are significantly fewer edges than vertices. We need to determine which representation uses less memory in this scenario.

#### First Principles Solution
Adjacency Matrix
An adjacency matrix (A) for a directed or undirected graph with (V) vertices is a (V \times V) 2D array.
Each entry (A[i][j]) holds either:
A value indicating the presence of an edge (1 for directed, or 0/1 for undirected).
Optionally, if there are weights on edges, it can hold that weight.

For a sparse graph, where the number of edges (E) is much smaller than (V^2), most entries in the matrix will be zero. However, the space complexity remains (O(V^2)) because we still allocate all possible entries for vertices.

Adjacency List
An adjacency list uses an array (or vector) of linked lists or arrays.
Each index in this array corresponds to a vertex and points to a list of its neighbors.
For each edge, the space required is proportional to (1) per edge rather than (V^2).

For a sparse graph, most vertices have very few edges (i.e., low degree). Therefore, the total memory usage will be dominated by the sum of the lengths of all neighbor lists.

#### Closing Summary
and Common Mistakes
Adjacency Matrix: Always uses (O(V^2)) space regardless of how sparse the graph is.
Adjacency List: Uses approximately (O(V + E)) space, making it highly efficient for sparse graphs. However, it requires a bit more overhead in terms of pointers and indirection.

#### Common Mistakes
Overlooking Sparse Characteristics: Failing to recognize that "sparse" implies many zero entries in the matrix.
Misunderstanding List Structure: Assuming adjacency list always uses (O(V^2)) space, when it typically uses much less for sparse graphs.

**Comparison:**

```
Vertex	Adjacency Matrix	Adjacency List
0	1	[3]
1	0	[]
2	1	[4]
3	0	[2,7]
4	0	[]
5	0	[6,8]
6	0	[5,9]
7	1	[3]
8	0	[]
9	0	[4]
```

**Code:**

```python
# Example of Adjacency Matrix and List for a simple graph
def adjacency_matrix(num_vertices, edges):
    matrix = [[0] * num_vertices for _ in range(num_vertices)]
    for u, v in edges:
        matrix[u][v] = 1  # or assign weight if needed
    return matrix

def adjacency_list(num_vertices, edges):
    adj_list = [[] for _ in range(num_vertices)]
    for u, v in edges:
        adj_list[u].append(v)  # append v to the list of neighbors of u
    return adj_list
```

**Example input / output:**

```text
Input:
Vertices: 10
Edges: [[0, 3], [2, 4], [6, 7]]

Adjacency Matrix (sparsity):
0 0 0 1 0 0 0 0 0 0
0 0 0 0 0 0 0 0 0 0
0 0 0 0 1 0 0 0 0 0
0 0 0 0 0 0 0 0 0 0
0 0 0 0 0 0 0 0 0 0
0 0 0 0 0 0 0 0 0 0
1 0 0 0 0 0 0 1 0 0
0 0 0 0 0 0 0 0 1 0
0 0 0 0 0 0 0 0 0 1
0 0 0 0 0 0 0 0 0 0

Adjacency List (space-efficient):
[ [3], [], [], [4], [], [2,7], [6], [4] ]
```

**Data:**

```json
// Example sparse graph data
{
  "vertices": 10,
  "edges": [
    [0, 3],
    [2, 4],
    [6, 7]
  ]
}
```

**Flow:**

1. Adjacency Matrix
2. O(V^2) space
3. Adjacency List
4. O(V + E) space

