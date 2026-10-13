---
type: "practice"
difficulty: "Intermediate"
---

# Find the length of the longest substring without repeating characters.

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Sliding Window Substrings](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Sliding_Window_Substrings.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

What is really being asked: Given a string, return the length of the longest contiguous substring that contains no repeated character. (Substring = contiguous; distinct from subsequence.) Example: "abcabcbb" → 3 ("abc"); "bbbbb" → 1; "pwwkew" → 3 ("wke"). The interviewer is probing: do you recognize this as a variable-size sliding window problem, can you get it to O(n), and can you handle the tricky pointer-jump optimization correctly.

Clarifying questions: What's the character set? (ASCII → I can use a fixed 128/256 array; Unicode → a hashmap.) Is the empty string valid? (Yes → answer 0.) Are we returning the length or the substring itself? (Length; but I'll note how to reconstruct.)

Plan: Brute force to establish the baseline and complexity → the sliding-window insight → two window variants (shrink-one-at-a-time vs. jump-left) → dry run → complexity + edge cases → follow-ups.

Brute force (baseline)

Enumerate all substrings: for each start i, extend j while s[i..j] has no duplicate, tracking the max length. Checking "no duplicate" needs a set. This is O(n²) substrings with O(1) amortized set work → O(n²) time, O(min(n, charset)) space. Correct but slow; state it, then improve.

Sliding window insight

