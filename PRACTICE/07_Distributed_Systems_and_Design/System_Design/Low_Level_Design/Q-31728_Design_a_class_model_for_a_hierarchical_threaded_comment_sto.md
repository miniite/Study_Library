---
type: "practice"
difficulty: "Deep"
---

# Design a class model for a hierarchical/threaded comment store that must return an entire subtree in one query; compare adjacency list, materialized path, and nested set — which do you pick for heavy writes and why?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Idempotent Consumer Design](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Idempotent_Consumer_Design/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a class model for a hierarchical/threaded comment store that returns an entire subtree in one query, we need to consider three main data storage models: adjacency list (AL), materialized path (MP), and nested set (NS). Each model has its strengths and weaknesses. For heavy write operations, the choice of model is critical as it affects performance and scalability.

#### Triage

The question asks us to compare these models based on their suitability for heavy writes.
We need to consider how each model handles inserts, updates, and deletes efficiently.

#### Assumptions
include a typical forum or social media application with frequent comments and replies.

#### First-principles Solution

Adjacency List (AL):

Structure: Each node contains pointers to its children nodes.

#### Pros
Simple to implement; easy to understand and query for immediate child relationships.

#### Cons
Expensive operations when traversing a subtree due to multiple queries needed. Insertions, updates, and deletions can propagate through the entire tree.

Materialized Path (MP):

Structure: Each node's path is stored in its path attribute, represented as a string of ancestor ids.

#### Pros
Efficient for query operations since the full path to any node is already known; no need for additional queries to traverse a subtree.

#### Cons
Heavy write operations required when nodes are inserted or moved. Path updates can be complex and require synchronization.

Nested Set (NS):

Structure: Each node has left and right attributes that define the range of nodes in its subtree.

#### Pros
Efficient for query operations; no need to traverse the tree structure.

#### Cons
Complicated to maintain, especially with frequent insertions and deletions. Updates can be complex and require recursive logic.

For heavy writes, we should choose the model that balances write efficiency with subtree retrieval speed. Here’s a detailed comparison:

Adjacency List (AL):

#### Pros
Simple structure, easy to implement.

#### Cons
Heavy write operations due to needing to adjust child pointers for every move or delete operation.

Materialized Path (MP):

#### Pros
Efficient reads due to stored paths. Only one query needed to retrieve a subtree.

#### Cons
Write operations are expensive as it requires updating the path of all affected nodes.

Nested Set (NS):

#### Pros
Efficient read operations, no need for recursive traversal.

#### Cons
Complicated write operations that require maintaining the nested set structure correctly. Can be error-prone and time-consuming to implement.

For a scenario with heavy writes, the Materialized Path (MP) model is often the best choice because it minimizes the number of writes needed by storing paths. However, this comes at the cost of more complex write operations.

#### Closing Summary

For a hierarchical comment store where subtree retrieval is frequent and heavyweight, materialized path is generally the best choice.
It balances well between read and write performance, making it suitable for scenarios with high read/write traffic.

#### Common Mistakes
Overlooking the complexity of maintaining paths in MP and not fully understanding the nested set model's intricacies can lead to suboptimal implementations.

**Code:**

```python
class CommentNode:
    def __init__(self, id):
        self.id = id
        self.children = []
        self.path = None  # Materialized Path

# Example of inserting a node in MP model
def insert_node(parent_id, new_id, nodes):
    parent_node = nodes[parent_id]
    path_parts = list(map(str, [parent_id] + [node.id for node in parent_node.children]))
    path_str = '/'.join(path_parts)
    new_node = CommentNode(new_id)
    new_node.path = path_str
    parent_node.children.append(new_node)

# Example of querying a subtree using MP model
def get_subtree(node_id, nodes):
    if not node_id:
        return []
    current_node = nodes[node_id]
    result = [current_node]
    for child in current_node.children:
        result += get_subtree(child.id, nodes)
    return result
```

**Example input / output:**

```text
Input:
- Node ID: 1

Output (Subtree):
[1, 2, 3, 4, 5]
```

**Data:**

```json
{
  "nodes": {
    "1": {"id": 1, "children": ["2", "3"]},
    "2": {"id": 2, "children": []},
    "3": {"id": 3, "children": ["4", "5"]},
    "4": {"id": 4, "children": []},
    "5": {"id": 5, "children": []}
  }
}
```

