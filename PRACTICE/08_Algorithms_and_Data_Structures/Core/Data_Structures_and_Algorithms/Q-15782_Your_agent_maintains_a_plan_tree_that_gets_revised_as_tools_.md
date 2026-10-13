---
type: "practice"
difficulty: "Deep"
---

# Your agent maintains a plan tree that gets revised as tools return; when a subtask fails you must re-plan only the affected branch in O(depth), not rebuild the whole tree; what immutable-tree or path-update approach enables that?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Cycle Detection in Graphs](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Cycle_Detection_in_Graphs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this problem, we need to design an immutable data structure and a set of algorithms that allow us to efficiently update only the affected branch in the plan tree when a subtask fails. The key is to leverage functional programming principles to ensure that updates are localized to minimize the impact on overall performance.

#### Triaging the Question

The question asks for an approach where, upon a failure of a subtask within a complex plan tree, we can re-plan only the affected branch in O(depth) time complexity rather than rebuilding the entire tree. This is critical for optimizing resource usage and ensuring that our agent remains responsive under dynamic conditions.

#### Assumptions
The plan tree is hierarchical with nodes representing tasks or subtasks.
Each node has a status indicating whether it is successful or failed.
When a subtask fails, we need to re-plan only the affected branch from the failure point downwards.

#### First Principles Solution
Immutable Tree Design: Use an immutable data structure where each change results in a new tree instance. This avoids modifying existing structures and allows for efficient updates via path updates.
Path Update Algorithm: Implement an algorithm that can identify the subtree starting from the failed node and update only those nodes.
Step-by-Step Solution

Immutable Node Representation:

Each node should store its status, children, and any necessary metadata.
PYTHON
class TreeNode:
    def __init__(self, task_id):
        self.task_id = task_id
        self.status = None  # Initially set to 'None' or 'Running'
        self.children = []
        self.parent = None

    def mark_failure(self):
        self.status = 'Failed'

    def update_subtree_status(self):
        if self.status == 'Failed':
            for child in self.children:
                child.mark_failure()
                child.update_subtree_status()

Path Identification and Update:

When a node fails, traverse the tree to identify all descendants starting from that node.
PYTHON
def update_failed_branch(node):
    if not node:
        return

    # Mark current node as failed
    node.mark_failure()

    # Recursively mark all children as failed
    for child in node.children:
        update_failed_branch(child)

Replanning Logic:

When a failure occurs, trigger the update_failed_branch function to re-plan only the affected branch.
PYTHON
def handle_failure(task_id):
    root = get_root_node()  # Assume this function fetches the root node of the plan tree
    current_node = find_node_by_task_id(root, task_id)
    if current_node:
        update_failed_branch(current_node)

Optimization Considerations:

Ensure that the tree structure supports efficient traversal and updates.
Use lazy evaluation or memoization techniques to avoid unnecessary recomputation.

#### Closing Summary

By leveraging immutable data structures and path-update algorithms, we can efficiently re-plan only the affected branch in O(depth) time complexity. This approach ensures that our agent remains responsive and resource-efficient even when dealing with complex and dynamic plan trees.

#### Common Mistakes
and Follow-Ups
Mistake: Failing to properly identify the subtrees that need updating.

#### Follow-Up
Discussing scenarios where multiple failures occur simultaneously and how to handle them without redundant work.
Advanced Discussion: Exploring more advanced data structures like Persistent Data Structures (PDS) or using functional programming libraries.

**Code:**

```python
class TreeNode:
    def __init__(self, task_id):
        self.task_id = task_id
        self.status = None  # Initially set to 'None' or 'Running'
        self.children = []
        self.parent = None

    def mark_failure(self):
        self.status = 'Failed'

    def update_subtree_status(self):
        if self.status == 'Failed':
            for child in self.children:
                child.mark_failure()
                child.update_subtree_status()

def update_failed_branch(node):
    if not node:
        return

    # Mark current node as failed
    node.mark_failure()

    # Recursively mark all children as failed
    for child in node.children:
        update_failed_branch(child)

def handle_failure(task_id):
    root = get_root_node()  # Assume this function fetches the root node of the plan tree
    current_node = find_node_by_task_id(root, task_id)
    if current_node:
        update_failed_branch(current_node)
```

**Flow:**

1. Root Node
2. (Task 1)
3. (Task 2)
4. (Subtask 1)
5. (Subtask 2)
6. (Subtask 3)
7. (Subtask 4)
8. C1

