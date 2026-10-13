---
type: "practice"
difficulty: "Deep"
---

# Find the lowest common ancestor in a general binary tree (not a BST).

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Binary Tree Traversal](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Binary_Tree_Traversal.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To find the lowest common ancestor (LCA) in a general binary tree, we need to understand the structure of the tree and the properties of LCA. The LCA is defined as the deepest node that has two given nodes as descendants. In this context, "descendants" can be any node below another node in the tree.

#### Triaging the Question

The question asks for an algorithm to find the LCA of two given nodes in a general binary tree. The interviewer is likely probing your understanding of tree traversal algorithms and data structures. We will assume that the tree has unique identifiers for each node, which helps in tracking nodes during our search.

#### First-Principles Solution

To solve this problem, we can use a recursive approach to traverse the tree and find both target nodes. The idea is to keep track of the path from the root to each target node. Once both paths are found, the LCA will be the last common node in these two paths.

Detailed Steps
Define Node Structure: Each node has an identifier (ID) and references to its left child and right child.
Traverse Tree: Use a recursive function to traverse the tree and find both target nodes.
Path Tracking: Maintain a path for each target node as we traverse.
Identify LCA: Once both paths are found, identify the last common node in these two paths.
Code Implementation

Here is the implementation of the solution:

PYTHON
class TreeNode:
    def __init__(self, val):
        self.val = val
        self.left = None
        self.right = None

def find_path(root, path, target_id):
    if root is None:
        return False

    # Append current node to path
    path.append(root)

    # Check if the current node is the target
    if root.val == target_id:
        return True

    # Recursively check left and right subtrees
    if (root.left and find_path(root.left, path, target_id)) or \
       (root.right and find_path(root.right, path, target_id)):
        return True

    # If not found in either subtree, remove current node from path
    path.pop()
    return False

def lowest_common_ancestor(root, node1_id, node2_id):
    path1 = []
    path2 = []

    if not find_path(root, path1, node1_id) or not find_path(root, path2, node2_id):
        return None

    # Find the last common node in both paths
    i = 0
    while i < len(path1) and i < len(path2):
        if path1[i] != path2[i]:
            break
        i += 1

    return path1[i-1]

#### Closing Summary
+ Common Mistakes/Follow-Ups

The LCA problem in a general binary tree can be solved using recursion to find paths and then identify the last common node. Common mistakes include not properly handling null nodes or not correctly identifying the last common node.

A follow-up could involve discussing performance optimizations, such as storing the paths in arrays instead of lists for better memory usage, or exploring other data structures like hash maps to speed up path comparison.

Self-Check

The solution is correct and complete.

