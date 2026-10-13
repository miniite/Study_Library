---
type: "practice"
difficulty: "Basic"
---

# Find the first non-repeating character in a string.

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ First Unique Character](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/First_Unique_Character.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

The question asks: given a string, find the first character that appears exactly once (the first non-repeating character). Because this is a "basic concept" question, the interviewer is checking that I can (1) reason clearly about a simple problem, (2) pick a clean data structure, (3) get the complexity right, and (4) handle edge cases. It's a warm-up, so clarity and correctness matter more than cleverness.

Clarifying questions I'd ask up front:

Do we return the character or its index? (I'll return the character, and mention the index variant.)
What counts as a character — case-sensitive? Are 'A' and 'a' different? (I'll assume case-sensitive and treat every character literally.)
What if no character is non-repeating (e.g., "aabb")? I'll return a sentinel — '_' or -1 for an index. I'll confirm the sentinel with the interviewer.
Character set: ASCII only, or full Unicode? (Affects whether I use a fixed 256-slot array or a hash map.)

Plan: start from the brute-force idea, then improve to the standard two-pass hash-map / frequency-count solution, state the complexity, and dry-run an example.

First principle: what makes a character "the answer"?

A character is the answer if two things are true, in this order:

It appears exactly once in the whole string.
Among all characters that appear once, it is the leftmost (earliest index).

So I need two pieces of information for each character: how many times it occurs, and the order in which characters appear. The string itself already preserves order (just scan left to right), so I only need to compute the counts.

Brute force (to show the baseline)

For each character, scan the entire rest of the string to see if it repeats. That's O(n²) time, O(1) extra space. Correct but wasteful — I mention it, then improve.

The clean solution: count, then scan

The efficient approach is two passes:

Pass 1 — build a frequency map. Walk the string once and count occurrences of each character in a hash map (char → count). O(n).

Pass 2 — find the first count-1. Walk the string again in original order; the first character whose count is 1 is the answer. Because I scan left to right, the first one I hit is automatically the leftmost non-repeating character. O(n).

Total: O(n) time. Space is O(k) where k is the number of distinct characters — bounded by the alphabet size (≤ 128 for ASCII, ≤ 256 for extended ASCII), so it's effectively O(1) for a fixed alphabet.

The two-pass ordering is the key insight: the counting pass ignores order, and the scanning pass supplies the order. Trying to do it in one pass is the classic mistake — you can't know a character is non-repeating until you've seen the whole string.

Small optimization: for ASCII I can replace the hash map with a fixed int[256] array indexed by the character's code. That removes hashing overhead and is what I'd write in a performance-sensitive setting. For Unicode I'd stick with a HashMap<Character,Integer> (or store code points).

Edge cases to handle (interviewers love these)
Empty string "" → no answer; return the sentinel.
All repeating "aabbcc" → return sentinel.
Single character "z" → returns 'z'.
All unique "abcd" → returns 'a' (the first).
Case sensitivity — "aA": if case-sensitive, 'a' is the answer because 'a' and 'A' are different. State this assumption explicitly.
Spaces / punctuation / digits — they're just characters; count them like any other unless told otherwise.
Complexity summary
Time: O(n) — two linear passes.
Space: O(k) distinct chars ≈ O(1) for a fixed alphabet (256 slots).

This is optimal: you must read every character at least once to be sure of its count, so O(n) time is a lower bound.

#### Follow-up
probes (and how I'd answer)
"Return the index instead of the character." Same algorithm; in pass 2, return the loop index instead of the char, and -1 if none. (See code.)
"What if the string is a stream and you must report the first non-repeater at any point?" That's a harder variant: maintain a linked structure / queue of candidate characters plus a count/state map, evicting characters as they become repeating, so you can answer in O(1) at each step. Worth mentioning to show depth, but it's beyond this basic version.
"Unicode-correctness?" Iterate by code point, not UTF-16 char, so surrogate-pair emoji aren't miscounted.
Summary — what a strong answer emphasizes

The clean answer is the two-pass frequency count: pass one builds char → count; pass two scans the original string left-to-right and returns the first character with count 1 (or a sentinel if none). It's O(n) time, O(1)/O(alphabet) space, which is optimal. I'd call out the assumptions (character vs. index, case sensitivity, no-answer sentinel, ASCII vs. Unicode) and note that the ordering — count first, then scan in original order — is exactly what guarantees "first" non-repeating rather than just "a" non-repeating.

**Comparison:**

```
Approach	Time	Space	Notes
Brute force (rescan for each char)	O(n²)	O(1)	Baseline; too slow
Two-pass hash map	O(n)	O(k) distinct	Standard, Unicode-friendly
Two-pass int[256]	O(n)	O(1)	Fastest for ASCII
Streaming (queue + state)	O(n) total, O(1)/query	O(k)	For "at any point in a stream" variant
```

**Code:**

```

// Return the first non-repeating character, or '' if none. Case-sensitive. // Time O(n), Space O(k) distinct chars (~O(1) for a fixed alphabet). public char firstNonRepeating(String s) { if (s == null || s.isEmpty()) return ''; // edge: empty / null

CODE
Copy
// Pass 1: count occurrences.
java.util.Map<Character, Integer> count = new java.util.LinkedHashMap<>();
for (char c : s.toCharArray()) {
    count.merge(c, 1, Integer::sum);
}

// Pass 2: scan in ORIGINAL order -> first count==1 is the leftmost answer.
for (char c : s.toCharArray()) {
    if (count.get(c) == 1) return c;
}
return '_';                                  // all characters repeat

}

CODE
Copy

```java
// Variant: return the INDEX (or -1). Fixed int[256] for speed (ASCII).
public int firstNonRepeatingIndex(String s) {
    int[] freq = new int[256];
    for (int i = 0; i < s.length(); i++) freq[s.charAt(i)]++;   // pass 1
    for (int i = 0; i < s.length(); i++)                        // pass 2
        if (freq[s.charAt(i)] == 1) return i;
    return -1;
}
PYTHON
Copy
# Python one-liner style using collections.Counter (preserves nothing about order,
# so we still scan the string for order). O(n) time, O(k) space.
from collections import Counter

def first_non_repeating(s: str) -> str:
    freq = Counter(s)                 # pass 1
    for ch in s:                      # pass 2, original order
        if freq[ch] == 1:
            return ch
    return "_"
```

**Example input / output:**

```text
Example 1:  s = "leetcode"
  Pass 1 counts: l:1 e:3 t:1 c:1 o:1 d:1
  Pass 2 (left->right): 'l' has count 1  -> answer = 'l'  (index 0)

Example 2:  s = "aabbccd"
  counts: a:2 b:2 c:2 d:1
  scan: a(2) a(2) b(2) b(2) c(2) c(2) d(1) -> answer = 'd' (index 6)

Example 3:  s = "aabb"       -> all counts >= 2 -> answer = '_'  (none)
Example 4:  s = ""           -> answer = '_'  (empty)
Example 5:  s = "aA"         -> case-sensitive: a:1, A:1 -> answer = 'a' (index 0)
Example 6:  s = "swiss"      -> counts s:3 w:1 i:1 -> answer = 'w' (index 1)
```

**Flow:**

1. yes
2. no
3. yes
4. no
5. yes
6. no
7. Start: string s
8. s empty?
9. return sentinel '_'
10. Pass 1: count each char intomap
11. Pass 2: scan s left to right
12. count of this char == 1?
13. return this char - leftmostnon-repeating
14. more chars?

