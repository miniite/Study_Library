---
type: "practice"
difficulty: "Other"
---

# Swap Adjacent Node Pairs in a Linked List

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Linked List](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Linked_List.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given the head of a singly linked list. Rearrange the list so that every pair of adjacent nodes is swapped: the first and second nodes trade places, then the third and fourth, and so on. Return the head of the modified list.

You must perform the swap by relinking the nodes themselves — you are not allowed to simply exchange the integer values stored inside the nodes. If the list has an odd number of nodes, the final leftover node keeps its original position.

Input

head: the head node of a singly linked list (possibly empty).

Output

The head of the list after every adjacent pair has been swapped.

#### Cons
traints

The number of nodes in the list is in the range [0, 100].
0 <= node value <= 100

#### Example
s & test cases
TEXT
Input: head = [8,3,6,1,7] -> Output: [3,8,1,6,7]
Why: Odd length; (8,3) and (6,1) swap while the trailing 7 stays in place.

Input: head = [] -> Output: []
Why: Empty list has nothing to swap.

Input: head = [9] -> Output: [9]
Why: A single node has no partner, so it is returned unchanged.

Input: head = [4,2] -> Output: [2,4]
Why: Exactly one pair swaps.

Input: head = [1,2,3,4,5,6] -> Output: [2,1,4,3,6,5]
Why: Even length with multiple pairs; every consecutive pair is swapped.

Input: head = [5,5,5,5] -> Output: [5,5,5,5]
Why: Duplicate values — the node links are swapped even though the printed values look identical.

#### Approach

Use a dummy sentinel node before the head so the first pair is handled uniformly. Keep a prev pointer to the node just before the pair being swapped. For each pair (first, second), rewire three links: first.next jumps to the node after second, second.next points back to first, and prev.next points to second (the new front of the pair). Advance prev to first (now the second element of the swapped pair) and repeat. The loop condition prev.next && prev.next.next guarantees there is a full pair left, so a lone trailing node is naturally left untouched. This is the classic in-place linked-list pointer-manipulation pattern with a sentinel head.

Complexity

Time: O(n) — each node is visited a constant number of times as we walk the list once, swapping one pair per iteration. Space: O(1) — only a fixed set of pointers (dummy, prev, first, second) is used; the swap is done in place with no recursion stack.

Python
PYTHON
class ListNode:
    def __init__(self, val=0, next=None):
        self.val = val
        self.next = next

def swapPairs(head):
    dummy = ListNode(0, head)
    prev = dummy
    while prev.next and prev.next.next:
        first = prev.next
        second = first.next
        first.next = second.next
        second.next = first
        prev.next = second
        prev = first
    return dummy.next
JavaScript
JAVASCRIPT
class ListNode {

#### cons
tructor(val = 0, next = null) {
        this.val = val;
        this.next = next;
    }
}

function swapPairs(head) {

#### cons
t dummy = new ListNode(0, head);
    let prev = dummy;
    while (prev.next && prev.next.next) {

#### cons
t first = prev.next;

#### cons
t second = first.next;
        first.next = second.next;
        second.next = first;
        prev.next = second;
        prev = first;
    }
    return dummy.next;
}
Java
JAVA
class ListNode {
    int val;
    ListNode next;
    ListNode(int val) { this.val = val; }
    ListNode(int val, ListNode next) { this.val = val; this.next = next; }
}

class Solution {
    public ListNode swapPairs(ListNode head) {
        ListNode dummy = new ListNode(0, head);
        ListNode prev = dummy;
        while (prev.next != null && prev.next.next != null) {
            ListNode first = prev.next;
            ListNode second = first.next;
            first.next = second.next;
            second.next = first;
            prev.next = second;
            prev = first;
        }
        return dummy.next;
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
    ListNode *swapPairs(ListNode *head) {
        ListNode dummy(0);
        dummy.next = head;
        ListNode *prev = &dummy;
        while (prev->next && prev->next->next) {
            ListNode *first = prev->next;
            ListNode *second = first->next;
            first->next = second->next;
            second->next = first;
            prev->next = second;
            prev = first;
        }
        return dummy.next;
    }
};
C
C
struct ListNode {
    int val;
    struct ListNode *next;
};

struct ListNode *swapPairs(struct ListNode *head) {
    struct ListNode dummy;
    dummy.next = head;
    struct ListNode *prev = &dummy;
    while (prev->next && prev->next->next) {
        struct ListNode *first = prev->next;
        struct ListNode *second = first->next;
        first->next = second->next;
        second->next = first;
        prev->next = second;
        prev = first;
    }
    return dummy.next;
}

#### Follow-up
questions
Generalize to reversing every group of k consecutive nodes instead of just pairs.
Solve it recursively rather than iteratively — what are the stack-space trade-offs?
Why is swapping the nodes (relinking) considered more instructive than just swapping the stored values?
Adapt the solution to a doubly linked list where the backward prev pointers must also be fixed.

