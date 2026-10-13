---
type: "practice"
difficulty: "Other"
---

# Split a String Into Maximum Independent Segments

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Two Pointers](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Two_Pointers.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given a string s made up of lowercase English letters. Split s into the largest possible number of contiguous segments such that no letter appears in more than one segment. Every character of s must belong to exactly one segment, and concatenating the segments back together in order must reproduce s exactly.

In other words, once a letter is placed in a segment, every other occurrence of that same letter must fall in the same segment. Among all valid ways to split the string, you want the one with the maximum number of segments.

Return a list of integers giving the length of each segment, in left-to-right order.

For instance, if s = "abbacdcefgfe", the answer is [4, 3, 5]: the string breaks into "abba", "cdc", and "efgfe", and within each piece every letter is confined to that piece alone. A split like "abbacdc", "efgfe" would also be valid but uses fewer segments, so it is not the answer.

Input: a string s of lowercase English letters.

Output: a list of integers — the sizes of the maximum-count set of segments, in order.

#### Cons
traints:

1 <= s.length <= 500
s consists only of lowercase English letters ('a' to 'z').

#### Example
s & test cases
TEXT
Input: s = "abbacdcefgfe" -> Output: [4, 3, 5]
Why: each letter is fully contained inside one of "abba", "cdc", "efgfe", and no finer split keeps that property.

Input: s = "xyzyx" -> Output: [5]
Why: 'x' appears at both the first and last positions, so the whole string must stay as a single segment.

Input: s = "abcde" -> Output: [1, 1, 1, 1, 1]
Why: every letter is distinct and occurs once, so each character forms its own segment.

Input: s = "aaaa" -> Output: [4]
Why: the single repeated letter forces one segment spanning all its occurrences.

Input: s = "z" -> Output: [1]
Why: minimum-length input; the lone character is its own segment.

Input: s = "abacbd" -> Output: [5, 1]
Why: 'a', 'b', 'c' interlock across indices 0..4 forming "abacb", then the final 'd' stands alone.

#### Approach

This is a greedy sweep with two pointers. First precompute, for every distinct letter, the index of its last occurrence. Then scan the string maintaining the current segment's start and a running end that is the maximum last-occurrence index among all letters seen since the segment began. That end marks the earliest position at which the current segment is allowed to close, because every letter encountered so far must be fully contained. When the scan index reaches end, no letter inside the current window appears later, so the segment is complete: record its length, then start a fresh segment right after. Extending greedily to the farthest required index and cutting at the first legal point guarantees the maximum number of segments.

Complexity

Time: O(n) where n is the length of s — one pass records the last index of each letter and a second pass sweeps through building the segments. Space: O(1) — the last-index table holds at most 26 entries for a fixed lowercase alphabet (O(k) for an alphabet of size k), not counting the output list.

Python
PYTHON
from typing import List

class Solution:
    def partitionLabels(self, s: str) -> List[int]:
        last = {c: i for i, c in enumerate(s)}
        ans = []
        start = end = 0
        for i, c in enumerate(s):
            end = max(end, last[c])
            if i == end:
                ans.append(i - start + 1)
                start = i + 1
        return ans
JavaScript
JAVASCRIPT
function partitionLabels(s) {

#### cons
t last = {};
    for (let i = 0; i < s.length; i++) last[s[i]] = i;

#### cons
t ans = [];
    let start = 0, end = 0;
    for (let i = 0; i < s.length; i++) {
        end = Math.max(end, last[s[i]]);
        if (i === end) {
            ans.push(i - start + 1);
            start = i + 1;
        }
    }
    return ans;
}
Java
JAVA
import java.util.*;

class Solution {
    public List<Integer> partitionLabels(String s) {
        int[] last = new int[26];
        for (int i = 0; i < s.length(); ++i)
            last[s.charAt(i) - 'a'] = i;

        List<Integer> ans = new ArrayList<>();
        int start = 0, end = 0;
        for (int i = 0; i < s.length(); ++i) {
            end = Math.max(end, last[s.charAt(i) - 'a']);
            if (i == end) {
                ans.add(i - start + 1);
                start = i + 1;
            }
        }
        return ans;
    }
}
C++
CPP
class Solution {
public:
    vector<int> partitionLabels(string s) {
        vector<int> last(26, 0);
        for (int i = 0; i < (int) s.size(); ++i)
            last[s[i] - 'a'] = i;

        vector<int> ans;
        int start = 0, end = 0;
        for (int i = 0; i < (int) s.size(); ++i) {
            end = max(end, last[s[i] - 'a']);
            if (i == end) {
                ans.push_back(i - start + 1);
                start = i + 1;
            }
        }
        return ans;
    }
};
C
C
#include <string.h>
#include <stdlib.h>

int* partitionLabels(char* s, int* returnSize) {
    int last[26];
    int len = (int) strlen(s);
    for (int i = 0; i < len; ++i)
        last[s[i] - 'a'] = i;

    int* ans = (int*) malloc(sizeof(int) * len);
    int count = 0, start = 0, end = 0;
    for (int i = 0; i < len; ++i) {
        int li = last[s[i] - 'a'];
        if (li > end) end = li;
        if (i == end) {
            ans[count++] = i - start + 1;
            start = i + 1;
        }
    }
    *returnSize = count;
    return ans;
}

#### Follow-up
questions
Return the actual substrings for each segment rather than just their lengths.
What changes if you must instead minimize the number of segments? (The whole string becomes one segment.)
Support an arbitrary or Unicode alphabet — replace the fixed 26-slot array with a hash map of last indices.
If the string arrives as a stream and you cannot precompute last indices, how would you adapt the approach?

