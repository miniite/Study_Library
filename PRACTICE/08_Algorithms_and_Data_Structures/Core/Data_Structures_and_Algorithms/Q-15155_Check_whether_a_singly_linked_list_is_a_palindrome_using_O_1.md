---
type: "practice"
difficulty: "Intermediate"
---

# Check whether a singly linked list is a palindrome using O(1) extra space.

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Linked List Techniques](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Linked_List_Techniques.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To check if a singly linked list is a palindrome with (O(1)) extra space, we need to find a way to compare elements from both ends of the list without using additional storage like an array or another linked list. The key idea is to use two pointers and potentially reverse half of the linked list in place.

#### Triage

The question asks us to check if a singly linked list is a palindrome with constant space complexity, meaning no extra data structures can be used apart from those already present in the list. We need to devise an algorithm that performs this operation efficiently.

#### Solution
Step 1: Identify Key Points
Palindrome Property: A linked list is a palindrome if it reads the same backward as forward.
Two Pointers Approach: Use two pointers to traverse the list, one slow and one fast. The slow pointer moves one step at a time while the fast pointer moves two steps at a time.
Step 2: Reverse Half of the List
Find the middle point of the linked list using the slow and fast pointer technique.
Reverse the second half of the linked list in place.
Compare elements from the beginning (first part) and the end (reversed second part) of the list.
Step 3: Reconstruct the List

After comparing, if the list is a palindrome, reconstruct the list to its original state by reversing the second half again.

Pseudocode
PYTHON
def reverse_linked_list(head):
    prev = None
    current = head
    while current:
        next_node = current.next
        current.next = prev
        prev = current
        current = next_node
    return prev

def is_palindrome(head):
    if not head or not head.next:
        return True

    # Find the middle of the list
    slow, fast = head, head
    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next

    # Reverse the second half of the list
    second_half_head = reverse_linked_list(slow)

    # Check if the first half equals the reversed second half
    first_half_node, second_half_node = head, second_half_head
    is_palindrome = True
    while second_half_node:
        if first_half_node.val != second_half_node.val:
            is_palindrome = False
            break
        first_half_node = first_half_node.next
        second_half_node = second_half_node.next

    # Restore the original order of the list
    reverse_linked_list(second_half_head)

    return is_palindrome

#### Closing Summary
and Common Mistakes/Follow-Ups

This approach ensures that we use (O(1)) extra space since only a few pointers are used, and no additional data structures like arrays or lists are created. The main challenge is managing the reversal of the second half of the list without losing track of the original order.

#### Common mistakes
include:

Forgetting to reverse the linked list back to its original state after checking for the palindrome.
Not correctly identifying the middle point, which can lead to incorrect handling of even-length lists.
Mismanaging pointers during the reversal process.

#### Follow-ups
could involve discussing edge cases like empty or single-element lists and optimizing further if necessary.

