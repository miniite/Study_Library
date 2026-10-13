---
type: "practice"
difficulty: "Other"
---

# Can a Word Fit Into the Letter Grid

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Matrix](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Matrix.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given an m x n grid board that represents the current state of a partially filled letter puzzle. Every cell holds one of three things:

a lowercase English letter, meaning that cell is already filled with that letter,
a single space ' ', meaning that cell is empty and can hold any letter,
a hash '#', meaning that cell is blocked and can never hold a letter.

You are also given a string word made of lowercase English letters. You must decide whether word can be placed into the grid. A placement lies entirely along one row (read left-to-right or right-to-left) or entirely along one column (read top-to-bottom or bottom-to-top), and it is valid only if all of the following hold:

None of the cells the word occupies is a blocked cell '#'.
Each cell the word occupies is either empty ' ', or already contains exactly the letter that lands on it.
If the word is placed horizontally, the cell immediately to the left of its first letter and the cell immediately to the right of its last letter must each be either a grid edge or a blocked cell '#' (they may not be empty or hold another letter).
If the word is placed vertically, the cell immediately above its first letter and the cell immediately below its last letter must each be either a grid edge or a blocked cell '#'.

In other words, the word must exactly fill one maximal open run of cells (a run bounded on both ends by a blocked cell or the border), matching every pre-filled letter it covers.

Return true if word can be placed somewhere in the grid, and false otherwise.

#### Cons
traints:

m == board.length
n == board[i].length
1 <= m * n <= 2 * 10^5
Each board[i][j] is ' ', '#', or a lowercase English letter.
1 <= word.length <= max(m, n)
word consists of lowercase English letters only.

#### Example
s & test cases
TEXT
Input: board = [['#',' ','#'],[' ',' ','#'],['#',' ','m']], word = "sum" -> Output: true
Why: Column 1 is a full open run of length 3 (cells ' ', ' ', 'm'). Placing "sum" top-to-bottom lands 'm' on the pre-filled 'm' and 's','u' on empty cells, so it fits.

Input: board = [[' ','#','p'],[' ','#','t'],[' ','#','p']], word = "pt" -> Output: false
Why: Every maximal open run is length 1 or length 3 (the left column of spaces is length 3, the right column p/t/p is length 3, each row splits into single cells). No run has length 2, so a 2-letter word cannot fit anywhere.

Input: board = [['#',' ','#'],[' ',' ','#'],['#',' ','g']], word = "go" -> Output: true
Why: Row 2 has the open run [' ', 'g'] of length 2. Reading it right-to-left gives the slot ['g',' '] which matches "go" ('g' matches the filled 'g', 'o' goes on the empty cell).

Input: board = [[' ']], word = "a" -> Output: true
Why: Minimal 1x1 grid with a single empty cell; a one-letter word fills it exactly.

Input: board = [['a','b','c']], word = "abcd" -> Output: false
Why: The only open run has length 3 while the word has length 4, so it can never fill a run exactly.

Input: board = [['a',' ','#']], word = "cb" -> Output: false
Why: The only open run of length 2 is ['a',' ']. Reading forward, 'a' must be 'c' (fails); reading backward, 'a' must be 'b' (fails). The pre-filled letter 'a' blocks both directions even though the run length matches, so the word cannot be placed.

#### Approach

The blocked cells '#' and the grid borders chop each row and each column into maximal open runs. A word can only be placed if it fills one such run exactly, so any run whose length differs from the word length is irrelevant. For every run of the correct length, test whether the word matches it reading forward (which covers left-to-right in a row / top-to-bottom in a column) or reading it reversed (which covers right-to-left / bottom-to-top); a position matches if the grid cell is empty ' ' or already equals the required letter. Scan all rows to cover horizontal placements, then scan all columns to cover vertical placements, using a sentinel '#' at the end of each line so the final run is always flushed. Return true as soon as any run accepts the word. The pattern is line segmentation plus a bidirectional exact-match check.

Complexity

Time: O(m * n) — each cell is read a constant number of times (once during the row pass, once during the column pass), and across all open runs the letter-by-letter fits comparisons total at most O(m * n) per orientation. Space: O(max(m, n)) for the current run/column buffer (O(1) if you compare in place).

Python
PYTHON
def can_place_word(board, word):
    m, n = len(board), len(board[0])
    L = len(word)

    def fits(cells):
        if len(cells) != L:
            return False
        forward = all(cells[i] == ' ' or cells[i] == word[i] for i in range(L))
        backward = all(cells[i] == ' ' or cells[i] == word[L - 1 - i] for i in range(L))
        return forward or backward

    def scan(lines):
        for line in lines:
            seg = []
            for ch in line:
                if ch == '#':
                    if fits(seg):
                        return True
                    seg = []
                else:
                    seg.append(ch)
            if fits(seg):        # flush the final open run of the line
                return True
        return False

    rows = [board[r] for r in range(m)]
    cols = [[board[r][c] for r in range(m)] for c in range(n)]
    return scan(rows) or scan(cols)
JavaScript
JAVASCRIPT
function canPlaceWord(board, word) {

#### cons
t m = board.length, n = board[0].length, L = word.length;

#### cons
t fits = (seg) => {
        if (seg.length !== L) return false;
        let forward = true, backward = true;
        for (let i = 0; i < L; i++) {
            if (seg[i] !== ' ' && seg[i] !== word[i])         forward = false;
            if (seg[i] !== ' ' && seg[i] !== word[L - 1 - i]) backward = false;
        }
        return forward || backward;
    };

#### cons
t scan = (line) => {
        let seg = [];
        for (let i = 0; i <= line.length; i++) {

#### cons
t ch = i < line.length ? line[i] : '#';   // sentinel flushes last run
            if (ch === '#') {
                if (fits(seg)) return true;
                seg = [];
            } else {
                seg.push(ch);
            }
        }
        return false;
    };

    for (let r = 0; r < m; r++) if (scan(board[r])) return true;
    for (let c = 0; c < n; c++) {

#### cons
t col = [];
        for (let r = 0; r < m; r++) col.push(board[r][c]);
        if (scan(col)) return true;
    }
    return false;
}
Java
JAVA
class Solution {
    private boolean fits(char[] seg, int segLen, String word) {
        int L = word.length();
        if (segLen != L) return false;
        boolean forward = true, backward = true;
        for (int i = 0; i < L; i++) {
            if (seg[i] != ' ' && seg[i] != word.charAt(i))         forward = false;
            if (seg[i] != ' ' && seg[i] != word.charAt(L - 1 - i)) backward = false;
        }
        return forward || backward;
    }

    private boolean scan(char[] line, String word) {
        char[] seg = new char[line.length];
        int len = 0;
        for (int i = 0; i <= line.length; i++) {
            char ch = (i < line.length) ? line[i] : '#';   // sentinel flushes last run
            if (ch == '#') {
                if (fits(seg, len, word)) return true;
                len = 0;
            } else {
                seg[len++] = ch;
            }
        }
        return false;
    }

    public boolean canPlaceWord(char[][] board, String word) {
        int m = board.length, n = board[0].length;
        for (int r = 0; r < m; r++) {
            if (scan(board[r], word)) return true;
        }
        for (int c = 0; c < n; c++) {
            char[] col = new char[m];
            for (int r = 0; r < m; r++) col[r] = board[r][c];
            if (scan(col, word)) return true;
        }
        return false;
    }
}
C++
CPP
#include <bits/stdc++.h>
using namespace std;

class Solution {
    static bool fits(const string& seg, const string& word) {
        int L = word.size();
        if ((int)seg.size() != L) return false;
        bool forward = true, backward = true;
        for (int i = 0; i < L; i++) {
            if (seg[i] != ' ' && seg[i] != word[i])         forward = false;
            if (seg[i] != ' ' && seg[i] != word[L - 1 - i]) backward = false;
        }
        return forward || backward;
    }

    bool scanLine(const string& line, const string& word) {
        string seg;
        for (char ch : line + '#') {          // trailing '#' flushes the last open run
            if (ch == '#') {
                if (fits(seg, word)) return true;
                seg.clear();
            } else {
                seg.push_back(ch);
            }
        }
        return false;
    }

public:
    bool canPlaceWord(vector<vector<char>>& board, string word) {
        int m = board.size(), n = board[0].size();
        for (int r = 0; r < m; r++) {
            string row(board[r].begin(), board[r].end());
            if (scanLine(row, word)) return true;
        }
        for (int c = 0; c < n; c++) {
            string col;
            for (int r = 0; r < m; r++) col.push_back(board[r][c]);
            if (scanLine(col, word)) return true;
        }
        return false;
    }
};
C
C
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <stdbool.h>

/* Does an open run `seg` of length segLen fit `word` reading forward or backward? */
static bool fits(const char* seg, int segLen, const char* word, int L) {
    if (segLen != L) return false;
    bool forward = true, backward = true;
    for (int i = 0; i < L; i++) {
        if (seg[i] != ' ' && seg[i] != word[i])         forward = false;
        if (seg[i] != ' ' && seg[i] != word[L - 1 - i]) backward = false;
    }
    return forward || backward;
}

bool canPlaceWord(char** board, int m, int n, const char* word) {
    int L = (int)strlen(word);
    int maxDim = m > n ? m : n;
    char* seg = (char*)malloc((size_t)maxDim + 1);
    bool ans = false;

    /* horizontal slots: scan each row, split on '#' (right edge acts as '#') */
    for (int r = 0; r < m && !ans; r++) {
        int len = 0;
        for (int c = 0; c <= n; c++) {
            char ch = (c < n) ? board[r][c] : '#';
            if (ch == '#') {
                if (fits(seg, len, word, L)) { ans = true; break; }
                len = 0;
            } else {
                seg[len++] = ch;
            }
        }
    }
    /* vertical slots: scan each column, split on '#' (bottom edge acts as '#') */
    for (int c = 0; c < n && !ans; c++) {
        int len = 0;
        for (int r = 0; r <= m; r++) {
            char ch = (r < m) ? board[r][c] : '#';
            if (ch == '#') {
                if (fits(seg, len, word, L)) { ans = true; break; }
                len = 0;
            } else {
                seg[len++] = ch;
            }
        }
    }
    free(seg);
    return ans;
}

#### Follow-up
questions
Instead of just true/false, return the starting coordinate and direction of a valid placement.
Count how many distinct valid placements exist for the word.
Support diagonal placements in addition to horizontal and vertical.
Given a list of words, return which ones can be placed (and handle very large grids efficiently).

