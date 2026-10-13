---
type: "practice"
difficulty: "Intermediate"
---

# Given a linked list, remove the nth node from the end in a single pass.

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Linked List Techniques](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Linked_List_Techniques.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

What's really being asked: Given a singly linked list, remove the node that is n-th from the end, and do it in a single pass (one traversal, O(n) time, O(1) extra space). The naive solution is two passes: pass one to count the length L, pass two to walk to node (L - n) and unlink. The "single pass" constraint is the whole point — the interviewer is probing whether I know the two-pointer / fixed-gap (fast–slow) technique and whether I handle the edge case of removing the head cleanly.

#### Assumptions
/ clarifying questions I'd state: (1) It's a singly linked list (no prev pointers). (2) n is valid, i.e., 1 ≤ n ≤ L. I'll ask what to do if n could exceed the length — I'll assume valid, but mention a guard. (3) "n-th from the end" is 1-indexed: n = 1 means remove the last node; n = L means remove the head. (4) Do I return the (possibly new) head? Yes — because removing the head changes it. (5) Is memory management relevant (C/C++ free)? I'll note it.

Plan: Use two pointers with a dummy head to uniformly handle the head-removal case. Advance a fast pointer n steps ahead, then move fast and slow together until fast hits the end; slow then sits just before the node to delete. Unlink and return dummy.next.

From first principles — why two pointers gives one pass

The difficulty is that a singly linked list has no random access and no back-pointer, so "n-th from the end" isn't directly reachable without knowing the length. The trick is to convert "distance from the end" into "distance from a scout that is already n ahead."

Key insight: if I put a fast pointer n nodes ahead of a slow pointer, then advance both one step at a time until fast reaches the end, the slow pointer will be exactly n nodes from the end — because the gap between them stays constant at n. To delete the n-th-from-end node, I actually want slow to stop one node before it, so I can relink slow.next = slow.next.next. I achieve that by anchoring slow at a dummy node placed before the head and stopping when fast becomes null (i.e., fast has walked off the last real node).

Step by step
Create a dummy node whose next is head. This is the single most important trick: it makes removing the real head identical to removing any interior node, so I don't need a special-case branch. slow starts at dummy.
Advance fast n steps from the head. (Equivalently, n + 1 steps from dummy — I'll start fast at dummy and step n + 1 times, or start fast at head and step n times; either is fine as long as the gap between slow=dummy and fast is n + 1 node-links.) I'll use: slow = dummy, fast = dummy, then step fast forward n + 1 times.
Move slow and fast together until fast is null. Now slow points to the node immediately before the target.
Unlink: slow.next = slow.next.next (and free the removed node in languages with manual memory).
Return dummy.next — the possibly-updated head.
Why the dummy matters (the edge case)

Without the dummy, removing the head (n = L) is a special case: there's no node before the head to relink from, so you'd have to write if target is head: return head.next. The dummy node is that "before the head" node, so the same slow.next = slow.next.next line works whether we delete the head, a middle node, or the tail. This single design choice eliminates a whole class of bugs and is exactly what interviewers look for.

Complexity
Time: O(L) — one traversal; fast visits each node once. Truly a single pass.
Space: O(1) — two pointers plus a dummy; no counting array, no recursion stack.
Correctness argument

When fast starts n + 1 links ahead of slow (with slow at dummy), the invariant "number of links between slow and fast = n + 1" holds through every joint step. fast becomes null after exactly L - n joint steps (it walks from position n+1 off the end at position L+1 counting the dummy as 0). At that moment slow has advanced L - n steps from dummy, landing on the node at 1-indexed position L - n — precisely the predecessor of the n-th-from-end node at position L - n + 1. Relinking removes exactly the intended node.

Summary — what a strong answer emphasizes

"Two pointers with a dummy head. Advance fast n nodes ahead, then move both together until fast falls off the end; slow lands one before the target, so slow.next = slow.next.next deletes it. The dummy makes head removal a non-special case. O(L) time, one pass, O(1) space." The two things that separate a strong answer from a shaky one are (a) the dummy node for the head case and (b) getting the off-by-one gap right so slow stops before the target, not on it.

#### Common mistakes
off-by-one in the fast-advance loop (stopping slow on the target instead of before it); not handling head removal (crashes or wrong head when n = L); dereferencing null if n > L (add a guard); and forgetting to return the possibly-new head. Follow-up probes: "What if n can exceed the length?" (guard: if fast is null after advancing, n > L → error or no-op). "Do it for a doubly linked list?" (still one pass; you can splice via prev). "Free the memory in C." "Can you do it recursively?" (yes, but that costs O(L) stack — contradicts O(1) space).

**Comparison:**

```
Approach	Passes	Time	Extra space	Handles head cleanly	Notes
Count length, then walk to L-n	2	O(L)	O(1)	needs special-case	simplest but two passes
Two pointers + dummy (chosen)	1	O(L)	O(1)	yes (dummy)	meets single-pass requirement
Recursion (unwind counts from end)	1 logical	O(L)	O(L) stack	yes	violates O(1) space
Store nodes in an array/deque	1	O(L)	O(L)	yes	wastes memory
```

**Code:**

```

class ListNode: def init(self, val=0, next=None): self.val = val self.next = next

def remove_nth_from_end(head: ListNode, n: int) -> ListNode: dummy = ListNode(0, head) # sentinel before head -> uniform head handling slow = fast = dummy

CODE
Copy
# Advance fast n+1 links so the gap between slow and fast is n+1.
for _ in range(n + 1):
    if fast is None:           # guard: n > length -> invalid, bail out
        return head
    fast = fast.next

# Move together; when fast is None, slow is the node BEFORE the target.
while fast is not None:
    slow = slow.next
    fast = fast.next

slow.next = slow.next.next     # unlink the n-th-from-end node
return dummy.next              # possibly-new head
CODE
Copy
```java
// Java equivalent
public ListNode removeNthFromEnd(ListNode head, int n) {
    ListNode dummy = new ListNode(0);
    dummy.next = head;
    ListNode slow = dummy, fast = dummy;
    for (int i = 0; i <= n; i++) {     // advance fast n+1 times
        if (fast == null) return head; // n > length guard
        fast = fast.next;
    }
    while (fast != null) { slow = slow.next; fast = fast.next; }
    slow.next = slow.next.next;         // unlink
    return dummy.next;
}
```

**Example input / output:**

```text
Example 1 — remove a middle node
Input:  1 -> 2 -> 3 -> 4 -> 5,  n = 2
Target: node 4 (2nd from end)
Output: 1 -> 2 -> 3 -> 5

Dry run (dummy D -> 1 -> 2 -> 3 -> 4 -> 5), n=2:
  advance fast n+1=3 links from D: fast -> 3   (slow=D)
  step together:
    slow=1 fast=4
    slow=2 fast=5
    slow=3 fast=null  -> stop
  slow=3, slow.next=4 -> slow.next = 5  => 1->2->3->5   ✓

Example 2 — remove the HEAD (n == length): why dummy matters
Input:  1 -> 2,  n = 2   (L = 2, remove head)
  advance fast 3 links: D->1->2->null, fast=null after 3 steps? D(0)->1(1)->2(2)->null(3): fast=null
  step-together loop doesn't run (fast already null)
  slow=D, slow.next=1 -> slow.next=2  => head becomes 2
Output: 2                                          ✓

Example 3 — remove the last node
Input:  1 -> 2 -> 3,  n = 1
Output: 1 -> 2                                     ✓

Example 4 — single node
Input:  1,  n = 1
Output: (empty list / null)                        ✓
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. dummy -> head;slow=fast=dummy
6. advance fast n+1 steps
7. fast == null during advance?
8. n > length: invalid, return headunchanged
9. fast != null?
10. slow = slow.next; fast =fast.next
11. slow.next = slow.next.next(unlink)
12. return dummy.next

