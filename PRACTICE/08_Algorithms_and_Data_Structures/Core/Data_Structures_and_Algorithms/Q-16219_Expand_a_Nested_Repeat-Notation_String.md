---
type: "practice"
difficulty: "Other"
---

# Expand a Nested Repeat-Notation String

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Recursion](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Recursion.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given a compressed string text written in a simple repeat notation and must return its fully expanded form.

The notation is count[segment], which means the string segment is written out exactly count times, where count is a positive integer. The segments may be nested, and a segment may contain plain letters, more count[...] groups, or both, in any order. Plain letters that sit outside any brackets are copied as-is.

You may assume the input is always well-formed: brackets are balanced and properly nested, there are no spaces, and digits appear only as repeat counts (so a group like count[...] always has its opening count immediately before [, and the original letters never contain digits). Expansion happens from the innermost group outward.

Input: a string text consisting of lowercase English letters, digits, and the bracket characters [ and ].

Output: the decoded string with every group expanded.

#### Cons
traints:

1 <= text.length <= 30
text consists only of lowercase English letters, digits 0–9, and the characters [ and ].
text is guaranteed to be valid input.
Every repeat count in text is in the range [1, 300].
The test data guarantees the expanded output length never exceeds 10^5.

#### Example
s & test cases
TEXT
Input: text = "2[ab]" -> Output: "abab"
Why: the segment "ab" is repeated twice.

Input: text = "3[x]2[yz]w" -> Output: "xxxyzyzw"
Why: groups sit side by side and the trailing plain letter "w" is copied through unchanged.

Input: text = "2[p3[q]]" -> Output: "pqqqpqqq"
Why: the inner group 3[q] expands to "qqq" first, giving "pqqq", which the outer 2[...] then doubles.

Input: text = "10[a]" -> Output: "aaaaaaaaaa"
Why: the count is a multi-digit number (10), so digit characters must be accumulated, not read one at a time.

Input: text = "abc" -> Output: "abc"
Why: with no brackets at all, the string passes through unchanged.

Input: text = "2[a2[b]c]" -> Output: "abbcabbc"
Why: the inner 2[b] becomes "bb", forming "abbc", and the outer 2[...] repeats that segment twice.

#### Approach

Use two stacks to handle the nesting: one for pending repeat counts and one for the partial string that preceded each [. Walk the input once, keeping a cur buffer for the segment being built and an integer num for the count currently being read. Digits accumulate into num (multiplying by 10 to support multi-digit counts). On [, push num and the current cur onto the stacks, then reset both to start a fresh inner segment. On ], pop the count and the saved prefix, and set cur = prefix + cur * count — this closes the innermost group and stitches its expansion back onto its parent. Plain letters append to cur. Because each ] resolves the innermost group first, the whole structure expands from the inside out, and cur holds the fully decoded string at the end. The pattern is classic stack-based parsing of nested/bracketed structure.

Complexity

Time: O(m) where m is the length of the decoded output — each output character is produced a constant number of times as segments are repeated and concatenated. Space: O(m) for the result, plus O(d) stack space for nesting depth d (bounded by the input length).

Python
PYTHON
class Solution:
    def expandText(self, text: str) -> str:
        stack = []  # (segment_before_bracket, repeat_count)
        cur = ''
        num = 0
        for c in text:
            if c.isdigit():
                num = num * 10 + int(c)
            elif c == '[':
                stack.append((cur, num))
                cur = ''
                num = 0
            elif c == ']':
                prev, rep = stack.pop()
                cur = prev + cur * rep
            else:
                cur += c
        return cur
JavaScript
JAVASCRIPT
function expandText(text) {

#### cons
t counts = [];

#### cons
t segments = [];
    let cur = '';
    let num = 0;
    for (const c of text) {
        if (c >= '0' && c <= '9') {
            num = num * 10 + (c.charCodeAt(0) - 48);
        } else if (c === '[') {
            counts.push(num);
            segments.push(cur);
            num = 0;
            cur = '';
        } else if (c === ']') {

#### cons
t rep = counts.pop();

#### cons
t prev = segments.pop();
            cur = prev + cur.repeat(rep);
        } else {
            cur += c;
        }
    }
    return cur;
}
Java
JAVA
import java.util.*;

class Solution {
    public String expandText(String text) {
        Deque<Integer> counts = new ArrayDeque<>();
        Deque<StringBuilder> segments = new ArrayDeque<>();
        StringBuilder cur = new StringBuilder();
        int num = 0;
        for (char c : text.toCharArray()) {
            if (Character.isDigit(c)) {
                num = num * 10 + (c - '0');
            } else if (c == '[') {
                counts.push(num);
                segments.push(cur);
                num = 0;
                cur = new StringBuilder();
            } else if (c == ']') {
                int rep = counts.pop();
                StringBuilder prev = segments.pop();
                for (int r = 0; r < rep; ++r) prev.append(cur);
                cur = prev;
            } else {
                cur.append(c);
            }
        }
        return cur.toString();
    }
}
C++
CPP
#include <string>
#include <stack>
#include <cctype>
using namespace std;

class Solution {
public:
    string expandText(string text) {
        stack<int> counts;
        stack<string> segments;
        string cur;
        int num = 0;
        for (char c : text) {
            if (isdigit((unsigned char)c)) {
                num = num * 10 + (c - '0');
            } else if (c == '[') {
                counts.push(num);
                segments.push(cur);
                num = 0;
                cur.clear();
            } else if (c == ']') {
                int rep = counts.top(); counts.pop();
                string prev = segments.top(); segments.pop();
                string repeated;
                repeated.reserve(cur.size() * rep);
                while (rep--) repeated += cur;
                cur = prev + repeated;
            } else {
                cur += c;
            }
        }
        return cur;
    }
};
C
C
#include <stdlib.h>
#include <string.h>

char* expandText(char* text) {
    int n = (int)strlen(text);

    int   numStack[64]; int numTop = 0;   /* pending repeat counts        */
    char* strStack[64]; int lenStack[64]; int capStack[64]; int strTop = 0;

    int cap = 16, len = 0;
    char* cur = (char*)malloc(cap);
    cur[0] = '\0';
    int curNum = 0;

    for (int i = 0; i < n; ++i) {
        char c = text[i];
        if (c >= '0' && c <= '9') {
            curNum = curNum * 10 + (c - '0');
        } else if (c == '[') {
            numStack[numTop++] = curNum;
            curNum = 0;
            strStack[strTop] = cur;
            lenStack[strTop] = len;
            capStack[strTop] = cap;
            strTop++;
            cap = 16; len = 0;
            cur = (char*)malloc(cap);
            cur[0] = '\0';
        } else if (c == ']') {
            int   rep     = numStack[--numTop];
            char* prev    = strStack[--strTop];
            int   prevLen = lenStack[strTop];
            int   prevCap = capStack[strTop];
            int   newLen  = prevLen + rep * len;
            while (prevCap <= newLen) prevCap *= 2;
            prev = (char*)realloc(prev, prevCap);
            for (int r = 0; r < rep; ++r) {
                memcpy(prev + prevLen, cur, len);
                prevLen += len;
            }
            prev[prevLen] = '\0';
            free(cur);
            cur = prev; len = prevLen; cap = prevCap;
        } else {
            if (len + 1 >= cap) { cap *= 2; cur = (char*)realloc(cur, cap); }
            cur[len++] = c;
            cur[len] = '\0';
        }
    }
    return cur;
}

#### Follow-up
questions
Solve it with explicit recursion (parse a segment, recursing on nested groups) instead of an explicit stack.
Allow a group with no leading count to default to a single copy (e.g. [ab] means ab).
Detect and report malformed input such as unbalanced brackets rather than assuming validity.
Return only the length of the decoded string without materializing it, so extremely large expansions can be measured.

