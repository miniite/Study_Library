---
type: "practice"
difficulty: "Other"
---

# Detect Whether a Singly Linked List Loops Back on Itself

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Two Pointers](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Two_Pointers.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given the first node (start) of a singly linked list. Every node holds an integer and a single next reference that points to the following node, or to nothing (null) at the end of a normal, straight list.

Because the next references are set programmatically, it is possible that some node's next points back to an earlier node instead of continuing forward. When that happens, walking the list by repeatedly following next never terminates — you keep looping through the same set of nodes forever. We call this a loop (or cycle).

Your job is to decide whether such a loop exists. Return true if, by starting at start and repeatedly following next, you would eventually revisit a node you have already seen; return false if following next instead reaches null (a proper end).

Important details:

You are given only the start reference. The position that the loop reconnects to is not provided to you — you must discover the loop's existence purely by traversing.
An empty list (start is null) has no loop, so the answer is false.
You should not modify the list. Aim to use only O(1) extra memory.

Input

start: the head node of a singly linked list (nodes connected through next links; in the examples a helper index link shows which node the tail's next reconnects to, -1 meaning the tail ends normally — this index is illustrative only and is not available to your function).

Output

A boolean: true if the list contains a loop, else false.

#### Cons
traints

The number of nodes is in the range [0, 10^4].
Each node's value is in the range [-10^5, 10^5].
link (used only to describe the examples) is -1 or a valid node index.

#### Example
s & test cases
TEXT
Input: values = [7, 4, 1, -8], link = 1 -> Output: true
Why: The tail node (-8) reconnects to index 1 (value 4), so following next cycles 4 -> 1 -> -8 -> 4 -> ... forever.

Input: values = [5, 9], link = 0 -> Output: true
Why: The second node points back to the first, forming a two-node loop that never reaches null.

Input: values = [11], link = -1 -> Output: false
Why: A single node whose next is null terminates immediately; there is no loop.

Input: values = [], link = -1 -> Output: false
Why: An empty list has no nodes to revisit, so the answer is false.

Input: values = [2, 6, 3, 8, 10], link = -1 -> Output: false
Why: Every next moves strictly forward and the last node ends in null, so traversal terminates without repeating.

Input: values = [1, 1, 1], link = 0 -> Output: true
Why: Duplicate values do not matter; identity, not value, defines a node, and the tail links back to the head to form a loop.

#### Approach

Use Floyd's two-speed pointer technique (the "tortoise and hare"). Advance a slow pointer one node at a time and a fast pointer two nodes at a time from the same starting point. If the list ends (fast or fast.next becomes null), no loop exists — a terminating end was reached. If a loop exists, the fast pointer eventually enters the cycle and, because it closes the gap to the slow pointer by exactly one node per iteration, it must land on the exact same node as the slow pointer; that collision proves a loop without ever recording which nodes were visited. This gives an O(1)-memory answer, which is why it is preferred over the alternative of storing seen nodes in a hash set (O(n) space). Pattern: fast-and-slow pointers / cycle detection on a linked structure.

Complexity

Time: O(n), where n is the number of nodes. If there is no loop the fast pointer reaches the end after at most n/2 hops; if there is a loop the two pointers are guaranteed to meet within one full traversal of the cycle, so the total work is linear. Space: O(1) — only two pointer variables are used regardless of list length.

Python
PYTHON
from typing import Optional

class ListNode:
    def __init__(self, val=0, next=None):
        self.val = val
        self.next = next

class Solution:
    def hasCycle(self, start: Optional[ListNode]) -> bool:
        slow = fast = start
        while fast and fast.next:
            slow = slow.next          # one step
            fast = fast.next.next     # two steps
            if slow is fast:          # pointers met -> loop
                return True
        return False                  # reached the end -> no loop
JavaScript
JAVASCRIPT
class ListNode {

#### cons
tructor(val = 0, next = null) {
        this.val = val;
        this.next = next;
    }
}

function hasCycle(start) {
    let slow = start;
    let fast = start;
    while (fast && fast.next) {
        slow = slow.next;             // one step
        fast = fast.next.next;        // two steps
        if (slow === fast) return true; // met -> loop exists
    }
    return false;                     // reached the end -> no loop
}
Java
JAVA
class ListNode {
    int val;
    ListNode next;
    ListNode(int x) { val = x; next = null; }
}

class Solution {
    public boolean hasCycle(ListNode start) {
        ListNode slow = start;
        ListNode fast = start;

        while (fast != null && fast.next != null) {
            slow = slow.next;          // one step
            fast = fast.next.next;     // two steps
            if (slow == fast)          // met -> loop exists
                return true;
        }
        return false;                  // fell off the end -> no loop
    }
}
C++
CPP
struct ListNode {
    int val;
    ListNode *next;
    ListNode(int x) : val(x), next(nullptr) {}
};

class Solution {
public:
    bool hasCycle(ListNode *start) {
        ListNode *slow = start;
        ListNode *fast = start;

        while (fast && fast->next) {
            slow = slow->next;         // one step
            fast = fast->next->next;   // two steps
            if (slow == fast)          // they met -> loop exists
                return true;
        }
        return false;                  // reached the end -> no loop
    }
};
C
C
#include <stdbool.h>
#include <stdlib.h>

struct ListNode {
    int val;
    struct ListNode *next;
};

bool hasCycle(struct ListNode *start) {
    struct ListNode *slow = start;
    struct ListNode *fast = start;

    while (fast && fast->next) {
        slow = slow->next;          /* one step  */
        fast = fast->next->next;    /* two steps */
        if (slow == fast)           /* pointers met -> loop */
            return true;
    }
    return false;                   /* fast fell off the end -> no loop */
}

#### Follow-up
questions
Beyond a yes/no answer, return the exact node where the loop begins (entry point of the cycle).
Report the length of the loop — how many nodes are in the cycle.
Detect the loop and then remove it so the list becomes a proper null-terminated list.
Contrast the O(1)-memory pointer method with a hash-set approach and discuss the trade-offs when nodes cannot be compared by identity.

