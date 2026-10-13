---
type: "practice"
difficulty: "Other"
---

# Restore All Possible Number Pairs from a Punctuation-Stripped String

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Backtracking](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Backtracking.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

A logging device records a pair of non-negative decimal numbers as a string of the form "(A, B)" — an opening parenthesis, the first number, a comma, a single space, the second number, and a closing parenthesis. Due to a formatting bug, every comma, decimal point, and space was deleted, leaving only the two parentheses with a run of digits between them.

For example, the pair "(4, 1.5)" collapses to "(415)", and "(20, 3)" collapses to "(203)".

You are given one such collapsed string s. Reconstruct and return every pair "(A, B)" that could have produced it, where both numbers obey the original formatting rules.

Formatting rules obeyed by the original numbers (every reconstruction you return must obey them too):

A number has no redundant leading zeros: it is either the single digit 0, or it starts with a non-zero digit. So "00", "05", and "012" are invalid, while "0" is valid, and a leading 0 is allowed only when it is immediately followed by a decimal point (e.g. "0.5").
A decimal point always has at least one digit before it, so a form like ".7" is invalid.
A number never has a trailing zero after the decimal point (no redundant trailing zeros), so "1.0", "2.50", and "0.0" are invalid.
A number may be a pure integer with no decimal point at all, such as "12" or "0".

Each returned pair must be formatted exactly as "(A, B)": opening parenthesis, the first number, a comma, a single space, the second number, and a closing parenthesis. The pairs may be returned in any order.

Input: a string s.

Output: a list of strings, each a valid reconstruction of the original pair.

#### Cons
traints:

4 <= s.length <= 12
s[0] == '(' and s[s.length - 1] == ')'.
Every other character of s is a digit '0'–'9' (so there are at least two digits inside the parentheses).

#### Example
s & test cases
TEXT
Input: s = "(415)" -> Output: ["(4, 1.5)", "(4, 15)", "(4.1, 5)", "(41, 5)"]
Why: normal case — the block "415" can be split as 4|15 (yielding "1.5" or "15" on the right) or as 41|5 (yielding "4.1" or "41" on the left).

Input: s = "(0203)" -> Output: ["(0, 2.03)", "(0, 20.3)", "(0, 203)", "(0.2, 0.3)"]
Why: leading-zero handling — "0" alone is a legal integer, "02" can only be "0.2", and a split like "020|3" is discarded because "020" has no valid form.

Input: s = "(1005)" -> Output: ["(1, 0.05)", "(10, 0.5)", "(100, 5)"]
Why: shows a decimal on the right in several positions plus the trailing-zero integer "100" being valid while "1.0"-style forms are not.

Input: s = "(00050)" -> Output: ["(0.005, 0)"]
Why: many-zeros edge case — the only legal split is "0005|0", giving "0.005" and the integer "0"; every other split contains an unusable zero run.

Input: s = "(11)" -> Output: ["(1, 1)"]
Why: minimal length — only one split point, and each side is the single legal integer "1".

Input: s = "(100)" -> Output: ["(10, 0)"]
Why: a trailing zero is fine in an integer ("10") but "1.0" and "0.0" are rejected, so only one reconstruction survives.

#### Approach

Strip the parentheses to get the digit block, then split it into a left number and a right number at every internal boundary (enumeration). For each side, enumerate every legal placement of a single decimal point — including "no point" (pure integer) — and keep only the forms that satisfy the validity rules: reject a redundant leading zero (a length-2+ integer part starting with 0) and reject a trailing zero after the decimal point. The valid forms of the left side times the valid forms of the right side, wrapped as "(A, B)", are the answer. The pattern is exhaustive enumeration / backtracking with a small, sharply-constrained validity filter, factored into a reusable helper that lists a digit run's legal numeric renderings.

Complexity

Let d be the number of digits inside the parentheses (d = s.length - 2 <= 10). Time: O(d^4) in the worst case — there are O(d) split positions, each side produces O(d) candidate number-forms, giving O(d) x O(d) x O(d) = O(d^3) pairs, and assembling each pair string costs O(d). Space: O(d^4) to hold the output (O(d^3) strings of length O(d)); working space per split is O(d). Because d <= 10, this is effectively constant time and space.

Python
PYTHON
from typing import List

class Solution:
    def restorePairs(self, s: str) -> List[str]:
        digits = s[1:-1]
        n = len(digits)

        def valid_numbers(part: str) -> List[str]:
            m = len(part)
            out = []
            for i in range(1, m + 1):          # i digits before the (optional) point
                if i > 1 and part[0] == '0':   # redundant leading zero
                    continue
                int_part = part[:i]
                frac_part = part[i:]
                if not frac_part:              # pure integer
                    out.append(int_part)
                elif frac_part[-1] != '0':     # no trailing zero after '.'
                    out.append(int_part + '.' + frac_part)
            return out

        ans = []
        for cut in range(1, n):
            for a in valid_numbers(digits[:cut]):
                for b in valid_numbers(digits[cut:]):
                    ans.append(f'({a}, {b})')
        return ans
JavaScript
JAVASCRIPT
function restorePairs(s) {

#### cons
t digits = s.slice(1, -1);

#### cons
t n = digits.length;

    function validNumbers(part) {

#### cons
t out = [];

#### cons
t m = part.length;
        for (let i = 1; i <= m; i++) {              // i digits before the (optional) point
            if (i > 1 && part[0] === '0') continue; // redundant leading zero

#### cons
t intPart = part.slice(0, i);

#### cons
t fracPart = part.slice(i);
            if (fracPart.length === 0) {
                out.push(intPart);                  // pure integer
            } else if (fracPart[fracPart.length - 1] !== '0') {
                out.push(intPart + '.' + fracPart); // no trailing zero
            }
        }
        return out;
    }

#### cons
t ans = [];
    for (let cut = 1; cut < n; cut++) {

#### cons
t lefts = validNumbers(digits.slice(0, cut));

#### cons
t rights = validNumbers(digits.slice(cut));
        for (const a of lefts)
            for (const b of rights)
                ans.push(`(${a}, ${b})`);
    }
    return ans;
}
Java
JAVA
import java.util.*;

class Solution {
    public List<String> restorePairs(String s) {
        String digits = s.substring(1, s.length() - 1);
        int n = digits.length();
        List<String> ans = new ArrayList<>();
        for (int cut = 1; cut < n; cut++) {
            List<String> lefts  = validNumbers(digits.substring(0, cut));
            List<String> rights = validNumbers(digits.substring(cut));
            for (String a : lefts)
                for (String b : rights)
                    ans.add("(" + a + ", " + b + ")");
        }
        return ans;
    }

    // Every legal number string the digit run could denote.
    private List<String> validNumbers(String part) {
        int m = part.length();
        List<String> out = new ArrayList<>();
        for (int i = 1; i <= m; i++) {                 // i digits before the (optional) point
            if (i > 1 && part.charAt(0) == '0') continue;  // redundant leading zero
            String intPart  = part.substring(0, i);
            String fracPart = part.substring(i);
            if (fracPart.isEmpty())
                out.add(intPart);                          // pure integer
            else if (fracPart.charAt(fracPart.length() - 1) != '0')
                out.add(intPart + "." + fracPart);         // no trailing zero
        }
        return out;
    }
}
C++
CPP
#include <string>
#include <vector>
using namespace std;

class Solution {
public:
    vector<string> restorePairs(const string& s) {
        string digits = s.substr(1, s.size() - 2);
        int n = digits.size();
        vector<string> ans;
        for (int cut = 1; cut < n; ++cut) {
            vector<string> lefts  = validNumbers(digits.substr(0, cut));
            vector<string> rights = validNumbers(digits.substr(cut));
            for (const string& a : lefts)
                for (const string& b : rights)
                    ans.push_back("(" + a + ", " + b + ")");
        }
        return ans;
    }

private:
    // Every legal number string the digit run could denote.
    vector<string> validNumbers(const string& part) {
        int m = part.size();
        vector<string> out;
        for (int i = 1; i <= m; ++i) {           // i digits before the (optional) point
            if (i > 1 && part[0] == '0') continue;   // redundant leading zero
            string intPart  = part.substr(0, i);
            string fracPart = part.substr(i);
            if (fracPart.empty())
                out.push_back(intPart);              // pure integer
            else if (fracPart.back() != '0')
                out.push_back(intPart + "." + fracPart); // no trailing zero
        }
        return out;
    }
};
C
C
#include <stdlib.h>
#include <string.h>
#include <stdio.h>

/* Build every legal number string for the digit run part[0..m).
   Returns a malloc'd array of malloc'd strings; *count receives its length. */
static char** validNumbers(const char* part, int m, int* count) {
    char** out = malloc(sizeof(char*) * (m + 1));
    int cnt = 0;
    for (int i = 1; i <= m; i++) {          /* i digits go before the (optional) point */
        if (i > 1 && part[0] == '0')        /* redundant leading zero in integer part */
            continue;
        int fracLen = m - i;
        if (fracLen == 0) {                 /* pure integer */
            char* num = malloc(i + 1);
            memcpy(num, part, i);
            num[i] = '\0';
            out[cnt++] = num;
        } else if (part[m - 1] != '0') {    /* decimal; no trailing zero allowed */
            char* num = malloc(m + 2);      /* i digits + '.' + fracLen digits + '\0' */
            memcpy(num, part, i);
            num[i] = '.';
            memcpy(num + i + 1, part + i, fracLen);
            num[m + 1] = '\0';
            out[cnt++] = num;
        }
    }
    *count = cnt;
    return out;
}

char** restorePairs(const char* s, int* returnSize) {
    int len = strlen(s);
    int n = len - 2;                 /* count of digits inside the parentheses */

#### cons
t char* digits = s + 1;      /* first digit */

    int cap = 256, size = 0;
    char** ans = malloc(sizeof(char*) * cap);

    for (int cut = 1; cut < n; cut++) {
        int lc, rc;
        char** lefts  = validNumbers(digits, cut, &lc);
        char** rights = validNumbers(digits + cut, n - cut, &rc);
        for (int a = 0; a < lc; a++) {
            for (int b = 0; b < rc; b++) {
                if (size == cap) {
                    cap *= 2;
                    ans = realloc(ans, sizeof(char*) * cap);
                }
                int la = strlen(lefts[a]), lb = strlen(rights[b]);
                char* pair = malloc(la + lb + 5);   /* "(" a ", " b ")" '\0' */
                sprintf(pair, "(%s, %s)", lefts[a], rights[b]);
                ans[size++] = pair;
            }
        }
        for (int a = 0; a < lc; a++) free(lefts[a]);
        for (int b = 0; b < rc; b++) free(rights[b]);
        free(lefts);
        free(rights);
    }

    *returnSize = size;
    return ans;
}

#### Follow-up
questions
Generalize from a pair to k comma-separated numbers packed inside the brackets.
Allow the numbers to be negative — a stripped minus sign could precede either value.
Only count how many valid reconstructions exist instead of materializing them all.
Permit scientific notation (an e exponent) or a thousands separator, and adjust the validity rules accordingly.

