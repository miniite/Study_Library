---
type: "practice"
difficulty: "Other"
---

# Transpose a Space-Separated Grid of Fields

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Matrix Manipulation](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Matrix_Manipulation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given the text content of a small table. Each line of the input is one row of the table, and within a row the fields are separated by a single space character. Every row has the same number of fields (columns).

Transpose the table: turn rows into columns and columns into rows. The value that was in row i, column j of the input must appear in row j, column i of the output. In the output, fields in a row are again joined by a single space, and there is one output line per original column.

Read the table from standard input and write the transposed table to standard output.

#### Example

Input:

CODE
city temp
delhi 34
paris 18

Output:

CODE
city delhi paris
temp 34 18

The first input column (city, delhi, paris) becomes the first output row, and the second input column (temp, 34, 18) becomes the second output row.

#### Cons
traints

The number of rows R satisfies 1 <= R <= 1000.
The number of columns C satisfies 1 <= C <= 1000.
Every row contains exactly C fields separated by single spaces; there are no leading or trailing spaces on a line.
Each field is a non-empty token of printable ASCII characters (no embedded spaces).

#### Example
s & test cases

Input:

CODE
city temp
delhi 34
paris 18

-> Output:

CODE
city delhi paris
temp 34 18

Why: a 3x2 grid transposes into a 2x3 grid; column j of the input becomes row j of the output.

Input:

CODE
a b c

-> Output:

CODE
a
b
c

Why: a single row of 3 columns becomes 3 single-field rows (a 1xC grid transposes to Cx1).

Input:

CODE
a
b
c

-> Output:

CODE
a b c

Why: a single column of 3 rows becomes one row of 3 fields (a Rx1 grid transposes to 1xR).

Input:

CODE
x

-> Output:

CODE
x

Why: a 1x1 grid is its own transpose (minimal edge case).

Input:

CODE
name age job
alice 21 eng
ryan 30 art
mia 27 med

-> Output:

CODE
name alice ryan mia
age 21 30 27
job eng art med

Why: a 4x3 grid transposes to 3x4, preserving the original left-to-right field order down each output row.

#### Approach

The core pattern is a two-dimensional index swap. Parse the input into a rectangular grid of tokens (row-major), remembering the column count from the first row. To transpose, iterate the output in column-major order of the original: the outer loop walks original columns c (these become output rows) and the inner loop walks original rows r, emitting grid[r][c] joined by single spaces. Because every row has the same width, indexing grid[r][c] is always valid and no padding is needed. Buffering the full grid first is required since the very first output line pulls one field from each input row.

Complexity

Time: O(R * C) — every field is read exactly once while parsing and written exactly once while emitting the transpose, so the work is linear in the total number of cells. Space: O(R * C) to hold the parsed grid in memory, which is unavoidable because producing the first output row requires the first field of every input row.

Python
PYTHON
import sys

def main():
    grid = [line.split() for line in sys.stdin.read().splitlines() if line]
    if not grid:
        return
    cols = len(grid[0])
    out = []
    for c in range(cols):
        out.append(' '.join(row[c] for row in grid))
    sys.stdout.write('\n'.join(out) + '\n')

if __name__ == '__main__':
    main()
JavaScript
JAVASCRIPT

#### cons
t fs = require('fs');

#### cons
t grid = fs.readFileSync(0, 'utf8')
  .split('\n')
  .map(l => l.replace(/\r$/, ''))
  .filter(l => l.length > 0)
  .map(l => l.split(' '));

if (grid.length) {

#### cons
t cols = grid[0].length;

#### cons
t out = [];
  for (let c = 0; c < cols; c++) {
    out.push(grid.map(row => row[c]).join(' '));
  }
  process.stdout.write(out.join('\n') + '\n');
}
Java
JAVA
import java.util.ArrayList;
import java.util.List;
import java.util.Scanner;

public class Main {
    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        List<String[]> grid = new ArrayList<>();
        while (sc.hasNextLine()) {
            String line = sc.nextLine();
            if (line.isEmpty()) continue;
            grid.add(line.split(" "));
        }
        if (grid.isEmpty()) return;

        int cols = grid.get(0).length;
        StringBuilder sb = new StringBuilder();
        for (int c = 0; c < cols; c++) {
            for (int r = 0; r < grid.size(); r++) {
                if (r > 0) sb.append(' ');
                sb.append(grid.get(r)[c]);
            }
            sb.append('\n');
        }
        System.out.print(sb);
    }
}
C++
CPP
#include <iostream>
#include <sstream>
#include <string>
#include <vector>
using namespace std;

int main() {
    vector<vector<string>> grid;
    string line;
    while (getline(cin, line)) {
        if (!line.empty() && line.back() == '\r') line.pop_back();
        if (line.empty()) continue;
        istringstream ss(line);
        vector<string> row;
        string tok;
        while (ss >> tok) row.push_back(tok);
        grid.push_back(move(row));
    }
    if (grid.empty()) return 0;

    int cols = (int)grid[0].size();
    for (int c = 0; c < cols; c++) {
        for (int r = 0; r < (int)grid.size(); r++) {
            if (r) cout << ' ';
            cout << grid[r][c];
        }
        cout << '\n';
    }
    return 0;
}
C
C
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

/* Read a space-separated grid from stdin and print its transpose. */
int main(void) {
    char **cells = NULL;          /* all tokens, stored row-major */
    int rows = 0, cols = 0, count = 0, cap = 0;
    char line[1 << 16];

    while (fgets(line, sizeof(line), stdin)) {
        size_t n = strlen(line);
        while (n > 0 && (line[n - 1] == '\n' || line[n - 1] == '\r'))
            line[--n] = '\0';
        if (n == 0) continue;

        int rowCols = 0;
        char *tok = strtok(line, " ");
        while (tok) {
            if (count == cap) {
                cap = cap ? cap * 2 : 16;
                cells = realloc(cells, (size_t)cap * sizeof(char *));
            }
            cells[count++] = strdup(tok);
            rowCols++;
            tok = strtok(NULL, " ");
        }
        if (rows == 0) cols = rowCols;
        rows++;
    }

    for (int c = 0; c < cols; c++) {
        for (int r = 0; r < rows; r++) {
            if (r) putchar(' ');
            fputs(cells[r * cols + c], stdout);
        }
        putchar('\n');
    }

    for (int i = 0; i < count; i++) free(cells[i]);
    free(cells);
    return 0;
}

#### Follow-up
questions
Handle ragged rows where columns may differ in count, padding missing cells with a placeholder.
Support an arbitrary single-character delimiter (tab, comma) passed as a parameter and preserve it on output.
Transpose a matrix too large to fit in memory by streaming through it in column blocks or using external buffers.
Return the transpose of an in-memory square matrix in place with O(1) extra space, then discuss why the rectangular case cannot.

