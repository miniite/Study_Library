---
type: "practice"
difficulty: "Deep"
---

# What is the difference in complexity between iterative and recursive tree traversals in terms of space?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Recursive vs Iterative Space](<../../../../08_Algorithms_and_Data_Structures/Core/Complexity_Analysis/Recursive_vs_Iterative_Space.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

What is really being asked: Compare the space complexity of iterative vs recursive tree traversals. The interviewer wants me to demonstrate that "recursion vs iteration" isn't a wash on space — that recursion consumes the call stack, an iterative version consumes an explicit stack (or queue), and that the shape of the tree determines whether that's O(h), O(log n), or O(n). At "deep understanding" level, the good answer goes further: recursion is not automatically worse, Morris traversal achieves O(1), BFS space differs from DFS, and the real-world reason iteration exists (stack-overflow safety) matters.

What they're probing for:

Do I distinguish tree height h from node count n, and know when h = log n vs h = n?
Do I understand that the recursion call stack IS auxiliary space (candidates often forget this)?
Can I reason about BFS vs DFS space, and about the O(1) Morris trick?
Do I know the practical consequence: deep/skewed trees blow the native call stack (StackOverflowError), which is the main reason to prefer iteration.

Plan: Establish the space model → analyze recursive DFS → analyze iterative DFS → contrast with BFS → present Morris O(1) → give the balanced-vs-skewed number → summarize the trade-off and when to pick which.

The space model: what counts

Space complexity for a traversal = the auxiliary (extra) space beyond the tree itself and the output. There are three consumers:

Recursion call stack — one activation frame per active recursive call. This is real memory on the native/thread stack; it absolutely counts.
Explicit data structure — a Stack (for iterative DFS) or Queue (for BFS) that you allocate on the heap.
Pointer/link rewiring — Morris traversal temporarily mutates the tree's null right-pointers, using O(1) extra.

The headline result: recursive DFS and iterative DFS have the same asymptotic space complexity — O(h) where h is the tree height. Recursion doesn't make it worse asymptotically; it just moves the stack from the heap to the call stack. What differs is the constant factor and the failure mode.

Recursive DFS: O(h)

In an in-order/pre-order/post-order recursion, at the deepest point of the traversal there is one live stack frame for every ancestor from the root down to the current node. That chain of live frames is exactly the current path length, whose maximum is the tree height h. So the call stack peaks at O(h) frames.

Balanced tree: h = O(log n) → space O(log n) (~30 frames for a billion nodes). Cheap.
Skewed / degenerate tree (essentially a linked list): h = O(n) → space O(n). For a million-node skewed tree that's a million stack frames — and native thread stacks are ~1 MB (Java default ~512 KB), so this overflows long before a million. This StackOverflow risk is the deep-understanding punchline.
Iterative DFS: also O(h), but on the heap

The iterative version replaces recursion with an explicit Stack. You push nodes as you descend; the stack holds at most the current root-to-node path (plus at most one sibling per level, depending on the traversal). So peak stack size is O(h) — same asymptotic class as recursive.

Why bother, then? Two reasons: (a) the explicit stack lives on the heap, which is far larger and grows dynamically, so a skewed tree of depth 1,000,000 won't crash — it just uses O(n) heap. (b) You control it (pause/resume, custom ordering). The cost: more verbose code and, for post-order, fiddly state tracking. So iterative isn't asymptotically cheaper — it's safer for adversarial/deep inputs.

BFS (level-order): a different space profile

BFS uses a queue and its space is O(w) where w is the maximum width of the tree, not height. For a balanced binary tree the last level holds ~n/2 nodes → O(n) space. So BFS is worse on space than DFS for balanced trees (O(n) vs O(log n)) but better for skewed trees (a skewed tree has width 1 → BFS is O(1)-ish while DFS recursion is O(n)). This width-vs-height duality is exactly what a strong candidate contrasts.

Morris traversal: O(1) space

The deep-dive move: Morris in-order traversal achieves O(1) auxiliary space by temporarily creating threaded links — it rewires the null right-pointer of a subtree's rightmost node to point back to the in-order successor, walks in, then restores the pointer. No stack, no recursion. Time stays O(n) (each edge is traversed at most a constant number of times; the successor-finding is amortized O(1)). Cost: it mutates the tree during traversal (restored before finishing, but not thread-safe and not usable on immutable/concurrent structures). It's the answer to "can you do it in constant space?"

Putting a number on it
Tree shape (n nodes)	height h	Recursive DFS	Iterative DFS	BFS
Balanced	log n	O(log n) call stack	O(log n) heap stack	O(n) queue (widest level)
Skewed (list-like)	n	O(n) call stack ⇒ overflow risk	O(n) heap stack (safe)	O(1) queue

For a balanced billion-node tree: recursion ≈ 30 frames — trivial. For a skewed million-node tree: recursion ≈ a million frames ⇒ StackOverflowError; iterative uses ~a million heap slots but survives; Morris uses O(1).

