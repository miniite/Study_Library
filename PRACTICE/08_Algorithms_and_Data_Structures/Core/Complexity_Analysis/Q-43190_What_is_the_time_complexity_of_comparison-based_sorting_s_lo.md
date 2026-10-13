---
type: "practice"
difficulty: "Deep"
---

# What is the time complexity of comparison-based sorting's lower bound and why can't it be faster than O(n log n)?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Sorting Linked Lists](<../../../../08_Algorithms_and_Data_Structures/Core/Complexity_Analysis/Sorting_Linked_Lists.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage
— what is really being asked

The interviewer wants two things: (1) state that the lower bound for comparison-based sorting is Ω(n log n) in the worst case (and on average), and (2) prove why — not just recite it. The probe is whether you understand that this bound comes from an information-theoretic / decision-tree argument, and whether you know the precise scope of the claim (it applies only to comparison-based sorts, which is why counting/radix sort can beat it). A strong candidate distinguishes "any algorithm that decides order purely by comparing pairs of elements" from algorithms that exploit the values themselves.

Clarifying assumptions I'd state up front:

We are sorting n elements using only comparisons like a < b, a <= b, a == b — no arithmetic on the keys, no using them as array indices.
"Faster" here means asymptotic worst-case (and average-case) comparison count. The bound is on comparisons, which lower-bounds total work.
The bound is a lower bound on the problem under the comparison model, not on one specific algorithm.
First principles — the decision-tree model

Model any comparison sort as a binary decision tree:

Each internal node is one comparison between two input elements (e.g. "is a[i] < a[j]?"). It has two outgoing edges: "yes" and "no".
The algorithm starts at the root and, based on each comparison's outcome, walks down one path.
Each leaf corresponds to one specific permutation of the input that the algorithm outputs as the sorted order.

Why binary? Each comparison yields one of two outcomes (ignoring ties, or folding == into one branch), so every node has at most 2 children. The length of the root-to-leaf path for a given input equals the number of comparisons the algorithm performs on that input. So the worst-case number of comparisons = the height h of the tree.

The counting argument

There are n distinct elements, so there are n! possible orderings (permutations) of the input. To sort correctly for every possible input, the algorithm must be able to produce every one of those n! permutations as an answer. Therefore the decision tree must have at least n! reachable leaves (each distinct correct output needs its own leaf; two different required permutations cannot share a leaf, because a leaf emits a fixed answer).

A binary tree of height h has at most 2^h leaves. To hold at least n! leaves we need:

CODE
2^h  >=  n!
=>  h  >=  log2(n!)

So the worst-case comparison count is at least log2(n!).

Turning log(n!) into n log n

Now bound log2(n!). Using Stirling's approximation, n! ≈ sqrt(2πn) · (n/e)^n, so:

CODE
log2(n!)  ≈  n·log2(n)  −  n·log2(e)  +  O(log n)
          =  Θ(n log n)

A cleaner elementary bound without Stirling: keep only the top half of the factorial's factors. In n! = n·(n−1)···2·1, the largest n/2 factors are each >= n/2. So:

CODE
n!  >=  (n/2)^(n/2)
log2(n!)  >=  (n/2)·log2(n/2)  =  (n/2)(log2 n − 1)  =  Ω(n log n)

Hence h = Ω(n log n). Any comparison sort must make Ω(n log n) comparisons in the worst case. The same argument gives the average case: average leaf depth in a binary tree with n! leaves is also Ω(log(n!)) = Ω(n log n), so you cannot even beat it on average.

This is a lower bound on the problem, so no cleverness within the comparison model escapes it. Mergesort and heapsort match it at O(n log n) worst case, so the bound is tight — Θ(n log n) is the true complexity of comparison sorting.

Why it "can't be faster" — the intuition

Sorting is really the task of identifying which of n! permutations the input is. Each yes/no comparison gives you at most 1 bit of information (it can at best halve the set of still-possible answers). You need enough bits to pin down one of n! possibilities, i.e. log2(n!) ≈ n log2 n bits, so you need at least that many comparisons. Fewer comparisons literally cannot distinguish all inputs — by pigeonhole, two different inputs requiring different orders would be forced into the same leaf, and one of them would be output wrong.

