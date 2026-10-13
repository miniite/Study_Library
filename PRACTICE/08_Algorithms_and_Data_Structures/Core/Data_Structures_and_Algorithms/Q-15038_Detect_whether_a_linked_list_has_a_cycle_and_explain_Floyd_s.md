---
type: "practice"
difficulty: "Intermediate"
---

# Detect whether a linked list has a cycle and explain Floyd's tortoise-and-hare approach.

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Linked List Techniques](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Linked_List_Techniques.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

What is really being asked: Given the head of a singly linked list, determine whether it contains a cycle (i.e., some node's next pointer loops back to an earlier node so a traversal never terminates). The interviewer explicitly wants Floyd's tortoise-and-hare (the "two-pointer / fast-slow" algorithm) and, just as importantly, why it works — not just that it does.

What the interviewer is probing for at the intermediate level:

Do you know the O(1)-space solution, not just the "put nodes in a hash set" one?
Can you give a clean, bug-free implementation (null-pointer safety on the fast pointer is the classic trap)?
Can you explain the invariant — why two pointers moving at different speeds must meet inside a cycle — rather than reciting it?
Bonus: can you extend it to finding the start of the cycle and the cycle length?

#### Assumptions
/ clarifying questions I'd state:

Singly linked list, one next per node. A "cycle" means a next pointer that revisits an already-visited node; it need not point back to the head.
I'm allowed to read pointers but not modify the list (a destructive trick like reversing or marking visited also works but mutates data and isn't thread-safe, so I avoid it).
Node values are irrelevant; identity of the node objects is what matters.

Plan: (1) baseline hash-set solution to frame the trade-off, (2) the tortoise-and-hare loop with careful null checks, (3) the correctness argument, (4) the extension to find the cycle's entry node (Floyd's second phase), (5) complexity and pitfalls.

Step 1 — The baseline, so the trade-off is clear

The obvious solution: walk the list, insert each node (by reference/identity) into a hash set. If you ever encounter a node already in the set, there's a cycle; if you reach null, there isn't.

Time: O(n). Space: O(n) — that extra space is the whole reason we look for something better.

The interviewer wants us to beat the space bound.

Step 2 — Floyd's tortoise and hare

Use two pointers starting at the head. The slow pointer (tortoise) advances one node per step; the fast pointer (hare) advances two.

If the list is acyclic, fast (or fast.next) reaches null and we return false.
If there is a cycle, fast can never fall off the end — it's trapped going around the loop — so it eventually laps slow and they land on the same node. When slow == fast, return true.

The critical implementation detail is the loop guard. Because fast moves two steps, we must verify both fast != null and fast.next != null before dereferencing fast.next.next. Getting this wrong is the #1 way this "one-liner" throws a NullPointerException.

CODE
slow = head, fast = head
while fast != null and fast.next != null:
    slow = slow.next
    fast = fast.next.next
    if slow == fast: return true
return false

Space is O(1) (two pointers), time is O(n). We meet the interviewer's bar.

Step 3 — Why they must meet (the correctness argument)

This is the part that separates a memorized answer from an understood one.

Suppose the cycle exists and has length C. Once both pointers are inside the cycle, consider the gap — the number of steps ahead the fast pointer is relative to slow, measured going around the loop. Each iteration, slow moves +1 and fast moves +2, so fast gains exactly 1 on slow every step. The gap therefore decreases by 1 each iteration (modulo C). A quantity that strictly decreases by 1 each step, wrapping around a finite ring of size C, must hit 0 within at most C steps. Gap 0 means they're on the same node → they meet. Because fast closes the gap by 1 (not 2 or more), it can never "jump over" slow without landing on it.

If there is no cycle, fast simply reaches the end first (fast or fast.next becomes null) and we terminate — no meeting, correctly returning false.

