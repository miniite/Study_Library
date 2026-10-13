---
type: "practice"
difficulty: "Other"
---

# Check Whether an Integer Is a Power of Three

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Recursion](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Recursion.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given a single 32-bit signed integer value. Determine whether it is a power of three: that is, decide whether there exists a non-negative integer exponent x such that value == 3^x.

The powers of three are 3^0 = 1, 3^1 = 3, 3^2 = 9, 3^3 = 27, and so on. Return true if value equals one of these, and false otherwise. Note that zero and all negative numbers are never powers of three.

Input

value: an integer in the 32-bit signed range.

Output

A boolean: true if value is a power of three, false otherwise.

#### Cons
traints

-2^31 <= value <= 2^31 - 1

Follow up: Can you decide it without any loop or recursion?

#### Example
s & test cases
TEXT
Input: value = 81 -> Output: true
Why: 81 = 3^4, a normal positive power of three.

Input: value = 1 -> Output: true
Why: 1 = 3^0; the exponent zero is allowed, so 1 counts.

Input: value = 0 -> Output: false
Why: no exponent x makes 3^x equal 0.

Input: value = -3 -> Output: false
Why: negative numbers can never be a power of three even though |value| is 3.

Input: value = 45 -> Output: false
Why: 45 = 3^2 * 5 carries a factor of 5, so it is not a pure power of three.

Input: value = 1162261467 -> Output: true
Why: 1162261467 = 3^19 is the largest power of three that fits in a 32-bit signed integer (an upper-boundary case).

#### Approach

Divisibility-by-the-largest-power trick. Within the 32-bit signed range the greatest power of three is 3^19 = 1162261467. Because 3 is prime, the only divisors of 3^19 are 3^0, 3^1, ..., 3^19 — every one of them a power of three. Therefore a positive value is a power of three if and only if it evenly divides 3^19. Checking value > 0 && 1162261467 % value == 0 answers the question in constant time with no loop or recursion, which is exactly what the follow-up asks. (The alternative loop repeatedly divides by 3 while divisible and checks that the result is 1, but the modulo trick is O(1).)

Complexity

Time: O(1) — a single sign check and one modulo operation, independent of the input size. Space: O(1) — no extra storage.

Python
PYTHON
def is_power_of_three(value: int) -> bool:
    # 3^19 = 1162261467 is the largest power of three within 32-bit signed range;
    # because 3 is prime, value is a power of three iff it divides 3^19.
    return value > 0 and 1162261467 % value == 0
JavaScript
JAVASCRIPT
function isPowerOfThree(value) {
    // 3^19 = 1162261467 is the largest power of three within 32-bit int range;
    // because 3 is prime, value is a power of three iff it divides 3^19.
    return value > 0 && 1162261467 % value === 0;
}
Java
JAVA
class Solution {
    public boolean isPowerOfThree(int value) {
        // 3^19 = 1162261467 is the largest power of three within 32-bit int range;
        // because 3 is prime, value is a power of three iff it divides 3^19.
        return value > 0 && 1162261467 % value == 0;
    }
}
C++
CPP
class Solution {
public:
    bool isPowerOfThree(int value) {
        // 3^19 = 1162261467 is the largest power of three within 32-bit int range;
        // because 3 is prime, value is a power of three iff it divides 3^19.
        return value > 0 && 1162261467 % value == 0;
    }
};
C
C
#include <stdbool.h>

/* 1162261467 == 3^19 is the largest power of three that fits in a 32-bit int.
   Since 3 is prime, every divisor of 3^19 is itself a power of three, so a
   positive value is a power of three exactly when it divides 3^19. */
bool isPowerOfThree(int value) {
    return value > 0 && 1162261467 % value == 0;
}

#### Follow-up
questions
Solve it strictly without any loop or recursion (the divisibility-by-3^19 approach).
Generalize the test to decide whether a number is a power of an arbitrary prime p.
How would you check for a power of four, where the "largest power divides" prime trick does not directly apply?
Extend the check to arbitrary-precision (big-integer) inputs that exceed 32 bits.

