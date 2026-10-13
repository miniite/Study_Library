---
type: "practice"
difficulty: "Deep"
---

# Probe: which sorting algorithm does your language's built-in sort use, and why might it matter?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Sorting Algorithms](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Sorting_Algorithms.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

What's really being asked: This is a "do you know what's under the hood" probe. The interviewer wants to know (a) which algorithm your language's standard-library sort actually uses, (b) why language designers picked it, and (c) when that choice becomes something you must reason about in production code — stability, worst-case behaviour, comparator cost, and adversarial inputs. It is a "deep understanding" DSA question, so the strong answer connects a concrete implementation fact to real engineering consequences.

What they're probing: Whether I understand that "just call .sort()" hides trade-offs (comparison vs non-comparison sorts, stability, O(n log n) average vs O(n²) worst case, cache behaviour, memory), and whether I can name failure modes: comparator that isn't a strict weak ordering, quadratic blow-up on crafted input, or a lost stable order that breaks a multi-key sort.

#### Assumptions
/ clarifying questions I'd raise: "Which language and which runtime version?" matters because the answer differs across V8 (Node/browser JS), Java, Python, C++, Go, and .NET, and some have changed algorithms across versions. I'll cover the major ones and then generalise.

From first principles: the families of sorts

A comparison sort only ever compares two elements; its information-theoretic lower bound is Ω(n log n) comparisons. Quicksort, mergesort, heapsort and their hybrids live here. A non-comparison sort (counting, radix, bucket) exploits key structure to hit O(n + k) but needs bounded integer-like keys. Standard library sort on arbitrary objects with a comparator must be a comparison sort, so O(n log n) is the target, and the real design question is which O(n log n) algorithm and how it degrades.

Two properties dominate the design choice:

Stability — equal elements keep their input order. This is essential for multi-key sorts ("sort by name, then stably by date").
Worst-case guarantee — pure quicksort is O(n²) on bad pivots; that is a denial-of-service risk if input is attacker-controlled.
What each standard library actually uses
JavaScript (V8, Node 12+ / modern browsers): Array.prototype.sort is TimSort — an adaptive, stable, hybrid of mergesort and insertion sort. Before ES2019 the spec did not require stability and V8 historically used an unstable insertion/quicksort mix for smaller arrays, which is exactly why old code that relied on stability broke across engines. ES2019 mandated stability.
Java: Arrays.sort(Object[]) and Collections.sort use TimSort (stable). Arrays.sort(int[]) on primitives uses a dual-pivot quicksort (Vladimir Yaroslavskiy) — not stable, because primitive equality makes stability meaningless, and quicksort is more cache-friendly with less memory.
Python: list.sort() and sorted() use TimSort (its birthplace), stable, worst case O(n log n).
C++: std::sort is typically introsort — quicksort that switches to heapsort when recursion depth exceeds ~2·log n (guaranteeing O(n log n) worst case) and to insertion sort for small runs. It is not stable. std::stable_sort is a separate, stable (merge-based) call.
Go: sort.Sort uses an introsort-style pdqsort (pattern-defeating quicksort) — not guaranteed stable; sort.Stable for stability. slices.Sort (generics) is pdqsort.
Rust: slice::sort is stable (TimSort-like); sort_unstable is pdqsort and is faster.
Why it matters — the engineering consequences

1. Stability changes correctness of multi-key sorts. If I sort records by city and then by lastLogin expecting the earlier city order preserved, an unstable sort silently reorders ties. Multi-key sorting relies on: sort by least-significant key first with a stable sort. On V8 pre-2019 or on Java primitive/std::sort, that pattern is a real bug.

2. Worst-case / adversarial input. Pure quicksort is O(n²) on inputs crafted to force bad pivots — a genuine algorithmic-complexity DoS if the data comes from users. This is why C++ chose introsort (heapsort fallback caps it) and Java added dual-pivot with defences. TimSort and introsort both guarantee O(n log n). Java's TimSort even had a famous 2015 bug where a broken invariant could throw ArrayIndexOutOfBoundsException, later patched — a reminder that "the library" is still code.

3. Comparator contract. A comparator must be a strict weak ordering (consistent, transitive, cmp(a,b) and cmp(b,a) agree). A subtly broken comparator (e.g. a - b on ints that overflows, or returning inconsistent results) yields undefined behaviour: Java throws IllegalArgumentException: Comparison method violates its general contract; C++ std::sort can read out of bounds. So the classic JS gotcha arr.sort() sorts lexicographically by string by default — [10, 2, 1].sort() → [1, 10, 2] — you must pass (a,b) => a-b.

4. Comparator cost dominates. TimSort minimises comparisons on partially-ordered or already-sorted data (adaptive, near-O(n) on runs), which is huge when each comparison is expensive (locale-aware string collation, deep object comparison). If comparisons are cheap and memory is tight, quicksort/introsort's in-place, cache-friendly behaviour wins — TimSort needs O(n) auxiliary space.

