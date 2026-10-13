---
type: "practice"
difficulty: "Basic"
---

# What is the space complexity of an algorithm that only uses a few variables regardless of input size?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Stack-based Complexity](<../../../../08_Algorithms_and_Data_Structures/Core/Complexity_Analysis/Stack-based_Complexity.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage
— what's really being asked

The question describes an algorithm that "only uses a few variables regardless of input size" and asks for its space complexity. The interviewer is testing whether you know (a) the definition of auxiliary space complexity, (b) that a fixed number of variables — no matter how large the input — is constant space, O(1), and (c) that O(1) does not mean "zero memory," it means "memory that does not grow with input size n." This is a fresher-level fundamental, so I'll define terms plainly and give clean examples of what counts and what doesn't.

Assumption/clarification I'd state: we usually mean auxiliary space — the extra memory the algorithm allocates beyond the input itself. The input array of size n occupies O(n), but if the algorithm doesn't allocate its own structures that scale with n, its auxiliary space is O(1).

Plan: define space complexity → apply the definition to "a few variables" → give the O(1) answer → contrast with O(n)/O(log n) so the concept is unambiguous → list the common traps.

From first principles

Space complexity measures how the extra memory an algorithm uses grows as the input size n grows. We express it in Big-O just like time.

If the algorithm declares a fixed set of variables — say a counter i, a running sum, a max, and a temporary temp — that's 4 variables whether n is 10 or 10 million. The memory footprint is a constant that does not depend on n. In Big-O terms, a constant is written O(1) — read "constant space" or "order one."

So the answer is: the space complexity is O(1) (constant space).

Why it's O(1) and not O(n)

The rule is about growth, not raw byte count. Four long variables use ~32 bytes; that's still O(1) because those 32 bytes stay 32 bytes no matter how big the input is. The moment memory usage would grow proportionally with n — for example if you allocated a new int[n] copy, or built a HashMap that ends up holding n keys — it would become O(n). "A few variables regardless of input size" is precisely the phrase that pins it to O(1).

The canonical O(1)-space pattern

The classic example is finding the maximum of an array:

CODE
max = a[0]
for i in 1..n-1:
    if a[i] > max: max = a[i]
return max

The loop reads n elements (that costs O(n) time) but only ever stores one variable, max (plus the loop index). No structure grows with n, so space is O(1) even though time is O(n). This decoupling — O(n) time, O(1) space — is exactly the point the question wants you to articulate: time and space are measured independently.

Making the concept sharp with contrasts
Algorithm behavior	Auxiliary space
A fixed handful of scalar variables (counter, sum, max)	O(1)
Swapping two array elements in place (bubble/selection sort's extra memory)	O(1)
Copying the input into a new array of size n	O(n)
Building a hash set/map that grows to ~n entries	O(n)
Recursion n levels deep (each call frame on the stack)	O(n)
Balanced recursion (binary search / balanced-tree) log n deep	O(log n)

Two subtle points a strong candidate raises:

Recursion consumes space too. Even if each recursive call declares only "a few variables," each pending call frame sits on the call stack. A recursion n deep is O(n) space, not O(1), because the stack grows with n. So "a few variables" is O(1) only for iterative code or recursion that isn't accumulating on the stack. (Tail-call-optimized languages can flatten this, but Java/Python generally don't.)

Input vs. auxiliary space. By convention we report auxiliary space and exclude the input storage. If the whole input is counted, everything is at least O(n) trivially, which isn't informative. So when we say an in-place algorithm is "O(1) space," we mean it adds only constant extra memory on top of the given input.

Summary — what a strong answer emphasizes

An algorithm that uses a fixed number of variables independent of n has O(1) — constant — space complexity. The core idea to voice: Big-O for space measures growth, not absolute bytes; if memory doesn't grow as the input grows, it's O(1), even if time is O(n). Give the max-of-array example (O(n) time, O(1) space) to prove you understand that time and space are separate axes.

#### Common mistakes
to avoid: (1) thinking O(1) means "no memory used" — it means constant, not zero; (2) forgetting that recursion adds O(depth) stack space, so a "few variables per call" recursive method can be O(n) space; (3) counting the input array against auxiliary space; (4) confusing this with time complexity. Follow-up probes to expect: "Is in-place reversal O(1) space?" (yes — swap with one temp); "What's the space of recursive factorial vs. an iterative one?" (O(n) vs O(1)); "Does storing a HashMap of size n change it?" (yes → O(n)).

**Comparison:**

```
Space complexity	Meaning	Example
O(1)	fixed memory, independent of n	sum/max/min over an array; in-place swap
O(log n)	memory grows with log of n	recursion of binary search (stack)
O(n)	memory grows linearly with n	copying input; hash set of n items
O(n²)	memory grows with square of n	building an n×n matrix
```

**Code:**

```java
// O(1) SPACE: only a fixed set of scalar variables, no matter how big 'a' is.
int findMax(int[] a) {
    int max = a[0];            // 1 variable
    for (int i = 1; i < a.length; i++) {  // 1 loop index
        if (a[i] > max) max = a[i];
    }
    return max;
    // Uses: max, i  -> constant count -> O(1) auxiliary space
    // (Time is O(n) because it reads all n elements.)
}

// CONTRAST -- O(n) SPACE: allocation grows with input size.
int[] copyArray(int[] a) {
    int[] b = new int[a.length];   // grows with n -> O(n) space
    for (int i = 0; i < a.length; i++) b[i] = a[i];
    return b;
}

// CONTRAST -- O(n) SPACE via recursion depth (n stacked call frames).
long factorial(int n) {
    if (n <= 1) return 1;
    return n * factorial(n - 1);   // call stack depth = n -> O(n) space
}
```

**Example input / output:**

```

Auxiliary memory used by findMax as input grows (constant — this is O(1)):

CODE
Copy
input size n = 10        -> variables stored: max, i  => 2 (constant)
input size n = 1,000     -> variables stored: max, i  => 2 (constant)
input size n = 1,000,000 -> variables stored: max, i  => 2 (constant)
=> memory does NOT grow with n  =>  O(1) space

Contrast — copyArray, memory grows with n (this is O(n)):

CODE
Copy
n = 10        -> allocates array of 10  => 10 slots
n = 1,000     -> allocates array of 1000 => 1000 slots
n = 1,000,000 -> allocates 1,000,000 slots
=> memory scales linearly with n  =>  O(n) space
```

**Flow:**

1. No: a fixed set of variables
2. Yes: array/map/recursionscales with n
3. Does extra memory grow asinput n grows?
4. Grows with n?
5. O(1) — constant space
6. O(n) / O(log n) / O(n²) ...

