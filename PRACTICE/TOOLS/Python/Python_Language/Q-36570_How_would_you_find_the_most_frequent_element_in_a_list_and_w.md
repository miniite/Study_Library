---
type: "practice"
difficulty: "Intermediate"
---

# How would you find the most frequent element in a list, and what is the time complexity of your approach?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Membership and Collection Complexity](<../../../../TOOLS/Python/Python_Language/Membership_and_Collection_Complexity.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

What is being asked: Given a list (say [1, 3, 3, 2, 3, 1]), find the element that appears most often (the mode), and state the time complexity of your approach.

What the interviewer is probing for (intermediate level): Not just "which function do I call," but whether you understand the counting model behind it, can reason about O(n) time vs. sorting-based O(n log n), know the idiomatic tools (collections.Counter, statistics.mode, max with a key), and can handle the messy real-world parts: ties, empty input, unhashable elements, and streaming/very-large data where you can't hold the whole list.

Clarifying questions a strong candidate asks:

If there's a tie (two elements share the top frequency), do we return one, all, or the first-seen?
Are the elements hashable (ints, strings, tuples) or could they be lists/dicts? Hashability decides whether a dict-based count works.
Is the list already sorted or could it be a stream too big for memory?
What should happen on an empty list — raise, or return None?

I'll assume hashable elements, return one mode (documenting the tie policy), and treat an empty list as an error case to handle explicitly.

From first principles — the counting model

Finding the most frequent element is a two-step problem: (1) count how many times each distinct value occurs, then (2) pick the value with the maximum count.

The natural data structure for counting is a hash map (Python dict): key = element, value = running count. One pass over the list fills the map — each insert/update is O(1) average — so counting is O(n) time and O(k) space, where k is the number of distinct elements (k <= n). Step (2), scanning the map for the max count, is O(k). Total: O(n) time, O(k) space — and you cannot do better than O(n) in the worst case, because you must look at every element at least once to be sure of the true mode.

Idiomatic Python: collections.Counter
PYTHON
from collections import Counter

def most_frequent(lst):
    if not lst:
        raise ValueError("empty list has no mode")
    counts = Counter(lst)                 # O(n) single pass
    return counts.most_common(1)[0][0]    # O(k) to find the top; returns the value

Counter(lst) builds the frequency map in one pass. most_common(1) returns a list like [(value, count)]; [0][0] pulls out the value. Under the hood most_common uses a heap, but for the single-max case it's effectively a linear scan.

Tie behavior worth knowing: Counter.most_common breaks ties by insertion order (the element that was inserted/first seen earliest wins) since Python 3.7. That's the detail interviewers love to poke at.

Equivalent without Counter
PYTHON
def most_frequent_max(lst):
    counts = {}
    for x in lst:
        counts[x] = counts.get(x, 0) + 1  # O(n) total
    return max(counts, key=counts.get)    # O(k), ties -> first max encountered

max(counts, key=counts.get) scans the keys and returns the one whose count is largest. This shows you understand what Counter does internally.

statistics.mode
PYTHON
import statistics
statistics.mode([1, 1, 2, 3])   # -> 1

Since Python 3.8, statistics.mode returns the first mode encountered on a tie and no longer raises on multimodal data. It's fine for quick use but less flexible than Counter when you need counts or the top-k.

Time / space complexity — the crisp statement
Hash-map / Counter approach: O(n) time, O(k) space (k = distinct elements). This is optimal for time.
Sort-then-scan: O(n log n) time, O(1)–O(n) space. Sort the list so equal elements are adjacent, then walk once tracking the longest run. Slower asymptotically, but uses no hash map — useful if elements are unhashable but comparable/sortable (e.g. sort of tuples), or if you must avoid a dict for memory reasons.
PYTHON
def most_frequent_sorted(lst):
    if not lst:
        raise ValueError("empty list")
    lst = sorted(lst)                 # O(n log n)
    best_val, best_len = lst[0], 1
    cur_val, cur_len = lst[0], 1
    for x in lst[1:]:                 # O(n)
        if x == cur_val:
            cur_len += 1
        else:
            cur_val, cur_len = x, 1
        if cur_len > best_len:
            best_val, best_len = cur_val, cur_len
    return best_val
