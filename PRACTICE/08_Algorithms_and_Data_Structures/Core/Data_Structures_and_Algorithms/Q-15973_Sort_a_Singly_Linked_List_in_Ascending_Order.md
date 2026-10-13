---
type: "practice"
difficulty: "Other"
---

# Sort a Singly Linked List in Ascending Order

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Divide and Conquer](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Divide_and_Conquer.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given the first node (start) of a singly linked list. Each node stores an integer value and a single next reference to the following node; the final node's next is null. The list may be empty (start is null).

Reorder the nodes so the list is sorted in non-decreasing (ascending) order of value, and return the head of the sorted list. Sort by relinking the existing nodes, not by copying their values into a separate container.

You should aim for an efficient, general-purpose sort: O(n log n) time. Ideally, achieve this using only O(1) extra memory beyond the list itself (i.e., without relying on recursion depth or an auxiliary array proportional to the number of nodes).

Values may be negative and may repeat; equal values may appear in any order relative to one another as long as the overall sequence is non-decreasing.

Input

start: the head node of a singly linked list, or null for an empty list.

Output

The head node of the list after its nodes have been reordered into non-decreasing value order (null if the input was empty).

#### Cons
traints

The number of nodes in the list is in the range [0, 5 * 10^4].
Each node's value is in the range [-10^5, 10^5].
Target O(n log n) time; the constant-space variant avoids recursion and per-element auxiliary storage.

#### Example
s & test cases
TEXT
Input: values = [4, 2, 1, 3] -> Output: [1, 2, 3, 4]
Why: A typical unsorted list is arranged into ascending order.

Input: values = [-1, 5, 3, 4, 0] -> Output: [-1, 0, 3, 4, 5]
Why: Mixed negative and positive values sort by signed order; the smallest (-1) comes first.

Input: values = [] -> Output: []
Why: An empty list has nothing to sort and is returned as-is (edge case).

Input: values = [8] -> Output: [8]
Why: A single node is already sorted (minimum non-empty case).

Input: values = [2, 2, 1, 2, 1] -> Output: [1, 1, 2, 2, 2]
Why: Duplicates are handled; both 1s precede all the 2s.

Input: values = [9, 7, 5, 3, 1, -2] -> Output: [-2, 1, 3, 5, 7, 9]
Why: A fully reversed list confirms the algorithm globally reorders rather than doing a local pass.

#### Approach

Use bottom-up (iterative) merge sort on the linked list. First measure the list length. Then, for run widths of 1, 2, 4, ... up to (but not reaching) the length, sweep the list once per width: repeatedly detach a left run of width nodes and a right run of width nodes with a split helper (which also null-terminates each block), merge the two sorted runs by relinking nodes in non-decreasing order, and append the merged run's tail to the output being rebuilt behind a dummy head. Because each pass doubles the size of the sorted runs, after ceil(log2 n) passes the whole list is one sorted run. Merging with <= keeps equal elements in their prior relative order. Doing this iteratively (rather than the more common top-down recursive split via a slow/fast midpoint pointer) removes the O(log n) recursion stack, achieving true O(1) auxiliary space. Pattern: divide-and-conquer merge sort adapted to linked lists, implemented bottom-up.

Complexity

Time: O(n log n), where n is the number of nodes. The bottom-up merge sort performs log2(n) passes (the run width doubles each pass: 1, 2, 4, ...), and each pass splits and merges the entire list in O(n) work, giving O(n log n) total. Space: O(1) — the algorithm is iterative, so there is no recursion stack, and merging only relinks existing nodes using a constant number of pointers; no auxiliary array proportional to n is allocated. This meets the O(n log n) time / O(1) memory target.

Python
PYTHON
from typing import Optional, Tuple

class ListNode:
    def __init__(self, val=0, next=None):
        self.val = val
        self.next = next

class Solution:
    def sortList(self, start: Optional[ListNode]) -> Optional[ListNode]:
        def length(node: Optional[ListNode]) -> int:
            n = 0
            while node:
                n += 1
                node = node.next
            return n

        # Detach first k nodes; terminate them, return head of the rest.
        def split(node: Optional[ListNode], k: int) -> Optional[ListNode]:
            while k > 1 and node:
                node = node.next
                k -= 1
            rest = node.next if node else None
            if node:
                node.next = None
            return rest

        def merge(a: Optional[ListNode],
                  b: Optional[ListNode]) -> Tuple[Optional[ListNode], ListNode]:
            dummy = ListNode(0)
            tail = dummy
            while a and b:
                if a.val <= b.val:
                    tail.next, a = a, a.next
                else:
                    tail.next, b = b, b.next
                tail = tail.next
            tail.next = a if a else b
            while tail.next:
                tail = tail.next
            return dummy.next, tail

        total = length(start)
        dummy = ListNode(0, start)

        width = 1
        while width < total:
            curr = dummy.next
            tail = dummy
            while curr:
                left = curr
                right = split(left, width)
                curr = split(right, width)
                merged_head, merged_tail = merge(left, right)
                tail.next = merged_head
                tail = merged_tail
            width *= 2

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

function sortList(start) {

#### cons
t getLength = (node) => {
        let n = 0;
        while (node) { n++; node = node.next; }
        return n;
    };

    // Detach first k nodes; terminate them, return head of the rest.

#### cons
t split = (node, k) => {
        while (--k > 0 && node) node = node.next;

#### cons
t rest = node ? node.next : null;
        if (node) node.next = null;
        return rest;
    };

#### cons
t merge = (a, b) => {

#### cons
t dummy = new ListNode(0);
        let tail = dummy;
        while (a && b) {
            if (a.val <= b.val) { tail.next = a; a = a.next; }
            else                { tail.next = b; b = b.next; }
            tail = tail.next;
        }
        tail.next = a ? a : b;
        while (tail.next) tail = tail.next;
        return [dummy.next, tail];
    };

#### cons
t length = getLength(start);

#### cons
t dummy = new ListNode(0, start);

    for (let width = 1; width < length; width *= 2) {
        let curr = dummy.next;
        let tail = dummy;
        while (curr) {

#### cons
t left = curr;

#### cons
t right = split(left, width);
            curr = split(right, width);

#### cons
t [head, mergedTail] = merge(left, right);
            tail.next = head;
            tail = mergedTail;
        }
    }
    return dummy.next;
}
Java
JAVA
class ListNode {
    int val;
    ListNode next;
    ListNode(int x) { val = x; next = null; }
    ListNode(int x, ListNode n) { val = x; next = n; }
}

class Solution {
    public ListNode sortList(ListNode start) {
        int length = getLength(start);
        ListNode dummy = new ListNode(0, start);

        for (int width = 1; width < length; width *= 2) {
            ListNode curr = dummy.next;
            ListNode tail = dummy;
            while (curr != null) {
                ListNode left = curr;
                ListNode right = split(left, width);
                curr = split(right, width);
                ListNode[] merged = mergeTwo(left, right); // [head, tail]
                tail.next = merged[0];
                tail = merged[1];
            }
        }
        return dummy.next;
    }

    private int getLength(ListNode head) {
        int len = 0;
        for (ListNode n = head; n != null; n = n.next) ++len;
        return len;
    }

    // Detach first k nodes; terminate them, return head of the rest.
    private ListNode split(ListNode head, int k) {
        while (--k > 0 && head != null) head = head.next;
        ListNode rest = (head == null) ? null : head.next;
        if (head != null) head.next = null;
        return rest;
    }

    private ListNode[] mergeTwo(ListNode a, ListNode b) {
        ListNode dummy = new ListNode(0);
        ListNode tail = dummy;
        while (a != null && b != null) {
            if (a.val <= b.val) { tail.next = a; a = a.next; }
            else                { tail.next = b; b = b.next; }
            tail = tail.next;
        }
        tail.next = (a != null) ? a : b;
        while (tail.next != null) tail = tail.next;
        return new ListNode[] { dummy.next, tail };
    }
}
C++
CPP
#include <utility>

struct ListNode {
    int val;
    ListNode *next;
    ListNode(int x) : val(x), next(nullptr) {}
    ListNode(int x, ListNode* n) : val(x), next(n) {}
};

class Solution {
public:
    ListNode* sortList(ListNode* start) {

#### cons
t int length = getLength(start);
        ListNode dummy(0, start);

        for (int width = 1; width < length; width *= 2) {
            ListNode* curr = dummy.next;
            ListNode* tail = &dummy;
            while (curr) {
                ListNode* left = curr;
                ListNode* right = split(left, width);
                curr = split(right, width);
                auto [head, mergedTail] = mergeTwo(left, right);
                tail->next = head;
                tail = mergedTail;
            }
        }
        return dummy.next;
    }

private:
    int getLength(ListNode* head) {
        int len = 0;
        for (; head; head = head->next) ++len;
        return len;
    }

    // Detach first k nodes; terminate them, return head of the rest.
    ListNode* split(ListNode* head, int k) {
        while (--k > 0 && head) head = head->next;
        ListNode* rest = head ? head->next : nullptr;
        if (head) head->next = nullptr;
        return rest;
    }

    std::pair<ListNode*, ListNode*> mergeTwo(ListNode* a, ListNode* b) {
        ListNode dummy(0);
        ListNode* tail = &dummy;
        while (a && b) {
            if (a->val <= b->val) { tail->next = a; a = a->next; }
            else                  { tail->next = b; b = b->next; }
            tail = tail->next;
        }
        tail->next = a ? a : b;
        while (tail->next) tail = tail->next;
        return {dummy.next, tail};
    }
};
C
C
#include <stddef.h>

struct ListNode {
    int val;
    struct ListNode *next;
};

static int getLength(struct ListNode* head) {
    int len = 0;
    for (; head; head = head->next) ++len;
    return len;
}

/* Detach the first k nodes of `head`; terminate that block and
   return the head of the remaining list. */
static struct ListNode* split(struct ListNode* head, int k) {
    while (--k > 0 && head) head = head->next;
    struct ListNode* rest = head ? head->next : NULL;
    if (head) head->next = NULL;
    return rest;
}

/* Merge two sorted lists; return the merged head and set *tailOut
   to the last node of the merged run. */
static struct ListNode* mergeTwo(struct ListNode* a, struct ListNode* b,
                                 struct ListNode** tailOut) {
    struct ListNode dummy;
    dummy.next = NULL;
    struct ListNode* tail = &dummy;
    while (a && b) {
        if (a->val <= b->val) { tail->next = a; a = a->next; }
        else                  { tail->next = b; b = b->next; }
        tail = tail->next;
    }
    tail->next = a ? a : b;
    while (tail->next) tail = tail->next;
    *tailOut = tail;
    return dummy.next;
}

struct ListNode* sortList(struct ListNode* start) {
    int length = getLength(start);
    struct ListNode dummy;
    dummy.next = start;

    for (int width = 1; width < length; width *= 2) {
        struct ListNode* curr = dummy.next;
        struct ListNode* tail = &dummy;
        while (curr) {
            struct ListNode* left = curr;
            struct ListNode* right = split(left, width);
            curr = split(right, width);
            struct ListNode* mergedTail;
            struct ListNode* mergedHead = mergeTwo(left, right, &mergedTail);
            tail->next = mergedHead;
            tail = mergedTail;
        }
    }
    return dummy.next;
}

#### Follow-up
questions
Explain precisely why bottom-up merge sort reaches O(1) space while the recursive top-down version uses O(log n) stack space, and when the recursive version is acceptable.
Sort the list in descending order, or make the merge stable and explain the role of <= versus <.
Contrast merge sort with quicksort on a linked list: why is merge sort usually preferred for lists (no random access, guaranteed O(n log n), no partition-around-pivot pointer juggling)?
Generalize to sort by a custom comparator or to sort a k-way set of already-sorted lists, and describe how the merge step scales.

