---
type: "practice"
difficulty: "Other"
---

# Determine Whether an Integer Is a Power of Four

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Bit Manipulation](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Bit_Manipulation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Given a single 32-bit signed integer value, decide whether it can be written as four raised to some non-negative integer exponent. In other words, return true if there exists an integer k >= 0 such that value == 4^k, and false otherwise.

The valid powers of four are 1 (= 4^0), 4, 16, 64, 256, and so on. Zero and all negative numbers are never powers of four. Note that a number can be a power of two without being a power of four (for example 8 = 2^3), and such numbers must return false.

Input / Output

Input: one integer value.
Output: a boolean — true if value is a power of four, otherwise false.

#### Cons
traints

-2^31 <= value <= 2^31 - 1

Follow up: Can you decide it in constant time, without any loops or recursion?

#### Example
s & test cases
TEXT
Input: value = 64 -> Output: true
Why: 64 = 4^3, a genuine power of four.

Input: value = 1 -> Output: true
Why: 1 = 4^0; the zero exponent is allowed.

Input: value = 12 -> Output: false
Why: 12 has more than one set bit (it is not even a power of two).

Input: value = 8 -> Output: false
Why: 8 = 2^3 is a power of two but NOT a power of four — its single set bit sits at an odd position.

Input: value = 0 -> Output: false
Why: zero cannot be expressed as 4^k for any exponent.

Input: value = -16 -> Output: false
Why: negative numbers are never powers of four even though 16 is.

#### Approach

A power of four is first of all a power of two, so it must be positive and have exactly one set bit; the classic constant-time test for that is value > 0 && (value & (value - 1)) == 0. Among powers of two, the powers of four are exactly those whose single set bit sits at an even position (bit 0, 2, 4, ...). The neat arithmetic filter is that 4^k - 1 is always divisible by 3: since 4^k - 1 = (2^k - 1)(2^k + 1) and among the three consecutive integers 2^k - 1, 2^k, 2^k + 1 exactly one is a multiple of 3 (and it cannot be the power of two 2^k), the factor 3 must divide one of the other two, hence 4^k - 1. So checking (value - 1) % 3 == 0 on top of the power-of-two test isolates the powers of four. (An equivalent trick masks with 0x55555555 to require the set bit at an even position.)

Complexity

Time: O(1) — a fixed number of bitwise and arithmetic operations, no loops. Space: O(1) — only the input is used.

Python
PYTHON
class Solution:
    def isPowerOfFour(self, value: int) -> bool:
        # Positive, single set bit (power of two), and (value - 1) % 3 == 0.
        return value > 0 and (value & (value - 1)) == 0 and (value - 1) % 3 == 0
JavaScript
JAVASCRIPT
function isPowerOfFour(value) {
    // Positive, single set bit (power of two), and (value - 1) % 3 === 0.
    return value > 0 && (value & (value - 1)) === 0 && (value - 1) % 3 === 0;
}
Java
JAVA
class Solution {
    public boolean isPowerOfFour(int value) {
        // Positive, single set bit (power of two), and (value - 1) % 3 == 0.
        return value > 0 && (value & (value - 1)) == 0 && (value - 1) % 3 == 0;
    }
}
C++
CPP
class Solution {
public:
    bool isPowerOfFour(int value) {
        // Positive, single set bit (power of two), and (value - 1) % 3 == 0.
        return value > 0 && (value & (value - 1)) == 0 && (value - 1) % 3 == 0;
    }
};
C
C
#include <stdbool.h>

bool isPowerOfFour(int value) {
    // Positive, exactly one set bit (power of two), and (value - 1) divisible by 3.
    return value > 0 && (value & (value - 1)) == 0 && (value - 1) % 3 == 0;
}

#### Follow-up
questions
Solve it with no loops and no recursion (the O(1) bit-plus-modulo trick shown here).
Replace the % 3 filter with a bitmask (value & 0x55555555) test and explain why it works.
Generalize the function to "is value a power of a given base b?"
Extend to 64-bit inputs and discuss where the 32-bit mask/overflow assumptions break.

