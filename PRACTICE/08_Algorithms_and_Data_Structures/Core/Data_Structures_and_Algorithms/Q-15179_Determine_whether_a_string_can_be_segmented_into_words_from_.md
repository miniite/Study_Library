---
type: "practice"
difficulty: "Intermediate"
---

# Determine whether a string can be segmented into words from a dictionary (word break).

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Tries and Word Building](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Tries_and_Word_Building.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

The problem is to determine if a given string can be broken down into substrings that are present in a provided dictionary of valid words. The approach involves using dynamic programming, ensuring we efficiently explore all possible segmentations without redundant checks.

#### Triaging the Question

This question tests your understanding of dynamic programming and recursive backtracking techniques. You need to ensure efficient computation while handling edge cases properly. Key assumptions include:

The input string can be up to a certain maximum length.
The dictionary is provided as an array or set of valid words.

#### First-Principles Solution

To solve this problem, we can use dynamic programming (DP) to avoid redundant checks and ensure efficient computation. Here’s the step-by-step approach:

Initialization: Create a boolean DP table where dp[i] indicates whether the substring from the start of the string up to index i-1 can be segmented into valid words.
Base Case: dp[0] = True, as an empty string is trivially segmentable.
State Transition: For each position i in the string, check all possible substrings ending at i. If a substring from j to i-1 exists in the dictionary and dp[j] is true, then set dp[i] = True.
Result: The final result will be stored in dp[len(string)], which indicates whether the entire string can be segmented.
Step-by-Step Solution

Here’s a Python implementation of the solution:

PYTHON
def word_break(s, word_dict):
    n = len(s)
    dp = [False] * (n + 1)
    dp[0] = True

    for i in range(1, n + 1):
        for j in range(i):
            if dp[j] and s[j:i] in word_dict:
                dp[i] = True
                break

    return dp[n]

#### Closing Summary
+ Common Mistakes/Follow-Ups

The word_break function uses dynamic programming to efficiently check all possible segmentations. A common mistake is not handling edge cases properly, such as an empty dictionary or a string with no valid segments.

#### Follow-ups

How would you handle very large strings or dictionaries?
Can you optimize the space complexity of this solution?

**Code:**

```python
def word_break(s, word_dict):
    n = len(s)
    dp = [False] * (n + 1)
    dp[0] = True

    for i in range(1, n + 1):
        for j in range(i):
            if dp[j] and s[j:i] in word_dict:
                dp[i] = True
                break

    return dp[n]
```

**Example input / output:**

```text
Input:
s = "leetcode"
word_dict = ["leet", "code"]

Output:
True

Explanation:
The string can be segmented as "lee" + "t" + "code".
```

