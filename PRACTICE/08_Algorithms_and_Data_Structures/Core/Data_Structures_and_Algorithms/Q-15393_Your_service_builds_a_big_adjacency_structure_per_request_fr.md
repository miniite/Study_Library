---
type: "practice"
difficulty: "Deep"
---

# Your service builds a big adjacency structure per request from normalized rows and it's slow; how do you decide between precomputing a materialized graph and building on the fly given the read/write ratio?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Set Intersection Structures](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Set_Intersection_Structures.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the challenge of building an adjacency matrix efficiently, we need to weigh the trade-offs between precomputing the graph and dynamically constructing it as needed. The key factors are the read/write ratio, memory footprint, and performance requirements. By understanding these dimensions, we can make informed decisions that optimize both performance and resource utilization.

Firstly, let's define what we're dealing with:

Adjacency Matrix: A matrix where each row represents a node in the graph, and each column represents an edge to another node.
Read/Write Ratio: The number of read operations compared to write operations. High read/write ratios suggest frequent updates, while low ratios imply mostly static data.

#### Triage

The core of the question is about balancing between precomputation and dynamic construction based on the access pattern. We need to analyze whether the overhead of precomputing the graph justifies its benefits or if it's better to build the structure on-the-fly when needed.

#### Assumptions
The adjacency matrix is a significant part of the data structure.
The service performs read operations more frequently than write operations, but this can vary based on specific use cases.

#### First-Principles Solution

To decide between precomputing and building dynamically, we need to consider several factors:

Frequency of Updates:

If updates (writes) are infrequent compared to reads, it might be more efficient to precompute the graph upfront.

Size of the Graph:

For small graphs with limited nodes, the overhead of precomputation might not be significant.
For large graphs, the memory footprint and time required for precomputation need careful consideration.

Latency Sensitivity:

If latency is critical, dynamic construction can reduce wait times because it avoids the initial processing delay associated with precomputation.

Resource Availability:

In environments with limited computational resources, precomputation might not be feasible due to high memory and CPU costs.

#### Cons
istency Requirements:

If consistency is a concern (e.g., ensuring data reflects recent changes), dynamic construction ensures the most up-to-date state.
Step-by-Step Analysis

Initial Assessment:

Calculate the size of the adjacency matrix based on the number of nodes and edges.
Estimate the time required for precomputation versus the potential performance benefits during reads.

Read/Write Ratio Analysis:

If the read/write ratio is significantly skewed towards reads, precomputing can be justified to reduce dynamic construction overhead.
Conversely, if writes are frequent, the graph might need to be updated more often, making dynamic construction preferable.

Cost-Benefit Analysis:

Measure the time and memory savings from precomputation versus the cost of maintaining a dynamic structure.

#### Cons
ider using data structures like an LRU cache for dynamic constructs to balance between immediate access and frequent updates.

Load Testing:

Implement both strategies in a controlled environment to observe performance metrics.
Use tools like profiling and tracing to identify bottlenecks and optimize accordingly.

#### Closing Summary

Deciding between precomputing and building an adjacency matrix dynamically depends on the specific use case, particularly the read/write ratio. For scenarios with infrequent writes and high read frequency, precomputation can provide significant performance benefits. However, for environments with frequent updates or limited resources, dynamic construction is more suitable. By carefully analyzing these factors through a cost-benefit lens and using empirical data from load testing, we can make an informed decision that optimizes both performance and resource utilization.

#### Common Mistakes/Follow-Ups
Ignoring Latency Sensitivity: Ensuring low-latency access is crucial for many applications; dynamic construction might be necessary in such cases.
Overlooking Memory Constraints: In memory-constrained environments, precomputing a large graph can lead to excessive resource usage and performance degradation.
Not Considering Future Growth: If the number of nodes or edges is expected to grow significantly, precomputation may become less viable.

**Comparison:**

```
Read/Write Ratio	Precomputation Advantages	Dynamic Construction Advantages
High Writes	High overhead, potential consistency issues	Lower overhead, dynamic updates
Low Writes	Reduced latency, optimized performance	Increased latency during initial construction
```

**Code:**

```python
# Example code for dynamic construction with an LRU cache

from collections import OrderedDict

class GraphBuilder:
    def __init__(self):
        self.graph = OrderedDict()

    def add_edge(self, node1, node2):
        if node1 not in self.graph:
            self.graph[node1] = []
        if node2 not in self.graph:
            self.graph[node2] = []
        self.graph[node1].append(node2)
        self.graph[node2].append(node1)

    def get_neighbors(self, node):
        return self.graph.get(node, [])
```

**Example input / output:**

```text
Input:
- Add edge from A to B
- Get neighbors for node A

Output:
- Neighbors of A: ['B']
```

**Data:**

```json
{
  "nodes": ["A", "B", "C", "D"],
  "edges": [
    {"from": "A", "to": "B"},
    {"from": "B", "to": "C"},
    {"from": "C", "to": "D"},
    {"from": "D", "to": "A"}
  ]
}
```

**Flow:**

1. High Reads
2. Low Reads
3. Initial Assessment
4. Read/Write Ratio?
5. Precompute Graph
6. Dynamically Constructon-the-fly
7. Optimize Memory Usage
8. Maintain Consistency

