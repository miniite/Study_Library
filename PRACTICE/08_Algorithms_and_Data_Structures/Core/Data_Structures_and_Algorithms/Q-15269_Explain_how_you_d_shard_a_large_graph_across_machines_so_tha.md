---
type: "practice"
difficulty: "Expert"
---

# Explain how you'd shard a large graph across machines so that most traversals stay local; what partitioning heuristic reduces cross-shard edges and how do you handle the ones that remain?

**Difficulty:** Expert · **Levels:** SDE-1/2, SDE-2/3 · [↑ Shortest Paths](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Shortest_Paths.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To shard a large graph across multiple machines while ensuring that most traversals stay local, we need to design an effective partitioning strategy that minimizes cross-shard edges. This involves understanding the nature of our graph and choosing appropriate data structures and algorithms for efficient traversal.

#### Triaging the Question

The core of this question lies in partitioning a graph effectively so that related nodes are colocated on the same machine as much as possible. The key aspects to focus on include:

Understanding Graph Properties: Knowing if the graph is directed, undirected, sparse, or dense.
Choosing Partitioning Heuristics: Selecting heuristics that minimize cross-shard edges.
Handling Remaining Cross-Edges: Ensuring efficient handling of edges that span different shards.

#### First Principles Solution

To begin, let’s assume we are dealing with an undirected and sparse graph. The goal is to partition the nodes into subgraphs such that most traversals (i.e., shortest path queries) remain local within a single shard.

Step 1: Graph Properties

Assume our graph has (N) nodes and (E) edges, where (E \ll N^2). We want to distribute these nodes across (M) machines.

Step 2: Partitioning Heuristics

A common and effective heuristic is Label-Based Partitioning. This involves grouping nodes based on their labels (or attributes) such that nodes with similar labels are colocated more often.

Node Labeling: Assign each node a label, which could be derived from its properties or metadata.
Hash-Based Sharding: Use a hash function to map these labels to machine IDs. For example, ( \text{machine_id} = \text{hash}(label) % M ).
Step 3: Handling Remaining Cross-Edges

Even with effective partitioning, some edges will remain between different shards. To handle these cross-edges efficiently:

Edge Indexing: Maintain an edge index that maps each shard to the set of edges that connect nodes in that shard to nodes in other shards.
Caching: Implement caching mechanisms for frequently accessed cross-shards.

#### Closing Summary
and Common Mistakes

In summary, we use label-based partitioning with hash functions to colocate related nodes on the same machine. This reduces cross-shard traffic significantly but may leave some edges unaddressed. Effective handling of these remaining edges involves maintaining an edge index and leveraging caching mechanisms.

#### Common mistakes
include not considering the graph’s structure adequately, failing to account for high-degree nodes that can skew partitioning, or neglecting cache efficiency for cross-shard access.

Common Follow-Ups
What if the graph is dense?
How do you handle dynamic graphs where nodes and edges change frequently?
What are the trade-offs in terms of memory usage and processing time?

**Comparison:**

```
Shard	Nodes
0	{0, 1}
1	{2, 3}
Shard	Cross-Edges
0	[(0, 1), (1, 0)]
1	[]
```

**Code:**

```python
def shard_graph(nodes, labels, num_shards):
    # Hash-based sharding
    shards = [[] for _ in range(num_shards)]

    for node_id, label in enumerate(labels):
        shard_id = hash(label) % num_shards
        shards[shard_id].append(node_id)

    return shards

def build_edge_index(shards, edges):
    edge_index = {shard: [] for shard in range(len(shards))}

    for u, v in edges:
        if u not in shards[u % len(shards)] and v not in shards[v % len(shards)]:
            # Cross-shard edge
            edge_index[shards[u % len(shards)]].append((u, v))
            edge_index[shards[v % len(shards)]].append((v, u))

    return edge_index

# Example usage
nodes = [0, 1, 2, 3]
labels = ['A', 'B', 'C', 'D']
edges = [(0, 1), (1, 2), (2, 3), (3, 0)]
num_shards = 4

shards = shard_graph(nodes, labels, num_shards)
edge_index = build_edge_index(shards, edges)

print("Shards:", shards)
print("Edge Index:", edge_index)
```

**Example input / output:**

```text
Shards: [[0, 1], [2, 3]]
Edge Index: {
    [0, 1]: [(0, 1), (1, 0)],
    [2, 3]: []
}
```

**Data:**

```json
{
    "nodes": [0, 1, 2, 3],
    "labels": ["A", "B", "C", "D"],
    "edges": [[0, 1], [1, 2], [2, 3], [3, 0]]
}
```

**Flow:**

1. Node Labeling
2. Hash-Based Sharding
3. Edge Indexing
4. Caching Mechanisms

