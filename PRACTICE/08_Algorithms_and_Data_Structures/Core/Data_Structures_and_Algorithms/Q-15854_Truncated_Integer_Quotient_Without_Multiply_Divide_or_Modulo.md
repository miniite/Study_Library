---
type: "practice"
difficulty: "Other"
---

# Truncated Integer Quotient Without Multiply, Divide, or Modulo

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Bit Manipulation](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Bit_Manipulation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given two 32-bit signed integers, numerator and denominator. Compute the quotient obtained by dividing numerator by denominator without using the multiplication (*), division (/), or modulo (%) operators anywhere in your solution.

The division must truncate toward zero, meaning any fractional part is discarded rather than rounded. For instance, a true result of 8.9 truncates to 8, and a true result of -3.6 truncates to -3 (it moves toward zero, not toward negative infinity).

Because the answer must be representable in the 32-bit signed integer range [-2^31, 2^31 - 1], apply the following clamping rule:

If the exact quotient is strictly greater than 2^31 - 1, return 2^31 - 1.
If the exact quotient is strictly less than -2^31, return -2^31.

In practice, the only case where the mathematical quotient overflows this range is numerator = -2^31 and denominator = -1, whose true quotient 2^31 must be clamped to 2^31 - 1.

Return the truncated quotient as a 32-bit signed integer.

#### Cons
traints:

-2^31 <= numerator, denominator <= 2^31 - 1
denominator != 0
You may not use *, /, or % on the operands to perform the division (bit shifts, addition, and subtraction are allowed).

#### Example
s & test cases
TEXT
Input: numerator = 22, denominator = 5 -> Output: 4
Why: 22 / 5 = 4.4, and truncating toward zero drops the fraction to give 4.

Input: numerator = 17, denominator = -4 -> Output: -4
Why: 17 / -4 = -4.25; truncating toward zero (not down to -5) gives -4.

Input: numerator = -2147483648, denominator = -1 -> Output: 2147483647
Why: the true quotient 2^31 overflows the 32-bit signed range, so it is clamped to 2^31 - 1.

Input: numerator = 0, denominator = 9 -> Output: 0
Why: zero divided by any nonzero value is zero (edge: dividend is the minimum magnitude).

Input: numerator = -15, denominator = 4 -> Output: -3
Why: -15 / 4 = -3.75; truncation toward zero yields -3.

Input: numerator = 7, denominator = 7 -> Output: 1
Why: equal magnitudes give exactly 1, no fractional part to truncate.

#### Approach

Long division in binary via repeated doubling. Convert both operands to nonnegative 64-bit magnitudes (64-bit so that |-2^31| does not overflow) and track the result sign separately using an XOR of the operand signs. Then greedily subtract the largest shifted multiple of the divisor that still fits: starting from the divisor, keep doubling it (chunk << 1) together with a counter k = 1 (also doubled) as long as the doubled chunk does not exceed the remaining dividend. Subtract that chunk once, add k to the answer, and repeat with the reduced dividend. This is exactly binary long division: each outer iteration fixes one group of high-order quotient bits. Handle the single overflow case (-2^31 / -1) up front by returning INT_MAX.

Complexity

Time: O((log N)^2) where N is the magnitude of the numerator — the outer loop peels off at most O(log N) chunks and each inner doubling loop runs O(log N) times. Space: O(1), only a few integer accumulators are used.

Python
PYTHON
class Solution:
    def divide(self, numerator: int, denominator: int) -> int:
        INT_MIN, INT_MAX = -2**31, 2**31 - 1

        # Only overflow case: -2^31 / -1 = 2^31, which is clamped.
        if numerator == INT_MIN and denominator == -1:
            return INT_MAX

        sign = -1 if (numerator > 0) ^ (denominator > 0) else 1
        dvd, dvs = abs(numerator), abs(denominator)
        ans = 0

        while dvd >= dvs:
            chunk, k = dvs, 1        # chunk = a doubled multiple of the divisor
            while dvd >= (chunk << 1):
                chunk <<= 1
                k <<= 1
            dvd -= chunk
            ans += k

        return sign * ans
JavaScript
JAVASCRIPT
function divide(numerator, denominator) {

#### cons
t INT_MIN = -(2 ** 31);

#### cons
t INT_MAX = 2 ** 31 - 1;

    // Only overflow case: -2^31 / -1 = 2^31, which is clamped.
    if (numerator === INT_MIN && denominator === -1) return INT_MAX;

#### cons
t sign = ((numerator > 0) ^ (denominator > 0)) ? -1 : 1;

    let dvd = Math.abs(numerator);
    let dvs = Math.abs(denominator);
    let ans = 0;

    // Use *2 (via addition-free doubling) rather than << to avoid 32-bit
    // truncation of intermediate values that can reach 2^31.
    while (dvd >= dvs) {
        let chunk = dvs;
        let k = 1;
        while (dvd >= chunk + chunk) {
            chunk += chunk;
            k += k;
        }
        dvd -= chunk;
        ans += k;
    }

    return sign * ans;
}
Java
JAVA
class Solution {
    public int divide(int numerator, int denominator) {
        // Only overflow case: -2^31 / -1 = 2^31, which does not fit in an int.
        if (numerator == Integer.MIN_VALUE && denominator == -1)
            return Integer.MAX_VALUE;

        final int sign = ((numerator > 0) ^ (denominator > 0)) ? -1 : 1;

        long dvd = Math.abs((long) numerator);
        long dvs = Math.abs((long) denominator);
        long ans = 0;

        while (dvd >= dvs) {
            long chunk = dvs;   // current multiple of the divisor
            long k = 1;         // how many divisors "chunk" represents
            while (dvd >= (chunk << 1)) {
                chunk <<= 1;
                k <<= 1;
            }
            dvd -= chunk;
            ans += k;
        }

        return (int) (sign * ans);
    }
}
C++
CPP
#include <climits>
#include <cstdlib>

class Solution {
 public:
  int divide(int numerator, int denominator) {
    // Only overflow case: -2^31 / -1 = 2^31, which does not fit in an int.
    if (numerator == INT_MIN && denominator == -1)
      return INT_MAX;

#### cons
t int sign = ((numerator > 0) ^ (denominator > 0)) ? -1 : 1;

    long long dvd = llabs(static_cast<long long>(numerator));
    long long dvs = llabs(static_cast<long long>(denominator));
    long long ans = 0;

    while (dvd >= dvs) {
      long long chunk = dvs;  // current multiple of the divisor
      long long k = 1;        // how many divisors "chunk" represents
      while (dvd >= (chunk << 1)) {
        chunk <<= 1;
        k <<= 1;
      }
      dvd -= chunk;
      ans += k;
    }

    return static_cast<int>(sign * ans);
  }
};
C
C
#include <limits.h>
#include <stdlib.h>

int divide(int numerator, int denominator) {
    // Only overflow case: -2^31 / -1 = 2^31, which does not fit in an int.
    if (numerator == INT_MIN && denominator == -1)
        return INT_MAX;

    int sign = ((numerator > 0) ^ (denominator > 0)) ? -1 : 1;

    // Work in 64-bit magnitudes so |INT_MIN| does not overflow.
    long long dvd = llabs((long long)numerator);
    long long dvs = llabs((long long)denominator);
    long long ans = 0;

    while (dvd >= dvs) {
        long long chunk = dvs;   // current multiple of the divisor
        long long k = 1;         // how many divisors "chunk" represents
        while (dvd >= (chunk << 1)) {
            chunk <<= 1;
            k <<= 1;
        }
        dvd -= chunk;
        ans += k;
    }

    return (int)(sign * ans);
}

#### Follow-up
questions
Solve it if only 32-bit arithmetic is available (no 64-bit type) — how do you avoid the |INT_MIN| overflow without casting up?
Return both quotient and remainder, still without using *, /, or %.
What if bit shifts were also disallowed and you could only use addition and subtraction? Discuss why that degrades to O(N).
Enumerate every input pair that can overflow and prove the code handles each.