Maintain a window [left, right] that is always duplicate-free (that's the invariant). Extend right one character at a time. When the incoming character s[right] is already inside the window, the window is no longer valid — we must move left rightward until the duplicate is removed. Because both pointers only move forward, each character enters and leaves the window at most once → O(n) total.

Two ways to implement the shrink:

Variant A — shrink one step at a time (set-based). Keep a HashSet of the window's characters. When s[right] is already in the set, remove s[left] and advance left in a while loop until s[right] can be added. Simple, clearly correct, O(n) (each char added/removed once).

Variant B — jump left directly (map of last index). Keep a map lastIndex[char] = its most recent index. When you see s[right] and it was seen at position p and p >= left (i.e., inside the current window), jump left = p + 1 in O(1) instead of stepping. This is the optimal, most-cited solution. The subtle correctness point is the guard p >= left: a stale index from before the current window must be ignored, or left could jump backwards and corrupt the answer.

Both are O(n); Variant B does fewer pointer moves and is the one to write in an interview, while narrating the p >= left guard.

Complexity
Time: O(n) — single pass; each character processed a constant number of times.
Space: O(min(n, |Σ|)) where |Σ| is the alphabet size. For ASCII, a fixed int[128] gives O(1) space and is faster than a hashmap (no hashing/boxing).
Edge cases
Empty string → 0.
Single character / all identical ("aaaa") → 1.
All distinct ("abcdef") → n.
Non-ASCII / Unicode → use a hashmap keyed by code point, not a fixed array.
Spaces and symbols count as characters ("a a" → 3 includes the space? — "a a" longest without repeat is " a" length 2, since 'a' repeats; good test of the guard).
Reconstructing the substring (follow-up)

Track the right index at each new best, storing bestStart = left and bestLen. Return s.substring(bestStart, bestStart + bestLen). Interviewers sometimes escalate to "return the substring" — mention you only need two extra variables.

#### Common mistakes
/ probes
Forgetting the p >= left guard in Variant B — the #1 bug. Without it, "abba" breaks: at the second 'a', its last index is 0, and if you don't guard, left jumps back to 1 even though the window already started at 2.
Using if instead of while in Variant A's shrink loop.
Confusing substring with subsequence — this problem is contiguous.
Off-by-one in length: right - left + 1.

Let me trace "abba" in Variant B to prove the guard matters: indices a=0,b=1,b=2,a=3. right=0 'a': last[a]=0, left=0, len=1, best=1. right=1 'b': len=2, best=2, last[b]=1. right=2 'b': last[b]=1 ≥ left(0) → left=2; last[b]=2; len=1. right=3 'a': last[a]=0 but 0 < left(2) → ignore (guard); left stays 2; last[a]=3; len=2, best=2. Answer 2 ("ab" at end). Without the guard, left would have jumped to 1 — wrong.

Summary — what a strong answer emphasizes

Recognize it as a variable sliding window with the invariant "window has no duplicates." Give the O(n²) brute force, then the O(n) window. Prefer the last-index-map jump version, and explicitly call out the p >= left guard (the classic correctness trap). State O(n) time / O(min(n, Σ)) space, use a fixed array for ASCII, and cover empty/all-same/all-distinct edge cases. Bonus points for showing how to reconstruct the actual substring.

**Comparison:**

```
Approach	Time	Space	Notes
Brute force (all substrings + set)	O(n²)	O(min(n,Σ))	baseline, too slow
Sliding window + set (Variant A)	O(n)	O(min(n,Σ))	shrink one step at a time, easy to reason
Sliding window + last-index map (Variant B)	O(n)	O(min(n,Σ))	jump left in O(1); needs p>=left guard
Input	Output	Why
"abcabcbb"	3	"abc"
"bbbbb"	1	all same
"pwwkew"	3	"wke" (substring, not subsequence "pwke")
""	0	empty
"abba"	2	guard stops left regressing
```

**Code:**

```java
// Variant B (optimal): last-index map, O(n) time, O(min(n, |Σ|)) space.
int lengthOfLongestSubstring(String s) {
    int[] last = new int[128];          // ASCII; last[c] = 1 + previous index of c, 0 = unseen
    int left = 0, best = 0;
    for (int right = 0; right < s.length(); right++) {
        char c = s.charAt(right);
        // if c seen at index >= left, jump left past it (guard: only if inside window)
        left = Math.max(left, last[c]);  // last[c] stores prevIndex+1; 0 when unseen => no jump
        best = Math.max(best, right - left + 1);
        last[c] = right + 1;             // store index+1 so "unseen" can stay 0
    }
    return best;
}

// Variant A (set-based, easiest to reason about), O(n):
int lengthOfLongestSubstringSet(String s) {
    java.util.Set<Character> win = new java.util.HashSet<>();
    int left = 0, best = 0;
    for (int right = 0; right < s.length(); right++) {
        while (win.contains(s.charAt(right))) {  // shrink until duplicate removed
            win.remove(s.charAt(left++));
        }
        win.add(s.charAt(right));
        best = Math.max(best, right - left + 1);
    }
    return best;
}
```

**Example input / output:**

```

Dry run of Variant B on "pwwkew" (last[] stores index+1; left jumps via max):

CODE
Copy
r=0 'p'  left=max(0, last[p]=0)=0   best=1  last[p]=1
r=1 'w'  left=max(0, last[w]=0)=0   best=2  last[w]=2   window "pw"
r=2 'w'  left=max(0, last[w]=2)=2   len=1   best=2  last[w]=3   window "w"
r=3 'k'  left=max(2, last[k]=0)=2   len=2   best=2  last[k]=4   window "wk"
r=4 'e'  left=max(2, last[e]=0)=2   len=3   best=3  last[e]=5   window "wke"
r=5 'w'  left=max(2, last[w]=3)=3   len=3   best=3  last[w]=6   window "kew"
=> 3   (longest is "wke" or "kew")

More cases:

CODE
Copy
""          -> 0
"bbbbb"     -> 1   ("b")
"abcabcbb"  -> 3   ("abc")
"abcdef"    -> 6   (whole string)
"abba"      -> 2   (guard prevents left moving backwards)
```

**Flow:**

1. yes
2. no
3. yes
4. no
5. left=0, best=0, right=0
6. c = s[right]
7. c seen at index >= left?
8. left = last_index_of_c + 1
9. left unchanged
10. best = max(best, right-left+1)
11. record last index of c = right
12. right < n-1?
13. right++
14. return best

