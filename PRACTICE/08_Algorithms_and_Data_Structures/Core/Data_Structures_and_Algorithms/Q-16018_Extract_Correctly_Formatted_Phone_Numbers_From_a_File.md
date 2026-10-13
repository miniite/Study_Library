---
type: "practice"
difficulty: "Other"
---

# Extract Correctly Formatted Phone Numbers From a File

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ String Processing](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/String_Processing.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given a text file that contains one contact string per line. Read the file line by line and print only those lines that hold a correctly formatted phone number, in the same order they appear.

A phone number is considered correctly formatted if — and only if — the entire line matches exactly one of these two shapes (each d stands for a single decimal digit 0–9):

ddd-ddd-dddd — three digits, a hyphen, three digits, a hyphen, four digits.
(ddd) ddd-dddd — an opening parenthesis, three digits, a closing parenthesis, a single space, three digits, a hyphen, four digits.

Any line that has extra characters, missing digits, wrong separators, or does not match one of the two shapes end-to-end is not a valid phone number and must be skipped. Assume no line has leading or trailing whitespace.

Read the input from standard input (one line per record) and write each valid phone number to standard output on its own line.

#### Cons
traints

The number of lines n satisfies 0 <= n <= 10^4.
Each line has length at most 40 characters.
Lines contain only printable ASCII characters and no leading/trailing whitespace.
The relative order of valid numbers in the output must equal their order in the input.

#### Example
s & test cases

Input:

CODE
415-882-0199
771 204 6653
(908) 337-1042

-> Output:

CODE
415-882-0199
(908) 337-1042

Why: line 1 matches ddd-ddd-dddd; line 3 matches (ddd) ddd-dddd; line 2 uses spaces instead of the required separators, so it is dropped.

Input:

CODE
(12) 345-6789
55-abc-1234

-> Output:

Why: first has only two digits in the parentheses (needs three) and second contains letters — neither matches a valid shape, so nothing is printed.

Input:

CODE
000-000-0000
(000) 000-0000

-> Output:

CODE
000-000-0000
(000) 000-0000

Why: only formatting is validated, not real-world reachability, so all-zero digits are accepted in both shapes.

Input:

CODE
415-882-0199
 415-882-0199
415-882-0199x

-> Output:

CODE
415-882-0199

Why: the match must span the whole line, so a leading space or a trailing extra character invalidates the otherwise-correct number.

Input (empty file):

-> Output:

Why: with zero lines there is nothing to validate, so the output is empty (edge case).

#### Approach

This is a line-oriented pattern-validation problem. The optimal idea is to compile a single anchored regular expression that captures both allowed shapes as an alternation (ddd-ddd-dddd or (ddd) ddd-dddd), then test each input line end-to-end with ^...$ so no partial or padded line slips through. Anchoring is the key correctness detail: without the ^ and $ anchors a valid substring inside a longer junk line would be wrongly accepted. Each line is processed and emitted independently, so the whole file streams in one pass.

Complexity

Time: O(N) where N is the total number of characters across all lines — each line is scanned once by a fixed-size regular expression whose cost is linear in the line length. Space: O(1) auxiliary beyond the current line buffer (output is streamed), since the compiled pattern uses constant state.

Python
PYTHON
import re
import sys

# Anchored so the whole line must match one of the two shapes.
PATTERN = re.compile(r'^(\d{3}-\d{3}-\d{4}|\(\d{3}\) \d{3}-\d{4})$')

def main():
    out = []
    for line in sys.stdin:
        line = line.rstrip('\n').rstrip('\r')
        if PATTERN.match(line):
            out.append(line)
    sys.stdout.write('\n'.join(out) + ('\n' if out else ''))

if __name__ == '__main__':
    main()
JavaScript
JAVASCRIPT

#### cons
t fs = require('fs');

// Anchored whole-line match against either accepted format.

#### cons
t pattern = /^(\d{3}-\d{3}-\d{4}|\(\d{3}\) \d{3}-\d{4})$/;

#### cons
t data = fs.readFileSync(0, 'utf8');

#### cons
t out = [];
for (const raw of data.split('\n')) {

#### cons
t line = raw.replace(/\r$/, '');
  if (pattern.test(line)) out.push(line);
}
process.stdout.write(out.length ? out.join('\n') + '\n' : '');
Java
JAVA
import java.util.Scanner;
import java.util.regex.Pattern;

public class Main {
    public static void main(String[] args) {
        Pattern pattern = Pattern.compile(
            "^(\\d{3}-\\d{3}-\\d{4}|\\(\\d{3}\\) \\d{3}-\\d{4})$");
        Scanner sc = new Scanner(System.in);
        StringBuilder sb = new StringBuilder();
        while (sc.hasNextLine()) {
            String line = sc.nextLine();
            if (pattern.matcher(line).matches())
                sb.append(line).append('\n');
        }
        System.out.print(sb);
    }
}
C++
CPP
#include <iostream>
#include <regex>
#include <string>
using namespace std;

int main() {
    // Whole-line match against either accepted shape.
    static const regex pattern(
        R"(^(\d{3}-\d{3}-\d{4}|\(\d{3}\) \d{3}-\d{4})$)");
    string line;
    while (getline(cin, line)) {
        if (!line.empty() && line.back() == '\r') line.pop_back();
        if (regex_match(line, pattern))
            cout << line << "\n";
    }
    return 0;
}
C
C
#include <stdio.h>
#include <string.h>
#include <regex.h>

/* Print each line whose entire content is a valid phone number. */
int main(void) {
    /* POSIX extended regex: ^(ddd-ddd-dddd | (ddd) ddd-dddd)$ */

#### cons
t char *pattern =
        "^([0-9]{3}-[0-9]{3}-[0-9]{4}|\\([0-9]{3}\\) [0-9]{3}-[0-9]{4})$";
    regex_t re;
    if (regcomp(&re, pattern, REG_EXTENDED) != 0) return 1;

    char line[256];
    while (fgets(line, sizeof(line), stdin)) {
        size_t n = strlen(line);
        while (n > 0 && (line[n - 1] == '\n' || line[n - 1] == '\r'))
            line[--n] = '\0';
        if (regexec(&re, line, 0, NULL, 0) == 0)
            printf("%s\n", line);
    }
    regfree(&re);
    return 0;
}

#### Follow-up
questions
Extend the accepted formats to also allow an optional +cc  country-code prefix and validate the digit count per region.
Instead of printing valid lines, output each invalid line with the reason it failed (wrong length, bad separator, non-digit).
Normalize every valid number into a single canonical form (e.g. ddddddddddd) before printing, de-duplicating repeats.
Handle a streaming source of billions of lines with bounded memory and report a running count of valid vs. invalid.

