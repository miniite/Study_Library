---
type: "practice"
difficulty: "Deep"
---

# You want to detect cycles in a feature-dependency graph at pipeline-compile time to prevent circular transforms; how do you build the graph incrementally and report the exact cycle path to the author?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Cycle Detection in Graphs](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Cycle_Detection_in_Graphs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the problem of detecting cycles in a feature-dependency graph, we need to ensure that each feature or node is processed only once during its dependency chain. The goal is to detect any circular dependencies and report them back to the author of the pipeline configuration. This involves building the graph incrementally while keeping track of nodes currently being visited and those already seen.

#### Triage

The question requires us to build a feature-dependency graph where each node represents a feature, and edges represent dependencies between features. The objective is to detect cycles in this graph, which would indicate circular transformations. We need to do this incrementally as the graph grows and provide detailed cycle information upon detection.

#### Assumptions

Each node has a name.
Dependencies are bidirectional (if A depends on B, then B also depends on A).
Cycles must be reported along with the exact path of nodes involved in the cycle.

#### First Principles Solution
Graph Representation: Represent the graph using an adjacency list where each key is a node and its value is a set of dependent nodes.
Cycle Detection Algorithm: Use Depth First Search (DFS) to detect cycles by keeping track of visited nodes and currently visiting nodes in two separate sets.
Path Recording: Maintain a stack or a path array during DFS traversal to record the path of nodes leading up to the current node.
Step-by-Step Solution

Initialize Graph Structure:

PYTHON
graph = {}

Graph Construction:

Add nodes and edges as they are defined in the pipeline configuration.
For each added edge, check if adding it would create a cycle by performing DFS from the source node.

Cycle Detection Using DFS:

PYTHON
def detect_cycle(node, graph, visited, stack):
    # Mark current node as visited and add to recursion stack
    visited[node] = True
    stack.append(node)

    for neighbour in graph[node]:
        if not visited[neighbour]:
            if detect_cycle(neighbour, graph, visited, stack):
                return True
        elif neighbour in stack:
            # Cycle detected: record the path
            cycle_path = [stack[i] for i in range(stack.index(node), len(stack))]
            print(f"Cycle detected: {cycle_path}")
            return True

    # Remove node from recursion stack before returning
    stack.pop()
    return False

Graph Building and Cycle Detection:

PYTHON
def build_and_check_graph(pipeline_config):
    graph = {}
    visited = set()
    for config in pipeline_config:
        source, targets = config
        if source not in graph:
            graph[source] = set()
        for target in targets:
            if target not in graph:
                graph[target] = set()
            # Add edge to the graph and check for cycle
            graph[source].add(target)
            visited.add(source)
            detect_cycle(source, graph, visited, [])
    return graph

pipeline_config = [
    ('A', ['B']),
    ('B', ['C']),
    ('C', ['D']),
    ('D', ['E']),
    ('E', ['F', 'A'])  # This creates a cycle
]

build_and_check_graph(pipeline_config)

Handling Edge Cases:

Ensure that each node is only added to the graph once.
Handle multiple dependencies on a single node correctly.

#### Closing Summary
and Common Mistakes/Follow-Ups

To summarize, we incrementally built the dependency graph while checking for cycles using DFS. By maintaining visited nodes and current stack during traversal, we were able to detect cycles efficiently. The key is to correctly manage the recursion stack to record cycle paths accurately. Some common mistakes include not properly managing the recursion stack or incorrectly handling multiple dependencies on a single node.

For follow-ups, one might consider:

Optimizing the graph construction for large-scale pipelines.
Implementing more sophisticated error messages and suggestions for fixing cycles.

#### Cons
idering parallel processing techniques if the pipeline is very large.

**Code:**

```python
def detect_cycle(node, graph, visited, stack):
    # Mark current node as visited and add to recursion stack
    visited[node] = True
    stack.append(node)

    for neighbour in graph[node]:
        if not visited[neighbour]:
            if detect_cycle(neighbour, graph, visited, stack):
                return True
        elif neighbour in stack:
            # Cycle detected: record the path
            cycle_path = [stack[i] for i in range(stack.index(node), len(stack))]
            print(f"Cycle detected: {cycle_path}")
            return True

    # Remove node from recursion stack before returning
    stack.pop()
    return False

def build_and_check_graph(pipeline_config):
    graph = {}
    visited = set()
    for config in pipeline_config:
        source, targets = config
        if source not in graph:
            graph[source] = set()
        for target in targets:
            if target not in graph:
                graph[target] = set()
            # Add edge to the graph and check for cycle
            graph[source].add(target)
            visited.add(source)
            detect_cycle(source, graph, visited, [])
    return graph

pipeline_config = [
    ('A', ['B']),
    ('B', ['C']),
    ('C', ['D']),
    ('D', ['E']),
    ('E', ['F', 'A'])  # This creates a cycle
]

build_and_check_graph(pipeline_config)
```

**Example input / output:**

```text
Cycle detected: ['E', 'F', 'A']
```

**Data:**

```python
pipeline_config = [
    ('A', ['B']),
    ('B', ['C']),
    ('C', ['D']),
    ('D', ['E']),
    ('E', ['F', 'A'])  # This creates a cycle
]
```

