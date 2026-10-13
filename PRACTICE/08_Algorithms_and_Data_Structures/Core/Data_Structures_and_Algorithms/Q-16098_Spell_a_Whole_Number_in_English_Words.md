---
type: "practice"
difficulty: "Other"
---

# Spell a Whole Number in English Words

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Recursion](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Recursion.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Given a non-negative whole number, produce its written-out form in English words.

You are given an integer value. Convert it into the exact English phrase a person would read aloud, using the standard scale words Hundred, Thousand, Million, and Billion. Individual words are separated by single spaces, every word begins with a capital letter, and the result has no leading or trailing spaces. The number zero is written simply as Zero.

Return the resulting phrase as a string.

The English naming rules you must follow:

Numbers below twenty each have their own word (One, Two, ..., Nineteen).
Multiples of ten from twenty to ninety have their own word (Twenty, Thirty, ..., Ninety), and a trailing units digit is appended after a space (for example 58 becomes "Fifty Eight").
A hundreds digit is spelled out followed by the word Hundred, then the remaining two-digit part (for example 305 becomes "Three Hundred Five").
The number is grouped from the right into blocks of three digits; each non-zero block is spelled out and tagged with its scale word (Thousand, Million, Billion). Blocks that are entirely zero contribute nothing.

#### Cons
traints:

0 <= value <= 2^31 - 1 (that is, at most 2,147,483,647).
The output uses ONLY the scale words Hundred, Thousand, Million, Billion. Do not insert the word "and", hyphens, or commas.
Because the maximum is just over two billion, Billion is the largest scale word you will ever need.

#### Example
s & test cases
TEXT
Input: value = 0 -> Output: "Zero"
Why: minimum input; zero is a special case that must not fall through the grouping logic.

Input: value = 9 -> Output: "Nine"
Why: single digit under twenty maps directly to its word.

Input: value = 58 -> Output: "Fifty Eight"
Why: a two-digit number above twenty; tens word plus units word joined by a space.

Input: value = 305019 -> Output: "Three Hundred Five Thousand Nineteen"
Why: mixes a hundreds part with a scale word and a low-teens remainder; checks that a zero tens digit ("Three Hundred Five", not "Three Hundred Zero Five") is handled.

Input: value = 20000012 -> Output: "Twenty Million Twelve"
Why: an all-zero middle block (the thousands group) must be skipped entirely.

Input: value = 2147483647 -> Output: "Two Billion One Hundred Forty Seven Million Four Hundred Eighty Three Thousand Six Hundred Forty Seven"
Why: the maximum 32-bit value; exercises every scale word up to Billion.

#### Approach

Break the number into three-digit groups from least significant to most significant (repeatedly take value % 1000 and divide by 1000). A helper spells any value 0..999 by handling the hundreds digit ("X Hundred"), then the tens/units using two lookup tables — one for 0..19 (where the teens are irregular) and one for the tens multiples. Each non-empty group is tagged with its scale word (Thousand / Million / Billion) chosen by the group index, and groups are concatenated from most significant to least. Zero groups are skipped so no spurious scale words appear, and value == 0 is special-cased to "Zero". The core technique is positional decomposition plus table lookup for the irregular English number words.

Complexity

Time: O(d) where d is the number of digits (equivalently O(log value)) — the number is processed in a fixed number of three-digit groups (at most four for a 32-bit value), and each group takes constant work. Space: O(d) for the assembled output string; the auxiliary lookup tables are constant size.

Python
PYTHON
class Solution:
    BELOW_TWENTY = ["", "One", "Two", "Three", "Four", "Five", "Six", "Seven",
                    "Eight", "Nine", "Ten", "Eleven", "Twelve", "Thirteen",
                    "Fourteen", "Fifteen", "Sixteen", "Seventeen", "Eighteen", "Nineteen"]
    TENS = ["", "", "Twenty", "Thirty", "Forty", "Fifty",
            "Sixty", "Seventy", "Eighty", "Ninety"]
    SCALE = ["", "Thousand", "Million", "Billion"]

    def numberToWords(self, value: int) -> str:
        if value == 0:
            return "Zero"

        result = ""
        i = 0
        while value > 0:
            if value % 1000 != 0:
                part = self._spell_three(value % 1000)
                if self.SCALE[i]:
                    part += " " + self.SCALE[i]
                result = part if not result else part + " " + result
            value //= 1000
            i += 1
        return result

    def _spell_three(self, n: int) -> str:
        s = ""
        if n >= 100:
            s += self.BELOW_TWENTY[n // 100] + " Hundred"
            n %= 100
            if n:
                s += " "
        if n >= 20:
            s += self.TENS[n // 10]
            n %= 10
            if n:
                s += " "
        if 0 < n < 20:
            s += self.BELOW_TWENTY[n]
        return s
JavaScript
JAVASCRIPT

#### cons
t BELOW_TWENTY = ["", "One", "Two", "Three", "Four", "Five", "Six", "Seven",
    "Eight", "Nine", "Ten", "Eleven", "Twelve", "Thirteen", "Fourteen",
    "Fifteen", "Sixteen", "Seventeen", "Eighteen", "Nineteen"];

#### cons
t TENS = ["", "", "Twenty", "Thirty", "Forty", "Fifty",
    "Sixty", "Seventy", "Eighty", "Ninety"];

#### cons
t SCALE = ["", "Thousand", "Million", "Billion"];

function spellThree(n) {
    let s = "";
    if (n >= 100) {
        s += BELOW_TWENTY[Math.floor(n / 100)] + " Hundred";
        n %= 100;
        if (n) s += " ";
    }
    if (n >= 20) {
        s += TENS[Math.floor(n / 10)];
        n %= 10;
        if (n) s += " ";
    }
    if (n > 0 && n < 20) s += BELOW_TWENTY[n];
    return s;
}

function numberToWords(value) {
    if (value === 0) return "Zero";

    let result = "";
    let i = 0;
    while (value > 0) {
        if (value % 1000 !== 0) {
            let part = spellThree(value % 1000);
            if (SCALE[i]) part += " " + SCALE[i];
            result = result === "" ? part : part + " " + result;
        }
        value = Math.floor(value / 1000);
        i++;
    }
    return result;
}
Java
JAVA
class Solution {
    private static final String[] BELOW_TWENTY = {
        "", "One", "Two", "Three", "Four", "Five", "Six", "Seven", "Eight", "Nine",
        "Ten", "Eleven", "Twelve", "Thirteen", "Fourteen", "Fifteen", "Sixteen",
        "Seventeen", "Eighteen", "Nineteen"};
    private static final String[] TENS = {
        "", "", "Twenty", "Thirty", "Forty", "Fifty", "Sixty", "Seventy", "Eighty", "Ninety"};
    private static final String[] SCALE = {"", "Thousand", "Million", "Billion"};

    public String numberToWords(int value) {
        if (value == 0) return "Zero";

        String result = "";
        int i = 0;
        while (value > 0) {
            if (value % 1000 != 0) {
                String part = spellThree(value % 1000);
                if (!SCALE[i].isEmpty()) part += " " + SCALE[i];
                result = result.isEmpty() ? part : part + " " + result;
            }
            value /= 1000;
            i++;
        }
        return result;
    }

    private String spellThree(int n) {
        StringBuilder s = new StringBuilder();
        if (n >= 100) {
            s.append(BELOW_TWENTY[n / 100]).append(" Hundred");
            n %= 100;
            if (n != 0) s.append(" ");
        }
        if (n >= 20) {
            s.append(TENS[n / 10]);
            n %= 10;
            if (n != 0) s.append(" ");
        }
        if (n > 0 && n < 20) s.append(BELOW_TWENTY[n]);
        return s.toString();
    }
}
C++
CPP
#include <string>
#include <vector>
using namespace std;

class Solution {
 public:
  string numberToWords(int value) {
    if (value == 0) return "Zero";

    string result;
    int i = 0;
    while (value > 0) {
      if (value % 1000 != 0) {
        string part = spellThree(value % 1000);
        if (!scale[i].empty()) part += " " + scale[i];
        result = result.empty() ? part : part + " " + result;
      }
      value /= 1000;
      ++i;
    }
    return result;
  }

 private:

#### cons
t vector<string> belowTwenty = {
      "", "One", "Two", "Three", "Four", "Five", "Six", "Seven", "Eight", "Nine",
      "Ten", "Eleven", "Twelve", "Thirteen", "Fourteen", "Fifteen", "Sixteen",
      "Seventeen", "Eighteen", "Nineteen"};

#### cons
t vector<string> tensWord = {
      "", "", "Twenty", "Thirty", "Forty", "Fifty", "Sixty", "Seventy", "Eighty", "Ninety"};

#### cons
t vector<string> scale = {"", "Thousand", "Million", "Billion"};

  string spellThree(int n) {
    string s;
    if (n >= 100) {
      s += belowTwenty[n / 100] + " Hundred";
      n %= 100;
      if (n) s += " ";
    }
    if (n >= 20) {
      s += tensWord[n / 10];
      n %= 10;
      if (n) s += " ";
    }
    if (n > 0 && n < 20) s += belowTwenty[n];
    return s;
  }
};
C
C
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

static const char* belowTwenty[] = {
    "", "One", "Two", "Three", "Four", "Five", "Six", "Seven", "Eight", "Nine",
    "Ten", "Eleven", "Twelve", "Thirteen", "Fourteen", "Fifteen", "Sixteen",
    "Seventeen", "Eighteen", "Nineteen"};
static const char* tensWord[] = {
    "", "", "Twenty", "Thirty", "Forty", "Fifty", "Sixty", "Seventy", "Eighty", "Ninety"};
static const char* scale[] = {"", "Thousand", "Million", "Billion"};

/* Spell a value 1..999 into buf (no trailing scale word). */
static void spellThree(int n, char* buf) {
    buf[0] = '\0';
    if (n >= 100) {
        strcat(buf, belowTwenty[n / 100]);
        strcat(buf, " Hundred");
        n %= 100;
        if (n) strcat(buf, " ");
    }
    if (n >= 20) {
        strcat(buf, tensWord[n / 10]);
        n %= 10;
        if (n) strcat(buf, " ");
    }
    if (n > 0 && n < 20) strcat(buf, belowTwenty[n]);
}

char* numberToWords(int value) {
    char* result = malloc(256);
    result[0] = '\0';
    if (value == 0) { strcpy(result, "Zero"); return result; }

    char groups[4][80];
    int gi = 0;
    while (value > 0) {
        int chunk = value % 1000;
        if (chunk != 0) {
            char part[80];
            spellThree(chunk, part);
            if (scale[gi][0] != '\0') {
                strcat(part, " ");
                strcat(part, scale[gi]);
            }
            strcpy(groups[gi], part);
        } else {
            groups[gi][0] = '\0';
        }
        value /= 1000;
        gi++;
    }

    int first = 1;
    for (int i = gi - 1; i >= 0; i--) {
        if (groups[i][0] != '\0') {
            if (!first) strcat(result, " ");
            strcat(result, groups[i]);
            first = 0;
        }
    }
    return result;
}

int main(void) {
    int tests[] = {0, 9, 58, 305019, 20000012, 2147483647};
    for (int i = 0; i < 6; i++) {
        char* s = numberToWords(tests[i]);
        printf("%d -> \"%s\"\n", tests[i], s);
        free(s);
    }
    return 0;
}

#### Follow-up
questions
Extend the input to signed 64-bit integers, adding a leading "Negative" and the Trillion/Quadrillion scale words.
Support the British reading style that inserts "and" (e.g., "One Hundred and Five").
Reverse the operation: parse an English words phrase back into the integer it represents.
Localize to another language whose grouping or teen rules differ (e.g., French vingt-et-un), and discuss what stays table-driven versus rule-driven.

