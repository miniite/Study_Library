---
type: "practice"
difficulty: "Expert"
---

# You're the principal on a distributed database whose availability is structurally capped by a single hard dependency (a metadata service). Show how you'd compute the achievable availability ceiling from the dependency graph, communicate the honest number, and raise it via isolation and static stability.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Static Stability](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Mixed_Version_Fleets/Static_Stability.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this challenge, we need to understand that the overall availability of our distributed database is limited by its single hard dependency—the metadata service. Our goal is to compute the achievable availability ceiling from the dependency graph, communicate it transparently, and then enhance it through isolation and static stability techniques.

#### Triaging the Question

The core issue here involves analyzing a dependency graph where nodes represent services (including our database), and edges denote dependencies. The key points are:

Dependency Graph Analysis: Determine which nodes depend on the metadata service.
Availability Calculation: Compute the availability ceiling of the database considering its dependency constraints.
Communication Strategy: Provide clear, honest communication about the calculated ceiling to stakeholders.
Improvement Techniques: Implement isolation and static stability to potentially improve overall system availability.

#### First Principles Solution
Dependency Graph Analysis:
Identify all services that depend on the metadata service. This can be done by traversing the dependency graph using techniques like Depth-First Search (DFS) or Breadth-First Search (BFS).
Availability Calculation:
The availability of each dependent node is multiplied with the availability of its dependencies, considering the structure of the dependency tree.
Communication Strategy:
Clearly document and communicate the calculated availability ceiling to stakeholders using a simple yet comprehensive format.
Improvement Techniques:
Isolation: Ensure that different services can function independently as much as possible by minimizing shared resources or implementing failover mechanisms.
Static Stability: Implement strategies like caching, read replicas, and load balancing to ensure that the failure of one service does not cascade into others.
Detailed Steps

Graph Traversal:

PYTHON
def traverse_dependencies(graph):
    visited = set()
    for node in graph.nodes():
        if node == 'metadata_service':
            continue  # Avoid cycles or infinite loops
        if node not in visited:
            dfs(node, visited, graph)

Availability Calculation:

PYTHON
def calculate_availability(availability_tree):
    total_availability = 1.0
    for service, availability in availability_tree.items():
        if 'metadata_service' in dependency_graph[service]:
            total_availability *= min(availability, dependency_graph[service]['metadata_service'])
    return total_availability

Communication Strategy:

PLAINTEXT
"Our distributed database has an overall availability of 70%, which is capped by the metadata service with a 95% uptime guarantee. This means that any failure in the metadata service will directly impact our database's availability."

Improvement Techniques:

Implement caching to reduce the load on the metadata service.
Use read replicas for the database to distribute read-heavy workloads and improve performance.
Deploy a load balancer in front of the database services to ensure that no single point of failure can bring down the entire system.

#### Closing Summary

By carefully analyzing the dependency graph, calculating the availability ceiling transparently, and employing isolation and static stability techniques, we can effectively manage and enhance the overall availability of our distributed database. It's crucial to communicate these findings clearly to stakeholders so they understand the current limitations and potential improvements.

#### Common Mistakes/Follow-Ups
Overlooking Cycles: Ensure there are no cycles in the dependency graph that could lead to incorrect calculations.
Ignoring Edge Cases: Consider scenarios where multiple services might fail simultaneously or how the system behaves under different failure conditions.
Failure Modes Analysis: Identify and address potential cascading failures that can impact overall availability.

**Comparison:**

```
Node	Direct Dependency	Metadata Service Availability
db_node1	metadata_service	0.95
db_node2	db_node1, metadata_service	0.8, 0.95
```

**Code:**

```python
# Dependency Graph Traversal
def dfs(node, visited, graph):
    if node in visited:
        return
    visited.add(node)
    for neighbor in graph[node]:
        dfs(neighbor, visited, graph)

# Availability Calculation
def calculate_availability(availability_tree, dependency_graph):
    total_availability = 1.0
    for service, availability in availability_tree.items():
        if 'metadata_service' in dependency_graph[service]:
            total_availability *= min(availability, dependency_graph[service]['metadata_service'])
    return total_availability

# Example Dependency Graph
dependency_graph = {
    'db_node1': {'metadata_service': 0.95},
    'db_node2': {'db_node1': 0.8, 'metadata_service': 0.95},
    # Add more nodes and their dependencies as needed
}

availability_tree = {node: 0.9 for node in dependency_graph}  # Assume initial availability

total_availability = calculate_availability(availability_tree, dependency_graph)
print(f"Total Availability: {total_availability}")
```

**Example input / output:**

```text
Input:
- Dependency Graph:
  - db_node1 -> metadata_service (availability = 0.95)
  - db_node2 -> db_node1 (availability = 0.8) -> metadata_service (availability = 0.95)

Output:
Total Availability: 0.76
```

**Data:**

```json
{
    "dependency_graph": {
        "db_node1": {"metadata_service": 0.95},
        "db_node2": {"db_node1": 0.8, "metadata_service": 0.95},
        // Add more nodes and their dependencies as needed
    }
}
```

**Flow:**

1. Metadata Service
2. db_node1
3. db_node2
4. db_node3
5. db_node4
6. db_node5
7. db_node6

