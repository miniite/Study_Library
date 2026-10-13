---
type: "practice"
difficulty: "Deep"
---

# A batch that computes shortest paths between all warehouse pairs is too slow with repeated Dijkstra; when would you switch to Floyd-Warshall or Johnson's algorithm, and how does graph density drive the choice?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Shortest Paths](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Shortest_Paths.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of a batch that computes shortest paths between all warehouse pairs being too slow due to repeated Dijkstra’s algorithm applications, we need to understand when switching to Floyd-Warshall or Johnson's algorithm is appropriate and how graph density influences this decision.

Firstly, let's summarize the key points: Dijkstra's algorithm is efficient for single-source shortest paths in graphs with non-negative edge weights, but its performance degrades significantly as the number of sources increases. Floyd-Warshall and Johnson’s algorithms are more general and can handle all pairs shortest paths (APSP) problems, making them suitable when multiple source-destination computations are required.

#### Triaging the Question

The core of this question involves understanding the trade-offs between Dijkstra's, Floyd-Warshall, and Johnson's algorithms. The interviewer is probing your knowledge of these algorithms' time complexities, their applicability to different graph structures (densely vs sparsely connected), and how to optimize for performance in a practical setting.

#### Assumptions
Graph Characteristics: The graph has non-negative edge weights.
Graph Density: The graph is dense or sparse.
Performance Requirements: We are prioritizing time efficiency, but also considering memory usage as an important factor.
Step-by-Step Solution
1. Dijkstra's Algorithm

Dijkstra’s algorithm works efficiently when the number of sources (vertices) is small and the graph is relatively sparse. Its time complexity is (O(E + V \log V)), where (E) is the number of edges, and (V) is the number of vertices. However, if we need to compute shortest paths for all pairs of warehouses, running Dijkstra’s algorithm from each warehouse (which would be (O(V^2 \cdot E + V^3 \log V))) can become prohibitively slow.

2. Floyd-Warshall Algorithm

Floyd-Warshall is a dynamic programming approach with a time complexity of (O(V^3)). It works well for dense graphs (i.e., those with many edges) and small to moderately sized vertex sets because the number of vertices (V) determines the overall complexity. For very dense graphs, this algorithm can be more efficient than repeatedly running Dijkstra’s.

3. Johnson's Algorithm

Johnson’s algorithm combines Dijkstra’s and Bellman-Ford algorithms. It first runs the Bellman-Ford algorithm to find a potential minimum weight for each vertex in (O(VE)) time. Then, it uses this information to transform the graph into one with non-negative weights and applies Dijkstra’s algorithm from each source. The overall complexity is (O(V^2 \log V + VE)). Johnson's algorithm can be advantageous when there are negative edge weights but no negative cycles.

Choosing Between Algorithms

Dijkstra’s vs Floyd-Warshall:

Sparse Graphs: If the graph is sparse, Dijkstra’s might still outperform Floyd-Warshall because its time complexity depends on (E), which can be much smaller than (V^2) in sparse graphs.
Dense Graphs: For dense graphs, Floyd-Warshall's (O(V^3)) complexity makes it a better choice.

Johnson’s Algorithm:

If the graph has negative edge weights but no negative cycles, Johnson’s algorithm can be an effective solution. It handles both sparse and dense graphs efficiently by leveraging Dijkstra’s strengths in non-negative weighted environments.
Graph Density Impact

The key factor is graph density. A sparse graph with many vertices and few edges might benefit from Dijkstra’s repeated runs due to its lower overall complexity when (E) is small compared to (V^2). Conversely, a dense graph will see Floyd-Warshall outperform repeated Dijkstra’s because of its cubic time complexity.

#### Closing Summary

In summary, the choice between Dijkstra's, Floyd-Warshall, or Johnson’s algorithm depends on the specific characteristics of the graph and the problem requirements. For sparse graphs with few sources, Dijkstra’s repeated runs can be faster. For dense graphs or when multiple sources need to compute shortest paths, Floyd-Warshall is more efficient. Johnson’s algorithm offers a versatile solution for handling negative weights in dense graphs.

#### Common Mistakes
Overlooking Graph Density: Assuming all graphs are sparse and using Dijkstra's repeatedly without considering the graph density.
Ignoring Negative Weights: Forgetting that Johnson’s algorithm can handle graphs with negative edge weights, but not cycles.
Not Considering Memory Usage: Focusing only on time complexity while ignoring memory constraints.

#### Conclusion

Understanding these algorithms’ trade-offs is crucial for making informed decisions in real-world applications. Always consider the graph density and specific problem requirements before choosing an algorithm.

**Comparison:**

```
Vertex	Dijkstra's Repeated Runs	Floyd-Warshall	Johnson’s Algorithm
Sparse Graphs	Efficient for small (V)	Less efficient due to cubic complexity	Effective if no negative cycles
Dense Graphs	Less efficient due to high (E)	More efficient with (O(V^3))	Can handle negative weights efficiently
```

**Code:**

```python
# Example of Floyd-Warshall Algorithm implementation
def floyd_warshall(graph):
    V = len(graph)
    dist = [[float('inf')] * V for _ in range(V)]

    # Initialize the distance matrix with graph values and set self-distances to 0
    for i in range(V):
        for j in range(V):
            if i == j:
                dist[i][j] = 0
            else:
                dist[i][j] = graph[i][j]

    # Floyd-Warshall Algorithm
    for k in range(V):
        for i in range(V):
            for j in range(V):
                dist[i][j] = min(dist[i][j], dist[i][k] + dist[k][j])

    return dist

# Example graph (adjacency matrix)
graph = [
    [0, 4, float('inf'), 3],
    [float('inf'), 0, 2, 5],
    [1, float('inf'), 0, float('inf')],
    [float('inf'), 6, 7, 0]
]

floyd_warshall(graph)
```

**Example input / output:**

```text
Input:
Graph = [
    [0, 4, null, 3],
    [null, 0, 2, 5],
    [1, null, 0, null],
    [null, 6, 7, 0]
]

Output:
[
    [0, 3, 3, 3],
    [null, 0, 2, 4],
    [1, null, 0, null],
    [null, 5, 8, 0]
]
```

**Data:**

```json
{
  "graph": [
    [0, 4, null, 3],
    [null, 0, 2, 5],
    [1, null, 0, null],
    [null, 6, 7, 0]
  ]
}
```

**Flow:**

1. Check Graph Density
2. (Sparse Graph)
3. (Dense Graph)
4. (Use Dijkstra's Repeated Runsif V is small)
5. (Use Floyd-Warshall orJohnson's Algorithm)
6. Graph Characteristics
7. (Non-negative Weights)
8. (Small Number of Sources)
9. (Dijkstra’s Repeated Runs)
10. (Many Sources or NegativeWeights)
11. Consider Floyd-Warshall orJohnson's Algorithm
12. (Non-negative Weights)
13. (Non-negative Weights)
14. (Small Number of Sources)
15. (Dijkstra’s Repeated Runs)
16. (Many Sources or NegativeWeights)
17. Consider Floyd-Warshall

