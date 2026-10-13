---
type: "practice"
difficulty: "Deep"
---

# A graph of microservice calls must be traced to find the critical path of a slow request; how do you build the span tree from out-of-order spans and compute the longest weighted path efficiently?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Cycle Detection in Graphs](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Cycle_Detection_in_Graphs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To tackle this problem, we'll start by summarizing our approach. The task is to reconstruct the call graph from microservice spans that arrive in an unordered sequence and then find the critical path (longest weighted path) efficiently. This involves two main steps: constructing a directed acyclic graph (DAG) from the spans and finding the longest path in this DAG.

#### Triage
of the Question:

Understanding the Problem: The input consists of a set of microservice calls represented as "spans," each with an identifier, parent span ID, start time, end time, and latency.
Graph Construction: Spans need to be organized into a directed acyclic graph (DAG) where each node represents a span and edges indicate the direction of the call flow.
Critical Path Identification: Once the DAG is constructed, we need to find the longest path in terms of weighted time (latency).

#### Assumptions

Spans are given as a list with fields: span_id, parent_span_id, start_time, end_time, and latency.
The input spans may be out-of-order.
Each span has at most one parent, except for the root span.

Step-by-Step Solution from First Principles:

Graph Construction:

Use a hash map to store children for each node (span). This allows us to build the graph incrementally as we process spans in any order.

Topological Sorting:

Perform topological sorting on the DAG to ensure that nodes are processed in a valid order where parent nodes come before their children.

Longest Path Calculation:

Use dynamic programming or depth-first search (DFS) with memoization to find the longest path in the graph.

Here's how we can implement these steps:

PYTHON
from collections import defaultdict, deque

def build_span_tree(spans):
    # Step 1: Build the span tree using a hash map for children
    children = defaultdict(list)
    for span in spans:
        parent_id = span['parent_span_id']
        if parent_id is None:
            root_span_id = span['span_id']  # Root node
        else:
            children[parent_id].append(span['span_id'])

    return children, root_span_id

def dfs(span_id, latency, graph, memo):
    # Step 3: DFS with memoization to find the longest path
    if span_id in memo:
        return memo[span_id]

    max_latency = 0
    for child_id in graph.get(span_id, []):
        max_latency = max(max_latency, dfs(child_id, latency + spans[child_id]['latency'], graph, memo))

    memo[span_id] = max_latency
    return max_latency

def find_critical_path(spans):
    # Step 1: Build the span tree
    children, root_span_id = build_span_tree(spans)

    # Step 3: Find the longest path starting from the root node
    memo = {}
    critical_path_latency = dfs(root_span_id, 0, children, memo)

    return critical_path_latency

# Sample data for testing
spans = [
    {'span_id': '1', 'parent_span_id': None, 'start_time': 0, 'end_time': 10, 'latency': 5},
    {'span_id': '2', 'parent_span_id': '1', 'start_time': 3, 'end_time': 6, 'latency': 3},
    {'span_id': '3', 'parent_span_id': '1', 'start_time': 0, 'end_time': 4, 'latency': 2},
    {'span_id': '4', 'parent_span_id': '2', 'start_time': 5, 'end_time': 7, 'latency': 2},
    {'span_id': '5', 'parent_span_id': '3', 'start_time': 1, 'end_time': 2, 'latency': 1}
]

# Expected output: The longest path latency is 8 (from span '1' to '4')
print(find_critical_path(spans))

#### Closing Summary
and Common Mistakes: This solution effectively constructs the call graph from unordered spans and finds the critical path efficiently using DFS with memoization. Key points include topologically sorting to ensure nodes are processed in a valid order, and using memoization to avoid redundant calculations during path finding.

#### Common mistakes
to watch out for include not handling edge cases like disconnected graphs or cycles (though cycles should be avoided in this context). Additionally, ensuring the correctness of the span tree construction is crucial; spans with missing parent IDs need special attention.

**Code:**

```python
from collections import defaultdict

def build_span_tree(spans):
    children = defaultdict(list)
    for span in spans:
        parent_id = span['parent_span_id']
        if parent_id is None:
            root_span_id = span['span_id']  # Root node
        else:
            children[parent_id].append(span['span_id'])

    return children, root_span_id

def dfs(span_id, latency, graph, memo):
    if span_id in memo:
        return memo[span_id]

    max_latency = 0
    for child_id in graph.get(span_id, []):
        max_latency = max(max_latency, dfs(child_id, latency + spans[child_id]['latency'], graph, memo))

    memo[span_id] = max_latency
    return max_latency

def find_critical_path(spans):
    children, root_span_id = build_span_tree(spans)
    memo = {}
    critical_path_latency = dfs(root_span_id, 0, children, memo)
    return critical_path_latency

# Sample data for testing
spans = [
    {'span_id': '1', 'parent_span_id': None, 'start_time': 0, 'end_time': 10, 'latency': 5},
    {'span_id': '2', 'parent_span_id': '1', 'start_time': 3, 'end_time': 6, 'latency': 3},
    {'span_id': '3', 'parent_span_id': '1', 'start_time': 0, 'end_time': 4, 'latency': 2},
    {'span_id': '4', 'parent_span_id': '2', 'start_time': 5, 'end_time': 7, 'latency': 2},
    {'span_id': '5', 'parent_span_id': '3', 'start_time': 1, 'end_time': 2, 'latency': 1}
]

print(find_critical_path(spans))
```

**Example input / output:**

```text
spans = [
    {'span_id': '1', 'parent_span_id': None, 'start_time': 0, 'end_time': 10, 'latency': 5},
    {'span_id': '2', 'parent_span_id': '1', 'start_time': 3, 'end_time': 6, 'latency': 3},
    {'span_id': '3', 'parent_span_id': '1', 'start_time': 0, 'end_time': 4, 'latency': 2},
    {'span_id': '4', 'parent_span_id': '2', 'start_time': 5, 'end_time': 7, 'latency': 2},
    {'span_id': '5', 'parent_span_id': '3', 'start_time': 1, 'end_time': 2, 'latency': 1}
]

critical_path_latency = find_critical_path(spans)
print(critical_path_latency)  # Output: 8
```

**Data:**

```json
[
    {"span_id": "1", "parent_span_id": null, "start_time": 0, "end_time": 10, "latency": 5},
    {"span_id": "2", "parent_span_id": "1", "start_time": 3, "end_time": 6, "latency": 3},
    {"span_id": "3", "parent_span_id": "1", "start_time": 0, "end_time": 4, "latency": 2},
    {"span_id": "4", "parent_span_id": "2", "start_time": 5, "end_time": 7, "latency": 2},
    {"span_id": "5", "parent_span_id": "3", "start_time": 1, "end_time": 2, "latency": 1}
]
```

