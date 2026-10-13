---
type: "practice"
difficulty: "Other"
---

# Splice a Second List into a Range of the First

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Two Pointers](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Two_Pointers.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given the head pointers of two singly linked lists, firstList and secondList, whose lengths are n and m respectively. You are also given two integer positions, start and end, using 0-based indexing.

Delete every node of firstList whose position lies in the inclusive range [start, end], and stitch the entire secondList into the gap that this deletion creates. In other words, the node that came just before position start must now point to the head of secondList, and the tail of secondList must point to whatever node originally followed position end.

Return the head of the resulting combined list.

Because the range [start, end] never includes the very first node (start >= 1) and never includes the very last node (end < n - 1), there is always a real node before the removed segment and a real node after it, so no head/tail edge handling is required.

#### Cons
traints
3 <= n <= 10^4 (length of firstList)
1 <= m <= 10^4 (length of secondList)
1 <= start <= end < n - 1
Each node value is an integer with 0 <= value <= 10^9.

#### Example
s & test cases
TEXT
Input: firstList = [7,4,8,2,9,3], start = 2, end = 3, secondList = [50,60,70] -> Output: [7,4,50,60,70,9,3]
Why: A normal splice — positions 2 and 3 (values 8 and 2) are removed and the whole second list drops into their place.

Input: firstList = [1,2,3,4,5,6,7], start = 1, end = 5, secondList = [100,200] -> Output: [1,100,200,7]
Why: The removed segment (5 nodes) is longer than the inserted list (2 nodes), so the result is shorter than the original.

Input: firstList = [9,9,9], start = 1, end = 1, secondList = [4] -> Output: [9,4,9]
Why: Minimal case with `start == end`; a single node is removed and a single node is inserted, with duplicate values around the seam.

Input: firstList = [0,5,10,15,20], start = 1, end = 3, secondList = [30,31,32,33,34] -> Output: [0,30,31,32,33,34,20]
Why: The inserted list is longer than the removed segment, so the result grows.

Input: firstList = [2,4,6,8,10], start = 3, end = 3, secondList = [11,12] -> Output: [2,4,6,11,12,10]
Why: Splice happens right before the last node, confirming the node after `end` is correctly reattached.

#### Approach

This is a pure pointer-relinking problem. Identify two anchor nodes: before, the node immediately preceding position start, and after, the node immediately following position end. Reach before with start - 1 steps from the head, then continue end - start + 2 steps to reach after. Point before.next at the head of the second list, walk the second list to its tail, and point that tail at after. The removed segment is now unreachable and the two lists are seamlessly joined. The underlying pattern is index-based traversal combined with in-place list surgery — the same "find-the-boundary-node-then-reconnect" technique used for reversing or removing sublists.

Complexity

Time: O(n + m). One linear walk over firstList to locate the two seam nodes (O(n) in the worst case), plus one walk to the end of secondList (O(m)). Space: O(1) — only a constant number of pointers are used; the two input lists are relinked in place with no extra allocation.

Python
PYTHON
class ListNode:
    def __init__(self, val=0, nxt=None):
        self.val = val
        self.next = nxt

def splice_lists(first, start, end, second):
    # Node just before the segment to remove.
    before = first
    for _ in range(start - 1):
        before = before.next

    # Node right after the removed segment (index end + 1).
    after = before
    for _ in range(end - start + 2):
        after = after.next

    before.next = second
    tail = second
    while tail.next:
        tail = tail.next
    tail.next = after

    return first
JavaScript
JAVASCRIPT
function ListNode(val, next) {
    this.val = val === undefined ? 0 : val;
    this.next = next === undefined ? null : next;
}

function spliceLists(first, start, end, second) {
    // Node just before the segment to remove.
    let before = first;
    for (let i = 0; i < start - 1; i++) before = before.next;

    // Node right after the removed segment (index end + 1).
    let after = before;
    for (let i = start - 1; i <= end; i++) after = after.next;

    before.next = second;
    let tail = second;
    while (tail.next) tail = tail.next;
    tail.next = after;

    return first;
}
Java
JAVA
class ListNode {
    int val;
    ListNode next;
    ListNode(int val) { this.val = val; }
}

class Solution {
    public ListNode spliceLists(ListNode first, int start, int end, ListNode second) {
        // Node just before the segment to remove.
        ListNode before = first;
        for (int i = 0; i < start - 1; i++) before = before.next;

        // Node right after the removed segment (index end + 1).
        ListNode after = before;
        for (int i = start - 1; i <= end; i++) after = after.next;

        before.next = second;
        ListNode tail = second;
        while (tail.next != null) tail = tail.next;
        tail.next = after;

        return first;
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
    ListNode* spliceLists(ListNode* first, int start, int end, ListNode* second) {
        // Node just before the segment to remove.
        ListNode* before = first;
        for (int i = 0; i < start - 1; ++i) before = before->next;

        // Node right after the removed segment (index end + 1).
        ListNode* after = before;
        for (int i = start - 1; i <= end; ++i) after = after->next;

        before->next = second;
        ListNode* tail = second;
        while (tail->next) tail = tail->next;
        tail->next = after;

        return first;
    }
};
C
C
#include <stdlib.h>

struct ListNode {
    int val;
    struct ListNode *next;
};

struct ListNode* spliceLists(struct ListNode* first, int start, int end,
                             struct ListNode* second) {
    /* Walk to the node just before position `start`. */
    struct ListNode* before = first;
    for (int i = 0; i < start - 1; i++) {
        before = before->next;
    }

    /* From `before` (index start-1), advance (end - start + 2) steps to reach
       the node that originally sat right after position `end`. */
    struct ListNode* after = before;
    for (int i = start - 1; i <= end; i++) {
        after = after->next;
    }

    /* Attach the second list, then link its tail to the node after `end`. */
    before->next = second;
    struct ListNode* tail = second;
    while (tail->next != NULL) {
        tail = tail->next;
    }
    tail->next = after;

    return first;
}

#### Follow-up
questions
How would you generalize the solution so that start could be the first node or end could be the last node? (Introduce a dummy head to remove the edge cases.)
What if secondList is allowed to be empty — how does the reconnection change so you simply delete the range?
Reverse the inserted segment before splicing it in without extra allocation.
Process firstList as a one-pass stream where you cannot revisit earlier nodes — can you still perform the splice?

