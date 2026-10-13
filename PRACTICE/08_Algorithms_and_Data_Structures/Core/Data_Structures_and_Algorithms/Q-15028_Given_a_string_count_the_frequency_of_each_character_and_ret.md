---
type: "practice"
difficulty: "Basic"
---

# Given a string, count the frequency of each character and return the most frequent one.

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Top-k and Sketches](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Top-k_and_Sketches.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

What the question is really asking: Given a string, count how many times each character appears, then return the character that appears most often. This is a classic frequency counting with a hash map warm-up. The interviewer wants a correct, clean O(n) solution and clear thinking about edge cases and tie-breaking.

What the interviewer is probing for: Can you reach for the right data structure (a hash map / count array) and produce a single-pass count plus a single-pass max? Do you handle edge cases (empty string, ties, case sensitivity, whitespace) and state your assumptions? Do you know the time/space complexity? This is easy to code but candidates lose points by ignoring ties or the empty input.

#### Assumptions
/ clarifying questions I'd state:

Is the count case-sensitive? ('A' vs 'a') — I'll assume yes (treat them as different) but mention how to fold case.
Do we count spaces/punctuation? — I'll assume yes (count every character); easy to filter if not.
On a tie (multiple characters share the max count), which to return? — I'll return the first one to reach the max (deterministic); I'd confirm the desired rule with the interviewer.
What if the string is empty? — return a sentinel (null / '\0') or throw; I'll return Optional/null and say so.
From first principles to the solution

1. The core idea — count, then find the max. Every character-frequency problem is two phases:

Count: walk the string once, incrementing a counter per character. A hash map <Character, Integer> is the natural store; for a known small alphabet (e.g., ASCII), a fixed-size int array of length 128/256 is faster and simpler.
Select: walk the counts once, tracking the character with the highest count.

2. Why a hash map / count array (not nested loops). A naive approach — for each character, scan the whole string to count its occurrences — is O(n²). Using a map, we count in a single O(n) pass because each map read/write is O(1) average. This is the key insight the interviewer is testing: trade a little memory for a big time win.

3. Building the count (phase 1). Initialize an empty map. For each character c in the string, do count[c] = count.getOrDefault(c, 0) + 1. After this pass, the map holds the exact frequency of every distinct character. Cost: O(n) time; O(k) space where k = number of distinct characters (≤ alphabet size).

4. Finding the most frequent (phase 2). Track best (the character) and bestCount (its frequency), initialized so the first entry wins. Iterate the map entries; whenever an entry's count strictly exceeds bestCount, update. Using strictly greater means we keep the first character that achieved the max — a deterministic, defensible tie-break. Cost: O(k).

5. Complexity. Overall O(n) time (n = string length; the second pass over k ≤ n entries is dominated by n) and O(k) space (distinct characters). For fixed ASCII, space is O(1) (a 128-int array). This is optimal — you must read every character at least once, so O(n) is a lower bound.

6. One-pass variant. You can even track the running max during the counting pass: after incrementing count[c], if count[c] > bestCount, update best. That folds both phases into one loop — same O(n), slightly cleaner. (Tie-break semantics differ subtly: the winner becomes whoever first crosses the current max during the scan.)

Edge cases & correctness details
Empty string: no characters → no answer. Return null/Optional.empty() or throw IllegalArgumentException; don't return a garbage character. State this.
All unique characters: every count is 1; return the first character (tie-break rule applies).
Case sensitivity: if case-insensitive is desired, normalize with Character.toLowerCase(c) before counting.
Non-letters / Unicode: an int[128] only covers ASCII; for full Unicode use a HashMap<Character,Integer> (or Integer code points for characters beyond the BMP). I'd note this if inputs may be Unicode.
Ties: be explicit about the rule. "First to reach max," "smallest character," or "any of the maxes" are all valid — just pick one and say it.
Worked example

Input "aabbbcc":

Count phase → {a:2, b:3, c:2}.
Select phase → best starts empty; a:2 → best=a(2); b:3 > 2 → best=b(3); c:2 not > 3 → unchanged.
Output: b (frequency 3).

Input "aabb" (a tie, 2 and 2): counts {a:2, b:2}; using strictly-greater, a is picked first and b:2 doesn't exceed it → Output a (first to reach max).

Summary — what a strong answer emphasizes

Use a hash map (or fixed-size count array) to tally frequencies in one O(n) pass, then scan the tallies once to return the character with the highest count — overall O(n) time, O(k) space (O(1) for a fixed ASCII alphabet). Beat the O(n²) nested-loop approach by trading memory for time. A strong answer states assumptions (case sensitivity, whitespace), handles the empty string, and defines a deterministic tie-break rule (I return the first character to reach the max). Bonus: mention the single-pass variant and the Unicode/int[]-vs-HashMap trade-off.

#### Common mistakes
/ follow-ups: using an O(n²) count-per-character loop; ignoring ties (undefined winner); crashing on empty input; assuming ASCII when input may be Unicode. Likely follow-ups: "Return the top k most frequent" → use a heap or bucket sort (bucket by frequency for O(n)); "What if the string doesn't fit in memory / is a stream?" → keep a running map (Misra–Gries / Count-Min Sketch for approximate heavy hitters at scale).

**Comparison:**

```
Approach	Time	Space	Notes
Nested loop (count each char by re-scanning)	O(n²)	O(1)	Simple but slow; avoid
HashMap frequency + max scan	O(n)	O(k) distinct	General; handles Unicode
int[128] / int[256] count array	O(n)	O(1)	Fastest for fixed ASCII alphabet
Bucket by frequency (for top-k)	O(n)	O(n)	Extends to "k most frequent"
```

**Code:**

```

import java.util.*;

// Returns the most frequent character; Optional.empty() for empty input. // Tie-break: the first character to reach the maximum count wins. static Optional<Character> mostFrequent(String s) { if (s == null || s.isEmpty()) return Optional.empty();

CODE
Copy
// Phase 1: count frequencies in one O(n) pass.
Map<Character, Integer> freq = new HashMap<>();
char best = s.charAt(0);
int bestCount = 0;

for (char c : s.toCharArray()) {
    int count = freq.merge(c, 1, Integer::sum); // count[c]++
    if (count > bestCount) {                    // running max (single pass)
        bestCount = count;
        best = c;
    }
}
return Optional.of(best);

}

CODE
Copy

```python
from collections import Counter

def most_frequent(s: str):
    if not s:
        return None
    freq = Counter(s)                 # O(n) count
    # max by count; on ties, Counter preserves first-seen order in CPython 3.7+
    return max(freq, key=lambda ch: freq[ch])
```

**Example input / output:**

```text
Input:  "aabbbcc"
Count:  {a:2, b:3, c:2}
Output: 'b'          (highest frequency = 3)

Input:  "aabb"       (tie)
Count:  {a:2, b:2}
Output: 'a'          (first to reach max, using strictly-greater compare)

Input:  ""           (empty)
Output: null / Optional.empty()

Input:  "hello"
Count:  {h:1, e:1, l:2, o:1}
Output: 'l'
```

**Flow:**

1. yes
2. no
3. yes
4. no
5. yes
6. no
7. Input string s
8. s empty?
9. return none
10. For each char c in s:freq[c]++
11. freq[c] > bestCount?
12. best = c;bestCount = freq[c]
13. keep current best
14. more chars?
15. return best

