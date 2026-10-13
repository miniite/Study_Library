---
type: "practice"
difficulty: "Other"
---

# Residue String Before the Final Alphabet Sweep

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Hash Table](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Hash_Table.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given a string w made up only of lowercase English letters.

Repeat the following sweep on w until it becomes empty:

Scan the letters of the alphabet in order from 'a' to 'z'. For each letter, if at least one copy of that letter is still present in w, delete its first (leftmost) remaining occurrence.

Each sweep therefore removes exactly one copy of every distinct letter that is currently present. Repeated sweeps eventually empty the string.

Return the contents of w immediately before the final sweep is applied — that is, the string that remains right before the last sweep that empties it.

Input

A single string w.

Output

The string that exists just before the last sweep is performed.

#### Cons
traints

1 <= len(w) <= 5 * 10^5
w consists only of lowercase English letters 'a'–'z'.

Key observation to guide your solution: the number of sweeps equals the maximum number of times any single letter appears. A letter survives into the last remaining string only if its frequency equals that maximum, and the copy that survives is its last occurrence in the original string.

#### Example
s & test cases
TEXT
Input: w = "ppqrqqrp" -> Output: "qp"
Why: 'p' and 'q' each appear 3 times (the max), 'r' twice. After 2 sweeps only one 'p' and one 'q' remain — their last occurrences — ordered by position gives "qp".

Input: w = "wxyz" -> Output: "wxyz"
Why: all letters are distinct, so a single sweep empties the string; the string before that only (final) sweep is the whole input.

Input: w = "aabbbc" -> Output: "b"
Why: 'b' appears 3 times (the max), 'a' twice, 'c' once. Only 'b' survives to the last sweep, leaving its last occurrence.

Input: w = "abcabc" -> Output: "abc"
Why: every letter appears twice (all tie for the max), so one of each survives (their last occurrences), giving "abc" in position order.

Input: w = "mmmm" -> Output: "m"
Why: single repeated letter; 3 sweeps remove three copies, and the string just before the 4th (final) sweep is "m".

Input: w = "z" -> Output: "z"
Why: single character; the only sweep is the final one, so the string before it is the input itself.

#### Approach

This is a frequency-counting / simulation-elimination problem. Each sweep removes one copy of every present letter, so the total number of sweeps equals the maximum letter frequency M. After M-1 sweeps, any letter with frequency below M is already gone, and any letter with frequency exactly M has exactly one copy left. Because sweeps always delete the earliest remaining copy first, the surviving copy of each max-frequency letter is its last occurrence in the original string. So the answer is simply: find the maximum frequency, then output the characters whose frequency equals that maximum, each at its last-occurrence position, in left-to-right order. This avoids any actual simulation (which would be quadratic) and runs in a single linear scan.

Complexity

Time: O(n) — two linear passes over the string (one to count frequencies and record last positions, one to build the answer). Space: O(1) — only fixed-size 26-element arrays for counts and last-seen indices, independent of input length.

Python
PYTHON
def residue_before_final_sweep(w: str) -> str:
    freq = [0] * 26
    last = [-1] * 26
    for i, ch in enumerate(w):
        c = ord(ch) - 97
        freq[c] += 1
        last[c] = i

    max_freq = max(freq)

    out = []
    for i, ch in enumerate(w):
        c = ord(ch) - 97
        if freq[c] == max_freq and last[c] == i:
            out.append(ch)
    return "".join(out)
JavaScript
JAVASCRIPT
function residueBeforeFinalSweep(w) {

#### cons
t freq = new Array(26).fill(0);

#### cons
t last = new Array(26).fill(-1);

  for (let i = 0; i < w.length; i++) {

#### cons
t c = w.charCodeAt(i) - 97;
    freq[c]++;
    last[c] = i;
  }

#### cons
t maxFreq = Math.max(...freq);

  let res = "";
  for (let i = 0; i < w.length; i++) {

#### cons
t c = w.charCodeAt(i) - 97;
    if (freq[c] === maxFreq && last[c] === i) res += w[i];
  }
  return res;
}
Java
JAVA
class Solution {
    public String residueBeforeFinalSweep(String w) {
        int[] freq = new int[26];
        int[] last = new int[26];
        java.util.Arrays.fill(last, -1);

        int n = w.length();
        for (int i = 0; i < n; i++) {
            int c = w.charAt(i) - 'a';
            freq[c]++;
            last[c] = i;
        }

        int maxFreq = 0;
        for (int f : freq) maxFreq = Math.max(maxFreq, f);

        StringBuilder sb = new StringBuilder();
        for (int i = 0; i < n; i++) {
            int c = w.charAt(i) - 'a';
            if (freq[c] == maxFreq && last[c] == i)
                sb.append(w.charAt(i));
        }
        return sb.toString();
    }
}
C++
CPP
#include <string>
#include <vector>
#include <algorithm>
using namespace std;

string residueBeforeFinalSweep(const string& w) {
    vector<int> freq(26, 0), last(26, -1);
    int n = (int)w.size();
    for (int i = 0; i < n; i++) {
        int c = w[i] - 'a';
        freq[c]++;
        last[c] = i;
    }

    int maxFreq = 0;
    for (int f : freq) maxFreq = max(maxFreq, f);

    string res;
    for (int i = 0; i < n; i++) {
        int c = w[i] - 'a';
        if (freq[c] == maxFreq && last[c] == i)
            res.push_back(w[i]);
    }
    return res;
}
C
C
#include <stdio.h>
#include <string.h>
#include <stdlib.h>

/* Returns the string remaining just before the last sweep. Caller frees. */
char* residueBeforeFinalSweep(const char* w) {
    int freq[26] = {0};
    int last[26];
    for (int i = 0; i < 26; i++) last[i] = -1;

    int n = (int)strlen(w);
    for (int i = 0; i < n; i++) {
        int c = w[i] - 'a';
        freq[c]++;
        last[c] = i;
    }

    int maxFreq = 0;
    for (int i = 0; i < 26; i++)
        if (freq[i] > maxFreq) maxFreq = freq[i];

    char* res = (char*)malloc((size_t)n + 1);
    int k = 0;
    for (int i = 0; i < n; i++) {
        int c = w[i] - 'a';
        if (freq[c] == maxFreq && last[c] == i)
            res[k++] = w[i];
    }
    res[k] = '\0';
    return res;
}

int main(void) {
    char* r = residueBeforeFinalSweep("ppqrqqrp");
    printf("%s\n", r);   /* qp */
    free(r);
    return 0;
}

#### Follow-up
questions
What if the answer required is the count of characters remaining before the last sweep rather than the string itself?
How would you handle a mixed-case or full-Unicode alphabet where the fixed 26-size arrays no longer apply?
Can you produce the entire trace (the string after every sweep) efficiently, or prove why naive simulation is O(n * M)?
What if the removal order were reversed (delete the last occurrence of each letter each sweep) — how does the surviving-copy rule change?

