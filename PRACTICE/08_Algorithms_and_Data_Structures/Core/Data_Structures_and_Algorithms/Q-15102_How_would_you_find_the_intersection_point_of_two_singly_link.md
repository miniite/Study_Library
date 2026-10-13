---
type: "practice"
difficulty: "Intermediate"
---

# How would you find the intersection point of two singly linked lists?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Linked List Techniques](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Linked_List_Techniques.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage
— what is really being asked

We have two singly linked lists that may merge: at some node they start sharing the same tail (same physical nodes, not just equal values). The task is to return the first common node — the intersection node — or null if they never merge. The interviewer is probing whether you understand the structure of a Y-shaped intersection, whether you can beat the brute-force O(m·n), and whether you handle the length-difference cleanly and edge cases correctly.

Key clarifications I'd state up front, because they define the problem:

Intersection means identity, not value. Two nodes intersect if they are the same object/reference (== on the node pointer), not if they hold equal data. This is critical — comparing values is a bug.
Because each list is singly linked and a node has only one next, once two lists share a node they share everything after it. You can't have a shape where they merge and then diverge again. So the intersection is a suffix — the classic "Y" shape.
Lists may be of different lengths before the merge; either may be empty; they may not intersect at all. Assume no cycles unless told otherwise.

Plan: give the brute-force baseline to frame the trade-off, then present two good solutions — the length-difference (align pointers) approach and the elegant two-pointer switch approach — both O(m+n) time, O(1) space. Then edge cases, why value comparison fails, and follow-ups (cycles, finding the length of the common part).

From first principles

Why the Y-shape matters. Draw it: list A has a unique nodes then a shared tail of c nodes; list B has b unique nodes then the same shared tail of c nodes. Total lengths are m = a + c and n = b + c. The intersection node is the first node of that shared tail. The whole difficulty is that a and b differ, so a naive "walk both from the head in lockstep" won't line up — the pointers reach the merge point at different steps.

Brute force (baseline). For each node in A, scan all of B looking for the same reference. O(m·n) time, O(1) space. Correct but slow; mention it to show you know the trade-off, then improve.

Hash-set approach. Put every node reference of A into a hash set, then walk B and return the first node already in the set. O(m+n) time but O(m) extra space. It's a valid answer and worth stating, but the interviewer usually wants O(1) space, so I'd present it as a stepping stone.

Optimal approach 1 — align by length difference. The Y-shape means the two shared tails have equal length c, so if we advance the pointer on the longer list by |m − n| first, both pointers are then equidistant from the intersection. After that, walk both one step at a time; the first place they point to the same node is the answer. Steps:

Compute lengths m and n (two passes).
Advance the head of the longer list by |m − n|.
Move both pointers together; return the node where pA == pB (or null when both hit the end).

Time O(m+n), space O(1). It's explicit and easy to justify.

Optimal approach 2 — two-pointer "switch heads" (the slick one). Start pA at head A, pB at head B. Advance both one node per step. When pA falls off the end of A, redirect it to head B; when pB falls off B, redirect it to head A. Each pointer now traverses a + c + b (or b + c + a) nodes total — the same count — so after at most m + n steps they arrive at the intersection node simultaneously. If there is no intersection, both become null at the same time and the loop ends returning null (this is why you must switch to the other head even when null, and check pA == pB including the null case).

Why the lengths equalize: pointer A walks a (rest of A) + b (all of B up to merge) + then reaches the shared node; pointer B walks b + a + then reaches it — both have covered a + b unique nodes before landing on the first shared node. Elegant, O(m+n) time, O(1) space, and only a handful of lines. This is the answer most interviewers hope to see, and I'd code this one while mentioning approach 1 as the intuition.

Correctness of the null case. If the lists don't intersect, after each pointer has traversed both lists (a + b steps for one, b + a for the other), both are simultaneously null. pA == pB is then null == null → true, loop exits, return null. That's why the switch must happen even at the end — it guarantees termination without an infinite loop.

Edge cases to call out
One or both lists empty → return null immediately (both approaches handle this since the loop never finds a match).
Intersection at the very first node (a = 0 or b = 0, lists fully overlap) → returned correctly; two-pointer finds it on the first alignment.
No intersection → return null; must not loop forever (handled by the null-switch).
Different lengths → the whole point; both approaches account for it.
Do NOT modify the lists or compare values — compare references. Comparing values would falsely "intersect" two lists that merely contain equal numbers.
Complexity
—	Time	Space
Brute force	O(m·n)	O(1)
Hash set	O(m+n)	O(m)
Length-diff align	O(m+n)	O(1)
Two-pointer switch	O(m+n)	O(1)
Summary — what a strong answer emphasizes

