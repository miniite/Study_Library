---
type: "practice"
difficulty: "Basic"
---

# Invert a binary tree and explain why this question became a meme in interview circles.

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Binary Tree Traversal](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Binary_Tree_Traversal.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To invert a binary tree, we need to swap all of its left and right children at every node. This problem has become somewhat of a meme in interview circles because it's simple enough to be quickly grasped but complex enough to probe several important concepts around data structures and algorithms.

#### Triage

The question is asking for an explanation and implementation of the binary tree inversion operation, also known as the "invert" or "reflect" operation. This problem tests your understanding of recursion, depth-first search (DFS), and the properties of a binary tree. The interviewer might be probing for clear code, an efficient algorithmic approach, and the ability to handle edge cases.

#### First-Principles Solution

To invert a binary tree, we can use a recursive approach that traverses each node in the tree. For every node, we swap its left child with its right child, then recursively do the same for both children until all nodes are processed. This ensures that the entire structure of the tree is inverted.

Step-by-Step
Base Case: If the current node is null, return immediately.
Recursive Case:
Swap the left and right children of the current node.
Recursively apply this inversion to the left child.
Recursively apply this inversion to the right child.
Code Implementation
JAVASCRIPT
function invertTree(root) {
  if (root === null) return null;

  // Swap the left and right children

#### cons
t temp = root.left;
  root.left = root.right;
  root.right = temp;

  // Recursively call on left and right subtrees
  invertTree(root.left);
  invertTree(root.right);

  return root;
}

#### Closing Summary
+ Common Mistakes/Follow-Ups

Inverting a binary tree is an excellent problem for interviews because it requires clear understanding of recursive functions, tree traversal techniques, and handling edge cases such as null nodes. A common mistake is not properly swapping the children or incorrectly implementing the base case.

#### Follow-ups
The interviewer might ask about optimizing the algorithm if multiple inversions are needed, discussing other data structures that could be used for similar operations, or explaining how to implement this in an iterative manner using a stack.

