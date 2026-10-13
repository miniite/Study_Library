---
type: "practice"
difficulty: "Other"
---

# Longest Balanced-Vowel Segment

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Bit Manipulation](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Bit_Manipulation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given a string text consisting only of lowercase English letters.

A contiguous segment (substring) of text is called balanced if every vowel — that is, each of a, e, i, o, and u — appears an even number of times inside that segment. Note that zero occurrences counts as even, so a segment with no vowels at all is balanced.

Return the length of the longest balanced segment of text. If no non-empty balanced segment exists, return 0.

Input:

text: a string of lowercase English letters.

Output:

A single integer: the maximum length of a contiguous balanced segment.

#### Cons
traints:

1 <= text.length <= 5 * 10^5
text contains only lowercase English letters (a–z).

#### Example
s & test cases
TEXT
Input: text = "xyzxyz" -> Output: 6
Why: The string has no vowels at all, so every vowel count is 0 (even). The whole string is balanced.

Input: text = "abbaee" -> Output: 6
Why: `a` appears twice and `e` appears twice; all vowel counts are even, so the entire string qualifies.

Input: text = "abcde" -> Output: 3
Why: The best balanced segment is "bcd" (no vowels). Extending in either direction pulls in `a` or `e` exactly once, making a vowel count odd.

Input: text = "iiii" -> Output: 4
Why: `i` occurs 4 times (even) and all other vowels 0 times, so the full string is balanced.

Input: text = "a" -> Output: 0
Why: The only non-empty substring "a" has one `a` (odd). No non-empty balanced segment exists.

Input: text = "aa" -> Output: 2
Why: `a` appears exactly twice (even), so the whole two-character string is balanced.

#### Approach

Encode the running parity (even/odd) of the five vowel counts as a 5-bit mask, where bit b toggles each time the matching vowel is read. A segment (j, i] has all vowels even exactly when the prefix parity at j equals the prefix parity at i — their XOR is zero. So for each mask value, remember the earliest index at which it occurred; whenever the same mask reappears at index i, the segment from that first index to i is balanced with length i - firstSeen[mask]. Seed the empty prefix (mask 0) at index -1 so segments starting at position 0 are counted. This is the prefix-XOR / bitmask-state pattern, giving a single linear pass.

Complexity

Time: O(n) — a single pass over the string, constant work per character. Space: O(1) — a fixed table of 2^5 = 32 entries regardless of input size.

Python
PYTHON
def longest_balanced_vowel_segment(text: str) -> int:
    vowel_bit = {'a': 0, 'e': 1, 'i': 2, 'o': 3, 'u': 4}
    first_seen = [-2] * 32     # first_seen[mask] = earliest index of this parity; -2 = unseen
    first_seen[0] = -1         # empty prefix lies before index 0

    mask = 0
    best = 0
    for i, ch in enumerate(text):
        if ch in vowel_bit:
            mask ^= 1 << vowel_bit[ch]
        if first_seen[mask] == -2:
            first_seen[mask] = i
        else:
            best = max(best, i - first_seen[mask])
    return best
JavaScript
JAVASCRIPT
function longestBalancedVowelSegment(text) {

#### cons
t bit = { a: 0, e: 1, i: 2, o: 3, u: 4 };

#### cons
t firstSeen = new Array(32).fill(-2); // -2 = parity mask not yet seen
    firstSeen[0] = -1;                        // empty prefix before index 0

    let mask = 0, best = 0;
    for (let i = 0; i < text.length; i++) {

#### cons
t c = text[i];
        if (c in bit) mask ^= 1 << bit[c];
        if (firstSeen[mask] === -2) firstSeen[mask] = i;
        else best = Math.max(best, i - firstSeen[mask]);
    }
    return best;
}
Java
JAVA
import java.util.Arrays;

class Solution {
    public int longestBalancedVowelSegment(String text) {
        int[] firstSeen = new int[32];   // 2^5 possible vowel-parity masks
        Arrays.fill(firstSeen, -2);      // -2 = not yet seen
        firstSeen[0] = -1;               // empty prefix before index 0

        int mask = 0, best = 0;
        for (int i = 0; i < text.length(); i++) {
            switch (text.charAt(i)) {
                case 'a': mask ^= 1 << 0; break;
                case 'e': mask ^= 1 << 1; break;
                case 'i': mask ^= 1 << 2; break;
                case 'o': mask ^= 1 << 3; break;
                case 'u': mask ^= 1 << 4; break;
                default: break;
            }
            if (firstSeen[mask] == -2) firstSeen[mask] = i;
            else best = Math.max(best, i - firstSeen[mask]);
        }
        return best;
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
    int longestBalancedVowelSegment(const string& text) {
        vector<int> firstSeen(32, -2); // -2 = parity mask not yet observed
        firstSeen[0] = -1;             // empty prefix sits before index 0

        int mask = 0, best = 0;
        for (int i = 0; i < (int)text.size(); i++) {
            switch (text[i]) {
                case 'a': mask ^= 1 << 0; break;
                case 'e': mask ^= 1 << 1; break;
                case 'i': mask ^= 1 << 2; break;
                case 'o': mask ^= 1 << 3; break;
                case 'u': mask ^= 1 << 4; break;
                default: break;
            }
            if (firstSeen[mask] == -2) firstSeen[mask] = i;
            else best = max(best, i - firstSeen[mask]);
        }
        return best;
    }
};
C
C
#include <string.h>

int longestBalancedVowelSegment(const char* text) {
    /* firstSeen[mask] = earliest index where the vowel-parity mask appeared. */
    int firstSeen[32];
    for (int k = 0; k < 32; k++) firstSeen[k] = -2; /* -2 = not yet seen */
    firstSeen[0] = -1;                              /* empty prefix, before index 0 */

    int mask = 0, best = 0;
    for (int i = 0; text[i] != '\0'; i++) {
        switch (text[i]) {
            case 'a': mask ^= 1 << 0; break;
            case 'e': mask ^= 1 << 1; break;
            case 'i': mask ^= 1 << 2; break;
            case 'o': mask ^= 1 << 3; break;
            case 'u': mask ^= 1 << 4; break;
            default: break; /* consonants do not change vowel parity */
        }
        if (firstSeen[mask] == -2) {
            firstSeen[mask] = i;            /* record earliest index of this parity */
        } else {
            int len = i - firstSeen[mask];  /* segment between equal parities is balanced */
            if (len > best) best = len;
        }
    }
    return best;
}

#### Follow-up
questions
What if the string can contain uppercase letters or non-letter characters — how do you decide which characters affect parity?
Generalize to requiring every one of the 26 letters to appear an even number of times; is the 2^k table still practical, and how large does k get?
Return the actual longest balanced segment (its start and end indices), not just its length.
Support a streaming input where characters arrive one at a time and you must report the best length so far after each character in O(1) amortized.

