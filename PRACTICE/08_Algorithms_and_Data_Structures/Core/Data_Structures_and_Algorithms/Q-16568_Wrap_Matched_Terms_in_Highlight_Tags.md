---
type: "practice"
difficulty: "Other"
---

# Wrap Matched Terms in Highlight Tags

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Hash Table](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Hash_Table.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given a list of search terms patterns and a single lowercase text string text. Your task is to surround every occurrence of any pattern inside text with the markup tags <b> (opening) and </b> (closing), so that all matched characters appear highlighted.

Occurrences may overlap, and two matched regions may sit right next to each other. Whenever highlighted characters are adjacent or overlapping, they must be covered by a single pair of tags rather than several — you have to use the fewest possible tags while still highlighting exactly the characters that belong to at least one matched pattern. A character is highlighted if and only if it lies inside at least one occurrence of some pattern.

Return the resulting string after the tags have been inserted. Characters that are not part of any match are left untouched.

Input

patterns: an array of strings (the terms to search for). This array may be empty.
text: the string in which matches are highlighted.

Output

A single string: text with <b>...</b> tags inserted around every maximal run of highlighted characters, using the minimum number of tags.

#### Cons
traints

1 <= text.length <= 500
0 <= patterns.length <= 50
1 <= patterns[i].length <= 10
text and every patterns[i] consist only of lowercase English letters.

Notes

If patterns is empty or nothing matches, return text unchanged.
A match is a contiguous substring equal to some pattern; the same character position may be covered by several patterns.

#### Example
s & test cases
TEXT
Input: patterns = ["on", "ne"], text = "money" -> Output: "m<b>one</b>y"
Why: "on" covers indices 1-2 and "ne" covers indices 2-3; the two overlapping runs merge into one highlighted block "one".

Input: patterns = ["ce", "el"], text = "excel" -> Output: "ex<b>cel</b>"
Why: "ce" covers indices 2-3 and "el" covers indices 3-4; adjacent/overlapping runs merge into a single tag pair.

Input: patterns = ["co", "in"], text = "coding" -> Output: "<b>co</b>d<b>in</b>g"
Why: two separate, non-adjacent matches each get their own tag pair.

Input: patterns = ["xy"], text = "abc" -> Output: "abc"
Why: no pattern occurs in the text, so it is returned unchanged.

Input: patterns = [], text = "hello" -> Output: "hello"
Why: an empty pattern list means nothing is highlighted.

Input: patterns = ["aa", "aaa"], text = "aaaa" -> Output: "<b>aaaa</b>"
Why: overlapping matches (including duplicates covering the same region) collapse into a single highlighted run spanning the whole string.

#### Approach

Reduce the problem to "which character positions must be highlighted," then merge. Build a boolean array bold of length n. For each pattern, find all of its occurrences in the text (via repeated substring search) and set bold[k] = true for every character index k inside each occurrence. This automatically unions overlapping and duplicate matches. Finally, walk the text once: open a <b> tag at the start of any maximal run of highlighted positions (a position that is bold while the previous one is not) and close </b> at the end of that run (bold while the next one is not). This "mark-then-emit maximal runs" technique is the interval-coverage / merge pattern applied on a per-character bitmap, and it guarantees the minimum number of tags because each contiguous highlighted block yields exactly one tag pair.

Complexity

Time: O(n * m * L), where n = length of text, m = number of patterns, and L = maximum pattern length — each pattern is scanned across the text and each match marks up to L positions. Space: O(n) for the boolean coverage array plus O(n) for the output. Justification: marking is dominated by scanning every start position for every pattern; emitting the tagged string is a single O(n) pass.

Python
PYTHON
from typing import List

class Solution:
    def highlight(self, patterns: List[str], text: str) -> str:
        n = len(text)
        bold = [False] * n

        # Mark every position covered by some pattern occurrence.
        for pat in patterns:
            if not pat:
                continue
            start = text.find(pat)
            while start != -1:
                for k in range(start, start + len(pat)):
                    bold[k] = True
                start = text.find(pat, start + 1)

        # Emit maximal runs wrapped in a single tag pair.
        res = []
        for i in range(n):
            if bold[i] and (i == 0 or not bold[i - 1]):
                res.append("<b>")
            res.append(text[i])
            if bold[i] and (i == n - 1 or not bold[i + 1]):
                res.append("</b>")
        return "".join(res)
JavaScript
JAVASCRIPT
function highlight(patterns, text) {

#### cons
t n = text.length;

#### cons
t bold = new Array(n).fill(false);

  // Mark every position covered by some pattern occurrence.
  for (const pat of patterns) {
    if (pat.length === 0) continue;
    let pos = text.indexOf(pat);
    while (pos !== -1) {
      for (let k = pos; k < pos + pat.length; k++) bold[k] = true;
      pos = text.indexOf(pat, pos + 1);
    }
  }

  // Emit maximal runs wrapped in a single tag pair.
  let res = "";
  for (let i = 0; i < n; i++) {
    if (bold[i] && (i === 0 || !bold[i - 1])) res += "<b>";
    res += text[i];
    if (bold[i] && (i === n - 1 || !bold[i + 1])) res += "</b>";
  }
  return res;
}
Java
JAVA
class Solution {
  public String highlight(String[] patterns, String text) {
    int n = text.length();
    boolean[] bold = new boolean[n];

    // Mark every position covered by some pattern occurrence.
    for (String pat : patterns) {
      int L = pat.length();
      if (L == 0) continue;
      int pos = text.indexOf(pat);
      while (pos != -1) {
        for (int k = pos; k < pos + L; ++k) bold[k] = true;
        pos = text.indexOf(pat, pos + 1);
      }
    }

    // Emit maximal runs wrapped in a single tag pair.
    StringBuilder sb = new StringBuilder();
    for (int i = 0; i < n; ++i) {
      if (bold[i] && (i == 0 || !bold[i - 1])) sb.append("<b>");
      sb.append(text.charAt(i));
      if (bold[i] && (i == n - 1 || !bold[i + 1])) sb.append("</b>");
    }
    return sb.toString();
  }
}
C++
CPP
#include <string>
#include <vector>
using namespace std;

class Solution {
 public:
  string highlight(vector<string>& patterns, string text) {

#### cons
t int n = (int)text.size();
    vector<bool> bold(n, false);

    // Mark every position covered by some pattern occurrence.
    for (const string& pat : patterns) {

#### cons
t int L = (int)pat.size();
      if (L == 0) continue;
      for (int i = 0; i + L <= n; ++i)
        if (text.compare(i, L, pat) == 0)
          for (int k = i; k < i + L; ++k) bold[k] = true;
    }

    // Emit maximal runs wrapped in a single tag pair.
    string res;
    for (int i = 0; i < n; ++i) {
      if (bold[i] && (i == 0 || !bold[i - 1])) res += "<b>";
      res += text[i];
      if (bold[i] && (i == n - 1 || !bold[i + 1])) res += "</b>";
    }
    return res;
  }
};
C
C
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <stdbool.h>

// Returns a newly malloc'd string; caller frees it.
char* highlight(char** patterns, int patternsSize, char* text) {
    int n = (int)strlen(text);
    bool* bold = (bool*)calloc(n > 0 ? n : 1, sizeof(bool));

    // Mark every position covered by at least one pattern occurrence.
    for (int p = 0; p < patternsSize; ++p) {
        int L = (int)strlen(patterns[p]);
        if (L == 0) continue;
        for (int i = 0; i + L <= n; ++i) {
            if (strncmp(text + i, patterns[p], L) == 0) {
                for (int k = i; k < i + L; ++k) bold[k] = true;
            }
        }
    }

    // Worst case each char gets <b> and </b> (3 + 4 extra chars).
    char* res = (char*)malloc((size_t)n * 8 + 1);
    int idx = 0;
    for (int i = 0; i < n; ++i) {
        if (bold[i] && (i == 0 || !bold[i - 1])) {
            res[idx++] = '<'; res[idx++] = 'b'; res[idx++] = '>';
        }
        res[idx++] = text[i];
        if (bold[i] && (i == n - 1 || !bold[i + 1])) {
            res[idx++] = '<'; res[idx++] = '/'; res[idx++] = 'b'; res[idx++] = '>';
        }
    }
    res[idx] = '\0';
    free(bold);
    return res;
}

#### Follow-up
questions
With many patterns, avoid the per-pattern rescan by building an Aho-Corasick automaton (or a trie) and matching all patterns in a single O(n + total pattern length + matches) pass over the text.
Instead of returning the tagged string, return the merged list of highlighted intervals [start, end).
Make matching case-insensitive, or extend to Unicode; how does that change the marking step?
If the text arrives as a stream too large to buffer, how would you highlight without holding the entire boolean array in memory?

