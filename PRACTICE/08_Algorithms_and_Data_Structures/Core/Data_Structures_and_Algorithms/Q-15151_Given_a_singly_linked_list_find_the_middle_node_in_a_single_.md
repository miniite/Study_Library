---
type: "practice"
difficulty: "Basic"
---

# Given a singly linked list, find the middle node in a single pass.

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Linked List Techniques](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Linked_List_Techniques.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To solve this problem of finding the middle node in a singly linked list with a single pass, we need to ensure that our algorithm traverses each node only once and efficiently identifies the middle point. This can be achieved using two pointers: a slow pointer and a fast pointer.

#### Triaging the Question
Understanding the Problem: The task is to find the middle node of a singly linked list in one pass.
Key Probing Points:
Algorithm efficiency (time complexity).
Handling edge cases like empty or single-node lists.
Ensuring a single traversal through the list.

#### First Principles Solution
Initialization: Start by initializing two pointers, slow and fast, both pointing to the head of the linked list.
Traversal: Move the fast pointer twice as fast as the slow pointer. This means that for every node slow visits, fast will visit two nodes.
Midpoint Identification: When fast reaches the end of the list (i.e., fast or fast.next is null), slow will be at the middle node.
Step-by-Step Solution
Start with Initialization:
PYTHON
slow = head
fast = head
Traverse the List:
While fast and fast.next are not null, move both pointers.
Move slow by one node: slow = slow.next
Move fast by two nodes: fast = fast.next.next
Identify the Middle Node: When the loop ends, slow will be at the middle node.

#### Closing Summary
and Common Mistakes
The key to this solution is recognizing that moving fast twice as fast allows it to traverse the entire list while slow traverses half of it.
A common mistake is not handling edge cases where the list might have an even number of nodes. In such cases, the middle node will be one before or after the point where fast would typically stop.

#### Example

#### Cons
ider a singly linked list with nodes [1 -> 2 -> 3 -> 4 -> 5]. The algorithm works as follows:

Start: slow = 1, fast = 1
After first iteration: slow = 2, fast = 3 (because fast.next.next is not null)
Continue until: fast = 5 and fast.next is null. At this point, slow will be at node 3.

#### Implementation
PYTHON
class ListNode:
    def __init__(self, x):
        self.val = x
        self.next = None

def find_middle_node(head):
    if not head:
        return None

    slow = fast = head
    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next

    return slow
Code Block
PYTHON
class ListNode:
    def __init__(self, x):
        self.val = x
        self.next = None

def find_middle_node(head):
    if not head:
        return None

    slow = fast = head
    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next

    return slow
Self-Check

The code is correct and covers all the necessary steps to identify the middle node in a single pass.

