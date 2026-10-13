---
type: "practice"
difficulty: "Other"
---

# Number at a Given Dictionary-Order Position

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Trie](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Trie.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Cons
ider all the integers from 1 to limit inclusive, written in the ordinary way (base 10, no leading zeros), and then sorted the way words are ordered in a dictionary — that is, by comparing them character by character as strings rather than by numeric value. For example, when limit is 15 the dictionary-order sequence is:

CODE
1, 10, 11, 12, 13, 14, 15, 2, 3, 4, 5, 6, 7, 8, 9

Notice that 10 comes right after 1 (because the string "10" starts with "1"), and every number that begins with 1 appears before 2.

Given two integers limit and position, return the integer that occupies the position-th slot (1-indexed) in this dictionary-order sequence.

Input: two integers limit and position.

Output: a single integer — the value found at index position (counting from 1) when 1..limit are sorted in dictionary order.

#### Cons
traints:

1 <= position <= limit <= 10^9.

Because limit can be as large as one billion, you cannot build or sort the full list explicitly; an efficient method must count how many numbers fall under a given leading prefix without enumerating them.

#### Example
s & test cases
TEXT
Input: limit = 15, position = 4 -> Output: 12
Why: the dictionary order is 1, 10, 11, 12, ...; the 4th entry is 12.

Input: limit = 1, position = 1 -> Output: 1
Why: there is only one number, so it occupies the single available slot.

Input: limit = 15, position = 1 -> Output: 1
Why: the smallest string is always "1", so the first slot is 1 for any limit >= 1.

Input: limit = 15, position = 8 -> Output: 2
Why: the numbers starting with 1 (1,10,11,12,13,14,15) fill the first seven slots, so slot 8 is the next prefix, 2.

Input: limit = 15, position = 15 -> Output: 9
Why: the very last entry in dictionary order is the largest single-digit prefix, 9.

Input: limit = 100, position = 3 -> Output: 100
Why: the order begins 1, 10, 100, ...; reaching slot 3 requires descending two levels into the "1" prefix, landing on 100.

#### Approach

Model the integers 1..limit as nodes of a 10-ary prefix tree (a denary trie): the root's children are the single digits 1..9, and appending a digit 0..9 to a node's value gives its children. A pre-order walk of this tree visits numbers in exactly dictionary order. The key primitive, countSteps(prefix), counts how many numbers in [1, limit] start with a given prefix by summing, over each depth, the count of values in the half-open range [prefix, prefix+1) scaled by 10 each level and clamped to limit + 1. Starting from curr = 1 (which already fills slot 1), repeatedly decide: if the whole subtree rooted at curr contains at most the remaining k numbers, skip it entirely (curr += 1, subtract that count); otherwise the answer lies inside, so descend (curr *= 10, consume one step for curr itself). When k reaches 0, curr is the answer. This is a greedy descent over an implicit trie driven by subtree-size counting; all arithmetic uses 64-bit values because prefixes can temporarily exceed limit.

Complexity

Time: O((log limit)^2). The prefix (the current number) grows by roughly one decimal digit per descent, so there are O(log limit) levels; at each level we may skip up to a few sibling prefixes, and each countSteps call itself loops O(log limit) times. Space: O(1) — only a handful of 64-bit counters are used; the 10-ary prefix tree is walked implicitly, never materialized.

Python
PYTHON
class Solution:
    def findKthNumber(self, limit: int, position: int) -> int:
        curr = 1
        k = position - 1                 # curr == 1 already occupies slot 1
        while k > 0:
            steps = self._count_steps(limit, curr, curr + 1)
            if steps <= k:               # skip the whole subtree under curr
                curr += 1
                k -= steps
            else:                        # descend into this subtree
                curr *= 10
                k -= 1
        return curr

    def _count_steps(self, limit: int, p1: int, p2: int) -> int:
        steps = 0
        while p1 <= limit:
            steps += min(limit + 1, p2) - p1
            p1 *= 10
            p2 *= 10
        return steps
JavaScript
JAVASCRIPT
function findKthNumber(limit, position) {
    let curr = 1;
    let k = position - 1;                // curr == 1 already occupies slot 1
    while (k > 0) {

#### cons
t steps = countSteps(limit, curr, curr + 1);
        if (steps <= k) {               // skip the whole subtree under curr
            curr += 1;
            k -= steps;
        } else {                        // descend into this subtree
            curr *= 10;
            k -= 1;
        }
    }
    return curr;
}

function countSteps(limit, p1, p2) {
    let steps = 0;
    while (p1 <= limit) {
        steps += Math.min(limit + 1, p2) - p1;
        p1 *= 10;
        p2 *= 10;
    }
    return steps;
}
Java
JAVA
class Solution {
    public int findKthNumber(int limit, int position) {
        long curr = 1;
        long k = (long) position - 1;   // curr == 1 already occupies slot 1
        while (k > 0) {
            long steps = countSteps(limit, curr, curr + 1);
            if (steps <= k) {           // skip the whole subtree under curr
                curr += 1;
                k -= steps;
            } else {                    // descend into this subtree
                curr *= 10;
                k -= 1;
            }
        }
        return (int) curr;
    }

    private long countSteps(long limit, long p1, long p2) {
        long steps = 0;
        while (p1 <= limit) {
            steps += Math.min(limit + 1, p2) - p1;
            p1 *= 10;
            p2 *= 10;
        }
        return steps;
    }
}
C++
CPP
#include <algorithm>
using namespace std;

class Solution {
public:
    int findKthNumber(int limit, int position) {
        long curr = 1;
        long k = (long)position - 1;   // curr == 1 already occupies slot 1
        while (k > 0) {
            long steps = countSteps(limit, curr, curr + 1);
            if (steps <= k) {          // skip the whole subtree under curr
                curr += 1;
                k -= steps;
            } else {                   // descend into this subtree
                curr *= 10;
                k -= 1;
            }
        }
        return (int)curr;
    }

private:
    long countSteps(long limit, long p1, long p2) {
        long steps = 0;
        while (p1 <= limit) {
            steps += min(limit + 1, p2) - p1;
            p1 *= 10;
            p2 *= 10;
        }
        return steps;
    }
};
C
C
/* How many integers in [1, limit] have a leading prefix in [p1, p2). */
static long countSteps(long limit, long p1, long p2) {
    long steps = 0;
    while (p1 <= limit) {
        long hi = (limit + 1 < p2) ? (limit + 1) : p2;
        steps += hi - p1;
        p1 *= 10;
        p2 *= 10;
    }
    return steps;
}

int findKthNumber(int limit, int position) {
    long curr = 1;
    long k = (long)position - 1;   /* curr == 1 already occupies slot 1 */
    while (k > 0) {
        long steps = countSteps(limit, curr, curr + 1);
        if (steps <= k) {          /* skip the whole subtree under curr  */
            curr += 1;
            k -= steps;
        } else {                   /* the target lives inside this subtree */
            curr *= 10;
            k -= 1;
        }
    }
    return (int)curr;
}

#### Follow-up
questions
Solve the inverse problem: given a value x, return its 1-indexed rank in the dictionary-order sequence.
Produce the dictionary-order sequence lazily as an iterator that yields the next number in O(log limit) without sorting.
Generalize the ordering to an arbitrary base b instead of base 10.
Handle the range [0, limit] or a general [a, limit] instead of [1, limit].

