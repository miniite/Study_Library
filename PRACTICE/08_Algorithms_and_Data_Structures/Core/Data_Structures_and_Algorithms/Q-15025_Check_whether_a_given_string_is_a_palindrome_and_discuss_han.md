---
type: "practice"
difficulty: "Basic"
---

# Check whether a given string is a palindrome and discuss handling of case and spaces.

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Palindromic Substrings](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Palindromic_Substrings.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

What is really being asked: Write code to determine whether a string reads the same forwards and backwards (a palindrome), and — importantly — discuss the choices around case sensitivity and whitespace/punctuation. This is a classic warm-up, but the interviewer is really watching how you handle ambiguity: a naive candidate reverses the string and compares; a strong candidate first asks "what counts as a palindrome here?" and handles the normalization explicitly.

What the interviewer is probing: Can you (1) clarify requirements before coding, (2) write clean, correct, O(n) code, and (3) reason about edge cases (empty string, single char, mixed case, spaces, punctuation, Unicode)? The whitespace/case discussion is the whole point — it separates a rote answer from a thoughtful one.

Clarifying questions I would ask the interviewer:

Do we treat it case-insensitively? ("Racecar" vs "racecar")
Do we ignore spaces and punctuation? ("A man, a plan, a canal: Panama")
Are we considering only alphanumeric characters, or every character?
Any Unicode/accent concerns?

Different answers to these change the code. A great candidate states an assumption and proceeds, rather than freezing.

From first principles: what "palindrome" means

A string is a palindrome if reversing it yields the same string — the first character matches the last, the second matches the second-to-last, and so on toward the middle. "level", "noon", "12321" are palindromes; "hello" is not.

The cleanest, most efficient check is the two-pointer technique: put one pointer left at the start and one right at the end; compare the characters they point to; if they match, move both inward; if they ever mismatch, it's not a palindrome. Stop when the pointers meet or cross. This is O(n) time and O(1) extra space — better than building a reversed copy, which uses O(n) extra space.

The case and whitespace discussion (the heart of the question)

The literal definition ("reversed equals original") gives surprising results on real input:

Case: "Racecar" reversed is "racecaR" — not equal character-for-character, so a strict check says it's not a palindrome. But humans read it as one. So we usually normalize case by lowercasing everything first.
Spaces and punctuation: "A man a plan a canal Panama" reversed is "amanaP lanac a nalp a nam A" — not equal, so strictly not a palindrome. But the famous phrase is a palindrome once you strip spaces, punctuation, and case. So we usually filter to alphanumeric characters and lowercase them, then check.

The right behavior depends on the spec. There are three common contracts:

Strict / literal — compare characters exactly as given. "Noon" is not a palindrome (capital N vs lowercase n). Rare in practice.
Case-insensitive only — lowercase, but keep spaces/punctuation. "Noon" → palindrome; "Race car" → not (the space breaks symmetry).
Alphanumeric, case-insensitive — the most common interview intent (and the LeetCode "Valid Palindrome" definition): keep only a–z and 0–9, lowercase, then check. This makes "A man, a plan, a canal: Panama" a palindrome.

Best practice: don't hard-code one behavior silently. Either ask, or expose it as a parameter/flag so the caller decides. In an interview I'd state: "I'll implement the alphanumeric, case-insensitive version since that matches the common phrase-palindrome intent, and I'll mention how to switch."

Two implementation approaches

#### Approach
A — two pointers with inline skipping (optimal): Walk left from the start and right from the end. Skip over any non-alphanumeric character. Compare the lowercased characters at the pointers. This is O(n) time, O(1) extra space, and processes the string in one pass without allocating a cleaned copy.

#### Approach
B — normalize then reverse-compare (simplest to read): Build a filtered, lowercased string, then check cleaned == cleaned[::-1]. Very readable, but uses O(n) extra space for the cleaned and reversed copies. Perfectly acceptable to present as the "clean and simple" version, then offer Approach A as the space-optimized one.

Edge cases to call out
Empty string "" — conventionally a palindrome (vacuously true).
Single character "a" — always a palindrome.
All non-alphanumeric ",. !" — after filtering it's empty → palindrome (by convention).
Unicode / accents — "Ève" vs "eVE": lowercasing rules vary by locale, and accented characters may need Unicode normalization (NFC/NFD) before comparison. Worth mentioning that ASCII lowercasing isn't universally correct.
Numbers/mixed — "12321" is a palindrome; the alphanumeric filter keeps digits.
Complexity
Time: O(n) — each character is examined at most once (or a constant number of times).
Space: O(1) for the two-pointer approach; O(n) for the normalize-and-reverse approach.
Summary / what a strong answer emphasizes

Start by clarifying the contract: case sensitivity and whether to ignore spaces/punctuation. Implement the check with two pointers converging from both ends for O(n) time and O(1) space, comparing normalized (lowercased, alphanumeric-only) characters. Explicitly enumerate edge cases (empty, single char, all-punctuation, Unicode) and note that the "right" normalization is a spec decision — ideally parameterized, not silently baked in.

#### Common mistakes
silently assuming one normalization and getting "wrong" answers on the interviewer's test string; building a reversed copy when O(1) space is easy; mishandling the empty string; forgetting Unicode/locale caveats; off-by-one in the pointer loop (while left < right).

#### Follow-up
probes: "Do it with O(1) extra space" (two pointers), "handle Unicode correctly," "check if a linked list is a palindrome" (find middle + reverse second half), "find the longest palindromic substring."

**Comparison:**

```
Contract	Normalization applied	"Racecar"	"A man a plan a canal Panama"	"Noon"
Strict / literal	none	False	False	False
Case-insensitive only	lowercase	True	False (spaces break it)	True
Alphanumeric + case-insensitive	lowercase + drop non-alnum	True	True	True
Approach	Time	Extra space	Readability	When to use
Two pointers (A)	O(n)	O(1)	Medium	When asked for optimal space
Normalize + reverse (B)	O(n)	O(n)	High	Quick, clean first pass
```

**Code:**

```python
# Approach A: two pointers, alphanumeric + case-insensitive. O(n) time, O(1) extra space.
def is_palindrome(s: str) -> bool:
    left, right = 0, len(s) - 1
    while left < right:
        while left < right and not s[left].isalnum():   # skip junk from the left
            left += 1
        while left < right and not s[right].isalnum():  # skip junk from the right
            right -= 1
        if s[left].lower() != s[right].lower():         # normalize case, compare
            return False
        left += 1
        right -= 1
    return True   # empty / single char / all-symbols all return True

# Approach B: normalize then reverse-compare. O(n) time, O(n) space, very readable.
def is_palindrome_simple(s: str) -> bool:
    cleaned = [c.lower() for c in s if c.isalnum()]
    return cleaned == cleaned[::-1]

# Configurable version: let the CALLER decide the contract instead of hard-coding it.
def is_palindrome_cfg(s: str, ignore_case=True, alnum_only=True) -> bool:
    chars = [c for c in s if (c.isalnum() or not alnum_only)]
    if ignore_case:
        chars = [c.lower() for c in chars]
    return chars == chars[::-1]
```

**Example input / output:**

```

Worked examples (alphanumeric + case-insensitive contract):

CODE
Copy
Input: "racecar"                       -> True   (odd length, mirrors around 'e')
Input: "Racecar"                       -> True   (case ignored -> "racecar")
Input: "A man, a plan, a canal: Panama"-> True   (strip punct/space+lower -> "amanaplanacanalpanama")
Input: "hello"                         -> False  (h vs o mismatch)
Input: "12321"                         -> True
Input: ""                              -> True   (empty = palindrome by convention)
Input: ".,!"                           -> True   (all non-alnum -> empty -> True)
Input: "No lemon, no melon"            -> True

Two-pointer dry-run on "A man a plan a canal Panama": left->'A', right->'a' : 'a'=='a' ok, move in left skips spaces, right skips spaces... each aligned pair (a/a, m/m, a/a...) matches until pointers cross -> return True

Strict-mode contrast (ignore_case=False, alnum_only=False): Input: "Noon" -> False ('N' != 'n') | Input: "noon" -> True
```

**Flow:**

1. no
2. yes
3. no
4. yes
5. Clarify contract: ignore case?ignore spaces/punct?
6. left = 0, right = len-1
7. left < right?
8. Return True
9. Skip non-alnum at left, then atright
10. lower s left == lower s right ?
11. Return False
12. left++, right--