Step 4 — Extension: where does the cycle start? (Floyd's phase 2)

A very common follow-up. Let the distance from head to the cycle's entry node be a, and let slow and fast first meet at distance b into the cycle (so b measured from the entry). When they meet:

slow has traveled a + b.
fast has traveled a + b + k·C for some integer k ≥ 1 (it went around k extra full loops).
fast traveled twice slow's distance: 2(a + b) = a + b + k·C ⇒ a + b = k·C ⇒ a = k·C − b.

k·C − b is exactly "the number of steps from the meeting point, going forward around the cycle, to reach the entry" (that's C − b for one lap, plus full laps). So: reset one pointer to the head, keep the other at the meeting point, and advance both one step at a time. They meet exactly at the cycle's entry node. That's the elegant a = distance-from-meeting-to-entry identity.

To get the cycle length, once you have a meeting point, keep one pointer fixed and walk the other until it returns — count the steps; that count is C.

Step 5 — Complexity, and why not the alternatives

#### Approach
Time	Space	Mutates list?
Hash set of visited nodes	O(n)	O(n)	No
Mark/visited flag on node	O(n)	O(1)	Yes (unsafe/concurrency-hostile)
Reverse-the-list trick	O(n)	O(1)	Yes
Floyd tortoise & hare	O(n)	O(1)	No

Floyd wins because it's O(1) space and non-destructive.

Summary — what a strong answer emphasizes

Lead with the two-pointer idea, write the loop with the correct fast != null && fast.next != null guard, and — crucially — explain the gap-closes-by-1 invariant so meeting is guaranteed, not hand-waved. Mention the O(1) space win over the hash set, and be ready with the phase-2 a = k·C − b derivation to locate the cycle's start and to compute its length.

#### Common mistakes
/ probes:

Dereferencing fast.next.next without the two-part null check → NPE on even-length acyclic lists.
Starting the pointers at different nodes (some start fast = head.next); it works but changes the meeting math — be consistent.
Claiming they "must eventually meet" without justifying why fast can't skip past slow.
Confusing "cycle detection" (meet-in-loop) with "cycle start" (needs the second phase from head).

**Comparison:**

```
Property	Value
Time complexity	O(n)
Space complexity	O(1)
Pointers used	2 (slow=+1, fast=+2)
Detection condition	slow == fast (same node reference)
No-cycle condition	fast == null or fast.next == null
Cycle-start identity	a = k·C − b → walk head & meeting-point +1 until equal
```

**Code:**

```java
// Detect a cycle — O(n) time, O(1) space
public boolean hasCycle(ListNode head) {
    ListNode slow = head, fast = head;
    while (fast != null && fast.next != null) {  // guard BOTH before .next.next
        slow = slow.next;         // tortoise: +1
        fast = fast.next.next;    // hare: +2
        if (slow == fast) return true;  // they met -> cycle
    }
    return false;                 // fast hit the end -> no cycle
}

// Follow-up: return the node where the cycle begins (or null)
public ListNode detectCycleStart(ListNode head) {
    ListNode slow = head, fast = head;
    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next.next;
        if (slow == fast) {                 // meeting point found
            ListNode p = head;
            while (p != slow) {             // a = k*C - b  =>  both walk +1 until they meet
                p = p.next;
                slow = slow.next;
            }
            return p;                       // cycle entry node
        }
    }
    return null;
}
```

**Example input / output:**

```

Dry run — list: 3 -> 2 -> 0 -> -4, where -4's next points back to node "2" (index 1). Cycle length C = 3 (nodes 2,0,-4).

CODE
Copy
step | slow (node) | fast (node)
init |     3       |     3
 1   |     2       |     0        (slow +1, fast +2)
 2   |     0       |     2        (fast wrapped past -4 -> 2)
 3   |    -4       |    -4   <-- slow == fast  => return true

Phase 2 (find start): reset p=head(3), keep slow=-4.

CODE
Copy
p: 3 -> 2       slow: -4 -> 2
p == slow at node "2"  => cycle starts at node with value 2.

Acyclic example: 1 -> 2 -> 3 -> null

CODE
Copy
init slow=1 fast=1; step1 slow=2 fast=3; step2 fast=3.next(null).next -> guard stops: fast.next==null
loop exits -> return false.
```

**Flow:**

1. No
2. Yes
3. Yes
4. No
5. slow=head, fast=head
6. fast != null AND fast.next !=null?
7. return false: no cycle
8. slow = slow.next; fast =fast.next.next
9. slow == fast?
10. return true: cycle detected

