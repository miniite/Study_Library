---
type: "practice"
difficulty: "Other"
---

# Integer Equal to the Sum of Its Proper Divisors

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Math](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Math.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

A positive integer is called self-balancing if it is exactly equal to the sum of all of its proper positive divisors — that is, every positive integer that divides it evenly, excluding the number itself.

For example, the divisors of 12 (other than 12) are 1, 2, 3, 4, 6, and their sum is 16, which is not 12, so 12 is not self-balancing. The divisors of 6 (other than 6) are 1, 2, 3, whose sum is 6, so 6 is self-balancing.

Given a single positive integer value, return true if value is self-balancing, and false otherwise.

Input

One integer value.

Output

A boolean: true if value equals the sum of its proper positive divisors, else false.

#### Cons
traints

1 <= value <= 10^8
The answer depends only on value; no external state is involved.

#### Example
s & test cases
TEXT
Input: value = 6 -> Output: true
Why: proper divisors 1 + 2 + 3 = 6, so it matches (smallest such number).

Input: value = 496 -> Output: true
Why: 1 + 2 + 4 + 8 + 16 + 31 + 62 + 124 + 248 = 496; a normal positive case.

Input: value = 12 -> Output: false
Why: 1 + 2 + 3 + 4 + 6 = 16 != 12; a number whose divisor sum exceeds it.

Input: value = 7 -> Output: false
Why: only proper divisor is 1, sum = 1 != 7; a prime falls short.

Input: value = 1 -> Output: false
Why: 1 has no proper divisors other than itself, so the sum is 0 (minimum edge case).

Input: value = 8128 -> Output: true
Why: a larger self-balancing number, exercising the sqrt-bounded scan.

#### Approach

Divisors come in pairs: if i divides value, then value / i does too. So iterate i from 2 up to sqrt(value); whenever i divides value, add both i and value / i to a running sum (seeded with 1, the universal proper divisor), taking care not to add the square root twice when i * i == value. Finally compare the accumulated sum to value. This pairing trick reduces the naive O(value) divisor scan to O(sqrt(value)). Values <= 1 are rejected up front since they have no qualifying divisor sum.

Complexity

Time: O(sqrt(value)) — divisors are found in complementary pairs, so we only scan i up to sqrt(value). Space: O(1) — a single running accumulator is used.

Python
PYTHON
def is_self_balancing(value: int) -> bool:
    if value <= 1:
        return False
    total = 1  # 1 is a proper divisor of every value > 1
    i = 2
    while i * i <= value:
        if value % i == 0:
            total += i
            pair = value // i
            if pair != i:      # do not count a square root twice
                total += pair
        i += 1
    return total == value
JavaScript
JAVASCRIPT
function isSelfBalancing(value) {
    if (value <= 1) return false;
    let sum = 1;                       // 1 divides every value > 1
    for (let i = 2; i * i <= value; i++) {
        if (value % i === 0) {
            sum += i;

#### cons
t pair = value / i;
            if (pair !== i) sum += pair; // add complement, skip duplicate for perfect squares
        }
    }
    return sum === value;
}
Java
JAVA
class Solution {
    public boolean isSelfBalancing(int value) {
        if (value <= 1) return false;
        long sum = 1;                       // 1 divides any value > 1
        for (int i = 2; (long) i * i <= value; i++) {
            if (value % i == 0) {
                sum += i;
                int pair = value / i;
                if (pair != i) sum += pair; // add the paired divisor, once for squares
            }
        }
        return sum == value;
    }
}
C++
CPP
class Solution {
public:
    bool isSelfBalancing(int value) {
        if (value <= 1) return false;
        long sum = 1;                       // 1 is always a proper divisor here
        for (int i = 2; (long)i * i <= value; ++i) {
            if (value % i == 0) {
                sum += i;
                int pair = value / i;
                if (pair != i) sum += pair; // avoid double-counting a perfect square root
            }
        }
        return sum == value;
    }
};
C
C
#include <stdbool.h>

bool isSelfBalancing(int value) {
    if (value <= 1) return false;      // 1 has no proper divisors summing to it
    long sum = 1;                       // 1 divides every value > 1
    for (int i = 2; (long)i * i <= value; i++) {
        if (value % i == 0) {
            sum += i;                   // i is a divisor
            int pair = value / i;
            if (pair != i) sum += pair; // add the complementary divisor once
        }
    }
    return sum == value;
}

#### Follow-up
questions
How would you adapt this if value can be as large as 10^18 (guard against overflow, use 64-bit accumulation)?
Instead of a boolean, return the actual list of proper divisors in sorted order.
Classify the number as deficient, self-balancing (equal), or abundant based on whether the divisor sum is less than, equal to, or greater than the value.
If you must answer many queries, how could precomputation (e.g., a sieve of divisor sums) help?

