---
type: "practice"
difficulty: "Other"
---

# Collapse Runs of Exactly K Equal Characters

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Stack](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Stack.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given a string text and an integer k. A single collapse operation consists of picking any k adjacent, equal characters in text and deleting all k of them; after deletion the characters immediately to the left and right of the removed block become adjacent and may themselves now form a new block of k equal characters.

You repeatedly perform collapse operations for as long as any block of k adjacent equal characters exists.

Return the final string once no further collapse is possible. It is guaranteed that the final result is unique regardless of the order in which collapses are performed.

Input

text: a string of lowercase English letters.
k: an integer, the run length that triggers a removal.

Output

The string remaining after all possible collapses have been applied (which may be empty).

#### Cons
traints

1 <= len(text) <= 10^5
2 <= k <= 10^4
text consists of only lowercase English letters.

#### Example
s & test cases
TEXT
Input: text = "xyzw", k = 2 -> Output: "xyzw"
Why: no two adjacent characters are equal, so there is nothing to collapse.

Input: text = "ppqqqppr", k = 3 -> Output: "pr"
Why: remove "qqq" to get "pppp r" wait — after removing "qqq" the two leading p's meet the following p forming "ppp", which collapses too, leaving "pr".

Input: text = "abbaccd", k = 2 -> Output: "d"
Why: remove "bb" -> "aaccd", then "aa" -> "ccd", then "cc" -> "d".

Input: text = "yyyyyy", k = 3 -> Output: ""
Why: six identical characters collapse in two rounds of three, removing everything.

Input: text = "lllm", k = 4 -> Output: "lllm"
Why: the longest run is only three l's, which is below k, so no collapse happens.

Input: text = "wwww", k = 2 -> Output: ""
Why: remove a pair to get "ww", then remove the remaining pair, leaving the empty string.

#### Approach

Use a stack of (character, run-length) pairs instead of physically deleting from the string. Scan left to right: if the current character equals the character on top of the stack, increment that entry's run length; the moment a run length reaches k, pop the entry (that block has fully collapsed), which automatically lets the previous block become adjacent to whatever comes next — this captures the cascading removals for free. If the current character differs from the top, push a fresh entry with count 1. After the scan, expand the surviving stack entries (each character repeated by its remaining count) to build the answer. This is the classic "run-length stack" / adjacent-group-collapse pattern and does everything in a single pass.

Complexity

Time: O(n), where n is the length of text; every character is pushed onto the stack once and removed at most once. Space: O(n) for the stack that holds the surviving characters and their run counts (worst case, no collapse occurs).

Python
PYTHON
def collapse_runs(text: str, k: int) -> str:
    stack = []  # each entry: [character, current run length]
    for c in text:
        if stack and stack[-1][0] == c:
            stack[-1][1] += 1
            if stack[-1][1] == k:
                stack.pop()
        else:
            stack.append([c, 1])
    return ''.join(ch * cnt for ch, cnt in stack)
JavaScript
JAVASCRIPT
function collapseRuns(text, k) {

#### cons
t stack = []; // [character, count]
    for (const c of text) {

#### cons
t top = stack[stack.length - 1];
        if (top && top[0] === c) {
            if (++top[1] === k) stack.pop();
        } else {
            stack.push([c, 1]);
        }
    }
    let result = "";
    for (const [ch, cnt] of stack) result += ch.repeat(cnt);
    return result;
}
Java
JAVA
class Solution {
    public String collapseRuns(String text, int k) {
        int n = text.length();
        char[] chars = new char[n];
        int[] counts = new int[n];
        int top = -1;
        for (int i = 0; i < n; i++) {
            char c = text.charAt(i);
            if (top >= 0 && chars[top] == c) {
                counts[top]++;
                if (counts[top] == k) top--;
            } else {
                top++;
                chars[top] = c;
                counts[top] = 1;
            }
        }
        StringBuilder sb = new StringBuilder();
        for (int i = 0; i <= top; i++)
            for (int j = 0; j < counts[i]; j++)
                sb.append(chars[i]);
        return sb.toString();
    }
}
C++
CPP
#include <string>
#include <vector>
#include <utility>
using namespace std;

string collapseRuns(const string& text, int k) {
    vector<pair<char,int>> st; // (character, current run length)
    for (char c : text) {
        if (!st.empty() && st.back().first == c) {
            if (++st.back().second == k) st.pop_back();
        } else {
            st.push_back({c, 1});
        }
    }
    string result;
    for (auto& p : st) result.append((size_t)p.second, p.first);
    return result;
}
C
C
#include <string.h>
#include <stdlib.h>

char* collapseRuns(const char* text, int k) {
    int n = (int)strlen(text);
    char* chars = (char*)malloc((size_t)(n > 0 ? n : 1));
    int* counts = (int*)malloc(sizeof(int) * (size_t)(n > 0 ? n : 1));
    int top = -1;
    for (int i = 0; i < n; i++) {
        char c = text[i];
        if (top >= 0 && chars[top] == c) {
            counts[top]++;
            if (counts[top] == k) top--;
        } else {
            top++;
            chars[top] = c;
            counts[top] = 1;
        }
    }
    int len = 0;
    for (int i = 0; i <= top; i++) len += counts[i];
    char* result = (char*)malloc((size_t)len + 1);
    int pos = 0;
    for (int i = 0; i <= top; i++)
        for (int j = 0; j < counts[i]; j++)
            result[pos++] = chars[i];
    result[pos] = '\0';
    free(chars);
    free(counts);
    return result;
}

#### Follow-up
questions
What changes if you must remove any run of length at least k rather than exactly k?
Can you achieve O(1) extra space by collapsing in place within the input character array, using a write pointer plus a parallel count array?
Suppose k can vary and is supplied as a list applied in sequence — how do you generalize the stack entry?
If the alphabet were huge (arbitrary Unicode code points), does the approach still hold, and what would the memory footprint be?

