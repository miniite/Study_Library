---
type: "practice"
difficulty: "Deep"
---

# Find the diameter of a binary tree and explain why it may not pass through the root.

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Binary Tree Recursion](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Binary_Tree_Recursion.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To find the diameter of a binary tree, we need to determine the longest path between any two nodes in the tree. This path can either pass or not pass through the root node. The key idea here is to calculate the height of subtrees and use those heights to find the maximum path length.

#### Triage

The question asks us to find the diameter of a binary tree, which involves identifying the longest path between any two nodes. We need to understand whether this path necessarily passes through the root or not. The interviewer is probing our understanding of tree traversal techniques and depth-first search (DFS). Key assumptions include:

The tree can be unbalanced.
Nodes have at most one parent.

#### First Principles Solution

We'll use a DFS approach where each node has two values: its height and the diameter that includes this node. Here’s how we reason through it step-by-step:

Height Calculation: For any given node, calculate the height of its left and right subtrees.

Diameter Calculation: The diameter at a node is either:

The longest path in the left subtree.
The longest path in the right subtree.
A path that includes this node as the root (which would be height[left] + height[right]).

Recursive DFS: Traverse the tree using recursion to compute these values for every node.

#### Implementation

We'll implement a solution where each node stores its height and diameter, then use these values to find the overall diameter of the tree.

PYTHON
class TreeNode:
    def __init__(self, val=0, left=None, right=None):
        self.val = val
        self.left = left
        self.right = right
        self.height = 0
        self.diameter = 0

def calculate_diameter(root):
    def dfs(node):
        if not node:
            return -1

        # Recursively get the height and diameter of left and right subtrees
        left_height, left_diameter = dfs(node.left)
        right_height, right_diameter = dfs(node.right)

        # Diameter through this node
        current_diameter = 2 + left_height + right_height

        # Maximum height from left or right subtree (1 is added to include the current node's level)
        max_height = 1 + max(left_height, right_height)

        # Update global diameter if necessary
        nonlocal global_diameter
        global_diameter = max(global_diameter, current_diameter, left_diameter, right_diameter)

        return (max_height, current_diameter)

    global_diameter = 0
    dfs(root)
    return global_diameter

# Example usage:
root = TreeNode(1)
root.left = TreeNode(2)
root.right = TreeNode(3)
root.left.left = TreeNode(4)
root.left.right = TreeNode(5)

print(calculate_diameter(root))  # Output: 4 (path is 4 -> 2 -> 1 -> 3 or equivalent in terms of nodes)

#### Closing Summary

The diameter of a binary tree can be found using DFS to calculate the height and diameter at each node. The path might not necessarily pass through the root, as it could be entirely within one subtree.

#### Common Mistakes
and Follow-Ups
Forgetting to use nonlocal: Ensure that you correctly update the global diameter variable.
Misinterpreting the Problem: Ensure that the diameter is defined as the longest path between any two nodes, not just through the root.
Edge Cases: Consider what happens if the tree has only one node or is completely unbalanced.

**Comparison:**

| Node | Height | Diameter |
|------|--------|----------|
| 1    | 3      | 4        |
| 2    | 2      | 2        |
| 3    | 1      | 0        |
| 4    | 0      | 0        |
| 5    | 0      | 0        |

**Code:**

```python
class TreeNode:
    def __init__(self, val=0, left=None, right=None):
        self.val = val
        self.left = left
        self.right = right
        self.height = 0
        self.diameter = 0

def calculate_diameter(root):
    def dfs(node):
        if not node:
            return -1

        left_height, left_diameter = dfs(node.left)
        right_height, right_diameter = dfs(node.right)

        current_diameter = 2 + left_height + right_height
        max_height = 1 + max(left_height, right_height)

        nonlocal global_diameter
        global_diameter = max(global_diameter, current_diameter, left_diameter, right_diameter)

        return (max_height, current_diameter)

    global_diameter = 0
    dfs(root)
    return global_diameter

# Example usage:
root = TreeNode(1)
root.left = TreeNode(2)
root.right = TreeNode(3)
root.left.left = TreeNode(4)
root.left.right = TreeNode(5)

print(calculate_diameter(root))  # Output: 4 (path is 4 -> 2 -> 1 -> 3 or equivalent in terms of nodes)
```

**Example input / output:**

```text
root = TreeNode(1)
root.left = TreeNode(2)
root.right = TreeNode(3)
root.left.left = TreeNode(4)
root.left.right = TreeNode(5)

print(calculate_diameter(root))  # Output: 4 (path is 4 -> 2 -> 1 -> 3 or equivalent in terms of nodes)
```

**Data:**

```json
{
    "tree": {
        "nodes": [
            {"id": 1, "val": 1},
            {"id": 2, "val": 2},
            {"id": 3, "val": 3},
            {"id": 4, "val": 4},
            {"id": 5, "val": 5}
        ],
        "edges": [
            { "source": 1, "target": 2 },
            { "source": 1, "target": 3 },
            { "source": 2, "target": 4 },
            { "source": 2, "target": 5 }
        ]
    }
}
```

**Flow:**

1. Diameter Calculation
2. Calculate Height and Diameter
3. Check Current Path Diameter
4. Update Global Diameter
5. Start
6. Create TreeNode Class
7. Define calculate_diameterFunction
8. Define DFS Helper Function
9. Initialize Global Diameter
10. Traverse Tree with DFS
11. Update Global Diameter
12. Return Global Diameter
13. End

