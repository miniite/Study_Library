---
type: "practice"
difficulty: "Intermediate"
---

# Explain how you'd detect an infinite loop in a data-cleaning recursion over linked records, and how to bound recursion depth safely.

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Recursion vs Explicit Stack](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Recursion_vs_Explicit_Stack.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To tackle the problem of detecting an infinite loop during recursive data cleaning over linked records, we need to ensure that our algorithm can gracefully handle such a scenario while also preventing excessive stack usage. The key idea is to maintain state information and carefully manage recursion depth.

First, let's break down what the question is asking:

Detection of Infinite Loop: We need to identify when the recursive function gets stuck in an infinite loop due to cyclic data structures.
Bounding Recursion Depth Safely: To avoid stack overflow or excessive memory usage, we must limit the number of recursive calls.

#### Triage
and Assumptions
The linked records can form cycles, leading to potential infinite loops if not handled properly.
We are dealing with a list-like structure (e.g., singly-linked list) where each node points to another node in the list.
Our goal is to clean or process data within this structure while avoiding infinite recursion.
Step-by-Step Solution
Maintain State Information: Use additional metadata for each node to track whether it has been processed. This can be achieved using a set or dictionary that stores nodes as they are visited.
Depth Tracking and Limitation: Implement a depth counter to limit the number of recursive calls. If the maximum allowed depth is exceeded, raise an error or return early.

#### First-Principles Solution

Initialization:

Initialize a set visited_nodes to keep track of nodes that have been visited.
Set a predefined maximum recursion depth, e.g., 1000 (this value can be tuned based on the application).

Recursive Function:

PYTHON
def clean_records(node, current_depth=0):
    if current_depth > max_depth:
        raise RecursionError("Exceeded maximum allowed depth")

    # Check if node is already visited to avoid infinite loops
    if node in visited_nodes:
        return  # Node has been processed; no need to reprocess

    visited_nodes.add(node)  # Mark the node as visited

    # Perform data cleaning or other operations on `node`

    # Recursively clean next nodes (if they exist)
    for next_node in get_next_nodes(node):
        clean_records(next_node, current_depth + 1)

Helper Function:

PYTHON
def get_next_nodes(node):
    return node.next  # Assuming a singly-linked list with `next` attribute

#### Closing Summary
and Common Mistakes/Follow-Ups

#### Common Mistakes
Failing to initialize the visited_nodes set can lead to false positives or infinite loops. Not setting a maximum depth limit may result in stack overflow.

#### Follow-Ups
How would you handle a doubly-linked list instead of a singly-linked list?
What are the implications of choosing different values for max_depth?
Can you optimize this solution further to minimize memory usage?

