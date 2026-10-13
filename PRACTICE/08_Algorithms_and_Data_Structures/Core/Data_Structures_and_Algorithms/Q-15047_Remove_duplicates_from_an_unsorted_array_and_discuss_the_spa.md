---
type: "practice"
difficulty: "Basic"
---

# Remove duplicates from an unsorted array and discuss the space-time trade-off of your approach.

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Sorting Algorithms](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Sorting_Algorithms.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage
— what's really being asked

The interviewer wants two things: (1) a working algorithm that takes an unsorted array and returns it with duplicates removed, and (2) an intelligent discussion of the space–time trade-off between the possible approaches. The phrase "discuss the space-time trade-off" is the real signal — this is not just a coding question, it's a judgment question. A weak candidate writes one solution and stops; a strong candidate presents at least two solutions, states their complexities precisely, and explains when they'd pick each one.

Before coding I'd clarify a few things (and you should say these out loud in an interview):

Do we need to preserve the original order of first appearance, or is any order acceptable? This changes the answer.
Can we modify the input array in place, or must we return a new one?
What is the element type — integers with a bounded range, arbitrary integers, strings, or objects? This decides whether a hash set or a bitset is viable.
Is memory tight (e.g., embedded/streaming) or is time the priority (typical web backend)?

I'll assume the common case first: arbitrary values, order-of-first-appearance should be preserved, extra memory is acceptable — then contrast with the memory-constrained alternative.

#### Approach
1 — Hash set (the default, order-preserving)

Walk the array once. Keep a HashSet of values already seen. For each element, if it's not in the set, add it to the set and append it to the result. If it is in the set, skip it.

Time: O(n) — one pass, and each hash-set contains/add is O(1) average.
Space: O(n) — in the worst case (all distinct) the set holds every element, plus the result array.
Order: preserved (first occurrence wins).

This is what you should reach for by default in a backend/fullstack setting. Memory is cheap, the code is clear, and O(n) time is optimal — you cannot remove duplicates without at least looking at every element once, so O(n) is a hard lower bound on time.

One subtlety to mention: the O(1) hash operations are average case. A pathological hash collision (or an adversary who crafts colliding keys) can degrade lookups toward O(n), making the whole thing O(n²). In practice modern hash sets randomize their seed to defend against this, but it's the kind of nuance an interviewer loves to hear.

#### Approach
2 — Sort first, then remove adjacent duplicates

Sort the array, then do a single linear scan collapsing runs of equal adjacent values (the classic two-pointer "remove duplicates from sorted array" step).

Time: O(n log n) — dominated by the sort.
Space: O(1) extra if you sort in place and dedup in place (ignoring the sort's own recursion stack, typically O(log n)).
Order: destroyed — output is sorted, not first-appearance order.

The trade-off is now explicit: you spend a log n factor in time to save the O(n) hash set. On a machine where memory is the bottleneck (large arrays, tight RAM, or you must operate in place), this is the better call. If the caller wanted sorted output anyway, this approach is strictly better because the sort is "free."

#### Approach
3 — Bitset / boolean array (when the value range is small and bounded)

If the elements are integers in a known small range, say 0..M, allocate a bit array of size M+1, mark each value's bit, and emit values in a single pass (skipping already-marked bits).

Time: O(n) (or O(n + M) if you iterate the bitset).
Space: O(M) bits — potentially far smaller than a hash set (1 bit vs a full hash-table entry per element).

This is the specialized winner when M is modest — e.g. deduping user ages, day-of-year, small enum codes. It fails when the range is huge or unbounded (a bitset for 64-bit integers is impossible).

The trade-off, summarized as a decision
Need	Best choice
Preserve first-appearance order, memory is fine	Hash set (O(n) time, O(n) space)
Memory-constrained / must dedup in place / output can be sorted	Sort + scan (O(n log n) time, O(1) extra space)
Small bounded integer range	Bitset (O(n) time, O(M) bits)

The single sentence to leave the interviewer with: "You're trading a logarithmic time factor against a linear space factor. Hash set buys you optimal O(n) time and order preservation at the cost of O(n) memory; sorting buys you near-zero extra memory at the cost of an O(log n) time factor and loss of order."

#### Common mistakes
& follow-ups
Claiming O(1) space for the hash-set approach — it's O(n). Don't confuse the lookup cost with the storage cost.
Forgetting order — many candidates hand back a sorted array and don't mention that order changed. Always state it.
Using list.contains() inside a loop instead of a set — that's O(n) per lookup → O(n²) total. A classic silent regression.

#### Follow-up
"do it in place with O(1) extra space and O(n) time" — for unsorted arbitrary data this is provably impossible without a hash structure; you must either accept sort's O(n log n) or a bitset's bounded-range assumption. Being able to say why it's impossible is the senior answer.

#### Follow-up
streaming / can't fit in memory — pivot to a probabilistic structure like a Bloom filter (allows tiny false-positive rate to save memory) or external sort.
Summary — what a strong answer emphasizes

Reach for the hash set as the primary answer: O(n) time, O(n) space, order preserved, clean code. Then proactively offer the sort-based alternative to demonstrate you understand the space–time dial, and mention the bitset for bounded ranges. Nail the complexities exactly, call out that O(n) time is a lower bound, and be explicit about the order-preservation consequence of each approach.

**Comparison:**

```
Approach	Time	Extra space	Order preserved	Best when
Hash set	O(n) avg	O(n)	Yes	Default; memory available; need first-appearance order
Sort + two-pointer scan	O(n log n)	O(1)*	No (output sorted)	Memory tight / in-place / sorted output acceptable
Bitset (bounded range 0..M)	O(n)	O(M) bits	Yes (scan in order)	Small, known integer range

* excluding the sort's O(log n) recursion/stack overhead.
```

**Code:**

```

// Approach 1: Hash set — O(n) time, O(n) space, preserves first-appearance order public static int[] removeDuplicates(int[] arr) { java.util.Set<Integer> seen = new java.util.HashSet<>(); java.util.List<Integer> result = new java.util.ArrayList<>(); for (int x : arr) { if (seen.add(x)) { // add() returns false if already present result.add(x); // first time we've seen x -> keep it } } return result.stream().mapToInt(Integer::intValue).toArray(); }

CODE
Copy

```java
// Approach 2: Sort + two-pointer scan — O(n log n) time, O(1) extra space (order NOT preserved)
public static int removeDuplicatesInPlace(int[] arr) {
    if (arr.length == 0) return 0;
    java.util.Arrays.sort(arr);
    int write = 1;                       // slow pointer: next slot to write
    for (int read = 1; read < arr.length; read++) {
        if (arr[read] != arr[write - 1]) // new distinct value
            arr[write++] = arr[read];
    }
    return write;                        // arr[0..write-1] are the unique values
}
PYTHON
Copy
# Approach 1 in Python — idiomatic, order-preserving (dict keeps insertion order in 3.7+)
def remove_duplicates(arr):
    return list(dict.fromkeys(arr))   # O(n) time, O(n) space, first-occurrence order
```

**Example input / output:**

```

Approach 1 (hash set), order preserved:

CODE
Copy
Input : [4, 2, 4, 7, 2, 9, 4]
Trace :
  4 -> not seen -> keep, seen={4},        result=[4]
  2 -> not seen -> keep, seen={4,2},      result=[4,2]
  4 -> seen     -> skip
  7 -> not seen -> keep, seen={4,2,7},    result=[4,2,7]
  2 -> seen     -> skip
  9 -> not seen -> keep, seen={4,2,7,9},  result=[4,2,7,9]
  4 -> seen     -> skip
Output: [4, 2, 4->skip ... ] = [4, 2, 7, 9]

Approach 2 (sort + scan), order NOT preserved:

CODE
Copy
Input : [4, 2, 4, 7, 2, 9, 4]
Sorted: [2, 2, 4, 4, 4, 7, 9]
Scan collapses runs -> [2, 4, 7, 9]   (returns length 4)
```

**Flow:**

1. Need first-appearanceorder,memory OK
2. Memory tight / in placeOR sorted output OK
3. Small boundedinteger range 0..M
4. Unsorted array
5. Constraints?
6. Hash set passO n time, O n space
7. Sort thentwo-pointer scanO n log n, O 1 space
8. Bitset mark & emitO n time, O M bits
9. Deduplicated result

