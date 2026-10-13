---
type: "practice"
difficulty: "Other"
---

# Increment a Number Stored as a Digit List

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Linked List](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Linked_List.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given a singly linked list whose nodes together represent one non-negative integer. Each node holds exactly one decimal digit, and the digits are ordered from most significant to least significant, so the head of the list holds the leading digit.

Add one to the integer represented by the list and return the head of the resulting list. The result must again be a linked list of single decimal digits with the most significant digit at the head.

For example, the list 4 -> 5 -> 9 represents the number 459; after adding one it becomes 460, i.e. 4 -> 6 -> 0.

Input

head: the first node of a singly linked list of digits (each node's value is a single digit 0..9), representing a non-negative integer with no leading zeros other than the single value 0 itself.

Output

The head of the linked list that represents the original number plus one.

#### Cons
traints

The number of nodes in the list is in the range [1, 100].
Each node value is in the range [0, 9].
The number represented by the list has no leading zeros, except for the number 0, which is represented by a single node with value 0.

#### Example
s & test cases
TEXT
Input: [4,5,9] -> Output: [4,6,0]
Why: 459 + 1 = 460; the carry from the trailing 9 stops at the 5. Normal case with one carry.

Input: [1,2,9] -> Output: [1,3,0]
Why: 129 + 1 = 130; carry propagates only through the last digit.

Input: [9,9,9] -> Output: [1,0,0,0]
Why: all digits are 9, so the carry runs off the front and a new most-significant 1 is prepended. Edge: list grows by one node.

Input: [0] -> Output: [1]
Why: smallest input; 0 + 1 = 1.

Input: [8] -> Output: [9]
Why: single digit with no carry.

#### Approach

Adding one can only propagate a carry across a run of trailing 9s. So the rightmost digit that is not a 9 is exactly where the increment lands: increase it by one, and every 9 to its right becomes 0. If every digit is a 9, the carry escapes the front and a new leading digit 1 is required. Using a dummy node in front of the head lets the all-nines case reuse that dummy as the new most-significant digit, unifying both branches without special casing. This is the classic "carry stops at the last non-9" trick, an O(1)-space alternative to a postorder recursive carry.

Complexity

Time: O(n) — a single pass to locate the rightmost non-9 node and another partial pass to zero the trailing nines, both linear in the number of digits n. Space: O(1) — only a fixed dummy node is allocated; the list is updated in place (the dummy is reused as the new leading digit when all digits are 9).

Python
PYTHON
class ListNode:
    def __init__(self, val=0, next=None):
        self.val = val
        self.next = next

class Solution:
    def plusOne(self, head: ListNode) -> ListNode:
        dummy = ListNode(0)
        dummy.next = head

        # Rightmost node whose digit is not 9 absorbs the carry.
        last_not_nine = dummy
        cur = head
        while cur:
            if cur.val != 9:
                last_not_nine = cur
            cur = cur.next

        last_not_nine.val += 1
        cur = last_not_nine.next
        while cur:
            cur.val = 0
            cur = cur.next

        return dummy.next if dummy.val == 0 else dummy
JavaScript
JAVASCRIPT
class ListNode {

#### cons
tructor(val = 0, next = null) {
        this.val = val;
        this.next = next;
    }
}

function plusOne(head) {

#### cons
t dummy = new ListNode(0);
    dummy.next = head;

    // Find the last node that is not a 9.
    let lastNotNine = dummy;
    for (let cur = head; cur !== null; cur = cur.next) {
        if (cur.val !== 9) lastNotNine = cur;
    }

    lastNotNine.val += 1;
    for (let cur = lastNotNine.next; cur !== null; cur = cur.next) {
        cur.val = 0;
    }

    return dummy.val === 0 ? dummy.next : dummy;
}
Java
JAVA
class ListNode {
    int val;
    ListNode next;
    ListNode(int val) { this.val = val; }
}

class Solution {
    public ListNode plusOne(ListNode head) {
        ListNode dummy = new ListNode(0);
        dummy.next = head;

        // Locate the last node whose digit is not 9.
        ListNode lastNotNine = dummy;
        for (ListNode cur = head; cur != null; cur = cur.next)
            if (cur.val != 9)
                lastNotNine = cur;

        lastNotNine.val += 1;
        for (ListNode cur = lastNotNine.next; cur != null; cur = cur.next)
            cur.val = 0;

        return dummy.val == 0 ? dummy.next : dummy;
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
    ListNode* plusOne(ListNode* head) {
        ListNode* dummy = new ListNode(0);
        dummy->next = head;

        // Rightmost node that is not a 9 is where the carry stops.
        ListNode* lastNotNine = dummy;
        for (ListNode* cur = head; cur; cur = cur->next)
            if (cur->val != 9)
                lastNotNine = cur;

        lastNotNine->val += 1;
        for (ListNode* cur = lastNotNine->next; cur; cur = cur->next)
            cur->val = 0;

        return dummy->val == 0 ? dummy->next : dummy;
    }
};
C
C
#include <stdlib.h>

struct ListNode {
    int val;
    struct ListNode *next;
};

struct ListNode* plusOne(struct ListNode* head) {
    struct ListNode* dummy = malloc(sizeof(struct ListNode));
    dummy->val = 0;
    dummy->next = head;

    /* Find the rightmost node whose digit is not 9. */
    struct ListNode* lastNotNine = dummy;
    for (struct ListNode* cur = head; cur != NULL; cur = cur->next) {
        if (cur->val != 9)
            lastNotNine = cur;
    }

    /* Increment it, and zero out every 9 that follows. */
    lastNotNine->val += 1;
    for (struct ListNode* cur = lastNotNine->next; cur != NULL; cur = cur->next)
        cur->val = 0;

    if (dummy->val == 0) {
        struct ListNode* res = dummy->next;
        free(dummy);
        return res;
    }
    return dummy; /* dummy became the new leading 1 */
}

#### Follow-up
questions
What changes if the digits are stored least-significant-first (the head is the ones digit)? Can you then do it in one forward pass?
Solve it recursively with a postorder DFS that returns the carry — compare stack usage against the O(1) iterative version.
Generalize to adding an arbitrary integer k, or adding two numbers each given as a digit list.
If the list may be extremely long, how do you avoid deep recursion and keep memory bounded?

