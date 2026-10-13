---
type: "practice"
difficulty: "Other"
---

# The p-th Arrangement of the First d Digits in Dictionary Order

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Recursion](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Recursion.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Cons
ider the digits 1, 2, 3, ..., d. There are exactly d! distinct ways to arrange all of them into an ordered string, and each such string uses every digit once.

Imagine writing out all d! arrangements sorted in increasing dictionary (lexicographic) order and numbering them starting from 1. For example, when d = 3 the ordered list is:

CODE
1. "123"
2. "132"
3. "213"
4. "231"
5. "312"
6. "321"

Given d and a position p, return the arrangement that sits at position p in this ordered list, as a string.

Input

An integer d — how many digits (1 through d) are being arranged.
An integer p — the 1-based rank of the desired arrangement.

Output

The string of length d that is the p-th arrangement in dictionary order.

#### Cons
traints

1 <= d <= 9
1 <= p <= d!

#### Example
s & test cases
TEXT
Input: d = 3, p = 3 -> Output: "213"
Why: The third entry in the ordered list of the six arrangements of 1,2,3.

Input: d = 4, p = 9 -> Output: "2314"
Why: With d! = 24 arrangements, the 9th in dictionary order fixes 2 as the leading digit.

Input: d = 3, p = 1 -> Output: "123"
Why: Position 1 is always the fully ascending arrangement.

Input: d = 1, p = 1 -> Output: "1"
Why: A single digit has exactly one arrangement.

Input: d = 4, p = 24 -> Output: "4321"
Why: The last position (p = d!) is always the fully descending arrangement.

Input: d = 9, p = 362880 -> Output: "987654321"
Why: Maximum-size case; p = 9! selects the final (descending) arrangement, exercising the largest factorial.

#### Approach

Use the factorial number system. Fixing the leading digit accounts for (d-1)! arrangements, so the leading digit is index p / (d-1)! into the sorted list of unused digits; the remainder p % (d-1)! becomes the rank within that block. Convert p to 0-based first. Precompute factorials, keep the remaining digits in ascending order, and at each step select idx = p / factorial[i], append that digit, remove it, and set p %= factorial[i]. This directly decodes the rank without enumerating permutations — the factorial-number-system / Lehmer-code technique.

Complexity

Time: O(d^2) — d iterations, each removing one element from the list of remaining digits in O(d). Space: O(d) for the factorial table, the digit list, and the output string.

Python
PYTHON
def find_arrangement(d, p):
    factorial = [1] * (d + 1)
    for i in range(1, d + 1):
        factorial[i] = factorial[i - 1] * i

    digits = [str(i) for i in range(1, d + 1)]
    result = []
    p -= 1  # 0-indexed
    for i in range(d - 1, -1, -1):
        idx = p // factorial[i]
        p %= factorial[i]
        result.append(digits.pop(idx))
    return ''.join(result)
JavaScript
JAVASCRIPT
function findArrangement(d, p) {

#### cons
t factorial = [1];
    for (let i = 1; i <= d; i++) factorial[i] = factorial[i - 1] * i;

#### cons
t digits = [];
    for (let i = 1; i <= d; i++) digits.push(i);

    let result = '';
    p -= 1; // 0-indexed
    for (let i = d - 1; i >= 0; i--) {

#### cons
t idx = Math.floor(p / factorial[i]);
        p %= factorial[i];
        result += digits[idx];
        digits.splice(idx, 1);
    }
    return result;
}
Java
JAVA
import java.util.ArrayList;
import java.util.List;

class Solution {
    public String findArrangement(int d, int p) {
        int[] factorial = new int[d + 1];
        factorial[0] = 1;
        for (int i = 1; i <= d; i++) factorial[i] = factorial[i - 1] * i;

        List<Integer> digits = new ArrayList<>();
        for (int i = 1; i <= d; i++) digits.add(i);

        StringBuilder sb = new StringBuilder();
        p -= 1; // 0-indexed
        for (int i = d - 1; i >= 0; i--) {
            int idx = p / factorial[i];
            p %= factorial[i];
            sb.append(digits.get(idx));
            digits.remove(idx);
        }
        return sb.toString();
    }
}
C++
CPP
#include <string>
#include <vector>
using namespace std;

string findArrangement(int d, long long p) {
    vector<long long> factorial(d + 1, 1);
    for (int i = 2; i <= d; ++i) factorial[i] = factorial[i - 1] * i;

    vector<int> digits;
    for (int i = 1; i <= d; ++i) digits.push_back(i);

    string result;
    p -= 1;  // 0-indexed
    for (int i = d - 1; i >= 0; --i) {
        int idx = (int)(p / factorial[i]);
        p %= factorial[i];
        result += to_string(digits[idx]);
        digits.erase(digits.begin() + idx);
    }
    return result;
}
C
C
#include <stdlib.h>

/* Returns a malloc'd string of length d; caller frees it. */
char* findArrangement(int d, long long p) {
    long long factorial[11];
    factorial[0] = 1;
    for (int i = 1; i <= d; ++i) factorial[i] = factorial[i - 1] * i;

    int digits[10];
    for (int i = 0; i < d; ++i) digits[i] = i + 1;
    int count = d;

    char* result = (char*)malloc((d + 1) * sizeof(char));
    p -= 1;  /* 0-indexed */
    int pos = 0;
    for (int i = d - 1; i >= 0; --i) {
        int idx = (int)(p / factorial[i]);
        p %= factorial[i];
        result[pos++] = (char)('0' + digits[idx]);
        for (int j = idx; j < count - 1; ++j) digits[j] = digits[j + 1];
        --count;
    }
    result[pos] = '\0';
    return result;
}

#### Follow-up
questions
Given an arrangement, return its 1-based rank (the inverse mapping via the Lehmer code).
Handle arranging a multiset of digits with duplicates in dictionary order.
Return the p-th arrangement of an arbitrary sorted list of distinct symbols, not just digits 1..d.
Discuss why building the answer greedily is far cheaper than repeatedly calling a next-permutation routine p times.

