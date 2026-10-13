---
type: "practice"
difficulty: "Other"
---

# Add Two Integers Stored as Most-Significant-First Digit Lists

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Linked List](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Linked_List.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given two non-empty singly linked lists. Each list encodes a single non-negative integer, storing exactly one decimal digit per node, with the most significant digit at the head of the list. For example, the list 6 -> 1 -> 5 represents the integer 615.

Add the two integers together and return their sum as a new singly linked list, using the same convention: one digit per node, most significant digit first.

Neither input list contains leading zeros, except for the integer zero itself, which is represented by a single node holding the value 0. Do not modify the input lists.

Input: the head pointers of two linked lists, listA and listB.

Output: the head pointer of a newly built linked list representing the sum.

#### Cons
traints:

The number of nodes in each list is in the range [1, 100].
Each node's value is a single digit in the range [0, 9].
Each input list represents an integer with no leading zeros (except the value 0).
The digits are ordered from most significant (head) to least significant (tail).

#### Example
s & test cases
TEXT
Input: listA = [6,1,5], listB = [4,2] -> Output: [6,5,7]
Why: 615 + 42 = 657; a normal case where the two numbers have different lengths.

Input: listA = [9,9,9], listB = [1] -> Output: [1,0,0,0]
Why: 999 + 1 = 1000; the carry propagates through every digit and the result is longer than either input.

Input: listA = [0], listB = [0] -> Output: [0]
Why: minimal case, both numbers are zero.

Input: listA = [5], listB = [5] -> Output: [1,0]
Why: single-digit addition that overflows into a new leading digit.

Input: listA = [7,3,4,2], listB = [8,5,6] -> Output: [8,1,9,8]
Why: 7342 + 856 = 8198; mixed-length addition with an interior carry.

#### Approach

Addition must start from the least significant digit, but the digits are stored most-significant-first, so we cannot walk the lists directly. Push every digit of each list onto its own stack; popping then yields digits from least significant to most significant. Repeatedly pop from both stacks (when available), add them plus the running carry, take the result digit modulo 10, and prepend a new node holding that digit to the output — prepending naturally rebuilds the answer in most-significant-first order. Continue while either stack is non-empty or a carry remains. This is the classic "reverse via stack" technique, which also solves the follow-up of not reversing the input lists.

Complexity

Time: O(m + n), where m and n are the lengths of the two lists — each digit is pushed once and popped once. Space: O(m + n) for the two stacks (the output list is unavoidable and not counted as auxiliary).

Python
PYTHON
class ListNode:
    def __init__(self, val=0, next=None):
        self.val = val
        self.next = next

class Solution:
    def addTwoNumbers(self, listA: ListNode, listB: ListNode) -> ListNode:
        stack_a, stack_b = [], []
        while listA:
            stack_a.append(listA.val)
            listA = listA.next
        while listB:
            stack_b.append(listB.val)
            listB = listB.next

        head = None
        carry = 0
        while stack_a or stack_b or carry:
            if stack_a:
                carry += stack_a.pop()
            if stack_b:
                carry += stack_b.pop()
            node = ListNode(carry % 10)
            node.next = head   # prepend so head remains most significant
            head = node
            carry //= 10
        return head
JavaScript
JAVASCRIPT
class ListNode {

#### cons
tructor(val = 0, next = null) {
        this.val = val;
        this.next = next;
    }
}

function addTwoNumbers(listA, listB) {

#### cons
t stackA = [], stackB = [];
    for (let p = listA; p !== null; p = p.next) stackA.push(p.val);
    for (let p = listB; p !== null; p = p.next) stackB.push(p.val);

    let head = null;
    let carry = 0;
    while (stackA.length || stackB.length || carry) {
        if (stackA.length) carry += stackA.pop();
        if (stackB.length) carry += stackB.pop();

#### cons
t node = new ListNode(carry % 10);
        node.next = head;   // prepend to keep most-significant digit first
        head = node;
        carry = Math.floor(carry / 10);
    }
    return head;
}
Java
JAVA
import java.util.ArrayDeque;
import java.util.Deque;

class ListNode {
    int val;
    ListNode next;
    ListNode(int x) { val = x; }
}

class Solution {
    public ListNode addTwoNumbers(ListNode listA, ListNode listB) {
        Deque<Integer> stackA = new ArrayDeque<>();
        Deque<Integer> stackB = new ArrayDeque<>();
        for (ListNode p = listA; p != null; p = p.next) stackA.push(p.val);
        for (ListNode p = listB; p != null; p = p.next) stackB.push(p.val);

        ListNode head = null;
        int carry = 0;
        while (!stackA.isEmpty() || !stackB.isEmpty() || carry > 0) {
            if (!stackA.isEmpty()) carry += stackA.pop();
            if (!stackB.isEmpty()) carry += stackB.pop();
            ListNode node = new ListNode(carry % 10);
            node.next = head;   // prepend to keep most-significant digit first
            head = node;
            carry /= 10;
        }
        return head;
    }
}
C++
CPP
#include <stack>
using namespace std;

struct ListNode {
    int val;
    ListNode* next;
    ListNode(int x) : val(x), next(nullptr) {}
};

class Solution {
public:
    ListNode* addTwoNumbers(ListNode* listA, ListNode* listB) {
        stack<int> stackA, stackB;
        for (ListNode* p = listA; p; p = p->next) stackA.push(p->val);
        for (ListNode* p = listB; p; p = p->next) stackB.push(p->val);

        ListNode* head = nullptr;
        int carry = 0;
        while (!stackA.empty() || !stackB.empty() || carry) {
            if (!stackA.empty()) { carry += stackA.top(); stackA.pop(); }
            if (!stackB.empty()) { carry += stackB.top(); stackB.pop(); }
            ListNode* node = new ListNode(carry % 10);
            node->next = head;   // prepend so head stays most significant
            head = node;
            carry /= 10;
        }
        return head;
    }
};
C
C
#include <stdlib.h>

struct ListNode {
    int val;
    struct ListNode *next;
};

/* Push both lists' digits into arrays used as stacks, then add from the
   least-significant end without reversing the input lists. */
struct ListNode* addTwoNumbers(struct ListNode* listA, struct ListNode* listB) {
    int stackA[128], stackB[128];
    int na = 0, nb = 0;

    for (struct ListNode* p = listA; p; p = p->next) stackA[na++] = p->val;
    for (struct ListNode* p = listB; p; p = p->next) stackB[nb++] = p->val;

    struct ListNode* head = NULL;
    int carry = 0;
    int i = na - 1, j = nb - 1;

    while (i >= 0 || j >= 0 || carry) {
        if (i >= 0) carry += stackA[i--];
        if (j >= 0) carry += stackB[j--];
        struct ListNode* node = (struct ListNode*)malloc(sizeof(struct ListNode));
        node->val = carry % 10;
        node->next = head;   /* prepend to keep most-significant digit first */
        head = node;
        carry /= 10;
    }
    return head;
}

#### Follow-up
questions
Could you solve it without using extra stacks and without reversing the input lists (for example, by computing lengths and using recursion)?
What if the digits were instead stored least-significant-first — how does the solution simplify?
If the lists could be millions of nodes long, how would the O(m+n) stack memory affect your design, and could you stream the result?
How would you handle numbers in an arbitrary base b instead of base 10?

