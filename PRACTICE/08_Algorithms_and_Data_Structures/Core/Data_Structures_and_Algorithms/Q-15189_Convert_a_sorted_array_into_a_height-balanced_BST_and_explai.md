---
type: "practice"
difficulty: "Intermediate"
---

# Convert a sorted array into a height-balanced BST and explain the divide step.

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Binary Search Trees](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Binary_Search_Trees.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To convert a sorted array into a height-balanced binary search tree (BST), we leverage the properties of both data structures: the sorted nature of the array and the balanced property of the BST. The key idea is to use a divide-and-conquer approach, where the middle element of the array serves as the root node, ensuring that the tree remains balanced.

#### Triaging the Question

The question is asking us to implement an algorithm that takes a sorted array and constructs a height-balanced BST from it. The interviewer is likely probing our understanding of binary search trees, divide-and-conquer strategies, and the properties of balanced trees. We need to assume the array is already sorted, as stated in the problem.

#### First-Principles Solution
Choose the Middle Element: Select the middle element of the array to be the root node. This ensures that both left and right subtrees will have nearly equal heights.
Divide the Array: Recursively apply the same logic on the left half of the array for the left subtree and on the right half for the right subtree.
Step-by-Step Reasoning
Base Case: If the input array is empty, return null.
Recursive Step:
Find the middle element of the current array segment.
Create a new node with this middle element as its value.
Recursively build the left subtree using the left half of the array (elements to the left of the middle).
Recursively build the right subtree using the right half of the array (elements to the right of the middle).

#### Example
Code Implementation

Let's walk through a Python implementation to illustrate this process.

PYTHON
class TreeNode:
    def __init__(self, val=0, left=None, right=None):
        self.val = val
        self.left = left
        self.right = right

def sorted_array_to_bst(arr):
    if not arr:
        return None

    mid = len(arr) // 2
    root = TreeNode(arr[mid])

    root.left = sorted_array_to_bst(arr[:mid])
    root.right = sorted_array_to_bst(arr[mid+1:])

    return root

# Example usage
arr = [1, 2, 3, 4, 5, 6, 7]
bst_root = sorted_array_to_bst(arr)

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, the divide step involves recursively partitioning the array by choosing the middle element as the root of each subtree. This ensures that the tree remains balanced.

#### Common mistakes
include:

Not correctly handling edge cases (e.g., empty arrays).
Failing to use recursion properly.
Not understanding the importance of the middle element in maintaining balance.

#### Follow-ups
could involve discussing time and space complexity, trade-offs with other approaches, or specific optimizations for different scenarios.

