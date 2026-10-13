---
type: "practice"
difficulty: "Intermediate"
---

# Walk me through analyzing the time and space complexity of a piece of code you wrote for a recent project.

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Stack-based Complexity](<../../../../08_Algorithms_and_Data_Structures/Core/Complexity_Analysis/Stack-based_Complexity.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

What is being asked: This is an "experience" question — walk through analyzing the time and space complexity of real code from a recent project, out loud, the way you would in code review. The interviewer wants a concrete, believable story, not a textbook definition.

What the interviewer is probing (intermediate): Can you (1) pick a realistic non-trivial piece of code, (2) identify the input-size variables correctly, (3) derive time and space from loops/recursion/data-structure operations, (4) spot a complexity problem and fix it, and (5) talk about the practical impact (latency, memory, cost)? They're checking that complexity analysis is a habit you apply while building, not just an interview trick.

Clarifying framing I'd state up front: I'll define my size variables clearly, distinguish auxiliary space from output space, note average vs worst case for hash-based structures, and mention how I verified it. Then I'll show the before/after because the interesting part is the optimization.

The concrete example (backend, realistic): On a recent project I built an endpoint that, for a given user, returns a "mutual connections" feed — for each of the user's orders/activities, enrich it with data joined from a second collection, and de-duplicate. The first version was an accidental O(n·m) and it showed up as a latency spike in production. That's the story I'll analyze.

Step 1 — Name the input sizes

Complexity analysis starts by naming what grows. Here:

n = number of items the user has (e.g., orders in the window).
m = number of candidate records I was matching each item against.
k = size of the final de-duplicated result (k ≤ n).

A common mistake is to say "O(n)" when there are really two independent dimensions. Being explicit about n and m is half the analysis.

Step 2 — The first (slow) version and its analysis

The naive version did, for each item, a linear scan of the candidate list to find matches, and used a list for de-duplication:

Outer loop over n items.
Inside, a for candidate in candidates scan → O(m) each.
Inside that, an if item.id in seen_list membership check on a Python list → O(k) each.

Time: the dominant nesting is n × m for the scan, so O(n·m) — and the in seen_list added a further O(n·k) term, worst case O(n²) when everything is unique. Combined worst case: O(n·m + n²). For a heavy user (n ≈ 5,000, m ≈ 20,000) that's ~100M+ operations per request — hundreds of milliseconds, and it scaled quadratically with account age. That's exactly the shape that passes tests on small fixtures and melts under real data.

Space: the seen_list, the result list, and a copy of candidates → O(n + m) auxiliary plus the O(k) output. Space was never the problem here; time was.

Step 3 — How I derived it (the method, generally)

The rules I apply, in order:

Loops: a loop that runs proportional to a size contributes a factor of that size; nested loops multiply (O(n·m)), sequential loops add (O(n + m)) and you keep the dominant term.
Data-structure ops: know the cost of each call — hash set/map lookup and insert are O(1) average (O(n) worst under pathological collisions/rehash); a list/array in check or .index() is O(n); sorting is O(n log n); a balanced tree op is O(log n); heap push/pop O(log n).
Recursion: write the recurrence and solve (e.g., divide-and-conquer T(n)=2T(n/2)+O(n) → O(n log n) by the Master Theorem); recursion depth adds O(depth) stack space.
Drop constants and lower-order terms for the asymptotic bound, but remember them when they matter in practice.
Separate auxiliary space from output space — the caller asked for the output, so the honest "extra" space excludes it.
Step 4 — The fix and its analysis

The insight: replace the linear scan with a hash index, and the list de-dup with a set.

Pre-build a dict by_key: id → record from candidates once: O(m) time, O(m) space.
Then each item does an O(1) average dict lookup and an O(1) average set membership check.
Total: O(m) to index + O(n) to process = O(n + m) time, down from O(n·m + n²).
Space: the index and the seen-set → O(n + m) auxiliary, O(k) output. I spent linear space to buy a quadratic-to-linear time win — a classic and usually correct trade at this scale.

The measured effect: p99 for heavy users dropped from ~400 ms to ~15 ms, and the curve went flat with account age instead of bending upward. I confirmed it with a quick doubling test (double the fixture size, time roughly doubled → linear) and a flame graph that no longer showed the scan as the hot frame.

Step 5 — Caveats I called out in review
Average vs worst case: hash lookups are O(1) average; I noted the O(n) worst case exists but is irrelevant for our key distribution (integer IDs).
The DB side: the real end-to-end cost also included the query — I made sure the candidate fetch was index-backed (O(log N + result)), otherwise a slow query would dominate and the in-memory Big-O wouldn't matter.
Output vs auxiliary space: if the caller streams results, I could cut output space; here we materialize the list, so O(k) output is inherent.
Summary / what a strong answer emphasizes

Name your size variables (n, m, k); read loops as multiply-when-nested / add-when-sequential; know the real cost of each data-structure op (the in list = O(n) trap is the classic bug); state time and space, separating auxiliary from output, and note average vs worst for hash structures. The compelling part is the arc: I found an O(n·m + n²) endpoint, traced it to a linear scan + list-membership, replaced them with a hash index + set to reach O(n + m) time / O(n + m) auxiliary space, and verified with profiling and a doubling test. That shows complexity analysis as a working habit tied to real latency and cost, which is exactly what an intermediate interviewer wants.

#### Common mistakes
collapsing two size dimensions into one n; forgetting that in list / .index() are linear; quoting hash ops as strictly O(1) with no average/worst caveat; counting output as "extra" space; ignoring that the DB query, not the in-memory loop, may be the true bottleneck.

#### Follow-up
probes: "What if m doesn't fit in memory?" (stream/batch the index, or push the join to the DB). "Worst case if IDs collide?" (O(n) per lookup — mitigate with a good hash / switch to a balanced tree for O(log n) guaranteed). "Could you avoid the extra space?" (only by giving up the time win or exploiting sortedness).

**Comparison:**

```
Aspect	Before (scan + list de-dup)	After (hash index + set)
Match step	Linear scan, O(m) per item	Dict lookup, O(1) avg
De-dup step	in list, O(k) per item	in set, O(1) avg
Total time	O(n·m + n²) worst	O(n + m)
Auxiliary space	O(n + m)	O(n + m)
Output space	O(k)	O(k)
Heavy-user p99	~400 ms	~15 ms
Operation	Structure	Time (avg / worst)
Membership / lookup	hash set/map	O(1) / O(n)
Membership	list / array	O(n) / O(n)
Insert	hash set/map	O(1) amortized / O(n) on rehash
Search	balanced BST	O(log n) / O(log n)
Sort	comparison sort	O(n log n)
```

**Code:**

```python
# BEFORE: O(n*m + n^2) time — nested scan + list-membership de-dup
def enrich_feed_slow(items, candidates):        # n = len(items), m = len(candidates)
    result, seen = [], []                       # 'seen' is a list  -> O(k) membership
    for it in items:                            # n iterations
        for c in candidates:                    #   * m each  -> O(n*m)
            if c.key == it.key and it.id not in seen:   # 'in list' -> O(k)
                seen.append(it.id)
                result.append(merge(it, c))
    return result

# AFTER: O(n + m) time, O(n + m) auxiliary space
def enrich_feed_fast(items, candidates):        # n = len(items), m = len(candidates)
    by_key = {}                                 # build hash index once: O(m) time/space
    for c in candidates:
        by_key.setdefault(c.key, c)             # first-wins; O(1) avg per insert
    result, seen = [], set()                    # set membership -> O(1) avg
    for it in items:                            # n iterations
        c = by_key.get(it.key)                  # O(1) avg lookup (was O(m) scan)
        if c is not None and it.id not in seen: # O(1) avg (was O(k))
            seen.add(it.id)
            result.append(merge(it, c))
    return result
```

**Example input / output:**

```

Quick doubling test used to confirm the analysis (illustrative):

CODE
Copy
version   n=1k, m=4k   n=2k, m=8k   n=4k, m=16k   ratio n->2n
slow      42 ms        168 ms       690 ms        ~4x  -> O(n*m) (quadratic-ish)
fast      1.1 ms       2.2 ms       4.5 ms        ~2x  -> O(n+m) (linear)

Doubling the input ~quadruples the slow version (nested scan) but only ~doubles the fast one (hash index) — the empirical curve matches the derived Big-O, and p99 for heavy users fell from ~400 ms to ~15 ms in production.
```

**Flow:**

1. yes
2. no
3. Name size variables: n items,m candidates, k results
4. Read control flow
5. Nested loops?
6. Multiply sizes -> O(n*m)
7. Add sizes, keep dominant ->O(n+m)
8. Add data-structure op costs(list 'in' = O(k))
9. Bottleneck: O(n*m + n^2)
10. Replace scan->hash index,list->set
11. O(n+m) time, O(n+m) auxspace
12. Verify: doubling test + flamegraph