Scope — where the bound does NOT apply

The bound assumes the only operation on keys is comparison. Algorithms that use the key values directly (as array indices or digit buckets) get more than one bit per operation and are outside the model:

Counting sort: O(n + k) for integer keys in range [0, k).
Radix sort: O(d·(n + b)) for d digits, base b. These aren't "counterexamples" to the theorem; they simply don't obey its "comparisons only" premise. This scoping is exactly what a good answer should call out.
Summary — what a strong answer emphasizes
Lower bound: Ω(n log n) comparisons, worst and average case; matched by mergesort/heapsort ⇒ Θ(n log n) is tight.
Proof: model as a binary decision tree; correctness ⇒ ≥ n! leaves; height h ≥ log2(n!) = Θ(n log n) (Stirling, or the (n/2)^(n/2) bound).
Information-theoretic view: each comparison ≤ 1 bit; you need log2(n!) bits to select one permutation.
Model-bound: applies only to comparison sorts; counting/radix escape it by exploiting key values, not comparing them.

#### Common mistakes
to avoid: claiming quicksort is O(n log n) worst case (it's O(n²)); saying the bound applies to all sorting (it doesn't — only comparison-based); confusing this comparison lower bound with a lower bound on total operations for value-based sorts; forgetting that the average case is also Ω(n log n), not just the worst case.

#### Follow-up
probes you should be ready for: "Prove log(n!) = Θ(n log n) without Stirling" (the top-half-of-factors trick above). "Does randomization help?" (No — even randomized comparison sorts have expected Ω(n log n); the leaf-count argument holds for the expected depth). "Why is 2^h the max leaves?" (binary tree height h). "Give an algorithm that beats it and say why that's allowed" (radix sort — non-comparison model).

**Comparison:**

```
Algorithm	Model	Best	Average	Worst	Beats n log n?
Merge sort	comparison	O(n log n)	O(n log n)	O(n log n)	No (matches, tight)
Heap sort	comparison	O(n log n)	O(n log n)	O(n log n)	No (matches)
Quick sort	comparison	O(n log n)	O(n log n)	O(n^2)	No
Insertion sort	comparison	O(n)	O(n^2)	O(n^2)	No
Counting sort	value-based (index)	O(n+k)	O(n+k)	O(n+k)	Yes — outside comparison model
Radix sort	value-based (digits)	O(d(n+b))	O(d(n+b))	O(d(n+b))	Yes — outside comparison model
```

**Example input / output:**

```

Worked micro-example: sort 3 distinct elements a, b, c.

Number of permutations = 3! = 6, so the decision tree needs >= 6 leaves. Minimum height h with 2^h >= 6 -> 2^2 = 4 < 6, 2^3 = 8 >= 6 -> h >= 3. So worst case needs at least 3 comparisons for n = 3.

A valid 3-comparison decision tree:

CODE
Copy
                 a < b ?
              yes/       \no
           b < c ?        b < c ?   ... (mirror)
        yes/    \no
   [a,b,c]     a < c ?
             yes/   \no
        [a,c,b]     [c,a,b]

Trace input a=2, b=1, c=3 (true order b<a<c, i.e. [b,a,c]): a<b? 2<1 -> no -> go to right subtree b<c? 1<3 -> yes -> ... a<c? 2<3 -> yes -> leaf outputs [b,a,c] (correct in 3 comparisons)

log2(3!) = log2(6) ~= 2.585, rounded up = 3 comparisons — matches the bound.
```

**Flow:**

1. Sort n elements (comparisonmodel)
2. Must distinguish all n!permutations
3. Model algorithm as binarydecision treenode = 1 comparison, leaf = 1output permutation
4. Correctness => >= n!reachable leaves
5. Binary tree height h: leaves <=2^h=> 2^h >= n! => h >=log2(n!)
6. log2(n!) = Theta(n log n)(Stirling / (n/2)^(n/2))
7. Worst-case comparisons >=Omega(n log n)matched bymergesort/heapsort =>Theta(n log n)

