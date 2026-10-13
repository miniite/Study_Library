---
type: "practice"
difficulty: "Deep"
---

# How would you merge k sorted linked lists efficiently and what is the complexity?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Skip Lists](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Skip_Lists.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To merge (k) sorted linked lists efficiently, we can use a priority queue (min-heap) to ensure that we always pick the smallest element among the heads of all lists at each step. This approach ensures an efficient merge with minimal comparisons and swaps.

#### Triage

The problem is asking for an efficient algorithm to merge multiple sorted linked lists into one single sorted list. The interviewer might be probing your understanding of data structures, particularly heaps, and how they can be used in algorithms. Assumptions include that the input lists are already sorted and we need to return a merged list.

#### Solution
Step-by-Step Explanation

Initialize a Min-Heap:

Create a min-heap (priority queue) where each node of the linked list is stored.
Each node in the heap contains not only its own value but also pointers back to its original list and position.

Insert Initial Nodes into Heap:

For each linked list, insert the first node into the heap. The heap will store nodes along with their corresponding lists and indices.

Extract Minimum Node and Insert Next Node:

Remove the minimum element from the heap.
Add the next node (if any) from the same list that was just removed back to the heap.
Repeat until the heap is empty, meaning all nodes have been processed.

#### Cons
truct Merged List:

As nodes are extracted from the heap and added to a new linked list, this merged list will naturally be sorted due to how the min-heap works.
Code Implementation
PYTHON
import heapq

class ListNode:
    def __init__(self, val=0, next=None):
        self.val = val
        self.next = next

    # For heap operations (min-heap)
    def __lt__(self, other):
        return self.val < other.val

def mergeKLists(lists):
    heap = []

    # Initialize the heap with the first node of each list
    for l in lists:
        if l:
            heapq.heappush(heap, (l.val, l))

    dummy = ListNode(0)
    current = dummy

    while heap:
        val, node = heapq.heappop(heap)  # Get minimum element
        current.next = ListNode(val)
        current = current.next

        if node.next:  # Add next node from the same list to heap
            heapq.heappush(heap, (node.next.val, node.next))

    return dummy.next

# Example usage:
list1 = ListNode(1, ListNode(4, ListNode(5)))
list2 = ListNode(1, ListNode(3, ListNode(4)))
list3 = ListNode(2, ListNode(6))

merged_list = mergeKLists([list1, list2, list3])

#### Closing Summary

The algorithm using a min-heap achieves an efficient solution with time complexity (O(N \log k)), where (N) is the total number of nodes across all lists and (k) is the number of lists. This approach minimizes the need for repeated sorting or merging operations by leveraging the properties of a heap to always find the smallest element quickly.

#### Common Mistakes
Incorrect Heap Implementation: Forgetting to implement the (\lt) method in ListNode can lead to incorrect ordering.
Heap Operations Overhead: Not using efficient heap operations might result in higher complexity than necessary.
Edge Cases: Forgetting to handle cases where a list is empty or all lists are empty.