Summary — what a strong answer emphasizes
Recursive and iterative DFS are both O(h) in space — recursion is not asymptotically worse; it just relocates the stack from heap to native call stack.
h is the swing factor: O(log n) for balanced trees, O(n) for skewed ones.
The real reason to prefer iterative is robustness: the native call stack is small and fixed (~1 MB), so deep/skewed inputs overflow it, whereas the heap stack scales — critical when the tree depth is attacker-controlled or unbounded.
DFS space depends on height; BFS space depends on width — pick based on tree shape and what you need (level info ⇒ BFS).
Morris is the O(1)-space DFS, trading pointer mutation for zero extra memory.

#### Common mistakes
saying recursion has "no extra space" (forgetting the call stack); claiming iterative is asymptotically cheaper (it isn't — same O(h)); confusing BFS width-space with DFS height-space; not knowing Morris. Follow-up probes: "Make it O(1)?" → Morris. "Why did prod crash on a linked-list-shaped tree?" → recursion depth = n → stack overflow → convert to iterative.

**Comparison:**

```
Traversal	Extra structure	Space complexity	Balanced tree	Skewed tree	Notes / risk
Recursive DFS	Call stack (native)	O(h)	O(log n)	O(n)	StackOverflow on deep trees
Iterative DFS	Explicit stack (heap)	O(h)	O(log n)	O(n)	Same class; survives deep trees
BFS level-order	Queue (heap)	O(w) = O(max width)	O(n)	O(1)	Space tied to width, not height
Morris in-order	None (thread pointers)	O(1)	O(1)	O(1)	Mutates tree; not thread-safe
Concept	Depends on	Value when balanced	Value when skewed
DFS space (h)	tree height	log n	n
BFS space (w)	max width	~n/2	1
```

**Code:**

```java
// 1) Recursive in-order DFS — space O(h) on the CALL STACK.
//    Balanced: O(log n). Skewed: O(n) -> StackOverflowError risk.
void inorderRecursive(TreeNode node, List<Integer> out) {
    if (node == null) return;
    inorderRecursive(node.left, out);   // frames stack up along the path
    out.add(node.val);
    inorderRecursive(node.right, out);
}

// 2) Iterative in-order DFS — space O(h) on an EXPLICIT HEAP STACK.
//    Same asymptotic space, but survives deep/skewed trees (heap, not thread stack).
List<Integer> inorderIterative(TreeNode root) {
    List<Integer> out = new ArrayList<>();
    Deque<TreeNode> stack = new ArrayDeque<>();
    TreeNode cur = root;
    while (cur != null || !stack.isEmpty()) {
        while (cur != null) { stack.push(cur); cur = cur.left; } // push the path
        cur = stack.pop();
        out.add(cur.val);
        cur = cur.right;
    }
    return out;
}

// 3) BFS level-order — space O(w) (max width). Balanced tree -> O(n).
List<Integer> bfs(TreeNode root) {
    List<Integer> out = new ArrayList<>();
    if (root == null) return out;
    Queue<TreeNode> q = new LinkedList<>();
    q.add(root);
    while (!q.isEmpty()) {
        TreeNode n = q.poll();
        out.add(n.val);
        if (n.left != null)  q.add(n.left);
        if (n.right != null) q.add(n.right);
    }
    return out;
}

// 4) Morris in-order — space O(1). Temporarily threads right-null pointers, then restores.
List<Integer> morrisInorder(TreeNode root) {
    List<Integer> out = new ArrayList<>();
    TreeNode cur = root;
    while (cur != null) {
        if (cur.left == null) { out.add(cur.val); cur = cur.right; }
        else {
            TreeNode pred = cur.left;
            while (pred.right != null && pred.right != cur) pred = pred.right;
            if (pred.right == null) { pred.right = cur; cur = cur.left; }   // thread
            else { pred.right = null; out.add(cur.val); cur = cur.right; }  // unthread
        }
    }
    return out;
}
```

**Example input / output:**

```

Tree used below:

CODE
Copy
        4
       / \
      2   6
     / \
    1   3

Recursive in-order dry run — peak call-stack depth = height = 3:

CODE
Copy
enter(4) -> enter(2) -> enter(1)      [3 live frames: 4,2,1  = path length]
  visit 1 ; return to 2 ; visit 2 ; enter(3) [frames: 4,2,3]
  visit 3 ; return to 4 ; visit 4 ; enter(6) [frames: 4,6]
  visit 6
output: 1 2 3 4 6      peak frames = 3 = h  ->  space O(h)

Skewed tree 1->2->3->...->1,000,000 (all right children):

CODE
Copy
Recursive: 1,000,000 nested frames  -> StackOverflowError (native stack ~1MB)
Iterative: heap Deque grows to ~1,000,000 entries -> completes, uses O(n) heap
Morris   : 0 extra frames                          -> completes, O(1) space

BFS on the balanced tree above:

CODE
Copy
queue peaks holding a full level; for a balanced n-node tree that is ~n/2 -> O(n)
output (level order): 4 2 6 1 3
```

**Flow:**

1. DFS
2. Yes, balanced
3. No, deep/skewedor attacker-controlled
4. Must be O(1) space
5. BFS / need levels
6. Need to traverse a tree —which space profile?
7. DFS or BFS?
8. Recursion depthbounded / tree shallow?
9. Recursive DFS: O(h)=O(log n)simplest code
10. Iterative DFS: O(h) on heapno stack overflow
11. Morris: O(1)but mutates tree
12. Queue: O(w)=O(max width)~O(n) for balanced

