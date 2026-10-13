---
type: "practice"
difficulty: "Other"
---

# Fewest Segments With a Bounded Numeric Value

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Two Pointers](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Two_Pointers.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given a string s made up of decimal digits, where every character is one of '1', '2', ..., '9' (there are no zero digits), together with a positive integer cap.

You must split s into one or more contiguous, non-overlapping pieces so that:

Every character of s belongs to exactly one piece (the pieces, read left to right, exactly reconstruct s).
When a piece is read as a base-10 integer, its value is at most cap. For example, the piece "27" has value 27 and the piece "3" has value 3.

Among all valid ways to split the string, return the minimum possible number of pieces. If no valid split exists (which happens exactly when some single digit is already larger than cap), return -1.

Input

s: a non-empty string of digits, each in '1'..'9'.
cap: a positive integer upper bound on the value of each piece.

Output

The minimum number of pieces in a valid split, or -1 if none exists.

#### Cons
traints

1 <= s.length <= 10^5
Each character of s is a digit from '1' to '9'.
1 <= cap <= 10^9

#### Example
s & test cases
TEXT
- Input: `s = "382919", cap = 40` -> Output: `3`
  Why: a minimal split is `"38" | "29" | "19"`; every piece is <= 40 and no split uses fewer than 3 pieces.
- Input: `s = "9", cap = 9` -> Output: `1`
  Why: the single digit `9` already satisfies `9 <= 9`, so one piece suffices (smallest input).
- Input: `s = "1111", cap = 1` -> Output: `4`
  Why: any two-digit piece is at least `11 > 1`, so each digit must stand alone, giving 4 pieces.
- Input: `s = "55", cap = 100` -> Output: `1`
  Why: the whole string as one piece has value `55 <= 100`, so a single piece is optimal.
- Input: `s = "47", cap = 3` -> Output: `-1`
  Why: the digit `4` alone already exceeds `cap = 3`, so no valid split can exist (impossible case).
- Input: `s = "999999", cap = 9` -> Output: `6`
  Why: `99 > 9`, so no two digits can share a piece; each of the six `9`s is its own piece.

#### Approach

This is a classic "minimum number of segments" greedy. Sweep left to right and grow the current piece as long as appending the next digit keeps its value <= cap (using the incremental update cur = cur * 10 + digit). The instant the next digit would overflow the cap, close the current piece, increment the count, and start a fresh piece at that digit. Making each piece as long as possible is provably optimal: any split point reachable after a shorter first piece is also reachable after the longest valid first piece (the greedy prefix is a superset of every alternative), so being greedy can never force more pieces later — an exchange argument. Impossibility is detected naturally: if the very first digit of a new piece already exceeds cap, then cur stays 0 and no split can ever place that digit, so return -1. Because cur and cap can reach 10^9, the running value is held in a 64-bit integer (long/long long) so cur * 10 cannot overflow.

Complexity

Time: O(n) where n = s.length — the index i only ever moves forward, so every character is visited a constant number of times across both loops. Space: O(1) — only the running piece value cur and a couple of counters are kept; no DP table or extra structure is needed. Justification: the greedy sweep touches each digit once.

Python
PYTHON
class Solution:
    def minSegments(self, s: str, cap: int) -> int:
        n = len(s)
        count = 0
        i = 0
        while i < n:
            cur = 0                                  # value of the piece being grown
            while i < n and cur * 10 + (ord(s[i]) - 48) <= cap:
                cur = cur * 10 + (ord(s[i]) - 48)    # extend the piece by one digit
                i += 1
            if cur == 0:                             # first digit already exceeds cap
                return -1
            count += 1                               # close off this piece
        return count
JavaScript
JAVASCRIPT
function minSegments(s, cap) {

#### cons
t n = s.length;
    let count = 0, i = 0;
    while (i < n) {
        let cur = 0;                                 // value of the piece being grown
        while (i < n && cur * 10 + (s.charCodeAt(i) - 48) <= cap) {
            cur = cur * 10 + (s.charCodeAt(i) - 48); // extend the piece by one digit
            i++;
        }
        if (cur === 0) return -1;                    // first digit already exceeds cap
        count++;                                      // close off this piece
    }
    return count;
}
Java
JAVA
class Solution {
  public int minSegments(String s, long cap) {
    int n = s.length(), count = 0, i = 0;
    while (i < n) {
      long cur = 0;
      while (i < n && cur * 10 + (s.charAt(i) - '0') <= cap) {
        cur = cur * 10 + (s.charAt(i) - '0');
        i++;
      }
      if (cur == 0) return -1;   // a single digit is bigger than cap
      count++;
    }
    return count;
  }
}
C++
CPP
class Solution {
 public:
  int minSegments(const string& s, long long cap) {
    int n = (int)s.size(), count = 0, i = 0;
    while (i < n) {
      long long cur = 0;
      while (i < n && cur * 10 + (s[i] - '0') <= cap) {
        cur = cur * 10 + (s[i] - '0');
        i++;
      }
      if (cur == 0) return -1;   // a single digit is bigger than cap
      count++;
    }
    return count;
  }
};
C
C
#include <string.h>

int minSegments(const char* s, long long cap) {
    int n = (int)strlen(s);
    int count = 0, i = 0;
    while (i < n) {
        long long cur = 0;                       /* value of the piece being grown */
        while (i < n && cur * 10 + (s[i] - '0') <= cap) {
            cur = cur * 10 + (s[i] - '0');       /* extend the current piece by one digit */
            i++;
        }
        if (cur == 0) return -1;                 /* first digit already exceeds cap */
        count++;                                 /* close off this piece */
    }
    return count;
}

#### Follow-up
questions
Prove formally that the greedy "longest valid prefix" choice yields the minimum number of pieces (exchange argument).
Modify the problem so the string may also contain the digit 0; how do leading zeros affect a piece's value and your greedy?
Instead of only the count, also return the actual list of pieces (or the split positions).
If the goal were to MAXIMIZE the number of pieces under the same cap, how would the strategy change, and is it still greedy?