State that intersection = shared node identity and that singly-linked lists can only merge into a common suffix (Y-shape). Then jump to an O(m+n) time, O(1) space solution: either align by length difference or, better, the two-pointer switch-heads trick, and be able to explain why the pointers meet (each covers a+b unique nodes). Nail the null / no-intersection termination.

#### Common mistakes
comparing node values instead of references; forgetting the case where lists never intersect (infinite loop if you don't switch to the other head at the end); off-by-one when advancing the longer list; and assuming lists are equal length. Follow-up probes I'd be ready for: What if a list has a cycle? (use Floyd's cycle detection first, then reason about whether they intersect inside/outside the loop). Can you also return the length of the shared tail? (once aligned, count steps to the end). What if you're not allowed extra pointers? (the two-pointer method already uses O(1)).

**Comparison:**

```
Approach	Time	Space	When to use
Brute force (scan B per node of A)	O(m·n)	O(1)	never in practice; baseline only
Hash set of A's node refs	O(m+n)	O(m)	when extra space is fine / lists huge & random access
Length-difference alignment	O(m+n)	O(1)	clear, easy to explain
Two-pointer switch heads	O(m+n)	O(1)	preferred; shortest, elegant
```

**Code:**

```

class ListNode: def init(self, val=0, next=None): self.val = val self.next = next

--- Approach 2: two-pointer "switch heads" (O(m+n) time, O(1) space) ---

def get_intersection_node(headA, headB): if headA is None or headB is None: return None pA, pB = headA, headB # Loop ends either at the intersection node or at None (no intersection). while pA is not pB: pA = pA.next if pA else headB # fell off A -> jump to head B pB = pB.next if pB else headA # fell off B -> jump to head A return pA # intersection node, or None

--- Approach 1: align by length difference (explicit intuition) ---

def get_intersection_by_length(headA, headB): def length(node): n = 0 while node: n += 1 node = node.next return n

CODE
Copy
m, n = length(headA), length(headB)
a, b = headA, headB
# advance the longer list's pointer by the difference
for _ in range(m - n if m > n else 0):
    a = a.next
for _ in range(n - m if n > m else 0):
    b = b.next
# now equidistant from the merge point
while a is not b:
    a, b = a.next, b.next
return a   # intersection node, or None
CODE
Copy
```java
// Java — two-pointer switch (identical logic)
public ListNode getIntersectionNode(ListNode headA, ListNode headB) {
    if (headA == null || headB == null) return null;
    ListNode a = headA, b = headB;
    while (a != b) {
        a = (a == null) ? headB : a.next;
        b = (b == null) ? headA : b.next;
    }
    return a; // node or null
}
```

**Example input / output:**

```

Example 1 — they intersect:

CODE
Copy
List A: 4 -> 1 \
                8 -> 4 -> 5      (shared tail begins at node with val 8)
List B: 5 -> 6 -> 1 /

a = 2 (nodes 4,1), b = 3 (nodes 5,6,1), c = 3 (nodes 8,4,5)
m = a+c = 5, n = b+c = 6

Two-pointer trace (nodes by value; * marks the shared node object 8):
 step:  pA path: 4,1,8,4,5,None,5,6,1, *8   (jumps to head B after None)
        pB path: 5,6,1,8,4,5,None,4,1, *8   (jumps to head A after None)
 Both reach the SAME node object 8 at the same step -> return node(8).

Expected output: reference to the node with value 8 (the first shared node).

Example 2 — no intersection:

CODE
Copy
List A: 2 -> 6 -> 4
List B: 1 -> 5
pA eventually None, jumps to head B; pB None, jumps to head A;
after a+b steps both are None simultaneously -> pA == pB (both None) -> return None.

Expected output: None.

Example 3 — full overlap (intersection is the head):

CODE
Copy
List A: 3 -> 7 -> 9   (headA)
List B: ----------^   (headB == node 3, i.e. b = 0)
Two-pointer meets immediately at node(3). -> return node(3).
```

**Flow:**

1. yes
2. no
3. pA = headA, pB = headB
4. pA == pB ?
5. return pA (intersection node ornull)
6. pA = pA.next if pA else headB
7. pB = pB.next if pB else headA
8. ASCII of the Y-shape:
9. A: a1 -> a2 ---\
10. c1 -> c2 -> c3 -> null   (c1 = intersection node)
11. B: b1 -> b2 -> /

