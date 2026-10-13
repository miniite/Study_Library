---
type: "practice"
difficulty: "Other"
---

# Flatten an Array of Integers Into Their Digit Sequence

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Math](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Math.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given an array numbers of positive integers. Build and return a new array digits that contains the individual decimal digits of every integer in numbers, expanded in place and preserving the original left-to-right order.

To expand an integer means to list its decimal digits from most significant to least significant, exactly as it is written. For example, the integer 40923 expands to [4, 0, 9, 2, 3], and a single-digit integer such as 7 expands to [7].

Process the integers in the order they appear in numbers, appending each integer's expanded digits to the result before moving on to the next integer.

Input

numbers: an array of positive integers.

Output

An array digits of single-digit integers (each between 0 and 9), formed by concatenating the digit expansion of each element of numbers in order.

#### Cons
traints

1 <= numbers.length <= 1000
1 <= numbers[i] <= 10^5

#### Example
s & test cases
TEXT
Input: numbers = [42, 7, 305] -> Output: [4, 2, 7, 3, 0, 5]
Why: 42 -> [4,2], 7 -> [7], 305 -> [3,0,5]; a normal mix of two-, one-, and three-digit values, and 305 exercises an interior zero.

Input: numbers = [9, 1] -> Output: [9, 1]
Why: Every element is a single digit, so the output is identical to the input; confirms no over-splitting.

Input: numbers = [100000] -> Output: [1, 0, 0, 0, 0, 0]
Why: The maximum allowed value 10^5 has trailing zeros; verifies leading digit plus all zeros are emitted, not dropped.

Input: numbers = [508, 60] -> Output: [5, 0, 8, 6, 0]
Why: Interior and trailing zeros across multiple numbers; ensures zeros are never skipped.

Input: numbers = [7] -> Output: [7]
Why: Minimum-length input with one single-digit value; smallest edge case.

#### Approach

Straightforward simulation: walk the input in order and, for each integer, emit its digits most-significant-first, appending them to a growing result. Two equivalent techniques appear across the languages. The string technique (Python, JS) converts the integer to its decimal string, which is already in most-significant-first order, then maps each character to its numeric value — no reversal needed and interior/trailing zeros are preserved automatically. The arithmetic technique (C, C++, Java) repeatedly takes x % 10 to peel the least-significant digit and x /= 10 to drop it, collecting digits in reverse and then flipping them (or filling from the back) so the output is most-significant-first. A do/while loop is used so that a value like 0 or any single-digit number still emits exactly one digit. The core pattern is digit extraction / base-10 decomposition, and the only subtlety is order (reverse the peeled digits) and not losing zeros.

Complexity

Time: O(N * D) where N is the number of integers and D is the maximum digit count (at most 6 here), i.e. linear in the total number of output digits. Space: O(total digits) for the output array (the C/C++/Java variants use only O(D) extra scratch beyond the result). Justification: each digit of each input integer is produced exactly once.

Python
PYTHON
from typing import List

def separate_digits(numbers: List[int]) -> List[int]:
    digits = []
    for x in numbers:
        for ch in str(x):          # str keeps most-significant-first order
            digits.append(int(ch))
    return digits
JavaScript
JAVASCRIPT
function separateDigits(numbers) {

#### cons
t digits = [];
    for (const x of numbers) {
        for (const ch of String(x)) {   // string preserves digit order, including zeros
            digits.push(Number(ch));
        }
    }
    return digits;
}
Java
JAVA
import java.util.ArrayList;
import java.util.List;

class Solution {
    public int[] separateDigits(int[] numbers) {
        List<Integer> tmp = new ArrayList<>();
        for (int x : numbers) {
            int start = tmp.size();
            do { tmp.add(x % 10); x /= 10; } while (x > 0);   // least-significant first
            for (int lo = start, hi = tmp.size() - 1; lo < hi; lo++, hi--) {
                int t = tmp.get(lo); tmp.set(lo, tmp.get(hi)); tmp.set(hi, t);  // reverse
            }
        }
        int[] digits = new int[tmp.size()];
        for (int i = 0; i < digits.length; i++) digits[i] = tmp.get(i);
        return digits;
    }
}
C++
CPP
#include <vector>
using namespace std;

class Solution {
public:
    vector<int> separateDigits(vector<int>& numbers) {
        vector<int> digits;
        for (int x : numbers) {
            int buf[8], c = 0;                 // extract least-significant first
            do { buf[c++] = x % 10; x /= 10; } while (x > 0);
            for (int t = c - 1; t >= 0; --t)   // then reverse to most-significant order
                digits.push_back(buf[t]);
        }
        return digits;
    }
};
C
C
#include <stdlib.h>

/* Returns a malloc'd array of all decimal digits of every element, in order. */
int* separateDigits(int* numbers, int numbersSize, int* returnSize) {
    int total = 0;
    for (int i = 0; i < numbersSize; i++) {
        int x = numbers[i], d = 0;
        do { d++; x /= 10; } while (x > 0);   /* count digits (handles single digits) */
        total += d;
    }
    int* digits = (int*)malloc((total > 0 ? total : 1) * sizeof(int));
    int idx = 0;
    for (int i = 0; i < numbersSize; i++) {
        int x = numbers[i];
        int buf[8], c = 0;                     /* max value 10^5 -> 6 digits, buf is safe */
        do { buf[c++] = x % 10; x /= 10; } while (x > 0);
        for (int t = c - 1; t >= 0; t--) digits[idx++] = buf[t];  /* reverse into output */
    }
    *returnSize = total;
    return digits;
}

#### Follow-up
questions
What if the integers can be arbitrarily large (beyond 64-bit), so you must treat the input as strings rather than ints?
How would you also return, alongside the flattened digits, the digit sum of each original integer?
Can you produce the result as a single flat array without any intermediate per-number buffer, writing digits directly into a pre-sized output?
If negative integers were allowed, how would you decide whether to emit a sign marker or take the absolute value first?

