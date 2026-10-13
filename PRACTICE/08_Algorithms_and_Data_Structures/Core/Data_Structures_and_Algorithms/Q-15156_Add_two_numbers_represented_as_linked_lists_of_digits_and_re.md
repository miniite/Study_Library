---
type: "practice"
difficulty: "Basic"
---

# Add two numbers represented as linked lists of digits and return the sum as a list.

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Linked List Arithmetic](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Linked_List_Arithmetic.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To solve this problem, you need to add two numbers that are represented as singly linked lists in reverse order (i.e., each node contains a single digit from 0-9). The goal is to return a new list representing the sum of these two numbers. This problem can be approached by simulating how addition works manually, considering carry-over values.

#### Triage

The interviewer is probing your understanding of linked lists and basic arithmetic operations. You need to consider edge cases like carrying over digits and handling the final result if there’s a remaining carry after processing all nodes.

#### First-Principles Solution
Initialize Variables: Start by initializing pointers for both input linked lists and an integer to hold any carry-over.
Traversal Loop: Traverse through both linked lists simultaneously, adding corresponding node values along with the current carry.
Handle Carry: If the sum is 10 or more, set a new carry and adjust the digit accordingly.
Add New Nodes: Append new nodes to the result list as necessary based on the current carry and summed value.
Final Check: After exiting the loop, check if there’s any remaining carry; if so, add one more node with this value.

#### Closing Summary
+ Common Mistakes

This problem requires careful handling of edge cases, such as unequal lengths of input lists and potential carries at the end. A common mistake is forgetting to handle a final carry that might occur after processing all nodes. Another pitfall is not correctly managing the direction in which digits are added (in this case, from the least significant digit).

