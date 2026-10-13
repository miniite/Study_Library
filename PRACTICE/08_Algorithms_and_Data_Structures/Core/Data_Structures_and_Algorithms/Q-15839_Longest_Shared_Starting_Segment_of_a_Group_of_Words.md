---
type: "practice"
difficulty: "Other"
---

# Longest Shared Starting Segment of a Group of Words

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Trie](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Trie.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given a collection of words. Determine the longest string that appears at the very start of every word in the collection — that is, the longest common leading segment shared by all of them.

If the words share no common leading segment, return an empty string "".

Input

words: an array of strings. The array has at least one element. Individual strings may be empty, and all non-empty strings consist only of lowercase English letters.

Output

A single string: the longest prefix common to all the words. Return "" when there is no common prefix (this includes the case where any word is the empty string).

#### Cons
traints

1 <= words.length <= 200
0 <= words[i].length <= 200
Each words[i] consists only of lowercase English letters when it is non-empty.

#### Example
s & test cases
TEXT
Input: words = ["glowworm", "glow", "globe"] -> Output: "glo"
Why: All three begin with `glo`; the fourth character differs (`w` vs the end vs `b`), so the shared segment stops there.

Input: words = ["sun", "raincoat", "car"] -> Output: ""
Why: The very first characters already disagree, so no common leading segment exists.

Input: words = ["repeat", "repeat", "repeat"] -> Output: "repeat"
Why: Identical words; the entire word is the common prefix.

Input: words = ["alpha"] -> Output: "alpha"
Why: A single word trivially shares its whole self as the prefix.

Input: words = ["mount", "mountain", ""] -> Output: ""
Why: One empty string forces the common prefix to length 0 regardless of the others.

Input: words = ["node", "nodes", "nod"] -> Output: "nod"
Why: The shortest word `nod` caps the answer; scanning must stop when the shortest string is exhausted.

#### Approach

Use vertical (column-by-column) scanning. Treat the first word as the candidate prefix and walk its characters left to right. For each character position i, check that same position in every other word: if any word is shorter than i+1 characters or has a different character there, the common prefix ends just before i, so return the first word's substring up to i. If the outer loop finishes without a mismatch, the entire first word is common to all, so return it. This is optimal because the answer can never be longer than the shortest word and never extend past the first disagreement, and vertical scanning stops at exactly that first point of divergence without extra data structures. The underlying pattern is character-column comparison across parallel strings.

Complexity

Time: O(S) where S is the total number of characters across all words — in the worst case every character of every string is compared once. Space: O(1) beyond the returned prefix, since only a handful of index variables are used.

Python
PYTHON
from typing import List

def longest_common_prefix(words: List[str]) -> str:
    first = words[0]
    for i in range(len(first)):
        c = first[i]
        for j in range(1, len(words)):
            # Word j ended, or its i-th character differs.
            if i == len(words[j]) or words[j][i] != c:
                return first[:i]
    return first   # first word is a prefix of all others
JavaScript
JAVASCRIPT
function longestCommonPrefix(words) {

#### cons
t first = words[0];
    for (let i = 0; i < first.length; i++) {

#### cons
t c = first[i];
        for (let j = 1; j < words.length; j++) {
            // Word j ended, or its i-th character differs.
            if (i === words[j].length || words[j][i] !== c) {
                return first.slice(0, i);
            }
        }
    }
    return first;   // first word is a prefix of all others
}
Java
JAVA
class Solution {
    public String longestCommonPrefix(String[] words) {
        for (int i = 0; i < words[0].length(); i++) {
            char c = words[0].charAt(i);
            for (int j = 1; j < words.length; j++)
                // Word j is too short, or mismatches at position i.
                if (i == words[j].length() || words[j].charAt(i) != c)
                    return words[0].substring(0, i);
        }
        return words[0];   // first word is a prefix of all others
    }
}
C++
CPP
#include <string>
#include <vector>
using namespace std;

class Solution {
public:
    string longestCommonPrefix(vector<string>& words) {
        for (int i = 0; i < (int)words[0].size(); ++i) {
            char c = words[0][i];
            for (int j = 1; j < (int)words.size(); ++j)
                // Word j ran out, or its i-th char differs.
                if (i == (int)words[j].size() || words[j][i] != c)
                    return words[0].substr(0, i);
        }
        return words[0];   // first word is a prefix of all others
    }
};
C
C
#include <stdlib.h>
#include <string.h>

/* Returns a malloc'd string holding the longest common prefix. */
char* longestCommonPrefix(char** words, int wordsSize) {
    /* Start by assuming the whole first word is the prefix. */
    int len = (int)strlen(words[0]);
    for (int i = 0; i < len; i++) {
        char c = words[0][i];
        for (int j = 1; j < wordsSize; j++) {
            /* Mismatch, or this word ended before position i. */
            if (words[j][i] == '\0' || words[j][i] != c) {
                char* out = (char*)malloc(i + 1);
                memcpy(out, words[0], i);
                out[i] = '\0';
                return out;
            }
        }
    }
    char* out = (char*)malloc(len + 1);
    memcpy(out, words[0], len + 1);   /* copy including terminator */
    return out;
}

#### Follow-up
questions
Could you sort the words first and compare only the lexicographically smallest and largest strings — why does that suffice?
How would you support fast repeated prefix queries over a fixed dictionary using a trie?
What changes if the strings may contain arbitrary Unicode characters rather than lowercase letters?
Can you find the longest common suffix instead by adapting the same scan?