5. When to bypass it. If keys are bounded integers or fixed-width, a radix/counting sort at O(n) beats any comparison sort — the standard .sort() is not always the right tool for hot paths on large integer keys.

Summary — what a strong answer emphasises

Name the algorithm precisely per language: TimSort (stable) for JS/Java-objects/Python/Rust-stable; dual-pivot quicksort for Java primitives; introsort for C++ std::sort; pdqsort for Go/Rust-unstable. Then explain why: TimSort is stable and adaptive (great for real-world partially-ordered data and expensive comparators); introsort/dual-pivot are in-place, cache-friendly, and cap the worst case at O(n log n). It matters because stability governs multi-key correctness, worst-case bounds govern DoS resistance, and the comparator contract governs whether you get correct results or undefined behaviour. Common follow-ups: "prove the Ω(n log n) bound," "when would you drop to radix sort," and "why is Java primitive sort not stable?" (answer: primitives have no identity beyond value, so stability is a no-op — and quicksort is faster and allocation-free).

**Comparison:**

```
Language / call	Algorithm	Stable?	Worst case	Notes
JS Array.sort (V8, ES2019+)	TimSort	Yes	O(n log n)	Default comparator sorts as STRINGS
Java Arrays.sort(Object[])	TimSort	Yes	O(n log n)	Comparator must be strict weak ordering
Java Arrays.sort(int[])	Dual-pivot quicksort	No	O(n log n)*	Stability moot for primitives; in-place
Python list.sort / sorted	TimSort	Yes	O(n log n)	Origin of TimSort
C++ std::sort	Introsort (QS→heapsort)	No	O(n log n)	Use std::stable_sort for stability
C++ std::stable_sort	Mergesort	Yes	O(n log n)	Needs O(n) aux (or O(n log² n) if no mem)
Go sort.Sort / slices.Sort	pdqsort	No	O(n log n)	sort.Stable for stability
Rust sort / sort_unstable	TimSort / pdqsort	Yes / No	O(n log n)	_unstable is faster

*Dual-pivot quicksort has known adversarial patterns; JDK hardens pivot selection.
```

**Code:**

```

// V8 gotcha: default sort is LEXICOGRAPHIC (string) — must pass a comparator. [10, 2, 1].sort(); // -> [1, 10, 2] (WRONG for numbers) [10, 2, 1].sort((a, b) => a - b); // -> [1, 2, 10] (correct)

// Multi-key sort relying on TimSort stability (ES2019+ guarantees it): users .sort((a, b) => a.lastLogin - b.lastLogin) // secondary key first .sort((a, b) => a.city.localeCompare(b.city)); // primary; ties keep lastLogin order

CODE
Copy

```java
// Stable, TimSort — objects:
Arrays.sort(people, Comparator.comparing(Person::city).thenComparing(Person::name));

// NOT stable — dual-pivot quicksort on primitives:
int[] a = {5, 3, 8, 3};
Arrays.sort(a); // stability is meaningless for equal ints

// Broken comparator -> "Comparison method violates its general contract":
// BAD: (x, y) -> x.score() - y.score()  // int overflow / not a strict weak ordering
// GOOD:
Comparator.comparingInt(Person::score);
CPP
Copy
// std::sort = introsort (quicksort -> heapsort fallback), NOT stable, O(n log n) worst case.
std::sort(v.begin(), v.end());
// Need stability? use the separate, merge-based call:
std::stable_sort(v.begin(), v.end());
```

**Example input / output:**

```

Worked example of why stability matters (multi-key sort):

Input records (already ordered by date, ascending):

CODE
Copy
{name: "Ann",  date: 1}
{name: "Bob",  date: 2}
{name: "Ann",  date: 3}

Now stably sort by name:

STABLE (TimSort): Ann(date1), Ann(date3), Bob(date2) -> Ann rows keep date order 1 then 3. CORRECT.
UNSTABLE (quicksort): could emit Ann(date3), Ann(date1), Bob(date2) -> tie order lost. BUG if you expected date order preserved.

JS default-comparator trap:

CODE
Copy
Input : [10, 2, 1]
sort() : [1, 10, 2]   // compares "10" < "2" as strings
sort((a,b)=>a-b) : [1, 2, 10]
```

**Flow:**

1. Yes, huge n
2. No
3. Yes
4. No
5. Yes
6. No
7. Need to sort
8. Keys boundedintegers/fixed-width?
9. Radix/Counting sort O(n+k) -beat comparison sort
10. Need stability - multi-key sort?
11. Use stable sort: TimSort /stable_sort / sort.Stable
12. Worst-case DoS risk onuntrusted input?
13. Introsort/pdqsort: heapsortfallback caps O(n log n)
14. Default library sort is fine

