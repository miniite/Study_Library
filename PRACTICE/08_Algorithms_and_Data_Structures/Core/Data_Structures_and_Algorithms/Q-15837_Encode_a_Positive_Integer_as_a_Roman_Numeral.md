---
type: "practice"
difficulty: "Other"
---

# Encode a Positive Integer as a Roman Numeral

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Hash Table](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Hash_Table.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Roman numerals use seven letters, each standing for a fixed value:

Letter	Value
I	1
V	5
X	10
L	50
C	100
D	500
M	1000

A number is written by combining these letters from the largest value down to the smallest, following these rules:

Letters are normally written in non-increasing order of value and their values are added up (for example XVIII = 10 + 5 + 1 + 1 + 1 = 18).
The letters I, X, C, and M may be repeated at most three times in a row. The letters V, L, and D are never repeated.
To write a digit that would otherwise need four repeats, use the subtractive form: a smaller-value letter placed immediately before a larger one means "subtract". Only these six subtractive combinations are allowed: IV = 4, IX = 9, XL = 40, XC = 90, CD = 400, CM = 900.
The conversion is done digit place by digit place (thousands, then hundreds, then tens, then ones), each place converted independently.

Given a positive integer value, return its Roman-numeral representation as a string.

Input

value: a positive integer.

Output

A string containing the Roman-numeral form of value.

#### Cons
traints

1 <= value <= 3999

#### Example
s & test cases
TEXT
Input: value = 2764 -> Output: "MMDCCLXIV"
Why: 2000 = MM, 700 = DCC, 60 = LX, 4 = IV (subtractive) — combines additive and subtractive forms.

Input: value = 9 -> Output: "IX"
Why: 9 uses the subtractive form (one less than ten), not `VIIII`.

Input: value = 44 -> Output: "XLIV"
Why: two subtractive forms in one number: 40 = XL and 4 = IV.

Input: value = 1 -> Output: "I"
Why: the minimum input; a single unit letter.

Input: value = 3888 -> Output: "MMMDCCCLXXXVIII"
Why: near the upper bound and the longest possible output (15 letters): 3000 = MMM, 800 = DCCC, 80 = LXXX, 8 = VIII.

Input: value = 400 -> Output: "CD"
Why: exercises the hundreds-place subtractive form on its own.

#### Approach

Greedy largest-first subtraction over a precomputed value-to-symbol table. List every emittable chunk — the seven base letters plus the six subtractive pairs — in strictly decreasing value order. Walk the table from the largest value down; while the remaining number is at least the current value, append that symbol and subtract its value. Including the subtractive pairs (900, 400, 90, 40, 9, 4) directly in the table means the ordinary "peel off the biggest coin that fits" greedy handles the 4-and-9 special cases automatically, with no digit-by-digit branching. This is the same pattern as canonical coin representation, which is greedy-optimal precisely because this denomination set is canonical.

Complexity

Time: O(1) — the value/symbol table has a fixed 13 entries and the total number of letters emitted is bounded (at most 15 for any input up to 3999), so the work does not grow with the input magnitude. Space: O(1) — the output string has bounded length and only the fixed table is stored.

Python
PYTHON
def to_roman(value: int) -> str:
    table = [(1000, "M"), (900, "CM"), (500, "D"), (400, "CD"), (100, "C"),
             (90, "XC"), (50, "L"), (40, "XL"), (10, "X"), (9, "IX"),
             (5, "V"), (4, "IV"), (1, "I")]
    parts = []
    for v, s in table:
        if value == 0:
            break
        count, value = divmod(value, v)
        parts.append(s * count)
    return "".join(parts)
JavaScript
JAVASCRIPT
function toRoman(value) {

#### cons
t table = [
        [1000, "M"], [900, "CM"], [500, "D"], [400, "CD"], [100, "C"],
        [90, "XC"], [50, "L"], [40, "XL"], [10, "X"], [9, "IX"],
        [5, "V"], [4, "IV"], [1, "I"],
    ];
    let out = "";
    for (const [v, s] of table) {
        while (value >= v) {
            out += s;
            value -= v;
        }
    }
    return out;
}
Java
JAVA
class Solution {
    public String toRoman(int value) {
        int[] vals = {1000, 900, 500, 400, 100, 90, 50, 40, 10, 9, 5, 4, 1};
        String[] syms = {"M", "CM", "D", "CD", "C", "XC",
                         "L", "XL", "X", "IX", "V", "IV", "I"};
        StringBuilder sb = new StringBuilder();
        for (int i = 0; i < vals.length; i++) {
            while (value >= vals[i]) {
                sb.append(syms[i]);
                value -= vals[i];
            }
        }
        return sb.toString();
    }
}
C++
CPP
#include <string>
#include <vector>
#include <utility>
using namespace std;

class Solution {
public:
    string toRoman(int value) {

#### cons
t vector<pair<int, string>> table{
            {1000, "M"}, {900, "CM"}, {500, "D"}, {400, "CD"}, {100, "C"},
            {90, "XC"},  {50, "L"},   {40, "XL"}, {10, "X"},   {9, "IX"},
            {5, "V"},    {4, "IV"},   {1, "I"}};
        string out;
        for (const auto& [v, s] : table) {
            while (value >= v) {
                out += s;
                value -= v;
            }
        }
        return out;
    }
};
C
C
#include <stdlib.h>
#include <string.h>

/* Returns a malloc'd string; longest result (3888) is 15 chars. */
char* toRoman(int value) {
    static const int vals[] = {1000, 900, 500, 400, 100, 90, 50, 40, 10, 9, 5, 4, 1};
    static const char* syms[] = {"M", "CM", "D", "CD", "C", "XC",
                                 "L", "XL", "X", "IX", "V", "IV", "I"};
    char* out = (char*)malloc(16);
    out[0] = '\0';
    for (int i = 0; i < 13; i++) {
        while (value >= vals[i]) {
            strcat(out, syms[i]);
            value -= vals[i];
        }
    }
    return out;
}

#### Follow-up
questions
Write the inverse: parse a Roman numeral string back into its integer value.
How would you validate whether a given string is a well-formed Roman numeral?
Extend the scheme to numbers above 3999 (for example using an overline/vinculum convention for thousands).
Why is the greedy approach guaranteed optimal here — for what denomination sets would greedy fail?

