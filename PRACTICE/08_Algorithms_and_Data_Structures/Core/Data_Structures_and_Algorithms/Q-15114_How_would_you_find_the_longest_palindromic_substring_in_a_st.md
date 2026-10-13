---
type: "practice"
difficulty: "Deep"
---

# How would you find the longest palindromic substring in a string?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Palindromic Substrings](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Palindromic_Substrings.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

What's really being asked: Given a string s, return the longest contiguous substring that reads the same forwards and backwards. "Substring" means contiguous (unlike subsequence). This is a "deep understanding" DSA question, so the interviewer wants more than the brute force — they want me to reason from O(n³) → O(n²) → O(n), justify the DP vs expand-around-center vs Manacher trade-off, and handle edge cases and the return contract precisely.

What they're probing: Can I recognise the palindrome structure (a palindrome of length k has a smaller palindrome at its core), pick the pragmatically best algorithm (expand-around-center for interviews, Manacher when O(n) is required), and reason about even- vs odd-length centers, which is the classic trap.

Clarifying questions I'd ask: (1) If there are multiple longest palindromes, return any or the first? (I'll return the first found.) (2) Return the substring or its start/end indices? (I'll return the substring.) (3) Empty string / single char? (Empty → ""; single char → itself.) (4) Case sensitivity and Unicode — I'll treat characters as-is (case-sensitive, code-unit level).

From first principles

A string is a palindrome iff s[i] == s[j] for every pair moving inward from the two ends. The key structural insight: a palindrome grows outward from a center. If s[i..j] is a palindrome, then s[i-1..j+1] is a palindrome iff s[i-1] == s[j+1]. There are two kinds of centers:

Odd length — centered on a single character (aba, center b).
Even length — centered between two characters (abba, center between the two bs).

That gives 2n - 1 possible centers for a string of length n.

#### Approach
1 — Brute force O(n³)

Check every substring (O(n²) of them) and verify each is a palindrome (O(n)). Correct but too slow; I'd mention it only to anchor the improvement.

#### Approach
2 — Expand Around Center, O(n²) time, O(1) space (my go-to)

Iterate over all 2n-1 centers. For each, expand outward while the characters match, tracking the longest span found. Handle odd centers (i, i) and even centers (i, i+1). This is O(n²) worst case (e.g. "aaaa..." where every center expands far) but O(1) extra space, simple, and virtually bug-free — the strongest interview answer for most companies. This is what I'd code first.

#### Approach
3 — Dynamic Programming, O(n²) time, O(n²) space

Let dp[i][j] = true if s[i..j] is a palindrome. Recurrence: dp[i][j] = (s[i] == s[j]) && (j - i < 2 || dp[i+1][j-1]). Fill by increasing length. Same time as expand-around-center but uses O(n²) memory, so I'd prefer expand-around-center unless the interviewer wants the DP formulation explicitly (it's a clean way to state the substructure).

#### Approach
4 — Manacher's Algorithm, O(n) time (the "deep" answer)

Manacher achieves linear time by reusing work across centers. It transforms s into t with sentinels between every char ("aba" → "^#a#b#a#$"), which unifies odd/even cases — every palindrome in t has odd length. It maintains an array p[] where p[i] is the radius of the palindrome centered at i, plus the current rightmost palindrome boundary (center C, right R). For a position i < R, its mirror mirror = 2*C - i already tells us a lower bound for p[i] (min(R - i, p[mirror])), so we skip re-comparing characters we've already proven. We only ever expand past known boundaries, and because R is monotonically non-decreasing, total work is O(n). The answer is the max radius (mapped back to original indices). I'd present Manacher when the constraints demand strict linear time (very large strings, tight latency).

Edge cases
Empty string → "".
Single character → that character (every single char is a palindrome of length 1).
All identical chars "aaaa" → whole string; this is expand-around-center's O(n²) worst case and Manacher's clearest win.
No palindrome longer than 1 (e.g. "abc") → return any single char, conventionally the first.
Even-length answer ("cbbd" → "bb") — must handle the between-chars center or you'll miss it (the classic bug).
Complexity summary

