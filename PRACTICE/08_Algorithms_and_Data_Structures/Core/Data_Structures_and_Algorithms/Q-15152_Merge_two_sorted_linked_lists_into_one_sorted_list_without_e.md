---
type: "practice"
difficulty: "Basic"
---

# Merge two sorted linked lists into one sorted list without extra allocation.

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Linked List Techniques](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Linked_List_Techniques.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Let's tackle the task of merging two sorted linked lists into a single sorted linked list without using any additional data structures. The key idea here is to traverse both lists simultaneously, comparing nodes and appending them in order to the result.

#### Triaging the Question

The core question is about efficiently merging two pre-sorted linked lists. This requires us to understand how to traverse these lists, compare elements, and link them appropriately without using extra space (apart from a few pointers). The interviewer might be probing your understanding of linked list traversal, conditional logic, and optimization techniques.

Step-by-Step Solution
Initialize Pointers: Start by initializing two pointers for each of the input lists.
Traverse Lists: Traverse both lists simultaneously, comparing the current nodes pointed to by these pointers.
Insert Smaller Element: Append the smaller node from the comparison to the result list and move its pointer forward.
Continue Until Completion: Repeat this process until one of the lists is fully traversed.
Append Remaining Nodes: Once one list is exhausted, append the remaining nodes from the other list directly.

#### Example
Code

Here's how you can implement the above logic in Python:

PYTHON
# Definition for singly-linked list.
class ListNode:
    def __init__(self, val=0, next=None):
        self.val = val
        self.next = next

def mergeTwoLists(l1: ListNode, l2: ListNode) -> ListNode:
    # Initialize a dummy node to simplify edge cases
    dummy = ListNode()
    current = dummy

    while l1 and l2:
        if l1.val < l2.val:
            current.next = l1
            l1 = l1.next
        else:
            current.next = l2
            l2 = l2.next
        current = current.next

    # Append the remaining nodes from either list
    current.next = l1 or l2

    return dummy.next

# Helper function to print linked list for verification
def printList(node: ListNode):
    while node:
        print(node.val, end=" -> ")
        node = node.next
    print("None")

# Example usage:
l1 = ListNode(1, ListNode(3, ListNode(5)))
l2 = ListNode(2, ListNode(4, ListNode(6)))

result = mergeTwoLists(l1, l2)
printList(result)

#### Closing Summary

This solution ensures that we only use a constant amount of additional space (pointers), making it efficient. The time complexity is O(n + m), where n and m are the lengths of the two lists respectively. This approach handles all cases, including when one or both lists are empty.

#### Common Mistakes/Follow-Ups
Mistake: Forgetting to handle edge cases where one list might be longer than the other.

#### Follow-Up
Discussing scenarios with unsorted inputs or exploring space complexity optimizations for extremely large datasets.

