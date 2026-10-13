---
type: "practice"
difficulty: "Basic"
---

# If an algorithm is O(2n + 100), what is its Big-O and why?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Big-O vs Constant Factors](<../../../../08_Algorithms_and_Data_Structures/Core/Complexity_Analysis/Big-O_vs_Constant_Factors.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage
— what is really being asked

The question gives an algorithm whose exact operation count is 2n + 100 and asks: what is its Big-O, and why? The answer is O(n) — linear. But the interviewer is not looking for just the two-letter answer; they are probing whether I actually understand the definition of Big-O and the two rules that flow out of it:

Drop constant multipliers (the 2 in 2n).
Drop lower-order / additive constant terms (the +100).

This is a foundational, fresher-level "do you understand asymptotic notation" question. So I will (a) restate the definition of Big-O, (b) apply it to 2n + 100 from first principles, (c) justify why the constants disappear with a concrete numeric example, and (d) call out the common mistakes people make here.

Step 1 — What Big-O actually means

Big-O describes how the running time (or number of operations) of an algorithm grows as the input size n grows toward infinity. It is an upper bound on the growth rate, not an exact operation count and not a measurement in seconds. Seconds depend on the machine, the language, the CPU; Big-O deliberately ignores all of that so we can compare algorithms independent of hardware.

Formally: f(n) = O(g(n)) if there exist positive constants c and n0 such that f(n) ≤ c · g(n) for all n ≥ n0. In plain English: past some input size, f(n) never grows faster than a constant multiple of g(n). This definition is exactly what lets us throw away constants and low-order terms.

Step 2 — Apply it to 2n + 100

Our function is f(n) = 2n + 100. I claim f(n) = O(n), i.e. g(n) = n.

Rule A — drop the constant multiplier. The 2 in 2n is a fixed factor. Whether an algorithm does the work once per element or twice per element, the shape of the growth is the same straight line — doubling n still roughly doubles the work. Big-O measures the shape of growth, not the slope, so 2n and n are the same class: O(n).

Rule B — drop the additive constant. The +100 does not depend on n at all. It is fixed setup cost (say, initializing 100 buckets). As n grows, this fixed 100 becomes an ever-smaller share of the total and eventually irrelevant.

Putting both rules together, 2n + 100 reduces to O(n).

Formal check against the definition. Pick c = 3 and n0 = 100. For every n ≥ 100: 2n + 100 ≤ 2n + n = 3n = c·n. Since 100 ≤ n when n ≥ 100, the inequality holds. So 2n + 100 ≤ 3·n for all n ≥ 100, which satisfies the definition with g(n) = n. Therefore 2n + 100 = O(n), proven, not just asserted.

Step 3 — Why the constants genuinely stop mattering (numeric intuition)

The abstract argument becomes obvious with numbers. Watch what fraction of the total the +100 accounts for as n grows:

n	2n	+100	total 2n+100	share from the +100
10	20	100	120	83%
100	200	100	300	33%
1,000	2,000	100	2,100	4.8%
1,000,000	2,000,000	100	2,000,100	0.005%

At n = 10 the constant dominates, but Big-O is about the limit as n → ∞, and by a million elements the +100 is rounding error. The 2× factor, meanwhile, is just a fixed scaling of the line — it never changes linear into anything else. This is why we confidently call it linear, O(n).

Step 4 — Contrast so the boundary is clear
2n + 100 → O(n) (linear): constants and lower-order terms dropped.
2n² + 100n + 5 → O(n²): keep only the fastest-growing term n²; drop 100n, 5, and the 2.
100 (no n at all) → O(1) (constant): work independent of input size.

The general rule the interviewer wants me to state: keep only the single fastest-growing term and strip its coefficient. For 2n + 100, the fastest-growing term is 2n, so after stripping the coefficient we get n → O(n).

#### Common mistakes
/ follow-up probes
Writing O(2n + 100). Technically not wrong (it's a valid upper bound), but it is not simplified; an interviewer expects the reduced form O(n). Big-O by convention is always given in simplest terms.
Confusing "constants don't matter in Big-O" with "constants never matter in practice." They do matter in the real world — an O(n) algorithm with a huge constant can be slower than an O(n log n) one for realistic n. Big-O only compares growth rates as n → ∞.
Keeping the +100 "to be safe." No — it is a constant, dropped by definition.

#### Follow-up
they might ask: "What's the Big-Ω / Big-Θ?" Here it's also Θ(n) (tight bound), since 2n + 100 is bounded both above and below by constant multiples of n.
Summary — what a strong answer emphasizes

2n + 100 is O(n), linear. A strong answer states the two reduction rules (drop constant coefficients like the 2; drop lower-order/constant terms like the +100), grounds them in the formal definition (∃ c, n0 : f(n) ≤ c·g(n) — e.g. c=3, n0=100), and gives the numeric intuition that the +100 becomes a vanishing fraction of the total as n grows. Keep only the fastest-growing term, strip its coefficient — that is the whole skill this question tests.

**Comparison:**

```
n	2n	+100	total (2n+100)	fraction from +100
10	20	100	120	83%
100	200	100	300	33%
1,000	2,000	100	2,100	4.8%
1,000,000	2,000,000	100	2,000,100	0.005%
Expression	Big-O	Reason
2n + 100	O(n)	drop coefficient 2 and constant 100
2n² + 100n + 5	O(n²)	keep fastest-growing term only
100	O(1)	no dependence on n
```

**Example input / output:**

```text
Worked reduction of f(n) = 2n + 100:

Step 1 (drop lower-order constant term):  2n + 100  →  2n
Step 2 (drop constant coefficient):       2n        →  n
Result:                                   O(n)

Formal verification (definition of Big-O):
  Claim:  2n + 100 = O(n)
  Choose: c = 3, n0 = 100
  For all n ≥ 100:  2n + 100 ≤ 2n + n = 3n = c·n   ✓  (since 100 ≤ n)
  Therefore the definition f(n) ≤ c·g(n) for n ≥ n0 holds  →  O(n).
```

