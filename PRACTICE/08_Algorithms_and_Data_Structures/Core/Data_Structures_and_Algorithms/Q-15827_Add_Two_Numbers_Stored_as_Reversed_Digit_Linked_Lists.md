---
type: "practice"
difficulty: "Other"
---

# Add Two Numbers Stored as Reversed Digit Linked Lists

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Linked List](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Linked_List.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Two non-empty singly linked lists each represent a single non-negative integer. Every node stores exactly one decimal digit, and the digits are kept in reverse order — the head node holds the least significant (ones) digit, the next node holds the tens digit, and so on.

Add the two numbers and return their sum as a linked list in the same reversed-digit format.

You may assume neither input contains leading zeros, except for the number 0 itself (which is represented by a single node with value 0).

Input

first: head of the first reversed-digit linked list.
second: head of the second reversed-digit linked list.

Output

The head of a new reversed-digit linked list representing first + second.

#### Cons
traints

The number of nodes in each list is in the range [1, 100].
0 <= Node.val <= 9
Each list represents a number that has no leading zeros.

#### Example
s & test cases
TEXT
Input: first = [3,4,2], second = [4,6,5] -> Output: [7,0,8]
Why: the lists read as 243 and 564 (reversed); 243 + 564 = 807, whose reversed digits are 7,0,8.

Input: first = [0], second = [0] -> Output: [0]
Why: minimal case, 0 + 0 = 0; the single zero node is preserved.

Input: first = [9,9,9], second = [1] -> Output: [0,0,0,1]
Why: 999 + 1 = 1000; a chain of carries lengthens the output beyond either input, testing the final trailing carry node.

Input: first = [5], second = [5,4] -> Output: [0,5]
Why: 5 + 45 = 50; the lists have different lengths, so the shorter one must be treated as ending in zeros.

Input: first = [1,8], second = [0] -> Output: [1,8]
Why: 81 + 0 = 81; adding zero leaves the number unchanged.

#### Approach

Elementary-school addition with a running carry. Because the digits are already stored least-significant-first, you can walk both lists front-to-back in lockstep. At each step add the two current digits (treating a missing/shorter list as contributing 0) plus the carry from the previous step; the new node's digit is sum % 10 and the carry passed forward is sum / 10. A dummy head node simplifies list construction so you never special-case the first node, and continuing the loop while a nonzero carry remains handles the case where the sum has more digits than either input. The pattern is single-pass linked-list simulation with a carry accumulator.

Complexity

Time: O(max(m, n)) where m and n are the list lengths — each node of each list is visited exactly once. Space: O(max(m, n)) for the newly built result list (O(1) auxiliary beyond the output).

Python
PYTHON
class ListNode:
    def __init__(self, val=0, next=None):
        self.val = val
        self.next = next

def add_two_numbers(first: ListNode, second: ListNode) -> ListNode:
    dummy = ListNode(0)
    curr = dummy
    carry = 0

    while first or second or carry:
        total = carry
        if first:
            total += first.val
            first = first.next
        if second:
            total += second.val
            second = second.next
        carry, digit = divmod(total, 10)
        curr.next = ListNode(digit)
        curr = curr.next

    return dummy.next
JavaScript
JAVASCRIPT
function ListNode(val, next) {
    this.val = val === undefined ? 0 : val;
    this.next = next === undefined ? null : next;
}

function addTwoNumbers(first, second) {

#### cons
t dummy = new ListNode(0);
    let curr = dummy;
    let carry = 0;

    while (first || second || carry) {
        let sum = carry;
        if (first)  { sum += first.val;  first  = first.next; }
        if (second) { sum += second.val; second = second.next; }
        carry = Math.floor(sum / 10);
        curr.next = new ListNode(sum % 10);
        curr = curr.next;
    }
    return dummy.next;
}
Java
JAVA
class ListNode {
    int val;
    ListNode next;
    ListNode(int x) { val = x; }
}

class Solution {
    public ListNode addTwoNumbers(ListNode first, ListNode second) {
        ListNode dummy = new ListNode(0);
        ListNode curr = dummy;
        int carry = 0;

        while (first != null || second != null || carry != 0) {
            int sum = carry;
            if (first != null)  { sum += first.val;  first  = first.next; }
            if (second != null) { sum += second.val; second = second.next; }
            carry = sum / 10;
            curr.next = new ListNode(sum % 10);
            curr = curr.next;
        }
        return dummy.next;
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
    ListNode* addTwoNumbers(ListNode* first, ListNode* second) {
        ListNode dummy(0);
        ListNode* curr = &dummy;
        int carry = 0;

        while (first || second || carry) {
            int sum = carry;
            if (first)  { sum += first->val;  first  = first->next; }
            if (second) { sum += second->val; second = second->next; }
            carry = sum / 10;
            curr->next = new ListNode(sum % 10);
            curr = curr->next;
        }
        return dummy.next;
    }
};
C
C
#include <stdlib.h>

struct ListNode {
    int val;
    struct ListNode* next;
};

struct ListNode* addTwoNumbers(struct ListNode* first, struct ListNode* second) {
    struct ListNode dummy;
    dummy.next = NULL;
    struct ListNode* curr = &dummy;
    int carry = 0;

    while (first || second || carry) {
        int sum = carry;
        if (first)  { sum += first->val;  first  = first->next; }
        if (second) { sum += second->val; second = second->next; }
        carry = sum / 10;

        struct ListNode* node = (struct ListNode*)malloc(sizeof(struct ListNode));
        node->val = sum % 10;
        node->next = NULL;
        curr->next = node;
        curr = node;
    }
    return dummy.next;
}

#### Follow-up
questions
What if the digits were stored in forward order (most significant first)? Would you reverse the lists, use stacks, or recurse?
Can you produce the result without mutating either input list?
How would you extend this to add more than two numbers, or a whole list of number-lists?
What changes if each node stores a digit in a base other than 10?

