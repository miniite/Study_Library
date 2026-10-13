---
type: "practice"
difficulty: "Other"
---

# Total Adjacent Character-Code Variation of a String

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ String](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/String.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given a string t consisting of lowercase English letters.

Define the variation total of t as the sum, taken over every pair of neighboring characters, of the absolute difference between their character codes (ASCII values). In other words, if t has characters t[0], t[1], ..., t[m-1], the variation total is:

CODE
|code(t[1]) - code(t[0])| + |code(t[2]) - code(t[1])| + ... + |code(t[m-1]) - code(t[m-2])|

where code(c) is the ASCII value of character c.

Return the variation total of t.

Input

t: a string of lowercase English letters.

Output

An integer, the variation total of t.

#### Cons
traints

2 <= t.length <= 100
t consists only of lowercase English letters.

#### Example
s & test cases
TEXT
- Input: `t = "abc"` -> Output: `2`
  Why: codes are `a=97, b=98, c=99`, so `|98-97| + |99-98| = 1 + 1 = 2` (simple increasing run).
- Input: `t = "aza"` -> Output: `50`
  Why: codes are `a=97, z=122`, so `|122-97| + |97-122| = 25 + 25 = 50` (large swings in both directions).
- Input: `t = "aa"` -> Output: `0`
  Why: identical adjacent characters contribute `|97-97| = 0`; minimum-length all-equal case.
- Input: `t = "abz"` -> Output: `25`
  Why: `|98-97| + |122-98| = 1 + 24 = 25` (mixes a small and a large gap).
- Input: `t = "ba"` -> Output: `1`
  Why: `|97-98| = 1`; a single decreasing pair, confirming the absolute value handles descending order.

#### Approach

Walk through the string once, and for each index i from 1 to m-1 add the absolute difference between the character codes of t[i] and t[i-1] to a running total. Character codes come directly from the character type (a char is its ASCII value), so no lookup table is needed. This is the straightforward linear-scan / adjacent-pair accumulation pattern; there is no cleverer approach because every adjacent pair independently contributes and all must be summed.

Complexity

Time: O(m) where m = t.length — a single pass visits each adjacent pair once. Space: O(1) — only a running accumulator is kept, independent of input size.

Python
PYTHON
class Solution:
    def variationTotal(self, t: str) -> int:
        return sum(abs(ord(t[i]) - ord(t[i - 1])) for i in range(1, len(t)))
JavaScript
JAVASCRIPT
function variationTotal(t) {
    let total = 0;
    for (let i = 1; i < t.length; ++i)
        total += Math.abs(t.charCodeAt(i) - t.charCodeAt(i - 1));
    return total;
}
Java
JAVA
class Solution {
  public int variationTotal(String t) {
    int total = 0;
    for (int i = 1; i < t.length(); i++)
      total += Math.abs(t.charAt(i) - t.charAt(i - 1));
    return total;
  }
}
C++
CPP
#include <string>
#include <cstdlib>
using namespace std;

class Solution {
 public:
  int variationTotal(string t) {
    int total = 0;
    for (size_t i = 1; i < t.size(); ++i)
      total += abs(t[i] - t[i - 1]);
    return total;
  }
};
C
C
int variationTotal(char* t) {
    int total = 0;
    for (int i = 1; t[i] != '\0'; ++i) {
        int d = (int)t[i] - (int)t[i - 1];
        total += d < 0 ? -d : d;     /* absolute difference of adjacent codes */
    }
    return total;
}

#### Follow-up
questions
How would the solution change if the string could contain uppercase letters, digits, or arbitrary Unicode characters?
Instead of the sum, return the maximum single adjacent difference and the pair of characters that produced it.
If you may delete exactly one character, which deletion minimizes (or maximizes) the resulting variation total?
Handle degenerate inputs gracefully: what should the answer be for a length-0 or length-1 string? (It is 0 — no adjacent pairs exist.)

