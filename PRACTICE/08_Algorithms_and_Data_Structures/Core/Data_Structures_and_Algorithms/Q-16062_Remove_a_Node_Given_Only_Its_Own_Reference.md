---
type: "practice"
difficulty: "Other"
---

# Remove a Node Given Only Its Own Reference

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Linked List](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Linked_List.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are working with a singly linked list of distinct integers. Each node stores an integer value and a pointer to the next node (or nothing, if it is the last node).

You are handed a direct reference to exactly one node in this list, called target, and you must remove target from the list. The catch: you are not given the head of the list, so you cannot walk from the front to locate the node that sits just before target. You only have target itself.

It is guaranteed that target is a genuine node of the list and that it is never the last node. "Removing" here does not literally mean freeing memory (though you may) — it means the list must end up as if target had never existed:

The value originally held by target must no longer appear anywhere in the list.
The total number of nodes must decrease by exactly one.
Every value that came before target stays in its original order.
Every value that came after target stays in its original order.

Your routine receives only the reference to target and returns nothing; it mutates the list in place.

#### Cons
traints:

The list contains between 2 and 1000 nodes.
Each node value v satisfies -1000 <= v <= 1000.
All values in the list are unique.
target is guaranteed to be a real node of the list and is guaranteed not to be the last (tail) node.

#### Example
s & test cases
TEXT
Input: list = [8, 3, 11, 6], target = node holding 3 -> Output: [8, 11, 6]
Why: the value 3 is removed and the remaining values keep their order.

Input: list = [8, 3, 11, 6], target = node holding 11 -> Output: [8, 3, 6]
Why: an interior node other than the head is removed cleanly.

Input: list = [8, 3, 11, 6], target = node holding 8 -> Output: [3, 11, 6]
Why: removing the second-to-none case — even the first node can be handled since it is not the tail.

Input: list = [-5, 20], target = node holding -5 -> Output: [20]
Why: minimum-length list of two nodes; removing the non-tail node leaves a single node.

Input: list = [1, 2, 3, 4, 5], target = node holding 4 -> Output: [1, 2, 3, 5]
Why: the removed value disappears while the node just after it is absorbed forward.

#### Approach

The core obstacle is that deletion of a node in a singly linked list normally requires the previous node so you can point it past the target — but here you only have the target itself. The trick is to delete a node by impersonation: instead of removing the target's memory cell, copy the value of its successor into the target, then splice the successor out (target.next = target.next.next). After this, the target node physically holds what used to be the next value, and the actual next node is unlinked. The net observable effect is identical to having removed the original target value, and the list length shrinks by one. This works only because the target is guaranteed not to be the tail (a tail has no successor to copy from). Pattern: overwrite-and-skip / "copy the next node onto this one."

Complexity

Time: O(1) — a fixed number of pointer/value assignments regardless of list length, since we never traverse. Space: O(1) — no extra data structures are used.

Python
PYTHON
class ListNode:
    def __init__(self, val=0, next=None):
        self.val = val
        self.next = next

class Solution:
    def deleteNode(self, node: 'ListNode') -> None:
        # node is guaranteed not to be the tail, so node.next is not None.
        node.val = node.next.val    # copy successor value into this node
        node.next = node.next.next  # unlink the successor
JavaScript
JAVASCRIPT
function ListNode(val, next = null) {
    this.val = val;
    this.next = next;
}

function deleteNode(node) {
    // node is guaranteed not to be the tail, so node.next exists.
    node.val = node.next.val;    // copy the successor's value forward
    node.next = node.next.next;  // splice the successor out
}
Java
JAVA
class ListNode {
    int val;
    ListNode next;
    ListNode(int x) { val = x; }
}

class Solution {
    public void deleteNode(ListNode node) {
        // node is guaranteed not to be the tail, so node.next exists.
        node.val = node.next.val;   // pull the successor's value into this node
        node.next = node.next.next; // splice the successor out of the list
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
    void deleteNode(ListNode* node) {
        ListNode* nextNode = node->next;   // guaranteed non-null: node is never the tail
        node->val = nextNode->val;         // copy successor value forward
        node->next = nextNode->next;       // bypass the successor
        delete nextNode;                   // free the detached successor
    }
};
C
C
#include <stdlib.h>

struct ListNode {
    int val;
    struct ListNode *next;
};

void deleteNode(struct ListNode* node) {
    struct ListNode* nextNode = node->next;   /* guaranteed non-NULL: node is not the tail */
    node->val = nextNode->val;                /* copy the successor's value into this node   */
    node->next = nextNode->next;              /* splice the successor out of the chain       */
    free(nextNode);                           /* reclaim the now-detached node               */
}

#### Follow-up
questions
Why does this approach break if target is the tail node, and what extra information would you need to handle that case?
Solve the ordinary version: given the head and a value, delete the first node with that value using traversal.
If the list values were not guaranteed unique, would the copy-forward trick still be correct? (Yes — it only depends on target not being the tail.)
Remove the k-th node from the end given only the head, in a single pass.

