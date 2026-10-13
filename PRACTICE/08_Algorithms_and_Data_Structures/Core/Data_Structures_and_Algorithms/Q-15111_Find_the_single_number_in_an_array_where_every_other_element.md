---
type: "practice"
difficulty: "Intermediate"
---

# Find the single number in an array where every other element appears twice using XOR.

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Single Number Bit Tricks](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Single_Number_Bit_Tricks.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

What the question is really asking: Given an integer array where every element appears exactly twice except one element that appears once, return that single element — using the XOR trick so it runs in O(n) time and O(1) extra space. This is the classic "Single Number" problem. The interviewer is probing whether you know the algebraic properties of XOR and can turn them into an elegant, allocation-free solution instead of reaching for a hash map or sorting.

Clarifying questions a strong candidate asks:

Is it guaranteed that exactly one element appears once and all others appear exactly twice? (Yes — that's the premise; XOR relies on it.)
Can values be negative or zero? XOR works for all of them (two's complement), so this doesn't change the solution.
What if the constraint were "every element appears three times except one" or "two elements appear once"? Those are important follow-up variants (covered at the end) — the plain single-XOR trick does not directly solve them.

Plan: recall the three XOR identities, show they make all paired elements cancel, fold the array with XOR, prove correctness, then contrast with the naive approaches and cover the standard follow-ups.

First principles: the three properties of XOR

XOR (^, exclusive-or) is a bitwise operator: each output bit is 1 iff the two input bits differ. Three algebraic facts are all we need:

Identity: x ^ 0 = x — XOR-ing with zero leaves a number unchanged.
Self-inverse: x ^ x = 0 — any number XOR-ed with itself is zero (every bit equals itself, so every output bit is 0).
Commutative + associative: a ^ b ^ c = c ^ a ^ b — order and grouping don't matter.

Combine them: if I XOR every element of the array together, associativity/commutativity let me freely reorder so each duplicated pair sits next to itself. Every pair collapses to 0 by property (2), and all those zeros collapse into the lone element by property (1). The single element is the only one without a partner to cancel it, so it survives.

Formally, for array A = [a, b, a, c, b]:

CODE
a ^ b ^ a ^ c ^ b
= (a ^ a) ^ (b ^ b) ^ c   (reorder — commutative/associative)
= 0 ^ 0 ^ c               (self-inverse)
= c                       (identity)
The algorithm

Initialize an accumulator result = 0 (the identity element for XOR), then XOR every element into it in a single pass. Whatever remains is the answer.

PYTHON
def single_number(nums):
    result = 0
    for x in nums:
        result ^= x
    return result

That's the whole solution. One loop, one integer of extra state.

Why this is optimal
Time: O(n) — exactly one pass over the array; each XOR is a constant-time CPU instruction.
Space: O(1) — a single accumulator variable, regardless of array size. Crucially, we do not allocate a hash set/map.
No sorting, no hashing, no extra allocation — it's the textbook example of using bit manipulation to beat the "obvious" data-structure solution on space.
Contrast with the naive approaches

The interviewer wants to see you reject the alternatives for a reason, not just recite XOR:

Hash map / set of counts: O(n) time but O(n) space; you scan and track counts, then find the one with count 1. Correct, but wasteful.
Set add/remove toggle: add on first sight, remove on second; the leftover element is the answer. Still O(n) space.
Sort then scan pairs: O(n log n) time and mutates/allocates. Slower.
Sum trick (2 * sum(set) - sum(arr)): clever but risks integer overflow on large inputs and needs a set (O(n) space). XOR has none of these downsides.

XOR wins on both time (O(n)) and space (O(1)) simultaneously, which no comparison-based or counting approach achieves.

#### Common mistakes
/ edge cases
Initializing the accumulator to nums[0] and looping from index 1 also works, but initializing to 0 is cleaner and handles a single-element array uniformly.
Assuming the premise holds: if some element appeared an odd number of times other than once, or the "single" element appeared an even number of times, the result would be wrong — XOR only isolates the element with an odd count when all others have even counts. State this assumption.
Negative numbers / zero: no special handling needed; two's-complement bit patterns XOR correctly.
Important follow-up variants (interviewers almost always push here)
Every element appears twice except two single elements → "Single Number III". XOR of all gives x ^ y. Find any set bit (e.g. diff & -diff isolates the lowest set bit) where x and y differ, partition the array by that bit, and XOR each partition separately to recover x and y. Still O(n)/O(1).
Every element appears three times except one → "Single Number II". Plain XOR fails because triples don't cancel. Use bit-counting mod 3: for each of the 32 bit positions, count how many numbers have that bit set; count % 3 reconstructs the answer's bits. Or the two-variable ones/twos finite-state-machine trick. O(n)/O(1).

Mentioning these shows you understand why the simple XOR works and where its assumptions break — exactly the depth expected at intermediate-experience level.

Summary — what a strong answer emphasizes

Lead with the three XOR identities (x^0=x, x^x=0, commutative/associative), explain that they make all duplicated pairs cancel to zero and leave the lone element, then give the one-line fold. Nail the O(n) time / O(1) space claim and explicitly contrast it against the O(n)-space hash approach. Finish by acknowledging the guaranteed-input assumption and the two standard follow-ups (two singles, and triples via bit-count mod 3) to demonstrate you understand the boundaries of the trick.

**Comparison:**

```
Approach	Time	Extra Space	Notes
XOR fold	O(n)	O(1)	optimal; no allocation; works for negatives/zero
Hash map counts	O(n)	O(n)	correct but wastes space
Set add/remove toggle	O(n)	O(n)	leftover element is the answer
Sort then scan pairs	O(n log n)	O(1)–O(n)	slower, mutates input
2*sum(set) - sum(arr)	O(n)	O(n)	overflow risk, needs a set
```

**Code:**

```
Single Number — every element appears twice except one. O(n) time, O(1) space.

def single_number(nums): result = 0 for x in nums: result ^= x # pairs cancel (x ^ x = 0); lone element survives (x ^ 0 = x) return result

Follow-up: exactly TWO elements appear once, all others twice.

def single_number_two(nums): xor_all = 0 for x in nums: xor_all ^= x # xor_all = a ^ b (the two singles) diff = xor_all & (-xor_all) # isolate lowest set bit where a and b differ a = 0 for x in nums: if x & diff: a ^= x # partition 1 -> isolates a return [a, xor_all ^ a] # b = (a ^ b) ^ a

CODE
Copy
```java
// Java equivalent of the core solution
int singleNumber(int[] nums) {
    int result = 0;
    for (int x : nums) result ^= x;
    return result;
}
```

**Example input / output:**

```text
Example 1 (basic):
  Input : nums = [4, 1, 2, 1, 2]
  Trace : 0
          0 ^ 4 = 4
          4 ^ 1 = 5
          5 ^ 2 = 7
          7 ^ 1 = 6
          6 ^ 2 = 4
  Output: 4

Example 2 (single element):
  Input : nums = [7]
  Output: 7            (0 ^ 7 = 7)

Example 3 (negatives / zero — still works):
  Input : nums = [-3, 0, -3]
  Trace : 0 ^ (-3) = -3 ; -3 ^ 0 = -3 ; -3 ^ (-3) = 0
  Output: 0

Bit-level view of Example 1 (why pairs cancel):
  4 = 100
  1 = 001
  2 = 010
  1 = 001   -> the two 001s XOR to 000
  2 = 010   -> the two 010s XOR to 000
  leftover  = 100 = 4
```

**Flow:**

1. yes
2. no
3. result = 0
4. next element x?
5. result = result XOR x
6. return result= the single number

