---
type: "practice"
difficulty: "Basic"
---

# Reverse a singly linked list iteratively and explain the pointer manipulation involved.

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Linked List Techniques](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Linked_List_Techniques.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

What the question is really asking: Reverse a singly linked list in place, iteratively, and clearly explain the three-pointer dance that flips each node's next pointer without losing the rest of the list. This is one of the most common warm-up interview questions; the real test is whether you can articulate the pointer manipulation precisely and avoid the classic "lost the tail" bug.

What the interviewer is probing for: Do you understand that reversing means reversing the direction of every link (not moving data)? Can you manage the pointers so you never orphan the remainder of the list? Can you state the invariant, handle edge cases (empty list, single node), and give O(n) time / O(1) space? Bonus: recognizing the recursive alternative and its O(n) stack cost.

#### Assumptions
/ clarifications I'd state: Singly linked list where each Node has a val and a next. I'll reverse it in place (rewire pointers, don't allocate a new list or copy values). Input may be empty (head == null) or a single node — both should return correctly. Target: O(n) time, O(1) extra space.

From first principles — why three pointers

To reverse, every node that currently points forward (to its successor) must instead point backward (to its predecessor). The danger: the moment you overwrite cur.next to point backward, you lose the reference to the rest of the list unless you saved it first. So we need to track three things at each step:

prev — the node that should become cur's new next (the part already reversed). Starts as null because the original head becomes the new tail, and a tail's next is null.
cur — the node we're currently reversing. Starts at head.
nxt — a temporary holding the original cur.next before we overwrite it, so we don't lose the unprocessed remainder.

The invariant: at the top of every loop iteration, everything from the original head up to (but not including) cur has already been reversed, and prev points to the head of that reversed portion.

The algorithm, step by step

For each node:

Save the next node: nxt = cur.next (so we can advance later without losing it).
Reverse the current link: cur.next = prev (point backward).
Advance prev: prev = cur (the node we just reversed is now the front of the reversed part).
Advance cur: cur = nxt (move to the node we saved).

Repeat until cur == null. At that point every link has been flipped, and prev points to what was the last node — the new head — so we return prev.

Order matters: you must save nxt (step 1) before overwriting cur.next (step 2), and you must move prev (step 3) before moving cur (step 4). Getting this order wrong is the #1 bug.

Dry run

List: 1 -> 2 -> 3 -> null.

Start: prev=null, cur=1.
Iter 1: nxt=2; 1.next=null; prev=1; cur=2. State: 1->null, remaining 2->3.
Iter 2: nxt=3; 2.next=1; prev=2; cur=3. Reversed so far: 2->1->null.
Iter 3: nxt=null; 3.next=2; prev=3; cur=null. Reversed: 3->2->1->null.
Loop ends (cur==null); return prev = node 3. Result: 3 -> 2 -> 1 -> null. Correct.
Edge cases
Empty list (head == null): loop body never runs, prev stays null, we return null. Correct.
Single node (1 -> null): iter 1 sets 1.next = null (unchanged), prev=1, cur=null; return 1. Correct.
Two nodes (1 -> 2): becomes 2 -> 1 -> null. Correct.

The same code handles all of these without special-casing — a sign the pointer logic is right.

Complexity
Time: O(n) — each of the n nodes is visited and rewired exactly once, constant work per node.
Space: O(1) — only three pointers (prev, cur, nxt), independent of list length. This is the key advantage of the iterative version.
Recursive alternative (mention it)

You can also reverse recursively: recurse to the end, then on the way back set head.next.next = head; head.next = null. It's elegant but uses O(n) stack space and can overflow on long lists, so the iterative O(1)-space version is preferred in interviews and production. Say this to show you know the trade-off.

Summary — what a strong answer emphasizes

Iterative reversal walks the list once with three pointers — prev (reversed part / eventual head, starts null), cur (current node, starts head), and nxt (saved successor). At each node you save cur.next, flip cur.next = prev, then slide prev = cur and cur = nxt; when cur hits null, prev is the new head. The load-bearing insight is saving nxt before overwriting the link so you never orphan the rest of the list, and starting prev = null so the old head correctly becomes the new tail. It's O(n) time, O(1) space, handles empty and single-node lists naturally, and beats the recursive version's O(n) stack usage.

#### Common mistakes
/ follow-ups: overwriting cur.next before saving nxt (loses the tail); initializing prev to head instead of null (creates a cycle / wrong tail); returning cur/head instead of prev; forgetting to set the old head's next to null. Typical follow-ups: "Do it recursively" (show the O(n)-stack version), "Reverse only nodes m..n" or "reverse in groups of k" (extends the same pointer discipline), and "How would you detect if reversal created a cycle?" (Floyd's tortoise-and-hare).

**Comparison:**

```
Approach	Time	Space	Notes
Iterative (three pointers)	O(n)	O(1)	Preferred; no recursion depth risk
Recursive	O(n)	O(n) stack	Elegant but can overflow on long lists
```

**Code:**

```

// Node definition class Node { int val; Node next; Node(int val) { this.val = val; } }

// Iterative in-place reversal: O(n) time, O(1) space. Node reverse(Node head) { Node prev = null; // reversed part; old head becomes new tail (next = null) Node cur = head; // node currently being reversed while (cur != null) { Node nxt = cur.next; // 1. save successor BEFORE we overwrite the link cur.next = prev; // 2. flip the pointer to face backward prev = cur; // 3. reversed front advances to current node cur = nxt; // 4. move on to the saved successor } return prev; // prev is the new head (old tail) }

CODE
Copy

```python
# Python equivalent
def reverse(head):
    prev, cur = None, head
    while cur:
        nxt = cur.next   # save successor
        cur.next = prev  # reverse link
        prev = cur       # advance prev
        cur = nxt        # advance cur
    return prev          # new head
```

**Example input / output:**

```text
Input : 1 -> 2 -> 3 -> null
Output: 3 -> 2 -> 1 -> null

Pointer trace (prev, cur, nxt) each iteration:
  init : prev=null cur=1
  it1  : nxt=2 ; 1.next=null ; prev=1 cur=2     -> 1->null
  it2  : nxt=3 ; 2.next=1    ; prev=2 cur=3     -> 2->1->null
  it3  : nxt=null;3.next=2   ; prev=3 cur=null  -> 3->2->1->null
  return prev = 3

Edge cases:
  Input: null       -> Output: null
  Input: 1 -> null  -> Output: 1 -> null
```

**Flow:**

1. No
2. Yes
3. prev = null, cur = head
4. cur != null?
5. return prev new head
6. nxt = cur.next save successor
7. cur.next = prev reverse link
8. prev = cur advance prev
9. cur = nxt advance cur

