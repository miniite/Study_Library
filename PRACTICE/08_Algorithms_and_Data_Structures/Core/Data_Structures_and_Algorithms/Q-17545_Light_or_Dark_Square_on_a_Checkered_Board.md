---
type: "practice"
difficulty: "Other"
---

# Light or Dark Square on a Checkered Board

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ String](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/String.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

An 8 by 8 checkered board is arranged so that its columns are labeled with the lowercase letters a through h from left to right, and its rows are labeled with the digits 1 through 8 from bottom to top. Every square on the board is colored either dark or light, and the colors alternate: any two squares that share an edge always have opposite colors. The square in the bottom-left corner — column a, row 1 — is dark.

You are given a two-character string cell that identifies exactly one square of the board. The first character is always a column letter in the range a..h, and the second character is always a row digit in the range 1..8. The string is guaranteed to name a valid square.

Return true if the identified square is light, and false if it is dark.

#### Cons
traints

cell.length == 2
cell[0] is a lowercase letter with 'a' <= cell[0] <= 'h'
cell[1] is a digit character with '1' <= cell[1] <= '8'
The input always denotes a valid square (letter first, digit second).

#### Example
s & test cases
TEXT
- Input: `cell = "a1"` -> Output: `false` — the bottom-left corner is defined as dark, so it is not light.
- Input: `cell = "h3"` -> Output: `true` — column index 7 plus row index 2 gives an odd sum, so the square is light.
- Input: `cell = "c7"` -> Output: `false` — column index 2 plus row index 6 gives an even sum, so the square is dark.
- Input: `cell = "b1"` -> Output: `true` — a square edge-adjacent to the dark `a1` corner must be the opposite color, light.
- Input: `cell = "h8"` -> Output: `false` — the top-right corner; indices 7 and 7 sum to an even number, so it is dark.

#### Approach

Map the column letter to an index 0..7 and the row digit to an index 0..7. On a checkerboard whose bottom-left corner is dark, a square is dark exactly when the sum of its column and row indices is even, and light when the sum is odd — because each step to an edge-adjacent square flips the parity and the color together. So the whole task reduces to a single parity (odd/even) check on col + row. This is the classic coordinate-parity / coloring-argument pattern.

Complexity

Time O(1) and space O(1): the answer depends only on the parity of two fixed-position characters, so a constant amount of work suffices.

Python
PYTHON
class Solution:
    def squareIsLight(self, cell: str) -> bool:
        col = ord(cell[0]) - ord('a')   # 0..7
        row = ord(cell[1]) - ord('1')   # 0..7
        return (col + row) % 2 == 1      # odd sum => light
JavaScript
JAVASCRIPT
function squareIsLight(cell) {

#### cons
t col = cell.charCodeAt(0) - 'a'.charCodeAt(0); // 0..7

#### cons
t row = cell.charCodeAt(1) - '1'.charCodeAt(0); // 0..7
    return ((col + row) & 1) === 1; // odd sum => light
}
Java
JAVA
class Solution {
    public boolean squareIsLight(String cell) {
        int col = cell.charAt(0) - 'a';   // 0..7
        int row = cell.charAt(1) - '1';   // 0..7
        return ((col + row) & 1) == 1;    // odd sum => light
    }
}
C++
CPP
#include <string>

class Solution {
public:
    bool squareIsLight(const std::string& cell) {
        int col = cell[0] - 'a';   // 0..7
        int row = cell[1] - '1';   // 0..7
        return ((col + row) & 1) == 1;  // odd sum => light
    }
};
C
C
#include <stdbool.h>

bool squareIsLight(const char* cell) {
    int col = cell[0] - 'a';   /* 0..7 */
    int row = cell[1] - '1';   /* 0..7 */
    /* dark when col+row is even (a1 => 0, dark); light when odd */
    return ((col + row) & 1) == 1;
}

#### Follow-up
questions
Generalize to an N by N board or to arbitrary rectangular boards, and answer for any coordinate in O(1).
Given two squares, decide in O(1) whether they share the same color (compare the parities of their index sums).
Extend to a 3-coloring or a torus wrap-around board and describe how the parity argument changes.
Return the color as a label without any conditional branching (e.g., derive it directly from the low bit of the summed character codes).

