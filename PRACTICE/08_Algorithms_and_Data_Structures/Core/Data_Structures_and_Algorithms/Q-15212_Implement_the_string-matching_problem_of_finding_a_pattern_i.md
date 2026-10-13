---
type: "practice"
difficulty: "Deep"
---

# Implement the string-matching problem of finding a pattern in text and mention KMP's advantage over naive.

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ First Unique Character](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/First_Unique_Character.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the string-matching problem, specifically implementing the Knuth-Morris-Pratt (KMP) algorithm to find a pattern within text, we need to understand its core concept and compare it with the naive approach. The KMP algorithm is known for its efficiency in handling the problem by utilizing the previously matched characters to avoid re-comparing them.

#### Triage
Restating the Question and Assumptions

The question asks us to implement a string-matching function using the KMP algorithm, which involves finding all occurrences of a pattern within a given text. The key points are:

Naive Approach: Simply compare every character in the text with the first character of the pattern.
KMP Algorithm: Uses partial match information (prefix table) to skip unnecessary comparisons.

#### Assumptions
include:

The input consists of two strings: text and pattern.
Both strings are case-sensitive.
We need to return all starting indices in the text where the pattern matches.

#### First-Principles Solution

To implement KMP, we first create a prefix table that stores the longest proper prefix which is also a suffix (LPS) for each position in the pattern. This helps us to avoid unnecessary comparisons by skipping already matched characters.

Step 1: Building the Prefix Table

The prefix table lps of length m (length of pattern) is built as follows:

Initialize len = 0 and set lps[0] = 0.
Iterate over each character in the pattern from index 1 to m - 1.
If pattern[i] == pattern[len], increment both i and len, then set lps[i] = len. Otherwise, keep decrementing len until a match is found or len becomes zero.
Step 2: Matching the Pattern

Using the prefix table, we can now scan through the text to find matches:

Initialize two pointers, i for the text and j for the pattern.
Compare characters from both strings; if they match, increment both pointers.
If a mismatch occurs or j reaches the end of the pattern, adjust j based on the prefix table. If lps[j] != 0, move i by j - lps[j].

This process ensures that we do not re-compare already matched characters, leading to an efficient solution with a time complexity of (O(n + m)).

#### Implementation

Here is the full implementation in Python:

PYTHON
def build_lps(pattern):
    m = len(pattern)
    lps = [0] * m
    j = 0
    for i in range(1, m):
        while j > 0 and pattern[i] != pattern[j]:
            j = lps[j - 1]
        if pattern[i] == pattern[j]:
            j += 1
        lps[i] = j
    return lps

def kmp_search(text, pattern):
    n, m = len(text), len(pattern)
    if m == 0:
        return [i for i in range(n)]

    lps = build_lps(pattern)
    result_indices = []
    j = 0

    for i in range(n):
        while j > 0 and text[i] != pattern[j]:
            j = lps[j - 1]
        if text[i] == pattern[j]:
            if j == m - 1:
                result_indices.append(i - m + 1)
                j = lps[j]
            else:
                j += 1

    return result_indices

# Example usage
text = "ABABDABACDABABCABAB"
pattern = "ABABCABAB"
print(kmp_search(text, pattern))  # Output: [10]

#### Closing Summary
and Common Mistakes/Follow-Ups

The KMP algorithm efficiently solves the string-matching problem by utilizing a precomputed prefix table. This approach ensures that we avoid unnecessary comparisons, making it significantly faster than the naive method which has a time complexity of (O(n \times m)).

#### Common mistakes
include:

Not correctly updating the j pointer when a mismatch occurs.
Failing to properly build and use the prefix table.

#### Follow-up
questions might involve optimizing for memory usage or adapting the algorithm to handle case-insensitive matches.

**Code:**

```python
def build_lps(pattern):
    m = len(pattern)
    lps = [0] * m
    j = 0
    for i in range(1, m):
        while j > 0 and pattern[i] != pattern[j]:
            j = lps[j - 1]
        if pattern[i] == pattern[j]:
            j += 1
        lps[i] = j
    return lps

def kmp_search(text, pattern):
    n, m = len(text), len(pattern)
    if m == 0:
        return [i for i in range(n)]

    lps = build_lps(pattern)
    result_indices = []
    j = 0

    for i in range(n):
        while j > 0 and text[i] != pattern[j]:
            j = lps[j - 1]
        if text[i] == pattern[j]:
            if j == m - 1:
                result_indices.append(i - m + 1)
                j = lps[j]
            else:
                j += 1

    return result_indices
```

**Example input / output:**

```text
text = "ABABDABACDABABCABAB"
pattern = "ABABCABAB"
print(kmp_search(text, pattern))  # Output: [10]
```