#### Approach
Time	Space
Brute force	O(n³)	O(1)
DP	O(n²)	O(n²)
Expand around center	O(n²)	O(1)
Manacher	O(n)	O(n)
What a strong answer emphasises

Lead with the center-expansion insight and the odd/even center distinction (that's the trap). Code expand-around-center cleanly (O(n²)/O(1)), state you can also frame it as DP, and then explain Manacher to reach O(n) — being able to articulate why it's linear (mirror reuse + monotone right boundary) is what separates "deep understanding" from rote. Nail the return contract and edge cases. Common follow-ups: "count all palindromic substrings" (same expansion, count each successful expansion — LeetCode 647), "longest palindromic subsequence" (different problem, 2D DP, O(n²)), and "why can't a greedy approach work?"

**Comparison:**

```
Approach	Time	Space	When to use
Brute force	O(n³)	O(1)	Only to motivate; never ship
DP dp[i][j]	O(n²)	O(n²)	To state the substructure cleanly
Expand around center	O(n²)	O(1)	Default interview answer; simple, low-memory
Manacher	O(n)	O(n)	Very large strings / strict linear-time requirement
```

**Code:**

```

// Expand Around Center — O(n^2) time, O(1) space. Returns the first longest palindrome. class Solution { public String longestPalindrome(String s) { if (s == null || s.length() < 1) return ""; int start = 0, end = 0; // [start, end] inclusive of best palindrome for (int i = 0; i < s.length(); i++) { int odd = expand(s, i, i); // odd-length center at i int even = expand(s, i, i + 1); // even-length center between i and i+1 int len = Math.max(odd, even); if (len > end - start + 1) { start = i - (len - 1) / 2; end = i + len / 2; } } return s.substring(start, end + 1); } // returns the length of the palindrome expanded from (left, right) private int expand(String s, int left, int right) { while (left >= 0 && right < s.length() && s.charAt(left) == s.charAt(right)) { left--; right++; } return right - left - 1; // width after the loop overshoots by 1 on each side } }

CODE
Copy

```python
# Manacher's Algorithm — O(n) time. Returns the longest palindromic substring.
def longest_palindrome(s: str) -> str:
    if not s:
        return ""
    # transform: "aba" -> "^#a#b#a#$" so every palindrome is odd-length
    t = "^#" + "#".join(s) + "#$"
    n = len(t)
    p = [0] * n          # p[i] = radius of palindrome centered at i in t
    c = r = 0            # current center and right boundary
    for i in range(1, n - 1):
        mirror = 2 * c - i
        if i < r:
            p[i] = min(r - i, p[mirror])   # reuse mirror's known radius
        while t[i + p[i] + 1] == t[i - p[i] - 1]:  # try to expand (sentinels stop it)
            p[i] += 1
        if i + p[i] > r:                   # moved right boundary out
            c, r = i, i + p[i]
    max_len = max(p)
    center = p.index(max_len)
    start = (center - max_len) // 2        # map back to original string index
    return s[start:start + max_len]
```

**Example input / output:**

```

Worked examples:

CODE
Copy
Input : "babad"   Output : "bab"   (or "aba" — both length 3; return the first found)
Input : "cbbd"    Output : "bb"    (even-length center between the two 'b's)
Input : "a"       Output : "a"
Input : ""        Output : ""
Input : "aaaa"    Output : "aaaa"  (expand-around-center worst case)

Dry run of expand-around-center on "cbbd":

CODE
Copy
i=0 'c': odd expand(0,0)->len1;  even expand(0,1) 'c'!='b' ->len0. best="c"
i=1 'b': odd expand(1,1)->len1;  even expand(1,2) 'b'=='b' -> expand to (0,3) 'c'!='d' stop
         even width = 3-0-1 = 2 -> len2. best updates: start=1-(2-1)/2=1, end=1+2/2=2 -> "bb"
i=2 'b': odd len1; even expand(2,3) 'b'!='d' len0. no update.
i=3 'd': len1. no update.
Result: s.substring(1,3) = "bb"
```

**Flow:**

1. yes
2. no
3. yes
4. no
5. For each index i in s
6. expand odd center (i, i)
7. expand even center (i, i+1)
8. len > best?
9. update start,end from center iand len
10. continue
11. more centers?
12. return s.substring(start, end+1)

