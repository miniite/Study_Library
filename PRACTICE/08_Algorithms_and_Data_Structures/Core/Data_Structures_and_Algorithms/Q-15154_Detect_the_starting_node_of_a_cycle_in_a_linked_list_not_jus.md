---
type: "practice"
difficulty: "Intermediate"
---

# Detect the starting node of a cycle in a linked list, not just whether one exists.

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Cycle Detection in Linked Lists](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Cycle_Detection_in_Linked_Lists.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To detect the starting node of a cycle in a linked list, we need to go beyond simply detecting if there is a cycle and identify where the cycle begins. This problem can be elegantly solved using Floyd's Tortoise and Hare algorithm, which leverages two pointers moving at different speeds through the linked list.

Firstly, let's summarize what the question is asking: given a singly linked list with one or more cycles, we need to find the node where the cycle starts. The interviewer may be probing your understanding of pointer manipulation, edge case handling, and algorithm efficiency. We'll assume that the linked list can have nodes with unique values.

Now, let's break down the solution step by step:

Initialization: Start with two pointers, slow and fast. Both will initially point to the head of the linked list.
Cycle Detection: Move slow one node at a time and fast two nodes at a time. If there is a cycle, these two pointers will eventually meet inside the cycle.
Finding the Cycle Start Node:
Once we detect a cycle (i.e., slow == fast), reset slow to the head of the list while keeping fast at the meeting point.
Move both pointers one node at a time; they will meet at the starting node of the cycle.

This approach works because by the time slow and fast meet, slow has traversed half the distance of the cycle. Resetting slow to the head and moving both at the same speed ensures that they will intersect exactly at the start of the cycle.

**Code:**

```python
class ListNode:
    def __init__(self, value=0, next=None):
        self.val = value
        self.next = next

def detectCycleStart(head: ListNode) -> ListNode:
    if not head or not head.next:
        return None  # No cycle detected

    slow = fast = head
    has_cycle = False

    # Step 1: Detect the cycle using Floyd's Tortoise and Hare algorithm
    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next

        if slow == fast:
            has_cycle = True
            break

    if not has_cycle:
        return None  # No cycle detected

    # Step 2: Find the starting node of the cycle
    slow = head
    while slow != fast:
        slow = slow.next
        fast = fast.next

    return slow
```

**Example input / output:**

```text
# Example usage
node1 = ListNode(3)
node2 = ListNode(2)
node3 = ListNode(0)
node4 = ListNode(-4)

# Creating a cycle: 1 -> 2 -> 3 -> 0 -> -4
node1.next, node2.next, node3.next, node4.next = node2, node3, node4, node1

print(detectCycleStart(node1).val)  # Output should be the value of node 3 (0)
```

**Flow:**

1. Yes
2. No
3. Head
4. Slow pointer
5. Fast pointer
6. Cycle Detected?
7. Reset Slow to Head
8. No Cycle
9. Move both pointers one step
10. Goto F
11. Return meeting point as cyclestart node