Real-world / scale considerations
Ties: decide and document the policy. To return all modes: maxc = max(counts.values()); modes = [v for v, c in counts.items() if c == maxc].
Unhashable elements (lists/dicts): a dict/Counter will raise TypeError. Convert to a hashable form (tuple(x)) or use the sort-based approach.
Streaming / data too big for RAM: you can't hold all counts if cardinality is huge. Use an approximate heavy-hitters algorithm — Count-Min Sketch or the Misra–Gries / Boyer–Moore majority variant — to find frequent items in O(1) space per counter with bounded error. Mentioning this signals seniority.
Distributed data: map-reduce — each shard emits partial counts, a reducer sums them; classic word-count pattern.
Summary — what a strong answer emphasizes

Lead with Counter(lst).most_common(1)[0][0], state it's O(n) time, O(k) space, and explain why that's optimal (must inspect every element). Show the manual dict version to prove you understand the mechanism, then contrast with the O(n log n) sort approach and say when you'd choose it (unhashable-but-sortable data, no-hash-map memory constraint). Close by handling ties, empty input, unhashable elements, and streaming/large-scale with Count-Min Sketch / majority-vote.

#### Common mistakes
claiming it's O(n log n) when the hash approach is O(n); forgetting the empty-list case; not knowing most_common breaks ties by insertion order; and using lst.count(x) inside a loop over the list, which is O(n²) — a classic performance trap.

**Comparison:**

```
Approach	Time	Space	When to use	Tie handling
Counter.most_common(1)	O(n)	O(k)	Default; hashable elements	Insertion order (first seen wins)
Manual dict + max(key=)	O(n)	O(k)	Show the mechanism / no imports	First max encountered
Sort + longest-run scan	O(n log n)	O(1)–O(n)	Unhashable-but-sortable; avoid hash map	First in sorted order
Boyer–Moore majority	O(n)	O(1)	Only if a strict >n/2 majority exists	N/A (single winner)
Count-Min Sketch / Misra–Gries	O(n)	O(1) per counter	Streaming / huge cardinality	Approximate
max(lst, key=lst.count) ❌	O(n²)	O(1)	Never (anti-pattern)	First max
```

**Code:**

```python
from collections import Counter

# Idiomatic — O(n) time, O(k) space
def most_frequent(lst):
    if not lst:
        raise ValueError("empty list has no mode")
    return Counter(lst).most_common(1)[0][0]

# Manual dict version (same complexity) — shows the mechanism
def most_frequent_max(lst):
    if not lst:
        raise ValueError("empty list")
    counts = {}
    for x in lst:
        counts[x] = counts.get(x, 0) + 1
    return max(counts, key=counts.get)

# Return ALL modes on a tie
def all_modes(lst):
    counts = Counter(lst)
    top = max(counts.values())
    return [v for v, c in counts.items() if c == top]

# O(1)-extra-space majority (only valid if a strict majority exists) — Boyer-Moore
def majority_element(lst):
    candidate, count = None, 0
    for x in lst:
        if count == 0:
            candidate, count = x, 1
        elif x == candidate:
            count += 1
        else:
            count -= 1
    return candidate  # verify with a second pass if a true majority isn't guaranteed

# ANTI-PATTERN — O(n^2), do NOT do this:
# max(lst, key=lst.count)   # lst.count scans the whole list for every element
```

**Example input / output:**

```text
most_frequent([1, 3, 3, 2, 3, 1])        -> 3        # 3 appears 3 times
most_frequent(['a','b','a','c','b','a']) -> 'a'      # 'a' appears 3 times
most_frequent([5])                       -> 5
most_frequent([])                        -> ValueError: empty list has no mode

# Tie: 1 and 2 both appear twice; most_common keeps insertion order -> 1 (seen first)
most_frequent([1, 2, 1, 2])              -> 1
all_modes([1, 2, 1, 2])                  -> [1, 2]

# Boyer-Moore majority (element appears > n/2 times):
majority_element([2, 2, 1, 2, 3, 2, 2])  -> 2
```

**Flow:**

1. Yes
2. No, but sortable
3. Streaming / huge
4. Input list
5. Elements hashable?
6. Counter/dict: one O(n) pass tocount
7. Scan counts O(k) for max
8. Return most frequentO(n) time, O(k) space
9. sort O(n log n)
10. Walk once, track longest run
11. Count-Min Sketch / majorityvoteO(1) space, approximate

