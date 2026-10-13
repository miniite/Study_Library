---
type: "practice"
difficulty: "Other"
---

# Selectable Subset Whose Bitwise OR Is Even

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Bit Manipulation](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Bit_Manipulation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given an array values of positive integers. Decide whether you can choose two or more of its elements so that the bitwise OR of the chosen elements is even — that is, its binary representation ends in at least one 0.

For intuition: the number 9 is 1001 in binary and ends in a 1 (no trailing zero), while 12 is 1100 and ends in two 0s (two trailing zeros). The bitwise OR of a group of numbers ends in a 0 exactly when every chosen number is even, because the lowest bit of an OR becomes 1 as soon as any single operand has its lowest bit set.

Return true if such a selection of two or more elements exists, and false otherwise.

Input

values: an array of positive integers.

Output

A boolean: true if some subset of size at least 2 has an even bitwise OR, otherwise false.

#### Cons
traints

2 <= values.length <= 100
1 <= values[i] <= 100

#### Example
s & test cases
TEXT
Input: values = [7, 2, 3, 6, 11] -> Output: true
Why: 2 and 6 are both even; 2 OR 6 = 6 ("110"), which ends in a zero. A normal case where two evens are present.

Input: values = [3, 5, 9, 13] -> Output: false
Why: every element is odd, so any OR keeps the lowest bit set and can never be even. All-odd edge case.

Input: values = [10, 20, 4] -> Output: true
Why: three even numbers; for example 10 OR 20 = 30 ("11110"), which is even. Shows more than two evens still works.

Input: values = [8, 3] -> Output: false
Why: only one even number (8); you need at least two, and 8 OR 3 = 11 ("1011") is odd. Exactly one even at minimum length.

Input: values = [2, 2] -> Output: true
Why: duplicate even values both qualify; 2 OR 2 = 2 ("10") is even. Duplicates at minimum length.

Input: values = [1, 1] -> Output: false
Why: both odd; the only pair ORs to 1. All-odd at the minimum array length.

#### Approach

The lowest bit of a bitwise OR is 0 only when every operand contributes a 0 in that position, i.e. when every chosen number is even. So the OR of a selected group is even if and only if all chosen numbers are even. The cheapest way to satisfy "two or more elements with an even OR" is therefore simply to pick two even numbers. The whole problem collapses to: does the array contain at least two even values? Count evens in one pass and return true the moment the count reaches two. The underlying pattern is reasoning about a bitwise operation at the single least-significant-bit level and reducing a subset-search question to a simple counting condition.

Complexity

Time: O(n) — a single pass over the array, stopping as soon as the second even number is seen. Space: O(1) — only a running counter of even numbers is kept.

Python
PYTHON
from typing import List

def has_even_bitwise_or(values: List[int]) -> bool:
    evens = 0
    for v in values:
        if v % 2 == 0:
            evens += 1
            if evens == 2:
                return True
    return False
JavaScript
JAVASCRIPT
function hasEvenBitwiseOr(values) {
    let evens = 0;
    for (const v of values) {
        if ((v & 1) === 0 && ++evens === 2) return true;
    }
    return false;
}
Java
JAVA
class Solution {
    public boolean hasEvenBitwiseOr(int[] values) {
        int evens = 0;
        for (int v : values) {
            if ((v & 1) == 0 && ++evens == 2) return true;
        }
        return false;
    }
}
C++
CPP
#include <vector>
using namespace std;

class Solution {
public:
    bool hasEvenBitwiseOr(vector<int>& values) {
        int evens = 0;
        for (int v : values) {
            if ((v & 1) == 0 && ++evens == 2) return true;
        }
        return false;
    }
};
C
C
#include <stdbool.h>

bool hasEvenBitwiseOr(int* values, int valuesSize) {
    int evens = 0;
    for (int i = 0; i < valuesSize; i++) {
        if ((values[i] & 1) == 0 && ++evens == 2) return true;
    }
    return false;
}

#### Follow-up
questions
Prove that exactly two even numbers always suffice — why is a larger even subset never required?
What if the OR had to have at least k trailing zeros instead of just one? How does the counting condition change per bit position?
Suppose instead you needed a subset whose OR is odd — what is the simplest condition then?
Can you answer in a single pass with O(1) extra space and no early sorting? (Yes — count evens and short-circuit at two.)

