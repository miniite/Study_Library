---
type: "practice"
difficulty: "Other"
---

# Length of the Longest Window with All Distinct Characters

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Sliding Window](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Sliding_Window.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given a string text. Find the length of the longest contiguous block of characters (a substring) in which no character appears more than once.

A substring is a run of consecutive characters taken from the original string, in order, without skipping any character in between. Among every substring whose characters are all pairwise distinct, return the maximum length. A subsequence (which may skip characters) does not count — the characters must be adjacent in the original string.

Input

text: a string that may contain letters, digits, punctuation, and spaces. It may be empty.

Output

An integer: the length of the longest substring of text in which every character is unique. If text is empty, the answer is 0.

#### Cons
traints

0 <= text.length <= 5 * 10^4
text consists of printable ASCII characters (uppercase and lowercase letters, digits, symbols, and spaces).
Matching is case-sensitive: 'a' and 'A' are different characters.

#### Example
s & test cases
TEXT
Input: text = "silverware" -> Output: 8
Why: "silverwa" (s,i,l,v,e,r,w,a) has all distinct characters; the next 'r' repeats an earlier 'r', so the window cannot grow further. A normal case where the best window is a long prefix.

Input: text = "zzzz" -> Output: 1
Why: every character is the same, so no window longer than a single character can be distinct. Edge case: all duplicates.

Input: text = "" -> Output: 0
Why: an empty string has no characters, so the longest distinct window has length 0. Edge case: empty input.

Input: text = "pig pen" -> Output: 6
Why: "ig pen" (i,g,space,p,e,n) is all distinct including the space; the leading 'p' is dropped because 'p' repeats later. Confirms spaces are counted as ordinary characters.

Input: text = "aA1a" -> Output: 3
Why: "aA1" (a,A,1) is distinct because matching is case-sensitive; the trailing 'a' repeats the first 'a'. Confirms case-sensitivity and mixed character classes.

Input: text = "tmux" -> Output: 4
Why: the entire string has no repeats, so the answer is its full length. Edge case: whole string already distinct.

#### Approach

Sliding window with last-seen positions. Maintain a window [left, right] that always holds distinct characters. As right scans forward, look up the last index at which the current character was seen. If that index falls inside the current window (last[c] >= left), the window would now contain a duplicate, so jump left to just past that previous occurrence in one step; otherwise the window simply extends. Record the character's new position and update the best length as right - left + 1. The pattern is the classic variable-size sliding window, sped up by storing each character's most recent index so the left boundary can leap directly instead of shrinking one step at a time.

Complexity

Time: O(n) — the right pointer advances once per character and the left pointer only ever moves forward, so each index is processed a constant number of times. Space: O(k) where k is the size of the character set (bounded by 256 for ASCII, i.e. O(1) extra space).

Python
PYTHON
def longest_distinct_window(text: str) -> int:
    last = {}                       # char -> last index seen
    ans = 0
    left = 0
    for right, c in enumerate(text):
        if c in last and last[c] >= left:   # char already in window
            left = last[c] + 1              # shrink window past the old copy
        last[c] = right
        ans = max(ans, right - left + 1)
    return ans
JavaScript
JAVASCRIPT
function longestDistinctWindow(text) {

#### cons
t last = new Map();          // char -> last index seen
    let ans = 0, left = 0;
    for (let right = 0; right < text.length; right++) {

#### cons
t c = text[right];
        if (last.has(c) && last.get(c) >= left)  // char already in window
            left = last.get(c) + 1;              // shrink window past old copy
        last.set(c, right);
        ans = Math.max(ans, right - left + 1);
    }
    return ans;
}
Java
JAVA
import java.util.Arrays;

class Solution {
    public int longestDistinctWindow(String text) {
        int[] last = new int[256];          // last index where each char appeared
        Arrays.fill(last, -1);
        int ans = 0, left = 0;
        for (int right = 0; right < text.length(); right++) {
            char c = text.charAt(right);
            if (last[c] >= left)            // char already in the window
                left = last[c] + 1;         // shrink window past the old copy
            last[c] = right;
            ans = Math.max(ans, right - left + 1);
        }
        return ans;
    }
}
C++
CPP
#include <string>
#include <vector>
#include <algorithm>
using namespace std;

class Solution {
public:
    int longestDistinctWindow(const string& text) {
        vector<int> last(256, -1);          // last index where each char appeared
        int ans = 0, left = 0;
        for (int right = 0; right < (int)text.size(); ++right) {
            unsigned char c = text[right];
            if (last[c] >= left)            // char already in the window
                left = last[c] + 1;         // shrink window past the old copy
            last[c] = right;
            ans = max(ans, right - left + 1);
        }
        return ans;
    }
};
C
C
/* Returns the length of the longest substring of `text` with all distinct chars. */
int longestDistinctWindow(char* text) {
    int last[256];                 /* last[c] = last index where char c was seen */
    for (int i = 0; i < 256; i++) last[i] = -1;

    int ans = 0, left = 0;
    for (int right = 0; text[right] != '\0'; right++) {
        unsigned char c = (unsigned char)text[right];
        if (last[c] >= left)       /* c already inside the current window */
            left = last[c] + 1;    /* jump left just past the previous copy */
        last[c] = right;
        int len = right - left + 1;
        if (len > ans) ans = len;
    }
    return ans;
}

#### Follow-up
questions
What if you also had to return the actual substring (or its start index), not just its length?
How would you handle a full Unicode alphabet instead of ASCII, and how does that change the space bound?
What if you needed the longest window containing at most K distinct characters instead of all-distinct?
Could you solve it if only the count of the longest window over a stream were needed, without storing the whole string?

