---
type: "practice"
difficulty: "Deep"
---

# Construct a binary tree from its inorder and preorder traversals.

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Binary Tree Traversal](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Binary_Tree_Traversal.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To construct a binary tree from its inorder and preorder traversals, we need to understand that these two traversal methods provide enough information to uniquely determine the structure of the tree. The key idea is that in a binary tree, the root node is always the first element in the preorder sequence, and it splits the inorder sequence into left and right subtrees.

In this problem, let's assume we are given the following sequences:

Preorder: [root, left_subtree_root, ...]
Inorder: [...left_subtree..., root, ...right_subtree...]

The strategy involves recursively building the tree by identifying the root from the preorder sequence and then splitting the inorder sequence into left and right subtrees based on this root. This process is repeated for each subtree until the entire tree is constructed.

#### Triage
What the question is asking: To reconstruct a binary tree given its inorder and preorder traversals.

#### Assumptions
The input lists are non-empty and valid.
The elements in both sequences are distinct.
Objective: Construct the binary tree, ensuring correct positioning of nodes based on traversal sequences.
Step-by-Step Solution
Identify Root Node: The first element in the preorder sequence is the root node.
Split Inorder Sequence: Find this root node's position in the inorder sequence to split it into left and right subtrees.
Recursive Construction:
For each subtree, the first element of its corresponding segment in the preorder list becomes its new root.
Recursively repeat steps 1-2 for these segments.

#### Example

Given:

Preorder: [A, B, D, E, C, F]
Inorder: [D, B, E, A, F, C]
Step-by-Step Construction

Root Identification: The root is A (first element of preorder).

Splitting:

Left Subtree of A: Preorder = [B, D, E], Inorder = [D, B, E]
Right Subtree of A: Preorder = [C, F], Inorder = [F, C]

Left Subtree Construction:

Root is B (first element of left preorder).
Splitting: Left subtree of B: Preorder = [D], Inorder = [D]
Node D.
Right subtree of B: Preorder = [E], Inorder = [E]
Node E.

Right Subtree Construction:

Root is C (first element of right preorder).
Splitting: No need for further splitting, only node C.
Resulting Tree
CODE
    A
   / \
  B   C
 / \
D   E

#### Closing Summary
and Common Mistakes/Follow-Ups

#### Common Mistakes

Incorrectly identifying the root or misunderstanding how to split the inorder sequence.
Not handling edge cases such as single-node trees or empty sequences.

#### Follow-Up
Questions:

How would you modify this approach if the inorder and preorder traversals were given in a different order?
What are the time complexities involved in constructing the tree using these methods?

**Code:**

```python
def build_tree(preorder, inorder):
    def construct_tree(pre_start, pre_end, in_start, in_end):
        if pre_start > pre_end or in_start > in_end:
            return None

        # Root is always the first element of preorder list
        root_val = preorder[pre_start]
        root = TreeNode(root_val)

        # Find position of root in inorder list to split it into left and right subtrees
        idx = index_map[root_val]

        # Recursively build left subtree
        root.left = construct_tree(pre_start + 1, pre_start + (idx - in_start), in_start, idx - 1)

        # Recursively build right subtree
        root.right = construct_tree(pre_start + (idx - in_start) + 1, pre_end, idx + 1, in_end)

        return root

    index_map = {val: idx for idx, val in enumerate(inorder)}
    return construct_tree(0, len(preorder) - 1, 0, len(inorder) - 1)

# Example usage
preorder = ["A", "B", "D", "E", "C", "F"]
inorder = ["D", "B", "E", "A", "F", "C"]

root = build_tree(preorder, inorder)
```

**Example input / output:**

```

Given:

Preorder: ["A", "B", "D", "E", "C", "F"]
Inorder: ["D", "B", "E", "A", "F", "C"]

Output:

CODE
Copy
A

/
B C /
D E
```

**Data:**

```json
{
  "preorder": ["A", "B", "D", "E", "C", "F"],
  "inorder": ["D", "B", "E", "A", "F", "C"]
}
```

