---
type: "practice"
difficulty: "Other"
---

# Reverse the Digits of a Machine Integer

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Math](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Math.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given a value n that fits inside a signed 32-bit integer, meaning it lies in the inclusive range [-2^31, 2^31 - 1].

Return the number formed by writing the decimal digits of n in reverse order while preserving its original sign. Any zeros that end up at the front after reversal are dropped naturally (for example, reversing 4560 gives 654, not 0654).

There is one important restriction: your solution must work as if no integer type wider than 32 bits is available. You may not accumulate the reversed value in a 64-bit type and simply check the range afterwards — you must detect that the result would leave the signed 32-bit range before it actually overflows.

If reversing n would produce a value outside [-2^31, 2^31 - 1], return 0 instead.

Input

A single integer n.

Output

The sign-preserving digit reversal of n, or 0 if that reversal does not fit in a signed 32-bit integer.

#### Cons
traints

-2^31 <= n <= 2^31 - 1 (that is, -2147483648 <= n <= 2147483647).
Only 32-bit-wide integer arithmetic may be used; no wider intermediate type and no big-integer accumulation-then-check.

#### Example
s & test cases
TEXT
Input: n = 574 -> Output: 475
Why: Straightforward positive number; digits 5-7-4 become 4-7-5.

Input: n = -418 -> Output: -814
Why: The sign must be preserved; only the digit order flips.

Input: n = 3900 -> Output: 93
Why: Trailing zeros become leading zeros on reversal and vanish (0093 -> 93).

Input: n = 0 -> Output: 0
Why: Zero reverses to itself; smallest-magnitude edge case.

Input: n = 1563847412 -> Output: 0
Why: The reversed digits form 2147483651, which is greater than 2^31 - 1, so the answer overflows and 0 is returned.

Input: n = -2147483648 -> Output: 0
Why: The most negative 32-bit value; its reversed magnitude (8463847412) is far out of range, so return 0.

#### Approach

Peel the number one decimal digit at a time from the low end using n % 10 (which carries n's sign in truncating languages) and n /= 10, and rebuild it in rev by shifting left one place each step (rev = rev * 10 + digit). The key subtlety is honoring the "no 64-bit storage" rule: before every rev * 10 + digit, compare rev against INT_MAX/10 and INT_MIN/10, and when rev equals that boundary, compare the incoming digit against the last digit of the limit (7 for the positive bound, -8 for the negative bound). This guard rejects any input whose reversal would overflow before the overflow can occur. Python uses unbounded integers, so it reverses the magnitude and range-checks once at the end, agreeing on every result.

Complexity

Time: O(d) where d is the number of decimal digits in n (at most 10 for a 32-bit integer), since each iteration strips one digit. Space: O(1) — only a constant number of scalar variables is used.

Python
PYTHON
class Solution:
    def reverse(self, n: int) -> int:
        INT_MIN, INT_MAX = -2**31, 2**31 - 1
        sign = -1 if n < 0 else 1
        n = abs(n)  # safe in Python: unbounded integers, no overflow on -2**31

        rev = 0
        while n:
            rev = rev * 10 + n % 10
            n //= 10

        rev *= sign
        return 0 if rev < INT_MIN or rev > INT_MAX else rev
JavaScript
JAVASCRIPT
function reverse(n) {

#### cons
t INT_MAX = 2147483647;   // 2**31 - 1

#### cons
t INT_MIN = -2147483648;  // -2**31
    let rev = 0;
    while (n !== 0) {

#### cons
t digit = n % 10;          // same sign as n
        n = Math.trunc(n / 10);        // truncate toward zero

        if (rev > 214748364 || (rev === 214748364 && digit > 7)) return 0;
        if (rev < -214748364 || (rev === -214748364 && digit < -8)) return 0;

        rev = rev * 10 + digit;
    }
    return rev;
}
Java
JAVA
class Solution {
    // Sign-preserving digit reversal; returns 0 on 32-bit overflow.
    // Overflow is caught before it happens, using only int arithmetic.
    public int reverse(int n) {
        int rev = 0;
        while (n != 0) {
            int digit = n % 10;    // same sign as n
            n /= 10;               // truncates toward zero

            if (rev > Integer.MAX_VALUE / 10 ||
                (rev == Integer.MAX_VALUE / 10 && digit > 7))
                return 0;
            if (rev < Integer.MIN_VALUE / 10 ||
                (rev == Integer.MIN_VALUE / 10 && digit < -8))
                return 0;

            rev = rev * 10 + digit;
        }
        return rev;
    }
}
C++
CPP
#include <climits>

class Solution {
 public:
    // Sign-preserving digit reversal; returns 0 on 32-bit overflow.
    // Detects overflow before it occurs, using only 32-bit arithmetic.
    int reverse(int n) {
        int rev = 0;
        while (n != 0) {
            int digit = n % 10;    // same sign as n
            n /= 10;               // truncates toward zero

            if (rev > INT_MAX / 10 || (rev == INT_MAX / 10 && digit > 7))
                return 0;
            if (rev < INT_MIN / 10 || (rev == INT_MIN / 10 && digit < -8))
                return 0;

            rev = rev * 10 + digit;
        }
        return rev;
    }
};
C
C
#include <limits.h>

/* Returns the sign-preserving digit reversal of n, or 0 on 32-bit overflow.
   Uses only 32-bit arithmetic: overflow is detected before it happens. */
int reverse(int n) {
    int rev = 0;
    while (n != 0) {
        int digit = n % 10;      /* has the same sign as n */
        n /= 10;                 /* truncates toward zero */

        /* Would rev * 10 + digit exceed INT_MAX? */
        if (rev > INT_MAX / 10 || (rev == INT_MAX / 10 && digit > 7))
            return 0;
        /* Would rev * 10 + digit fall below INT_MIN? */
        if (rev < INT_MIN / 10 || (rev == INT_MIN / 10 && digit < -8))
            return 0;

        rev = rev * 10 + digit;
    }
    return rev;
}

#### Follow-up
questions
How would your solution change if a 64-bit type were allowed — would you still prefer the pre-multiplication overflow check, and why?
Reverse the digits of an arbitrary-precision (big) integer instead, where no fixed range applies.
Reverse the number in a given base b (e.g., base 2 or base 16) rather than base 10.
Explain exactly why n % 10 and n / 10 behave correctly for negative inputs in truncating languages, and how you handle the language where % follows the divisor's sign.

