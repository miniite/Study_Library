---
type: "practice"
difficulty: "Other"
---

# Reverse the Order of a Singly Linked Chain

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Linked List](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Linked_List.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given the head node of a singly linked chain. Each node stores an integer value and a pointer to the next node; the last node points to nothing (null). Reverse the direction of every link so that the node that was last becomes the new head and the node that was the head becomes the tail, then return the head of the reversed chain.

You must reverse the chain in place by re-wiring the existing nodes' next pointers — do not build a second list of freshly allocated nodes.

Input

head: the first node of a singly linked chain, or null if the chain is empty. Each node has fields val (an integer) and next (the following node or null).

Output

The head node of the chain after all links have been reversed. If the input chain is empty, return null.

#### Cons
traints

The number of nodes lies in the range [0, 5000].
-5000 <= node.val <= 5000.
The reversal should ideally use O(1) extra space.

#### Example
s & test cases
TEXT
- Input: head = [7, 3, 9, 4, 1] -> Output: [1, 4, 9, 3, 7]
  Why: standard multi-node chain; the entire order is flipped.
- Input: head = [8, 2] -> Output: [2, 8]
  Why: two-node chain checks the basic pointer swap between exactly two elements.
- Input: head = [] -> Output: []
  Why: empty chain edge case; nothing to reverse, return null.
- Input: head = [42] -> Output: [42]
  Why: single node; reversal leaves it unchanged, and the tail's null terminator must be preserved.
- Input: head = [-5, 0, -5, 6] -> Output: [6, -5, 0, -5]
  Why: negative and duplicate values confirm the reversal is by position, independent of the stored values.

#### Approach

Reversing a singly linked chain in place is the canonical three-pointer technique. Keep prev (the reversed part built so far, initially null), curr (the node being processed, initially the head), and a temporary next pointer. On each step, first save curr.next into next so the forward link isn't lost, then redirect curr.next to point back at prev, then slide both prev and curr one step forward. When curr falls off the end (null), prev holds the former tail, which is the new head. This runs in a single pass with constant extra space. The same result can be achieved recursively by reversing the sublist after head, then making head.next.next = head and cutting head.next, but that costs O(n) stack space; the iterative version is preferred as the optimal solution. Pattern: in-place pointer manipulation on linked lists.

Complexity

Time: O(n) — each of the n nodes is visited exactly once and re-wired in O(1). Space: O(1) — only three fixed pointers (prev, curr, next) are used, regardless of chain length. (A recursive formulation is also O(n) time but uses O(n) stack space for the call chain.)

Python
PYTHON
class ListNode:
    def __init__(self, val=0, next=None):
        self.val = val
        self.next = next

class Solution:
    def reverseChain(self, head: 'ListNode') -> 'ListNode':
        # Three-pointer in-place reversal; prev ends up as the new head.
        prev = None
        curr = head
        while curr:
            next_node = curr.next
            curr.next = prev
            prev = curr
            curr = next_node
        return prev
JavaScript
JAVASCRIPT
class ListNode {

#### cons
tructor(val, next = null) {
        this.val = val;
        this.next = next;
    }
}

function reverseChain(head) {
    // Iteratively flip each pointer; prev is the new head at the end.
    let prev = null;
    let curr = head;
    while (curr !== null) {

#### cons
t nextNode = curr.next;
        curr.next = prev;
        prev = curr;
        curr = nextNode;
    }
    return prev;
}
Java
JAVA
class ListNode {
    int val;
    ListNode next;
    ListNode(int val) { this.val = val; }
}

class Solution {
    public ListNode reverseChain(ListNode head) {
        // prev trails behind curr, collecting the reversed portion.
        ListNode prev = null;
        ListNode curr = head;
        while (curr != null) {
            ListNode nextNode = curr.next;
            curr.next = prev;
            prev = curr;
            curr = nextNode;
        }
        return prev;
    }
}
C++
CPP
struct ListNode {
    int val;
    ListNode* next;
    ListNode(int x) : val(x), next(nullptr) {}
};

class Solution {
public:
    ListNode* reverseChain(ListNode* head) {
        // Iterative three-pointer reversal in O(1) extra space.
        ListNode* prev = nullptr;
        ListNode* curr = head;
        while (curr != nullptr) {
            ListNode* nextNode = curr->next;
            curr->next = prev;
            prev = curr;
            curr = nextNode;
        }
        return prev;
    }
};
C
C
struct ListNode {
    int val;
    struct ListNode* next;
};

struct ListNode* reverseChain(struct ListNode* head) {
    // Walk the chain once, flipping each node's next pointer to its predecessor.
    struct ListNode* prev = NULL;
    struct ListNode* curr = head;
    while (curr != NULL) {
        struct ListNode* nextNode = curr->next; // remember what's ahead
        curr->next = prev;                       // re-wire backwards
        prev = curr;                             // advance the "reversed" head
        curr = nextNode;                         // advance the cursor
    }
    return prev; // prev is the old tail = new head
}

#### Follow-up
questions
Can you also give a recursive implementation, and what is its space cost compared to the iterative one?
How would you reverse only the nodes between positions m and n, leaving the rest intact?
How would you reverse the chain in groups of exactly k consecutive nodes?
If the structure were a doubly linked list, what additionally must change during reversal?

