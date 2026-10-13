---
type: "practice"
difficulty: "Intermediate"
---

# What is the time complexity of the classic two-sum solved with a hash map, and what's its space complexity?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Time-space Trade-offs](<../../../../08_Algorithms_and_Data_Structures/Core/Complexity_Analysis/Time-space_Trade-offs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

What is being asked? State — and justify — the time and space complexity of the canonical two-sum solved with a hash map: given an array nums and a target, return indices of the two numbers that add up to target. The expected answer is O(n) time, O(n) space, but at intermediate level the interviewer wants the reasoning, the assumptions behind O(1) hash operations, and the trade-off against the brute-force version.

What are they probing for?

Do I know the algorithm well enough to reason about each operation's cost, not just recite "O(n)"?
Do I understand why a hash map gives amortized O(1) lookups — and when that assumption breaks (worst-case collisions → O(n) per op)?
Can I articulate the classic space-for-time trade-off?

Clarifying questions I'd ask: Is there exactly one solution (LeetCode's guarantee) or must I handle none/multiple? Can I use the same element twice (no — distinct indices)? Are there duplicates or negatives (map still works)? Do I return indices or values? These don't change complexity but show rigor.

Plan: describe the one-pass hash-map algorithm, count the cost of each step, sum to get overall time and space, then state the assumptions and worst case.

The algorithm (one pass)

Walk the array once. For each element nums[i], compute the complement need = target - nums[i]. If need is already in the hash map, we've found the pair — return [map.get(need), i]. Otherwise, store nums[i] → i in the map and continue. Because we check the map before inserting the current element, we never pair an element with itself, and we naturally return the earliest valid pair.

Key insight: instead of searching the rest of the array for a matching partner (which is the linear inner scan of brute force), we ask the hash map "have I already seen the number I need?" — a single O(1) lookup.

Time complexity: O(n)
We iterate the array once → n iterations.
Inside each iteration we do a constant number of O(1) operations: one subtraction, one hash-map has/get lookup, and at most one set.
Hash-map lookup and insert are amortized O(1) (average case), assuming a good hash function and a load factor kept low by resizing.

So total = n × O(1) = O(n) time. This is optimal: any correct two-sum must inspect each element at least once, so you cannot beat Ω(n).

The caveat that separates a strong answer: amortized O(1) is average case. In the pathological worst case — adversarial keys all colliding into one bucket — each operation degrades to O(n) and the whole algorithm becomes O(n²). In practice, with integer keys and language-provided hash maps (Java HashMap, Python dict, JS Map), this essentially never happens and we quote O(n) confidently. Also, resizing the map costs O(n) occasionally, but amortized over all insertions it's O(1) per insert, which is why the average holds.

Space complexity: O(n)
The hash map stores up to n entries (in the worst case we scan almost the entire array before finding the pair, e.g., the answer is the last two elements, or there's no answer).
No other data structure grows with input.

So auxiliary space = O(n). The output array of two indices is O(1) and usually not counted.

The trade-off (the point of the question)

The brute force is O(n²) time, O(1) space — two nested loops, no extra structure. The hash map buys us O(n) time at the cost of O(n) space. That's the textbook space-for-time trade-off: we spend memory to remember what we've already seen so we can answer "is my complement here?" in constant time instead of re-scanning. Whether that trade is worth it depends on context — for large n it's almost always yes (a linear pass vs. quadratic is the difference between milliseconds and seconds at n = 100,000); when memory is extremely tight and n small, brute force is fine.

Edge cases (don't affect complexity, show correctness)
Duplicates ([3,3], target 6): works because we check-then-insert, so the second 3 finds the first 3's index.
Negatives / zeros: arithmetic is identical; hash map handles any integer key.
No solution: loop ends, return an empty/sentinel result — still O(n).
Overflow: in fixed-width integer languages, target - nums[i] could overflow; use a wider type if inputs span the int range. (Not an issue in Python/JS numbers for typical ranges.)
Summary — what a strong answer emphasizes

O(n) time, O(n) space, achieved with a single pass and amortized-O(1) hash-map lookups. The candidate should: (1) explain the complement trick, (2) justify O(1) per lookup and note the O(n²) adversarial worst case, (3) name the O(n) space as the price of the O(n) time, and (4) frame it explicitly as a space-for-time trade-off versus the O(n²)/O(1) brute force. Mentioning the check-before-insert ordering (avoids self-pairing, handles duplicates) is the detail that signals real understanding.

#### Common mistakes
claiming O(1) space (forgetting the map grows to n); ignoring the worst-case collision degradation; inserting before checking (which can mis-handle duplicates or self-pair); saying it's "faster than O(n)" (impossible — you must read the input).

**Comparison:**

```
Metric	Value	Why
Time (average)	O(n)	Single pass × amortized-O(1) map ops
Time (worst, adversarial collisions)	O(n²)	Every map op degrades to O(n)
Space (auxiliary)	O(n)	Map holds up to n value→index entries
Output space	O(1)	Two indices
Lower bound	Ω(n)	Must read every element once
```

**Code:**

```
One-pass hash map: O(n) time, O(n) space.

def two_sum(nums, target): seen = {} # value -> index, amortized O(1) ops for i, x in enumerate(nums): need = target - x # complement, O(1) if need in seen: # check BEFORE inserting -> no self-pair, handles dups return [seen[need], i] seen[x] = i # remember this value's index return [] # no valid pair

CODE
Copy

```java
// Java equivalent
import java.util.*;
class Solution {
    public int[] twoSum(int[] nums, int target) {
        Map<Integer, Integer> seen = new HashMap<>(); // O(n) space
        for (int i = 0; i < nums.length; i++) {       // O(n) passes
            int need = target - nums[i];
            if (seen.containsKey(need))               // amortized O(1)
                return new int[]{ seen.get(need), i };
            seen.put(nums[i], i);
        }
        return new int[]{};
    }
}
```

**Example input / output:**

```text
INPUT : nums = [2, 7, 11, 15], target = 9
OUTPUT: [0, 1]

Trace (seen = {} initially):
 i=0 x=2  need=9-2=7  -> 7 not in seen -> seen = {2:0}
 i=1 x=7  need=9-7=2  -> 2 IS in seen (idx 0) -> return [0, 1]   (2 iterations, 1 lookup each)

Duplicate case:
INPUT : nums = [3, 3], target = 6
 i=0 x=3  need=3 -> not in seen -> seen = {3:0}
 i=1 x=3  need=3 -> in seen (idx 0) -> return [0, 1]   (correct because we check before insert)

No-solution case:
INPUT : nums = [1, 2, 4], target = 100 -> loop completes -> return []  (still O(n))
```

**Flow:**

1. no
2. yes
3. yes
4. no
5. Start: seen = empty map
6. More elements?
7. Return empty: no pair
8. x = nums[i]; need = target - x
9. need in seen?
10. Return seen[need], i
11. seen[x] = i

