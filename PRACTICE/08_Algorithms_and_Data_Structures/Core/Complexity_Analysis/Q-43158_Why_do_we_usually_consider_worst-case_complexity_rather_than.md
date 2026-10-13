---
type: "practice"
difficulty: "Intermediate"
---

# Why do we usually consider worst-case complexity rather than best-case when comparing algorithms?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Big-O vs Constant Factors](<../../../../08_Algorithms_and_Data_Structures/Core/Complexity_Analysis/Big-O_vs_Constant_Factors.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

What the question is really asking: When we say "algorithm A is O(n log n)" and use that to pick it over another, we are almost always quoting the worst case. The interviewer wants to know whether you understand why the worst case is the default lens for comparison — and, importantly, whether you know when it is not the right lens (i.e., you don't over-apply the rule).

What the interviewer is probing: Do you understand the three canonical cases (best / average / worst)? Can you connect "which case to use" to real engineering concerns — SLAs, tail latency, adversarial inputs, and predictability? A junior answer says "worst case is safest." A strong intermediate answer explains the guarantee semantics, contrasts it with average-case, and names concrete situations where average-case or amortized analysis is actually the honest choice.

#### Assumptions
/ clarifying questions I'd state: I'll assume we're comparing algorithms for a production system where we care about correctness of the bound and about latency behavior, not a one-off script. I'd clarify: Is the input adversarial or trusted? Do we care about the single slowest call, or aggregate throughput over many calls? Those answers change whether worst-case, average-case, or amortized is the right yardstick.

Plan: Define the three cases, explain the property that makes worst-case attractive (it's an upper-bound guarantee), give the practical/engineering reasons, then honestly cover the cases where worst-case misleads and average/amortized wins.

From first principles

For a given input size n, an algorithm can run faster or slower depending on the specific input of that size. Complexity analysis captures this with three functions:

Best case — the fastest the algorithm can run for any input of size n. Example: linear search finds the target at index 0 → O(1).
Average case — the expected running time over some probability distribution of inputs. Example: linear search on a uniformly random position → about n/2 comparisons → O(n).
Worst case — the slowest the algorithm can run for any input of size n. Example: linear search where the target is absent or last → n comparisons → O(n).

Big-O by itself is just "asymptotic upper bound as n grows"; it says nothing about which input you feed it. So we must additionally pick a case. The default is worst case, and here is why.

Reason 1 — Worst case is a guarantee, the others are only hopes

The worst case is an unconditional promise: "no matter what input arrives, the algorithm will not take longer than this." When you design a system, you build against guarantees, not averages. If I tell you an operation is O(log n) worst case, you can size capacity and set timeouts with confidence. Average-case says "usually it's fine" — and "usually" is exactly what breaks in production at 2 a.m.

Reason 2 — SLAs, tail latency, and capacity planning are worst-case questions

Real systems commit to p99/p99.9 latency and to timeouts. A request that hits the pathological path is your tail. If a hash-map operation is O(1) average but O(n) worst, and one unlucky request degrades to O(n), that single request can blow the timeout and cascade (retries → thundering herd). Users experience the tail, not the mean. Sizing servers, thread pools, and deadlines is inherently a "what's the worst that can happen" exercise, so the worst-case bound is the number you actually plan around.

Reason 3 — Adversarial inputs make the worst case reachable on purpose

Average-case analysis assumes inputs are drawn from some benign distribution. But if an attacker chooses the input, they will steer you straight into the worst case. The classic example is a hash-collision DoS: hash tables are O(1) average, but if an attacker crafts keys that all collide, every insert/lookup degrades to O(n), and a table of n entries costs O(n²) to fill — a few KB of crafted request data can pin a CPU. Similarly, naive quicksort with a fixed pivot is O(n log n) average but O(n²) worst, and an attacker who knows the pivot rule can feed the killer input. On any system exposed to untrusted input, the "average" case is a fiction; the worst case is the real case.

Reason 4 — It makes comparison well-defined and pessimism-safe

Average-case analysis needs an assumed input distribution, and two engineers can pick different distributions and get different answers — it's not portable. Worst case needs no such assumption; it's a single, unambiguous, distribution-free number, which makes cross-algorithm comparison clean and reproducible. And being pessimistic is the safe direction to be wrong in: if the true behavior turns out better than the worst-case bound, you're pleasantly surprised; if you'd bet on average and hit worst, you're paged.

Why not best case (ever, for comparison)

Best case is almost useless for choosing an algorithm because it's trivially gameable and non-representative — nearly every algorithm has some lucky input. "Bubble sort is O(n) best case (already-sorted input)" tells you nothing about whether to ship bubble sort. Best case is only interesting as a lower-bound sanity check or to characterize early-exit optimizations.

When worst-case is the wrong lens (the nuance that makes an answer strong)

Blindly using worst-case can make you reject a genuinely better algorithm. Know these exceptions:

Average-case dominates in practice for trusted inputs. Quicksort's worst case is O(n²), yet it usually beats O(n log n) mergesort/heapsort on real hardware because of cache locality and low constants, and randomized pivoting makes the worst case astronomically unlikely. Hash tables are chosen everywhere despite O(n) worst case because average O(1) is what you get in practice. Here the honest metric is expected time.

Amortized analysis is the right tool when a single operation is occasionally expensive but the sequence is cheap. A dynamic array (ArrayList/vector) push_back is O(n) worst case on the resize, but amortized O(1) across many pushes because doubling spreads the cost. Judging it by the O(n) resize alone would be misleading. Amortized gives a per-operation guarantee over any sequence — stronger than average-case (no distribution assumed), and it's the correct comparison basis for such structures.

Randomized algorithms report expected worst case: randomized quicksort is O(n log n) expected regardless of input, because the randomness (not the input distribution) provides the guarantee. That expectation is a guarantee against adversaries, so it's a legitimate comparison basis.

The mature stance: default to worst case because it's the guarantee you can plan and defend against, but switch to amortized or expected/average analysis when it more honestly reflects the algorithm's behavior over realistic or adversary-proofed workloads — and always be explicit about which case you're quoting.

Summary — what a strong answer emphasizes
Worst case is the upper-bound guarantee — it holds for every input, so it's what you build SLAs, timeouts, and capacity against; averages are hopes, and users feel the tail.
It's distribution-free, so comparisons are unambiguous, and being pessimistic is the safe direction to err.
Adversarial/untrusted input makes the worst case reachable on demand (hash-collision DoS, quicksort killer inputs), so "average" isn't safe on public surfaces.
Best case is (almost) never used to compare — it's trivially gameable.
The nuance: amortized (dynamic array push) and expected/average (quicksort, hash tables, randomized algorithms) are sometimes the correct lens; a strong engineer names which case they're quoting and why.

#### Common mistakes
saying "worst case is always what matters" (ignoring amortized/expected), confusing average with amortized, or forgetting that adversarial inputs turn average into worst.

#### Follow-up
probes to expect: "Give an example where you'd deliberately prefer an algorithm with a worse worst case" (answer: quicksort/hash tables). "What's the difference between amortized and average-case?" (amortized makes no assumption about input distribution; it's a guarantee over any sequence).

**Comparison:**

```
Case	Meaning	Guarantee?	Needs input distribution?	Typical use
Best	Fastest possible for size n	No (lucky input)	No	Rarely used; characterizes early-exit
Average	Expected time over input distribution	No — only "usually"	Yes (assumption-dependent)	Trusted inputs, practical choice (quicksort, hashing)
Worst	Slowest possible for size n	Yes — holds for every input	No (distribution-free)	Default for comparison, SLAs, adversarial input
Amortized	Avg per op over any sequence	Yes — over a sequence	No	Dynamic array push, structures with rare costly ops
Structure/algorithm	Best	Average	Worst	Which case you actually rely on
Hash table lookup	O(1)	O(1)	O(n) (all collide)	Average in practice; worst on adversarial input
Quicksort	O(n log n)	O(n log n)	O(n²)	Average/expected (randomized pivot)
Mergesort	O(n log n)	O(n log n)	O(n log n)	Worst == average → predictable
Dynamic array push	O(1)	O(1)	O(n) (resize)	Amortized O(1)
Balanced BST (search)	O(log n)	O(log n)	O(log n)	Worst — guaranteed
```

**Example input / output:**

```

Linear search for target x in array of size n — the three cases:

CODE
Copy
Input : a = [x, ...], target = x            -> 1 comparison   -> BEST    O(1)
Input : a = [..., x] (x last) OR x absent    -> n comparisons  -> WORST   O(n)
Input : x uniformly random position          -> ~n/2 compares  -> AVERAGE O(n)

Quicksort (fixed first-element pivot) — why worst-case matters vs average:

CODE
Copy
Input : already-sorted [1,2,3,...,n]  -> partitions are size (n-1),(n-2),...
        -> sum of work ~ n + (n-1) + ... + 1 = n(n+1)/2  -> WORST  O(n^2)
Input : random permutation             -> balanced-ish splits
        -> recurrence T(n)=2T(n/2)+O(n) -> AVERAGE O(n log n)
An attacker who knows the pivot rule can FORCE the sorted input -> O(n^2).
```

**Flow:**

1. Adversarial / public API
2. Trusted
3. Yes
4. No
5. Yes
6. No, care about typicalthroughput
7. Choosing which case tocompare on
8. Input trusted or adversarial?
9. Use WORST caseattacker forces thepathological input
10. Single op occasionally costlybut sequence cheap?
11. Use AMORTIZEDe.g. dynamic array push
12. Need a hard guaranteeSLA / timeout / p99?
13. Use AVERAGE / EXPECTEDe.g. quicksort, hashing
14. Distribution-free, safe,plannable = default

