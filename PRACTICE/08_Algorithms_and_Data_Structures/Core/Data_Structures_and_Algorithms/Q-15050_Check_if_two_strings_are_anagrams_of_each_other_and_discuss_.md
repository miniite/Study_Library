---
type: "practice"
difficulty: "Basic"
---

# Check if two strings are anagrams of each other and discuss two different approaches.

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Anagram Problems](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Anagram_Problems.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

What is being asked: Given two strings, determine whether they are anagrams of each other — i.e., one is a rearrangement of the other using exactly the same letters, the same number of times. Classic examples: "listen" and "silent" are anagrams; "hello" and "world" are not. The interviewer also explicitly wants two different approaches, so I must present both and compare them.

What the interviewer is probing for (basic-concept level): Do I know what an anagram is, can I translate the definition into code, do I recognize the two canonical solutions (sort-and-compare vs. frequency-count), and can I reason about their time/space complexity? At this level they want clear fundamentals, correct code, and a clean comparison — not exotic optimizations.

Clarifying questions / assumptions I would state up front:

Case sensitivity — is "Listen" an anagram of "Silent"? I'll assume case-insensitive is not required unless stated, but I'll mention how to handle it (lowercase both first).
Whitespace / punctuation — for phrase anagrams like "dormitory" / "dirty room" we usually strip spaces. I'll assume simple single-word strings unless told otherwise.
Character set — plain ASCII lowercase letters (a–z)? Or full Unicode? This changes the space bound of the counting approach. I'll cover both.
Empty strings — two empty strings are trivially anagrams.

Plan: Establish the core invariant (anagrams ⇔ identical multiset of characters), give the two standard approaches, code each, dry-run one, then compare complexity and pick a recommendation.

First principle: the invariant

Two strings are anagrams if and only if they have the exact same characters with the exact same counts. An immediate, cheap consequence: if their lengths differ, they cannot be anagrams — so the first line of any solution is a length check that lets us bail out in O(1).

From that invariant, two natural strategies fall out:

If two multisets of characters are equal, then sorting each string produces the identical sequence → sort and compare.
Equal multisets also means each character's frequency is equal → count and compare frequencies.

#### Approach
1 — Sort and compare

Sort both strings; if they're anagrams the sorted forms are byte-for-byte identical.

Steps: (1) if lengths differ, return false; (2) sort both strings; (3) compare the sorted results for equality.

Complexity: Sorting dominates at O(n log n) time. Space is O(n) (or O(log n)) depending on whether the language's sort is out-of-place or in-place, plus the char arrays.

#### Pros
Extremely short, obviously correct, works for any character set (Unicode included) with no assumptions, and needs no auxiliary map. Cons: The n log n factor is unnecessary — we can do better linearly.

#### Approach
2 — Frequency count (the preferred answer)

Count how many times each character appears in the first string, then subtract using the second string. If every count lands back at zero, they're anagrams.

Steps: (1) length check; (2) build a frequency table from string A (increment); (3) walk string B decrementing — if any count goes negative, a character appears more in B than in A → return false; (4) all counts zero ⇒ anagram. (Because lengths are equal and no count went negative, "all zero at the end" is guaranteed, so a single pass with the negative-check suffices.)

Complexity: O(n) time (two linear passes). Space is O(k) where k is the alphabet size. For lowercase ASCII you can use a fixed int[26], which is O(1) space. For arbitrary Unicode, use a hash map: O(k) space bounded by the number of distinct characters.

#### Pros
Linear time — the optimal general answer. Cons: Needs an auxiliary structure and a decision about the character set.

Handling the real-world variants
Case-insensitive: lowercase both strings before processing.
Ignore spaces/punctuation: filter to alphabetic characters first (common for "phrase anagram" versions like dormitory / dirty room).
Unicode correctness: naive per-char counting can misbehave with surrogate pairs (emoji, some CJK) and combining accents. If Unicode-correct behavior matters, normalize (e.g., NFC) and iterate by code points / grapheme clusters, not UTF-16 chars. At interview level, mentioning this awareness scores points.
Complexity summary and recommendation

The frequency-count approach is the one I'd ship: it's O(n) time and, for a bounded alphabet, O(1) space — strictly better than sorting's O(n log n). The sort approach is worth knowing because it's a one-liner, trivially correct, and handles any character set without thinking about alphabet size — a good "I can write it in 10 seconds" fallback and a great sanity cross-check.

#### Common mistakes
to avoid:

Forgetting the length check (frequency count without it can wrongly return true when B is a strict prefix's rearrangement, though the negative-check guards most cases — still, the length check is the clean, cheap guard).
Claiming O(1) space for the counting approach without stating the bounded-alphabet assumption — for full Unicode it's O(k).
Ignoring case/whitespace when the examples clearly imply phrase anagrams.
Comparing character sets instead of counts ("aab" vs "abb" share the same set {a,b} but are not anagrams).

Likely follow-ups: "Group all anagrams in a list together" (use the sorted string or a count-signature as a hash-map key → O(N·k) grouping), "check if two strings are anagrams ignoring one allowed edit," or "find all anagrams of a pattern inside a text" (sliding-window frequency count, O(n)).

What a strong answer emphasizes: the multiset invariant, the O(1) length pre-check, both approaches with correct complexities, the bounded-alphabet caveat on space, and a clear recommendation (frequency count) with the sort approach as the simple alternative.

**Comparison:**

```
Approach	Time	Space	Charset handling	When to use
Sort & compare	O(n log n)	O(n) (or O(log n) in-place)	Any (Unicode-safe by default)	Quick to write; cross-check; unknown alphabet
Frequency count (hash map)	O(n)	O(k) distinct chars	Any	General optimal answer
Frequency count (int[26])	O(n)	O(1)	Lowercase a–z only	Bounded alphabet, fastest
```

**Code:**

```
Approach 1: Sort and compare — O(n log n) time, O(n) space, any charset

def is_anagram_sort(a: str, b: str) -> bool: if len(a) != len(b): return False return sorted(a) == sorted(b)

Approach 2: Frequency count — O(n) time, O(1) space for a bounded alphabet

def is_anagram_count(a: str, b: str) -> bool: if len(a) != len(b): return False counts = {} for ch in a: counts[ch] = counts.get(ch, 0) + 1 for ch in b: if ch not in counts: # char in b never seen in a return False counts[ch] -= 1 if counts[ch] < 0: # char appears more often in b than in a return False return True # equal lengths + no negatives => all zero

CODE
Copy

```java
// Java: fixed int[26] frequency count for lowercase a-z — O(n) time, O(1) space
boolean isAnagram(String a, String b) {
    if (a.length() != b.length()) return false;
    int[] count = new int[26];
    for (int i = 0; i < a.length(); i++) {
        count[a.charAt(i) - 'a']++;
        count[b.charAt(i) - 'a']--;   // single pass: add for a, subtract for b
    }
    for (int c : count) if (c != 0) return false;
    return true;
}
```

**Example input / output:**

```

Worked examples (Approach 2, dry-run on a = "listen", b = "silent"):

CODE
Copy
a = "listen", b = "silent"
lengths equal (6 == 6) -> continue

Build counts from a="listen":
  l:1, i:1, s:1, t:1, e:1, n:1

Walk b="silent", decrementing:
  's' -> s:0
  'i' -> i:0
  'l' -> l:0
  'e' -> e:0
  'n' -> n:0
  't' -> t:0
No count went negative -> return True   (anagram)

More cases:

CODE
Copy
is_anagram_count("hello", "world")  -> False   (length equal, but 'w' not in "hello")
is_anagram_count("aab",   "abb")    -> False   ('b' count goes negative)
is_anagram_count("",      "")       -> True    (both empty)
is_anagram_count("rat",   "car")    -> False   ('c' not in "rat")
is_anagram_sort("dormitory".replace(" ",""), "dirty room".replace(" ","")) -> True
```

**Flow:**

1. No
2. Yes
3. Yes
4. No
5. Start: strings a, b
6. len a == len b?
7. Return false
8. Build freq count from a
9. For each char in b: decrementcount
10. Any count < 0 or char unseen?
11. Return true: anagram

