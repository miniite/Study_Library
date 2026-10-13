---
type: "practice"
difficulty: "Deep"
---

# Find the longest common subsequence of two strings and outline the DP table.

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Subsequence DP](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Subsequence_DP.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

The interviewer wants me to (1) solve the Longest Common Subsequence (LCS) of two strings and (2) explicitly outline and explain the DP table that produces the answer — not just quote the formula. At "deep understanding" level, they are probing whether I know the difference between a subsequence and a substring, can derive the recurrence from optimal substructure, can read the 2-D table, can reconstruct the actual subsequence (not just its length), and can discuss complexity and space optimisation. I will state assumptions, derive the recurrence, build a full table on a concrete example, backtrack to recover the string, and cover complexity and follow-ups.

Clarifying / assumptions: A subsequence keeps relative order but need not be contiguous ("ace" is a subsequence of "abcde"), unlike a substring which must be contiguous. Characters are compared case-sensitively unless told otherwise. We return the length (and, if asked, one actual LCS — there can be several of equal length).

First principles: why DP

Brute force would enumerate all 2^m subsequences of string X and test each against Y — exponential. But LCS has the two DP hallmarks:

Optimal substructure: Look at the last characters. If X[i-1] == Y[j-1], that matching character must be usable as the last character of an LCS of the prefixes, and LCS(X[0..i], Y[0..j]) = 1 + LCS(X[0..i-1], Y[0..j-1]). If they differ, the LCS ignores at least one of those two characters, so it is the better of dropping X's last char or dropping Y's last char.
Overlapping subproblems: The recursion repeatedly asks for the LCS of the same prefix pairs (i, j), so we cache them in a table indexed by prefix lengths.
The DP table and recurrence

Let dp[i][j] = length of the LCS of the first i characters of X (length m) and the first j characters of Y (length n). The table has size (m+1) × (n+1). Row/column 0 represent the empty prefix, so:

CODE
dp[0][j] = 0   for all j      (empty X → LCS 0)
dp[i][0] = 0   for all i      (empty Y → LCS 0)

Transition, for i from 1..m and j from 1..n:

CODE
if X[i-1] == Y[j-1]:   dp[i][j] = dp[i-1][j-1] + 1          // extend the diagonal match
else:                  dp[i][j] = max( dp[i-1][j],          // drop last char of X
                                       dp[i][j-1] )         // drop last char of Y

The final answer is dp[m][n]. Intuitively, each cell looks at three neighbours: the diagonal (i-1,j-1) when characters match, or the max of up (i-1,j) and left (i,j-1) when they do not. Every cell is O(1), and there are (m+1)(n+1) cells, so time = O(m·n) and space = O(m·n).

Reading and reconstructing the LCS from the table

The length alone is dp[m][n]. To recover the actual subsequence, walk backward from dp[m][n]:

If X[i-1] == Y[j-1], this character is part of the LCS — prepend it and move diagonally to (i-1, j-1).
Otherwise move to whichever neighbour holds the larger value: up (i-1, j) if dp[i-1][j] >= dp[i][j-1], else left (i, j-1).

Stop when i == 0 or j == 0. Because we prepend, the characters come out in correct order. This backtracking is O(m + n). (If multiple LCS exist, the tie-break at the else branch decides which one you get.)

Worked example (see input_output)

For X = "AGGTAB", Y = "GXTXAYB", the table fills to dp[6][7] = 4, and backtracking yields "GTAB". This shows the LCS need not be contiguous in either string.

Complexity and space optimisation
Time: O(m·n). Space: O(m·n) for the full table.
Space-optimised length only: row i depends solely on row i-1, so keep two rows (or one row + a saved diagonal), giving O(min(m, n)) space — but you then lose the ability to backtrack the actual string. So: use the full table if the interviewer wants the subsequence, the rolling array if they only want the length. There is also Hirschberg's algorithm, which recovers the actual LCS in O(m·n) time but only O(min(m, n)) space via divide-and-conquer — a strong point to raise for very long strings.

#### Follow-ups
a strong candidate anticipates
LCS vs Longest Common Substring: the substring version resets dp[i][j] to 0 on a mismatch (contiguity required) and tracks the global max cell — a different recurrence, easy to confuse.
Edit distance / diff tools: git diff, diff, and DNA sequence alignment are essentially LCS/edit-distance DP; LCS length relates to edit distance by deletions + insertions = m + n - 2·LCS.
Longest Increasing Subsequence connection: LIS of an array equals LCS of the array and its sorted-unique copy.
All LCS strings: counting or enumerating all distinct LCS is a harder DP layered on top of this table.
Summary — what a strong answer emphasises

Define subsequence vs substring, give the dp[i][j] recurrence with the match → diagonal+1, mismatch → max(up, left) rule and the zero base row/column, state O(m·n) time, show the filled table on a concrete example, and backtrack to reconstruct the actual LCS. For depth, mention O(min(m,n)) rolling-array space (length only) and Hirschberg (actual string, linear space), plus the substring/edit-distance/LIS connections. Common mistakes: confusing subsequence with substring, off-by-one indexing between the 1-based table and 0-based strings, and claiming you can backtrack from a rolling 2-row array (you cannot — you need the full table or Hirschberg).

**Comparison:**

```
Question	LCS (subsequence)	Longest Common Substring
Contiguous required?	No	Yes
Mismatch rule	dp = max(up, left)	dp = 0 (reset)
Answer cell	dp[m][n]	global max over all cells
Order preserved?	Yes	Yes (contiguous)
Goal	Time	Space	Can recover the string?
Length only, full table	O(m·n)	O(m·n)	yes
Length only, rolling rows	O(m·n)	O(min(m,n))	no
Actual LCS, Hirschberg	O(m·n)	O(min(m,n))	yes
```

**Code:**

```python
def lcs(X: str, Y: str):
    m, n = len(X), len(Y)
    # dp[i][j] = LCS length of X[:i] and Y[:j]; row/col 0 = empty prefix = 0
    dp = [[0] * (n + 1) for _ in range(m + 1)]
    for i in range(1, m + 1):
        for j in range(1, n + 1):
            if X[i - 1] == Y[j - 1]:
                dp[i][j] = dp[i - 1][j - 1] + 1        # match -> extend diagonal
            else:
                dp[i][j] = max(dp[i - 1][j], dp[i][j - 1])  # mismatch -> best of up/left

    # backtrack to recover one actual LCS string
    i, j, out = m, n, []
    while i > 0 and j > 0:
        if X[i - 1] == Y[j - 1]:
            out.append(X[i - 1])
            i, j = i - 1, j - 1
        elif dp[i - 1][j] >= dp[i][j - 1]:
            i -= 1
        else:
            j -= 1
    return dp[m][n], ''.join(reversed(out))

# Space-optimised LENGTH ONLY -> O(min(m,n)) space (cannot backtrack the string)
def lcs_len(X: str, Y: str):
    if len(X) < len(Y):
        X, Y = Y, X                       # make Y the shorter -> smaller row
    prev = [0] * (len(Y) + 1)
    for i in range(1, len(X) + 1):
        curr = [0] * (len(Y) + 1)
        for j in range(1, len(Y) + 1):
            curr[j] = prev[j - 1] + 1 if X[i - 1] == Y[j - 1] else max(prev[j], curr[j - 1])
        prev = curr
    return prev[len(Y)]
```

**Example input / output:**

```

Input: X = "AGGTAB", Y = "GXTXAYB"

Full DP table (rows = X prefixes "", A, G, G, T, A, B; cols = Y prefixes "", G, X, T, X, A, Y, B):

CODE
Copy
        ""  G   X   T   X   A   Y   B
   ""    0  0   0   0   0   0   0   0
   A     0  0   0   0   0   1   1   1
   G     0  1   1   1   1   1   1   1
   G     0  1   1   1   1   1   1   1
   T     0  1   1   2   2   2   2   2
   A     0  1   1   2   2   3   3   3
   B     0  1   1   2   2   3   3   4   <- dp[6][7] = 4

Backtrack from dp[6][7]=4: B matches (take B) -> A matches (take A) -> T matches (take T) -> G matches (take G). Output: LCS length = 4, one LCS = "GTAB"

Second example (subsequence, not substring): X = "ABCBDAB", Y = "BDCAB" -> LCS length = 4, e.g. "BCAB" or "BDAB".
```

**Flow:**

1. match
2. mismatch
3. Compute dp[i][j] for prefixesX[:i], Y[:j]
4. X[i-1] == Y[j-1] ?
5. dp[i][j] = dp[i-1][j-1] + 1(diagonal)
6. dp[i][j] = max(dp[i-1][j],dp[i][j-1]) (up / left)
7. Answer length = dp[m][n];backtrack for the string

