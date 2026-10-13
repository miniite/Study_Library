---
type: "practice"
difficulty: "Deep"
---

# What is the space complexity of reversing a linked list recursively versus iteratively?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Recursive vs Iterative Space](<../../../../08_Algorithms_and_Data_Structures/Core/Complexity_Analysis/Recursive_vs_Iterative_Space.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

What is really being asked: Compare the space cost of the two standard ways to reverse a singly linked list — recursive vs iterative — and explain why they differ. The crisp answer is recursive = O(n) auxiliary space, iterative = O(1) auxiliary space (both are O(n) time). At the "deep understanding" level the interviewer wants the mechanism behind the O(n): the recursion call stack. They may push on tail-call optimization, stack-overflow risk, and when the recursive version is nevertheless acceptable.

What's being probed:

Do you understand that recursion's space cost lives on the call stack, and that stack depth = recursion depth = n here?
Can you distinguish auxiliary space (extra memory the algorithm consumes) from the input/output list itself?
Do you know whether tail-call optimization (TCO) would help, and in which languages it actually applies?
Do you understand the practical consequence: stack overflow on long lists.

#### Assumptions
I'd state: singly linked list, reversing in place (reuse nodes). Space = auxiliary space (we don't count the n input nodes). "Recursive" means the natural head-recursion that recurses to the tail then rewires while unwinding.

Plan: define auxiliary space → analyze iterative → analyze recursive by counting stack frames → discuss TCO nuance → give the practical trade-off and verdict.

Iterative: O(1) auxiliary space

The iterative reversal keeps exactly three pointers — prev, curr, next — and rewires each node's next field in place as it walks the list once. That set of variables is fixed in size: it doesn't grow whether the list has 10 or 10 million nodes. No new nodes are allocated, and there's no recursion, so nothing accumulates. Constant scratch memory = O(1) auxiliary space. Time is O(n) (one pass).

Recursive: O(n) auxiliary space — and why

The recursive reversal is elegant: recurse to the end of the list, then on the way back up, make each node's successor point to it. But look at the memory. To reach the tail, the function calls itself once per node before any base case returns. Each pending call occupies a stack frame — storing the parameter (head/node reference), the return address, and any locals. For a list of n nodes, you have n frames stacked up simultaneously at the deepest point of the recursion, before the unwinding begins.

That simultaneous depth is the space cost. Recursion depth = n, each frame is O(1), so total auxiliary space is O(n). This is not the "n nodes of the list" (those are input) — it is genuinely extra memory the runtime spends managing the recursion. The iterative version has no such stack because it loops instead of recursing.

So the whole difference reduces to one idea: iterative reuses one loop frame; recursive spawns one frame per node.

The nuance: tail-call optimization

A natural follow-up: "Isn't the recursion tail-recursive, so couldn't the compiler make it O(1)?" Two things to say:

The naïve head recursion (reverse(head.next) first, then rewire) is not tail-recursive — the rewiring work happens after the recursive call returns, so the frame must be kept. As written, it's genuinely O(n).
You can rewrite it as a tail-recursive helper reverse(node, prev) where the recursive call is the last action. That is TCO-eligible in principle. But whether it actually becomes O(1) depends entirely on the language/runtime: Scheme, and some functional languages, guarantee TCO; the JVM and CPython do not perform it, so it stays O(n) stack in Java/Python regardless. A senior answer names this: "tail form is O(1) only if the runtime guarantees TCO — Java and Python don't, so in those it's still O(n) and can StackOverflow."
The practical consequence

O(n) stack space isn't just a theoretical footnote — it's a stack-overflow bug waiting to happen. Default stack sizes hold only tens of thousands of frames (Python's default recursion limit is ~1000; the JVM overflows around tens of thousands deep depending on frame size). Reverse a list of a million nodes recursively and it crashes; the iterative version handles it fine in O(1) space. That's why production code prefers the iterative reversal for unbounded input.

Verdict / trade-off
Both are O(n) time.
Space is the differentiator: iterative O(1), recursive O(n) (call stack).
Recursive code is arguably cleaner/more readable and fine for bounded, short lists.
For arbitrary-length input, choose iterative to avoid stack overflow.
	Iterative	Recursive (naïve)	Recursive (tail form)
Time	O(n)	O(n)	O(n)
Aux space	O(1)	O(n) stack	O(1) only with TCO, else O(n)
Stack-overflow risk	none	yes (deep lists)	none only with TCO
Readability	medium	high	medium
Summary — what a strong answer emphasizes

Lead with "same O(n) time; recursive is O(n) auxiliary space, iterative is O(1)", then immediately explain the why: the recursive version stacks n frames (depth = list length) before unwinding, whereas the iterative version reuses three pointers in a single loop frame. Elevate it by naming the stack-overflow risk on long lists and the TCO caveat (tail form helps only in runtimes that guarantee TCO — not JVM/CPython). Conclude with the engineering call: iterative for unbounded input.

#### Common mistakes
claiming recursive is O(1) "because it's in place" (in place refers to node reuse, not stack); claiming iterative is O(n) because there are n nodes (those are input); asserting TCO fixes it in Java/Python (it doesn't).

#### Follow-up
probes to expect: "What's the max list length your recursive version survives?" (roughly the runtime's stack-depth limit). "How would you make the recursion O(1) space?" (rewrite iteratively, or rely on guaranteed TCO). "Reverse in groups of k?" (iterative: O(n) time, O(1) space).

**Comparison:**

```
Metric	Iterative	Recursive (naïve)
Time	O(n)	O(n)
Auxiliary space	O(1) (3 pointers)	O(n) (n call-stack frames)
Space source	loop, reused frame	recursion depth = list length
Stack overflow on long list	No	Yes
Tail-call optimizable	n/a	naïve form: no; tail form: only with guaranteed TCO
```

**Code:**

```java
// ITERATIVE — O(n) time, O(1) auxiliary space
ListNode reverseIterative(ListNode head) {
    ListNode prev = null, curr = head;
    while (curr != null) {              // one loop frame reused
        ListNode next = curr.next;
        curr.next = prev;
        prev = curr;
        curr = next;
    }
    return prev;                        // three pointers, size independent of n
}

// RECURSIVE (naïve head recursion) — O(n) time, O(n) stack space
ListNode reverseRecursive(ListNode head) {
    if (head == null || head.next == null) return head; // base case at the tail
    ListNode newHead = reverseRecursive(head.next);     // n frames stack up here
    head.next.next = head;             // rewire while unwinding (work AFTER the call
    head.next = null;                  //  => NOT tail-recursive => frame must persist)
    return newHead;
}

// TAIL-RECURSIVE form — O(1) ONLY if the runtime guarantees TCO (JVM/CPython do NOT)
ListNode reverseTail(ListNode node, ListNode prev) {
    if (node == null) return prev;
    ListNode next = node.next;
    node.next = prev;
    return reverseTail(next, node);    // last action => TCO-eligible, but JVM ignores it
}
```

**Example input / output:**

```text
List: 1 -> 2 -> 3 -> 4 -> null   (n = 4)

RECURSIVE — stack depth at the moment the base case is hit (peak auxiliary space):
  frame reverse(1)
    frame reverse(2)
      frame reverse(3)
        frame reverse(4)
          frame reverse(null) -> returns          <- 5 pending frames ~ O(n)
  ... then unwind, rewiring 4->3, 3->2, 2->1
  Peak simultaneous frames grows with n  =>  O(n) space.

ITERATIVE — variables live at any instant: prev, curr, next  (always 3)
  independent of n  =>  O(1) space.

Both produce: 4 -> 3 -> 2 -> 1 -> null   in O(n) time.
```

