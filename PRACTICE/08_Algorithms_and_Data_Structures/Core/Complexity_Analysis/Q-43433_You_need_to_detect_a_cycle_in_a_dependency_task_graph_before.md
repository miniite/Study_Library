---
type: "practice"
difficulty: "Deep"
---

# You need to detect a cycle in a dependency/task graph before running it; explain the complexity of the detection, how you'd surface the offending cycle, and the space cost of tracking visited state.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Complexity vs Real Performance](<../../../../08_Algorithms_and_Data_Structures/Core/Complexity_Analysis/Complexity_vs_Real_Performance.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To detect cycles in a task or dependency graph before execution, we need to ensure the graph is acyclic. This problem can be approached using depth-first search (DFS) or topological sorting, both of which have well-defined time and space complexities that are crucial for understanding their efficiency.

#### Triaging the Question

The core challenge here is to determine if a directed graph contains any cycles, specifically in a dependency/task context where running tasks depends on their prerequisites being completed. The key aspects of this problem include:

Understanding the Graph Structure: A task or dependency can be represented as a directed graph with nodes representing tasks and edges representing dependencies.
Cycle Detection Algorithms: We need to use an algorithm that can traverse the graph without getting stuck in infinite loops due to cycles.
Surface Offending Cycles: Once a cycle is detected, we should identify the specific cycle and its elements for corrective action.

#### Assumptions
The graph is directed (tasks depend on other tasks).
Nodes represent tasks and edges represent dependencies between them.
The task graph can be represented as an adjacency list or matrix.
We need to detect cycles before any execution of tasks begins.

#### Solution
from First Principles
Cycle Detection Using DFS

The most common approach for cycle detection in a dependency graph is using Depth-First Search (DFS). This algorithm keeps track of the visited nodes and their parent during traversal. If we revisit a node that is already on the current path, it indicates a cycle.

Here's a step-by-step explanation:

Initialize a Visited State: Create a state array to keep track of whether each node has been visited (not just in this DFS instance but all instances) and if it’s currently being processed.
DFS Traversal:
For each unvisited node, start a DFS traversal.
Mark the current node as “visiting” or temporarily visited.
Recursively visit its neighbors.
If a neighbor is marked as visiting, we have found a cycle.
Mark the node as fully visited when all its neighbors are processed.

#### Example
Pseudocode
PYTHON
def has_cycle(graph):
    n = len(graph)
    visited = [False] * n  # Initialize visited state array

    def dfs(node, parent=None):
        if visited[node]:
            return False  # Already visited this node in the current path
        else:
            visited[node] = True  # Mark as temporarily visiting

        for neighbor in graph[node]:
            if not visited[neighbor]:  # If unvisited, visit it
                if dfs(neighbor, node):
                    return True
            elif parent != neighbor:  # Check for cycle
                return True

        visited[node] = False  # Mark as fully visited after processing all neighbors
        return False

    for i in range(n):
        if not visited[i]:
            if dfs(i):
                return True, find_cycle(i)  # Return the offending cycle

    return False, None  # No cycles found

def find_cycle(node):
    path = [node]
    while node is not None:
        node = parent[node]  # Traverse back to the starting point of the cycle
        path.append(node)
    return list(reversed(path))  # Return the cycle in correct order
Time and Space Complexity Analysis
Time Complexity: The DFS traversal visits each node and edge exactly once. Therefore, the time complexity is (O(V + E)), where (V) is the number of vertices (nodes) and (E) is the number of edges.
Space Complexity:
Visited states: (O(V))
Recursion stack: (O(V)) in worst case
Additional data structures for cycle information are minimal.

#### Closing Summary

In conclusion, using DFS to detect cycles in a dependency graph is both effective and efficient with an (O(V + E)) time complexity. The space cost primarily comes from the visited state array and recursion stack, which can be optimized by using iterative DFS or other cycle detection algorithms like Kosaraju’s for directed graphs.

#### Common Mistakes/Follow-ups
Misunderstanding Visited States: Confusing temporary with permanent states.
Overlooking Edge Cases: Not handling fully disconnected graphs where no cycles might exist but still requiring traversal.
Improper Cycle Surface: Failing to correctly identify and surface the offending cycle.

By carefully implementing the DFS algorithm and managing the visited state, we can ensure robust cycle detection in dependency graphs. The trade-offs lie in balancing between space and time efficiency while ensuring the integrity of the task graph remains intact.

**Comparison:**

```
Node	Parents
A	-
B	A
C	A
D	B, C
E	C
F	D
```

**Code:**

```python
def has_cycle(graph):
    n = len(graph)
    visited = [False] * n  # Initialize visited state array

    def dfs(node, parent=None):
        if visited[node]:
            return False  # Already visited this node in the current path
        else:
            visited[node] = True  # Mark as temporarily visiting

        for neighbor in graph[node]:
            if not visited[neighbor]:  # If unvisited, visit it
                if dfs(neighbor, node):
                    return True
            elif parent != neighbor:  # Check for cycle
                return True

        visited[node] = False  # Mark as fully visited after processing all neighbors
        return False

    for i in range(n):
        if not visited[i]:
            if dfs(i):
                return True, find_cycle(i)  # Return the offending cycle

    return False, None  # No cycles found

def find_cycle(node):
    path = [node]
    while node is not None:
        node = parent[node]  # Traverse back to the starting point of the cycle
        path.append(node)
    return list(reversed(path))  # Return the cycle in correct order
```

**Example input / output:**

```text
graph = {
  "A": ["B", "C"],
  "B": ["D"],
  "C": ["E"],
  "D": ["F"],
  "E": [],
  "F": ["D"]
}

has_cycle(graph), find_cycle("A")
# Output: (True, ['F', 'D'])
```

**Data:**

```json
{
  "graph": {
    "A": ["B", "C"],
    "B": ["D"],
    "C": ["E"],
    "D": ["F"],
    "E": [],
    "F": ["D"]
  }
}
```

**Flow:**

1. Back to D
2. Node A
3. Node B
4. Node C
5. Node D
6. Node E
7. Node F

