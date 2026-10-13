---
type: "practice"
difficulty: "Deep"
---

# Your app state tree is deeply nested and every update clones the whole thing; how would you use structural sharing or a normalized flat map to make updates and selectors efficient?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Cycle Detection in Graphs](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Cycle_Detection_in_Graphs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of inefficiency in updating and selecting from a deeply nested app state tree, we can leverage structural sharing and normalization techniques. This approach ensures that only necessary parts of the state are updated, significantly improving performance.

#### Triaging the Question

The core challenge here is to optimize memory usage and reduce unnecessary recomputations when updating or querying a complex state structure in an application. The key questions revolve around:

Understanding the Current Issue: Why cloning the entire tree is inefficient?
Exploring Structural Sharing: How can we share structures between different versions of the state without duplicating memory usage?
Implementing Normalization Techniques: Can we flatten and normalize the state to make updates more efficient?

#### Assumptions
The current state tree structure is deeply nested, leading to significant overhead when updating or querying.
We aim for a solution that minimizes both time complexity (when performing operations) and space complexity (when storing states).

#### First Principles Solution
1. Understanding the Problem

When every update to a deep state tree involves cloning the entire structure, it leads to exponential memory usage and runtime overhead. This is because each update creates a new instance of the entire tree, which can be prohibitively expensive.

2. Structural Sharing (Shallow Copy)

Instead of creating a full copy, we can use structural sharing by only copying references that have changed. For example, if an inner node remains unchanged, its reference is reused in the new state version. This approach significantly reduces memory usage but requires careful tracking and management.

3. Normalized Flat Map (Deep Copy with Sharing)

A more advanced technique involves normalizing the state to a flat map where each key maps to values or other keys that refer back to the original structure. This allows us to create new versions of the tree without full duplication, optimizing both memory and computational resources.

#### Implementation
Steps
Identify Shared Nodes: Traverse the current state and identify nodes that are shared across multiple states.
Track Changes: For each update, only mark the nodes that have changed and their dependencies.
Rebuild State: When creating a new state version, reuse existing nodes where possible and create new ones only when necessary.

#### Example
Code
PYTHON
class Node:
    def __init__(self, value, children=None):
        self.value = value
        self.children = children or []

def update_state(current_tree, changes):
    updated_tree = current_tree  # Start with a shallow copy

    for change in changes:
        node_path, new_value = change.split('=')

        path_parts = node_path.strip().split('.')
        curr_node = updated_tree

        # Traverse to the target node
        for part in path_parts[:-1]:
            if not hasattr(curr_node, 'children'):
                raise KeyError(f"Node {part} does not exist")
            child = next((child for child in curr_node.children if child.key == part), None)
            if not child:
                raise KeyError(f"Child node {part} of {curr_node.value} does not exist")
            curr_node = child

        target_key = path_parts[-1]
        if target_key == 'value':
            if isinstance(curr_node, Node):
                # For complex nodes, share the rest of the structure
                updated_tree.children[curr_node.key] = update_state(curr_node.children[0], changes)
            else:
                curr_node.value = new_value
        else:
            raise KeyError(f"Unknown key {target_key}")

    return updated_tree

# Example usage
root = Node('root', [Node('child1'), Node('child2')])
changes = ['child1.value=updated']
new_root = update_state(root, changes)

#### Closing Summary
and Common Mistakes

#### Common Mistakes
Forgetting to track shared nodes can lead to excessive memory usage. Incorrectly assuming that all updates require a full clone.

#### Follow-Ups
How would you handle concurrent updates? What about performance optimizations for selectors (read operations)?

**Comparison:**

```
Node Key	Value	Children
root		child1
		child2
child1	updated
child2	original
```

**Code:**

```

class Node: def init(self, value, children=None): self.value = value self.children = children or []

CODE
Copy

```python
def update_state(current_tree, changes):
    updated_tree = current_tree  # Start with a shallow copy

    for change in changes:
        node_path, new_value = change.split('=')

        path_parts = node_path.strip().split('.')
        curr_node = updated_tree

        # Traverse to the target node
        for part in path_parts[:-1]:
            if not hasattr(curr_node, 'children'):
                raise KeyError(f"Node {part} does not exist")
            child = next((child for child in curr_node.children if child.key == part), None)
            if not child:
                raise KeyError(f"Child node {part} of {curr_node.value} does not exist")
            curr_node = child

        target_key = path_parts[-1]
        if target_key == 'value':
            if isinstance(curr_node, Node):
                # For complex nodes, share the rest of the structure
                updated_tree.children[curr_node.key] = update_state(curr_node.children[0], changes)
            else:
                curr_node.value = new_value
        else:
            raise KeyError(f"Unknown key {target_key}")

    return updated_tree

# Example usage
root = Node('root', [Node('child1'), Node('child2')])
changes = ['child1.value=updated']
new_root = update_state(root, changes)
```

**Example input / output:**

```text
# Input: Deeply nested state and changes
root = Node('root', [Node('child1'), Node('child2')])
changes = ['child1.value=updated']

# Output: Updated state with shared nodes
new_root = update_state(root, changes)
```

**Data:**

```json
{
  "root": {
    "value": "root",
    "children": [
      { "key": "child1", "value": "original" },
      { "key": "child2", "value": "original" }
    ]
  }
}
```

**Flow:**

1. State Tree
2. Update Node Path
3. Track Changes
4. Rebuild Updated State
5. Return New State

