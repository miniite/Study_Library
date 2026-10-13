---
type: "practice"
difficulty: "Other"
---

# Sum of Divisors for Values Having Exactly Four Factors

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Math](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Math.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given an array of positive integers values. A positive integer is said to be four-factored if it has exactly four distinct positive divisors (including 1 and the number itself).

For every element of values that is four-factored, add up all four of its divisors. Return the grand total of those divisor sums, taken over every four-factored element in the array (an element that appears more than once is counted each time it appears).

If no element of values is four-factored, return 0.

Input
values: an array of positive integers.
Output
A single integer: the sum of divisors, aggregated across every four-factored element of values.

#### Cons
traints
1 <= values.length <= 10^4
1 <= values[i] <= 10^5
Notes
The four divisors of a number x are the four distinct positive integers that divide x evenly.
A number is four-factored in exactly two situations: it is the cube of a prime (p^3, divisors 1, p, p^2, p^3), or it is the product of two distinct primes (p * q, divisors 1, p, q, p*q). Any other value has a different divisor count.

#### Example
s & test cases
TEXT
Input: values = [15, 4, 7] -> Output: 24
Why: 15 has divisors 1, 3, 5, 15 (exactly four, sum 24); 4 has three divisors and 7 has two, so neither contributes.

Input: values = [15, 15] -> Output: 48
Why: 15 is four-factored and appears twice, so its divisor sum 24 is counted twice.

Input: values = [1, 2, 3, 4, 5] -> Output: 0
Why: divisor counts are 1, 2, 2, 3, 2 respectively; none equals four, so the answer is 0.

Input: values = [8] -> Output: 15
Why: 8 = 2^3 has divisors 1, 2, 4, 8 (a prime cube, exactly four), summing to 15.

Input: values = [6] -> Output: 12
Why: 6 = 2 * 3 has divisors 1, 2, 3, 6 (product of two distinct primes), summing to 12.

Input: values = [12] -> Output: 0
Why: 12 has divisors 1, 2, 3, 4, 6, 12 (six divisors), which is more than four.

#### Approach

For each value, enumerate divisors by trial division only up to sqrt(v): whenever i divides v, it contributes the pair i and v/i (or a single divisor when i*i == v). Track both the running divisor count and their running sum, and break the moment the count exceeds four (an early-exit optimization, since any value with more than four divisors can never qualify). If the final count is exactly four, add that value's divisor sum into the answer. The underlying pattern is classic O(sqrt(n)) divisor enumeration with paired counting.

Complexity

Time: O(n * sqrt(M)), where n is the array length and M is the maximum value; each element is factored by trial division up to its square root, with an early break once more than four divisors appear. Space: O(1) extra beyond the input.

Python
PYTHON
class Solution:
    def sumFourFactorDivisors(self, values):
        total = 0
        for v in values:
            count = 0
            div_sum = 0
            i = 1
            while i * i <= v:
                if v % i == 0:
                    if i * i == v:          # square root divisor counted once
                        count += 1
                        div_sum += i
                    else:                   # i and v // i are two distinct divisors
                        count += 2
                        div_sum += i + v // i
                    if count > 4:
                        break
                i += 1
            if count == 4:
                total += div_sum
        return total
JavaScript
JAVASCRIPT
function sumFourFactorDivisors(values) {
    let total = 0;
    for (const v of values) {
        let count = 0;
        let divSum = 0;
        for (let i = 1; i * i <= v; ++i) {
            if (v % i === 0) {
                if (i * i === v) { count += 1; divSum += i; }
                else             { count += 2; divSum += i + v / i; }
                if (count > 4) break;
            }
        }
        if (count === 4) total += divSum;
    }
    return total;
}
Java
JAVA
class Solution {
    public int sumFourFactorDivisors(int[] values) {
        int total = 0;
        for (int v : values) {
            int count = 0;
            long divSum = 0;
            for (long i = 1; i * i <= v; ++i) {
                if (v % i == 0) {
                    if (i * i == v) { count += 1; divSum += i; }
                    else            { count += 2; divSum += i + v / i; }
                    if (count > 4) break;
                }
            }
            if (count == 4) total += (int) divSum;
        }
        return total;
    }
}
C++
CPP
#include <vector>
using namespace std;

class Solution {
public:
    int sumFourFactorDivisors(vector<int>& values) {
        int total = 0;
        for (int v : values) {
            int count = 0;
            long long divSum = 0;
            for (long long i = 1; i * i <= v; ++i) {
                if (v % i == 0) {
                    if (i * i == v) { count += 1; divSum += i; }
                    else            { count += 2; divSum += i + v / i; }
                    if (count > 4) break;
                }
            }
            if (count == 4) total += static_cast<int>(divSum);
        }
        return total;
    }
};
C
C
#include <stdio.h>

int sumFourFactorDivisors(int* values, int valuesSize) {
    int total = 0;
    for (int k = 0; k < valuesSize; ++k) {
        int v = values[k];
        int count = 0;
        long long divSum = 0;
        for (long long i = 1; i * i <= v; ++i) {
            if (v % i == 0) {
                if (i * i == v) {          /* perfect-square factor counted once */
                    count += 1;
                    divSum += i;
                } else {
                    count += 2;            /* i and v/i are two distinct factors */
                    divSum += i + v / i;
                }
                if (count > 4) break;      /* early exit: too many divisors */
            }
        }
        if (count == 4) total += (int)divSum;
    }
    return total;
}

#### Follow-up
questions
Generalize to "exactly k divisors" for an arbitrary k — how does the early-break threshold and reasoning change?
If the same query must run for millions of values bounded by 10^5, how would a precomputed smallest-prime-factor sieve let you answer each value in near-constant time?
Return the actual four divisors (not just their sum) for every qualifying value.
What is the tightest characterization of four-factored numbers, and how can it replace divisor enumeration entirely?

