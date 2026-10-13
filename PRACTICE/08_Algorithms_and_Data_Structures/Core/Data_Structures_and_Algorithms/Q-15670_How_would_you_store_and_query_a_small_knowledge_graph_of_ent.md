---
type: "practice"
difficulty: "Deep"
---

# How would you store and query a small knowledge graph of entities and relations extracted by an LLM, and why is an adjacency list a reasonable start?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Spatial Indexes](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Spatial_Indexes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To store and query a small knowledge graph of entities and relations extracted by an LLM, we need to consider both efficiency and flexibility in representing relationships between entities. An adjacency list can be a reasonable starting point due to its simplicity and ease of traversal. Here's how you would approach this problem:

Firstly, let's define what our knowledge graph is: it consists of entities (nodes) and their relations (edges). Each edge represents a connection or interaction between two nodes.

The core idea behind using an adjacency list is that each node points to its neighboring nodes, making it easy to traverse the graph. This structure is particularly suitable for small graphs where we want fast querying capabilities without excessive memory overhead.

Let's delve into the specifics of implementing this solution step by step:

Define Nodes and Edges: Each entity in our knowledge graph will be represented as a node with properties like id, type, and name. Relations between nodes can be stored as edges, each with attributes such as relation_type and weight.

Adjacency List Representation:

Each node stores a list of its neighbors along with the relation type.
This allows for efficient traversal (e.g., BFS or DFS) to explore connected components.

Querying the Graph: Using an adjacency list, querying can be optimized by traversing from one node to its known neighbors and following the edges as needed.

#### Example
Data:

PYTHON
nodes = {
    1: {"type": "Person", "name": "Alice"},
    2: {"type": "Organization", "name": "Acme Inc."},
    3: {"type": "Location", "name": "New York"}
}

edges = {
    1: [(2, "works_for"), (3, "lives_in")],
    2: [],
    3: []
}

Graph Traversal Example:

PYTHON
def get_neighbors(node_id):
    return nodes[node_id]

def bfs(start_node_id):
    visited = set()
    queue = [start_node_id]

    while queue:
        current_node = queue.pop(0)
        if current_node not in visited:
            print(nodes[current_node])
            visited.add(current_node)
            for neighbor, relation in edges[current_node]:
                queue.append(neighbor)

bfs(1)

This approach ensures that we can efficiently store and query the knowledge graph while keeping memory usage low. However, there are some common pitfalls to avoid:

#### Performance Considerations
While adjacency lists are efficient for many operations, they may not be as good as matrix representations for certain types of queries (e.g., finding shortest paths).
Edge Cases: Ensure that edge cases like self-loops and multiple edges between the same nodes are handled appropriately.
Scalability: For larger graphs, consider using more advanced data structures or optimizing algorithms.

In summary, an adjacency list is a reasonable starting point for storing and querying a small knowledge graph due to its simplicity and efficiency. It allows for straightforward traversal and can be extended with optimizations as needed.

**Code:**

```python
nodes = {
    1: {"type": "Person", "name": "Alice"},
    2: {"type": "Organization", "name": "Acme Inc."},
    3: {"type": "Location", "name": "New York"}
}

edges = {
    1: [(2, "works_for"), (3, "lives_in")],
    2: [],
    3: []
}
```

**Example input / output:**

```text
def get_neighbors(node_id):
    return nodes[node_id]

def bfs(start_node_id):
    visited = set()
    queue = [start_node_id]

    while queue:
        current_node = queue.pop(0)
        if current_node not in visited:
            print(nodes[current_node])
            visited.add(current_node)
            for neighbor, relation in edges[current_node]:
                queue.append(neighbor)

bfs(1)
```

