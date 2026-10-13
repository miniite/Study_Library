---
type: "practice"
difficulty: "Other"
---

# Read a String Written in a Zigzag Across Rows

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ String](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/String.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

A string is written onto a grid in a zigzag (boustrophedon) pattern across a fixed number of rows, then read back one full row at a time from top to bottom.

The writing rule: place characters going straight down the rows one per row (row 0, row 1, ... down to the last row); when the bottom row is reached, move back up diagonally toward the top row (each up step is one row higher and one column to the right); when the top row is reached, go down again. Keep alternating down and up until every character of the string has been placed.

For example, with 3 rows the string "ABCDEFGHIJKL" lands like this:

CODE
A       E       I
B   D   F   H   J   L
C       G       K

Reading row by row gives "AEI" + "BDFHJL" + "CGK" = "AEIBDFHJLCGK".

Given the string text and the integer rowCount, return the string obtained by reading the zigzag arrangement row by row.

Input

text: a non-empty string of English letters (both cases), commas, and periods.
rowCount: the number of rows used for the zigzag.

Output

The string formed by concatenating the rows of the zigzag from top to bottom.

#### Cons
traints

1 <= text.length <= 1000
text consists of English letters (lowercase and uppercase), ',', and '.'.
1 <= rowCount <= 1000
When rowCount is 1 the "zigzag" is a single straight row, so the string is returned unchanged.

#### Example
s & test cases
TEXT
Input: text = "ABCDEFGHIJKL", rowCount = 3 -> Output: "AEIBDFHJLCGK"
Why: the standard multi-row case; row 0 collects A,E,I, the middle row collects the diagonal characters B,D,F,H,J,L, and row 2 collects C,G,K.

Input: text = "ABCDEF", rowCount = 2 -> Output: "ACEBDF"
Why: with two rows the pattern just alternates rows, so evens A,C,E form row 0 and odds B,D,F form row 1 — the simplest zigzag.

Input: text = "HELLO", rowCount = 1 -> Output: "HELLO"
Why: a single row means no zigzag at all; the string must be returned exactly as given.

Input: text = "ABC", rowCount = 5 -> Output: "ABC"
Why: more rows than characters — the walk never reaches the bottom row, so each character occupies its own row and reading top-to-bottom reproduces the input.

Input: text = "Z", rowCount = 1 -> Output: "Z"
Why: minimum-length input with a single row; the lone character is returned unchanged.

#### Approach

Row-bucketing by simulated vertical direction. Keep one buffer per row and a "current row" pointer that walks down and up. Start heading downward; each time the pointer hits the top row (index 0) or the bottom row (index rowCount - 1), flip the direction. Append each character to the buffer for the current row, then step the pointer by the current direction. Because the pointer traces exactly the zigzag path, the buffers end up holding each row in reading order, so concatenating them top-to-bottom yields the answer in a single linear pass. The pattern is a one-pass simulation with a bouncing index — no coordinate math or grid allocation is required.

Complexity

Time: O(n), where n is the length of text — each character is appended to exactly one row once, and concatenating the rows visits each character once more. Space: O(n) for the per-row buffers that together hold all n characters (plus the output string).

Python
PYTHON
def convert(text: str, row_count: int) -> str:
    if row_count == 1:
        return text
    rows = [''] * row_count
    k, direction = 0, -1               # flips to +1 at the top row on entry
    for c in text:
        rows[k] += c
        if k == 0 or k == row_count - 1:
            direction = -direction
        k += direction
    return ''.join(rows)
JavaScript
JAVASCRIPT
function convert(text, rowCount) {
    if (rowCount === 1) return text;

#### cons
t rows = new Array(rowCount).fill('');
    let k = 0, direction = -1;          // flips to +1 at the top row on entry
    for (const c of text) {
        rows[k] += c;
        if (k === 0 || k === rowCount - 1) direction = -direction;
        k += direction;
    }
    return rows.join('');
}
Java
JAVA
class Solution {
    public String convert(String text, int rowCount) {
        if (rowCount == 1) return text;
        StringBuilder[] rows = new StringBuilder[rowCount];
        for (int r = 0; r < rowCount; r++) rows[r] = new StringBuilder();

        int k = 0, direction = -1;          // flips to +1 at the top row on entry
        for (int i = 0; i < text.length(); i++) {
            rows[k].append(text.charAt(i));
            if (k == 0 || k == rowCount - 1) direction = -direction;
            k += direction;
        }

        StringBuilder result = new StringBuilder();
        for (StringBuilder row : rows) result.append(row);
        return result.toString();
    }
}
C++
CPP
#include <string>
#include <vector>
using namespace std;

class Solution {
public:
    string convert(string text, int rowCount) {
        if (rowCount == 1) return text;
        vector<string> rows(rowCount);
        int k = 0, direction = -1;         // flips to +1 at the top row on entry
        for (char c : text) {
            rows[k] += c;
            if (k == 0 || k == rowCount - 1) direction = -direction;
            k += direction;
        }
        string result;
        for (const string& row : rows) result += row;
        return result;
    }
};
C
C
#include <stdlib.h>
#include <string.h>

/* Returns a malloc'd, null-terminated zigzag row-reading of text. */
char* convert(char* text, int rowCount) {
    int n = (int)strlen(text);
    char* result = (char*)malloc(n + 1);
    if (rowCount == 1) { memcpy(result, text, n); result[n] = '\0'; return result; }

    char** rows = (char**)malloc(rowCount * sizeof(char*));
    int* lens = (int*)calloc(rowCount, sizeof(int));
    for (int r = 0; r < rowCount; r++) rows[r] = (char*)malloc(n + 1);

    int k = 0, direction = -1;      /* flips to +1 at the top row on entry */
    for (int i = 0; i < n; i++) {
        rows[k][lens[k]++] = text[i];
        if (k == 0 || k == rowCount - 1) direction = -direction;
        k += direction;
    }

    int pos = 0;
    for (int r = 0; r < rowCount; r++) {
        memcpy(result + pos, rows[r], lens[r]);
        pos += lens[r];
        free(rows[r]);
    }
    result[pos] = '\0';
    free(rows);
    free(lens);
    return result;
}

#### Follow-up
questions
Can you compute each character's destination row with a direct index formula over the cycle length 2*(rowCount-1), and write straight into the output without per-row buffers?
How would you invert the process — reconstruct the original string given the row-by-row reading and the row count?
What changes if the up-going characters must also occupy their true diagonal columns in a full 2D grid rather than being packed into a row buffer?
How does the result behave as rowCount grows beyond the string length, and can you short-circuit that case?

