---
type: "practice"
difficulty: "Intermediate"
---

# What is the time complexity of the naive recursive Fibonacci and why is it so bad?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Recursive vs Iterative Space](<../../../../08_Algorithms_and_Data_Structures/Core/Complexity_Analysis/Recursive_vs_Iterative_Space.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

The question is "What is the time complexity of the naive recursive Fibonacci and why is it so bad?" The expected answer is exponential — O(φⁿ) ≈ O(1.618ⁿ), commonly stated as O(2ⁿ) — and, crucially, the why: the recursion tree re-computes the same subproblems an exponential number of times (overlapping subproblems with no memoization). This is a two-part question: state the complexity precisely, then explain the mechanism (redundant recomputation) and how to fix it (memoization → O(n), bottom-up → O(n) time O(1) space, matrix/fast-doubling → O(log n)). A strong candidate derives the bound from the recurrence rather than just quoting "2ⁿ," and quantifies the waste. Let me work through it.

The naive algorithm and its recurrence
CODE
fib(n) = fib(n-1) + fib(n-2),  with fib(0)=0, fib(1)=1

Let T(n) be the number of operations (calls). Then:

CODE
T(n) = T(n-1) + T(n-2) + O(1)

This recurrence is itself the Fibonacci recurrence. Its solution grows like the golden ratio φ = (1+√5)/2 ≈ 1.618:

T(n) = Θ(φⁿ) ≈ Θ(1.618ⁿ), which is exponential. It is commonly (loosely) quoted as O(2ⁿ) as a simple upper bound.

A neat sanity check: the number of leaves in the recursion tree equals fib(n+1), and the total number of nodes (calls) is about 2·fib(n+1) − 1 — again exponential. So the algorithm makes on the order of φⁿ calls.

Why it is so bad: overlapping subproblems recomputed

The badness is not the branching per se — it's that the two branches overlap and recompute identical subproblems from scratch, exponentially many times. Draw the tree for fib(5):

CODE
                 fib(5)
             /            \
         fib(4)            fib(3)
        /      \          /      \
    fib(3)    fib(2)   fib(2)   fib(1)
    /   \     /   \    /   \
 fib(2) f1  f1   f0  f1   f0
 /  \
f1  f0

fib(3) is computed twice, fib(2) three times, fib(1) five times, and so on. As n grows, the count of times a small subproblem is recomputed itself grows like Fibonacci. There is no memory between branches — the left subtree's work is thrown away and redone in the right subtree. That redundancy compounds multiplicatively down the tree, producing the exponential blow-up. Concretely, fib(50) naively is ~2.7×10¹⁰ calls (seconds-to-minutes), and fib(100) is astronomically infeasible (~10²¹ calls).

Space, for contrast, is only O(n): at any instant the deepest chain of pending calls is n (going down the fib(n-1) side), so the call stack holds at most ~n frames. Exponential time, linear space — a classic and important distinction the interviewer may probe ("if it makes 2ⁿ calls, isn't space 2ⁿ?" → no, calls complete and pop; only the current path is live).

How to fix it (must include this)
Top-down memoization — cache each fib(k) the first time it's computed; subsequent hits are O(1). Each of the n distinct subproblems is solved once → O(n) time, O(n) space (cache + stack). This is the single most important insight: it turns exponential into linear by eliminating recomputation.
Bottom-up (tabulation), two variables — iterate 0..n keeping only the last two values → O(n) time, O(1) space. Best simple choice; no stack, no overflow.
Matrix exponentiation / fast doubling — using the identity [[1,1],[1,0]]ⁿ or the doubling identities with fast (binary) exponentiation → O(log n) arithmetic operations. Best asymptotically for huge n (ignoring big-integer cost).

The through-line: the exponential cost came purely from overlapping subproblems + no memoization, which is exactly the signature that says "use dynamic programming." Fibonacci is the canonical teaching example of DP for this reason.

Summary — what a strong answer emphasizes
Time = Θ(φⁿ) ≈ Θ(1.618ⁿ) (loosely O(2ⁿ)) — exponential.
Why bad: the recurrence T(n)=T(n-1)+T(n-2) grows like Fibonacci itself; the recursion tree has ~fib(n) leaves and recomputes the same subproblems exponentially many times (overlapping subproblems, no caching).
Space is only O(n) (max recursion depth) — don't confuse call count with simultaneous stack depth.
Fix: memoization or tabulation → O(n) time; matrix/fast-doubling → O(log n). This is the textbook motivation for dynamic programming.

#### Common mistakes
saying it's O(n) (that's the number of distinct values, not the naive cost); saying space is exponential; quoting O(2ⁿ) without knowing the tight bound is φⁿ; not naming overlapping subproblems as the root cause. Follow-ups: "Prove the exponential bound" (T(n) ≥ 2·T(n-2) ≥ 2^{n/2}, and ≤ 2ⁿ, so between 2^{n/2} and 2ⁿ, tight Θ(φⁿ)) · "Space of the naive version?" (O(n)) · "Reduce to O(1) space?" (two-variable loop) · "Best for very large n?" (fast doubling, O(log n)).

**Comparison:**

```
Approach	Time	Space	When to use
Naive recursion	Θ(φⁿ) ≈ Θ(1.618ⁿ) (exp)	O(n) stack	never in practice; teaching only
Top-down memoization	O(n)	O(n) (cache + stack)	recursive style, natural DP intro
Bottom-up, two variables	O(n)	O(1)	default; safe, no overflow
Matrix expo / fast doubling	O(log n)	O(1) / O(log n)	very large n
```

**Code:**

```java
// NAIVE: O(phi^n) ~ O(2^n) TIME (exponential), O(n) SPACE (max stack depth n)
int fib(int n) {
    if (n < 2) return n;
    return fib(n - 1) + fib(n - 2);   // recomputes overlapping subproblems every time
}

// MEMOIZED (top-down DP): O(n) TIME, O(n) SPACE — each subproblem solved once
int fibMemo(int n, Integer[] memo) {
    if (n < 2) return n;
    if (memo[n] != null) return memo[n];      // cache hit -> O(1)
    return memo[n] = fibMemo(n - 1, memo) + fibMemo(n - 2, memo);
}

// BOTTOM-UP two-variable: O(n) TIME, O(1) SPACE — best simple version, no stack
long fibIter(int n) {
    if (n < 2) return n;
    long a = 0, b = 1;
    for (int i = 2; i <= n; i++) { long c = a + b; a = b; b = c; }
    return b;
}
```

**Example input / output:**

```

Number of fib calls made by the NAIVE version (calls = 2*fib(n+1)-1) — this IS the exponential blow-up:

CODE
Copy
n     naive calls        distinct subproblems (what memo/iter does)
5     15                 6
10    177                11
20    ~21,891            21
30    ~2.69 million      31
40    ~331 million       41
50    ~40.7 billion      51

Same result fib(10)=55 either way — but naive did 177 calls vs. 11 with memoization. That gap (exponential vs. linear) is the whole point.
```

**Flow:**

1. Recursion tree for fib(5) — note the REPEATED nodes (recomputed work):
2. fib(5)
3. /        \
4. fib(4)        fib(3)      <-- fib(3) computed AGAIN here
5. /     \        /    \
6. fib(3)  fib(2)  fib(2)  fib(1)
7. /   \    /  \    /  \
8. fib(2) f1  f1  f0  f1  f0
9. /  \
10. f1  f0
11. fib(3): x2   fib(2): x3   fib(1): x5   -> recomputation grows like Fibonacci => exponential

