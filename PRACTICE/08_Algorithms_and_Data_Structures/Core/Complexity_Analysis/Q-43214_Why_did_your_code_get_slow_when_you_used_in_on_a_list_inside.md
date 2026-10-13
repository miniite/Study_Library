---
type: "practice"
difficulty: "Intermediate"
---

# Why did your code get slow when you used `in` on a list inside a loop? What's the fix?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Nested Loop Complexity](<../../../../08_Algorithms_and_Data_Structures/Core/Complexity_Analysis/Nested_Loop_Complexity.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

What the question is really asking: This is a scenario/debugging question. You wrote a loop that, on each iteration, does x in some_list. It ran slowly on larger inputs. Explain why the slowdown happens (the complexity blow-up) and give the concrete fix.

What the interviewer is probing for (intermediate experience): Can you recognize the classic O(n·m) trap — a linear membership scan (in on a list is O(n)) nested inside a loop of m iterations — and fix it by switching the lookup structure to a set/dict (O(1) average membership), turning the whole thing into O(n + m)? This is one of the most common real-world performance bugs, so they want to see you diagnose it, not just recite Big-O.

Key clarifying questions: "How big does the list get, and how many times is the loop running?" — the bug is invisible at n=10 and catastrophic at n=100k. "Do we look up the same list repeatedly?" (if yes, build the set once outside the loop). "Do the elements change during the loop?" (affects whether we can precompute).

Plan: (1) Pin down the cost of in on a list. (2) Show how nesting it multiplies. (3) Give the fix and its new complexity. (4) Cover the gotcha: build the set once, outside the loop.

Why it got slow

x in a_list is O(n) — a Python list is a contiguous array, so membership must scan element-by-element until it finds x or exhausts the list (worst case, and always when x is absent, it checks all n).

Now put that inside a loop that runs m times:

PYTHON
for x in queries:        # m iterations
    if x in big_list:    # each check is O(n)
        ...

Each of the m iterations pays an O(n) scan, so the total work is O(m · n). If queries and big_list are both size n (a very common shape — e.g., "find common items", "filter out seen ids"), that's O(n²).

The reason it "suddenly" got slow is that O(n²) growth is quiet for small inputs and brutal for large ones:

n = 1,000 → ~1 million comparisons → milliseconds. Fine.
n = 100,000 → ~10 billion comparisons → tens of seconds to minutes. Not fine.

10× the data → 100× the time. That superlinear cliff is the signature of a linear scan hidden inside a loop.

The fix

Membership testing is exactly what a hash-based structure is for. Convert the list you keep searching into a set (or a dict if you also need an associated value) once, then each in check is O(1) average instead of O(n):

PYTHON
lookup = set(big_list)     # O(n) once, up front
for x in queries:          # m iterations
    if x in lookup:        # O(1) average each
        ...

New cost: O(n) to build the set + O(m) for the checks = O(n + m) total — linear. For the n = 100,000 / n = 100,000 case that collapses ~10 billion operations down to ~200,000. The set is O(n) extra space; that space-for-time trade is almost always worth it here.

The critical gotcha: build the set OUTSIDE the loop

The optimization only works if you construct the set once, before the loop. A very common half-fix reintroduces the whole problem:

PYTHON
# STILL O(m*n): rebuilding the set every iteration wastes the O(1) lookup
for x in queries:
    if x in set(big_list):   # set() is O(n) and runs m times
        ...

set(big_list) is itself O(n); doing it inside the loop makes the total O(m·n) again (plus constant overhead). Hoist it out. If the list is mutated during the loop, keep the set in sync incrementally (lookup.add(...)/lookup.discard(...)) rather than rebuilding.

Other angles worth mentioning
If you need order or the original list as well, keep the list and maintain a companion set purely for lookups (seen = set() alongside the list).
If you only need to test membership against a sorted list and can't afford the set's memory, bisect gives O(log n) per check via binary search — better than O(n), worse than O(1), but no extra hash table. (Plain in does not use sortedness.)
The same pattern applies beyond in: list.index(), list.count(), and list.remove() are all O(n) and dangerous in loops for the same reason.
Summary / what a strong answer emphasizes
Diagnosis: in on a list is O(n); inside an m-iteration loop that's O(m·n) (often O(n²)) — fine on small data, cliff-edge slow on large data.
Fix: precompute a set(...) (or dict) once outside the loop → membership becomes O(1) average → total O(n + m), at O(n) extra space.
Guard against the anti-fix: don't rebuild the set inside the loop.

#### Common mistakes
rebuilding the set each iteration; assuming sorting the list speeds up in (it doesn't — you'd need bisect); forgetting elements must be hashable to go in a set; over-optimizing a loop that only runs a handful of times (measure first). Follow-up probes: "What's the space cost of the fix?" (O(n)); "What if the list changes each iteration?" (maintain the set incrementally); "How would you find the intersection of two lists efficiently?" (convert one to a set, iterate the other → O(n + m), or set(a) & set(b)).

**Comparison:**

```
Version	Membership per check	Loop iterations	Total time	Extra space
in on list inside loop	O(n)	m	O(m·n) (≈O(n²))	O(1)
Convert to set once, then check	O(1) avg	m	O(n + m)	O(n)
Rebuild set(list) inside loop (anti-fix)	O(1) but preceded by O(n) build	m	O(m·n) — no gain	O(n) transient
Sorted list + bisect per check	O(log n)	m	O(n log n + m log n)	O(1)
```

**Code:**

```python
# --- BEFORE: O(m * n) -> quadratic, gets slow on big inputs ---
def filter_known_bad(events, blocklist):        # blocklist is a list
    kept = []
    for e in events:                            # m iterations
        if e.user_id not in blocklist:          # O(n) linear scan each time
            kept.append(e)
    return kept

# --- AFTER: O(n + m) -> linear. Build the set ONCE, outside the loop ---
def filter_known_bad_fast(events, blocklist):
    blocked = set(blocklist)                     # O(n), once
    return [e for e in events if e.user_id not in blocked]  # O(1) avg per check

# --- ANTI-FIX (do NOT do this): set rebuilt every iteration -> still O(m * n) ---
def still_slow(events, blocklist):
    return [e for e in events if e.user_id not in set(blocklist)]  # BUG: O(n) inside loop
```

**Example input / output:**

```

Setup: len(events)=m=100_000, len(blocklist)=n=100_000

BEFORE (list in inside loop): work ~= m * n = 100_000 * 100_000 = 1e10 comparisons -> ~tens of seconds

AFTER (set built once): build set: n = 100_000 checks: m*1 = 100_000 total ~= n + m = 200_000 operations -> ~milliseconds

Speedup ~ 50_000x on this shape.

CODE
Copy

Tiny trace showing the fix behaves identically:

events user_ids = [1, 2, 3, 4] blocklist = [2, 4] blocked = {2, 4} 1 not in blocked -> keep 2 in blocked -> drop 3 not in blocked -> keep 4 in blocked -> drop result kept ids = [1, 3] # same as the slow version
```

**Flow:**

1. wrong
2. Loop does 'x in list' eachiteration
3. 'in' on list = O(n) scan
4. m iterations x O(n) = O(m*n) ~O(n^2)
5. Fix
6. Build set/dict ONCE, beforethe loop (O(n))
7. 'x in set' = O(1) average percheck
8. Total O(n + m), O(n) extraspace
9. set(list) INSIDE loop -> stillO(m*n)

