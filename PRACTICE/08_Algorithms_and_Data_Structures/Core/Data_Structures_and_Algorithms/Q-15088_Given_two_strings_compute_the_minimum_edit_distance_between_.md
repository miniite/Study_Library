---
type: "practice"
difficulty: "Deep"
---

# Given two strings, compute the minimum edit distance between them.

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Edit Distance](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Edit_Distance.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

The question is the classic minimum edit distance (Levenshtein distance): given two strings A (length m) and B (length n), find the minimum number of single-character insertions, deletions, and substitutions to transform A into B. Since this is tagged "deep understanding" at a top-tier bar (Amazon/Google/Microsoft/Meta), the interviewer wants more than "it's a 2D DP." They want:

Why it's a dynamic-programming problem — the optimal-substructure / recurrence derivation from first principles.
A correct recurrence with correct base cases, and a clear explanation of what each of the three transitions means.
Complexity (time and space) and the standard O(min(m,n)) space optimization.
Reconstruction of the actual edit operations, not just the number.
Awareness of variants (weighted costs, the Damerau transposition, why substitution counts as 1 not 2) and real-world uses.

Clarifying questions I'd raise: Are the three operations all cost 1 (classic Levenshtein), or do insert/delete/substitute have different weights (e.g., DNA alignment)? Is transposition of adjacent chars a single edit (Damerau–Levenshtein)? Do I need just the distance or also the sequence of edits? I'll solve the classic unit-cost Levenshtein and note the generalizations.

Plan: define the subproblem → derive the recurrence → base cases → fill order → dry-run → complexity → space optimization → reconstruction → variants → summary.

Step 1 — Define the right subproblem

The whole trick to any string-alignment DP is choosing the subproblem. Define:

dp[i][j] = minimum edit distance between the first i characters of A (i.e., A[0..i-1], a prefix of length i) and the first j characters of B (B[0..j-1]).

The answer we want is dp[m][n]. Indices run 0..m and 0..n (note the +1 — we include the empty prefix), so the table is (m+1) × (n+1).

Why prefixes? Because edits happen at the ends when we align greedily from the front: any optimal transformation of A into B can be described in terms of what we do with the last characters of the two current prefixes. That gives optimal substructure.

Step 2 — Derive the recurrence from the last characters

Look at A[i-1] and B[j-1] (the last chars of the two prefixes). There are exactly two cases:

Case A: the last characters match (A[i-1] == B[j-1]). Then we don't need to spend an edit on them; align them for free and the problem reduces to the smaller prefixes:

CODE
dp[i][j] = dp[i-1][j-1]

Case B: they differ. Some edit must reconcile them. There are exactly three ways, and we take the cheapest:

Substitute A[i-1] → B[j-1]: cost 1, then align the remaining A[0..i-2] with B[0..j-2] → 1 + dp[i-1][j-1].
Delete A[i-1] from A: cost 1, then align A[0..i-2] with B[0..j-1] → 1 + dp[i-1][j].
Insert B[j-1] into A: cost 1, then align A[0..i-1] with B[0..j-2] → 1 + dp[i][j-1].

So:

CODE
dp[i][j] = min( dp[i-1][j-1] + 1,   // substitute
                dp[i-1][j]   + 1,   // delete from A
                dp[i][j-1]   + 1 )  // insert into A

A clean way to write both cases together: let cost = (A[i-1] == B[j-1]) ? 0 : 1, then

CODE
dp[i][j] = min( dp[i-1][j-1] + cost,   // match (cost 0) or substitute (cost 1)
                dp[i-1][j]   + 1,      // delete
                dp[i][j-1]   + 1 )     // insert

The intuition worth verbalizing: each cell only depends on its top, left, and diagonal neighbors — the diagonal is the "match/substitute" move, up is "delete," left is "insert." This dependency pattern is why we fill the table row by row.

Step 3 — Base cases
dp[0][0] = 0 — two empty strings need no edits.
dp[i][0] = i — turning a length-i prefix into the empty string needs i deletions.
dp[0][j] = j — turning the empty string into a length-j prefix needs j insertions.

Getting these right is where many candidates slip; they encode "the cost of pure inserts/deletes to/from empty."

Step 4 — Fill order and complexity

Fill the table with i from 0..m, j from 0..n. Every cell is O(1). So:

Time: O(m·n) — fill every cell once.
Space: O(m·n) for the full table.

This is essentially optimal for computing the distance in general; there's no known strongly-subquadratic algorithm for arbitrary strings (and under the Strong Exponential Time Hypothesis, none exists — a nice senior-level nugget). Special cases are faster: if the distance is bounded by k, the Ukkonen band trick computes it in O(k·min(m,n)) by only filling a diagonal band of width 2k+1.

Step 5 — Space optimization

Because dp[i][j] only depends on the current and previous row, I don't need the whole table to get the number. Keep two rows (or even one row plus a saved diagonal value) and roll them. That reduces space to O(min(m,n)) (make the shorter string the columns). This is the version I'd write in production when I only need the distance and not the edit script.

The catch: with rolling rows you lose the ability to backtrack the actual operations, because reconstruction needs the full table (or Hirschberg's divide-and-conquer, which recovers the alignment in O(m·n) time and O(min(m,n)) space — worth naming for a senior panel).

Step 6 — Reconstructing the edit operations

To output which edits (not just how many), keep the full dp table and backtrack from dp[m][n] to dp[0][0]:

If A[i-1]==B[j-1] and dp[i][j]==dp[i-1][j-1] → match, move diagonally.
Else if dp[i][j]==dp[i-1][j-1]+1 → substitute, move diagonally.
Else if dp[i][j]==dp[i-1][j]+1 → delete, move up.
Else (dp[i][j]==dp[i][j-1]+1) → insert, move left.

Reverse the collected operations to get the transformation script.

Step 7 — Variants and subtleties (shows depth)
Why substitution is 1, not 2. In classic Levenshtein a substitution is a single primitive. If you only allowed insert+delete (i.e., substitution costs 2), you'd get the LCS-based distance — a different metric.
Weighted / general costs. DNA/spell-check often use per-operation or per-character-pair costs (a substitution matrix). Replace the +1/+cost with the specific weights; the DP structure is unchanged. This is the Needleman–Wunsch global-alignment framework.
Damerau–Levenshtein adds transposition of two adjacent characters as one edit — add a fourth transition dp[i-2][j-2]+1 when A[i-1]==B[j-2] && A[i-2]==B[j-1]. Common in typo correction.
It's a true metric — non-negative, symmetric, satisfies the triangle inequality — which lets you use it with metric-space data structures (BK-trees) for fuzzy search.
Real-world uses

Spell-checkers and autocomplete ("did you mean…"), git diff and diff/patch tooling, DNA/protein sequence alignment (bioinformatics), fuzzy record matching/deduplication, OCR post-correction, and plagiarism/near-duplicate detection. Naming a couple signals that I understand why this problem matters.

Dry run (see input_output)

For A = "horse", B = "ros" the answer is 3: horse → rorse (sub h→r) → rose (del r) → ros (del e). The table confirms dp[5][3] = 3.

Summary — what a strong answer emphasizes

Define dp[i][j] as the edit distance between length-i and length-j prefixes; base cases dp[i][0]=i, dp[0][j]=j; recurrence takes dp[i-1][j-1] free on a match, else 1 + min(substitute=dp[i-1][j-1], delete=dp[i-1][j], insert=dp[i][j-1]). That's O(m·n) time, reducible to O(min(m,n)) space for the number (Hirschberg for the alignment in linear space). I'd derive the recurrence from the last characters, get the base cases right, show reconstruction via backtracking, and demonstrate breadth by naming weighted costs, Damerau transposition, the Ukkonen band for small k, and the metric property.

#### Common mistakes
off-by-one in the (m+1)×(n+1) sizing; wrong base cases; forgetting that a match costs 0 (writing +1 everywhere); and conflating Levenshtein with LCS-distance. A good follow-up probe is "reduce space to O(min(m,n)) but still recover the alignment" — the answer is Hirschberg's algorithm.

**Comparison:**

```
Move in table	Meaning	Cell used
Diagonal, chars equal	Match (free)	dp[i-1][j-1] + 0
Diagonal, chars differ	Substitute	dp[i-1][j-1] + 1
Up	Delete from A	dp[i-1][j] + 1
Left	Insert into A	dp[i][j-1] + 1
Variant	Operations allowed	Notes
Levenshtein (this problem)	ins, del, sub (each 1)	Classic
LCS-distance	ins, del only (sub = 2)	Longest common subsequence based
Damerau–Levenshtein	+ adjacent transposition	Typo correction
Needleman–Wunsch	weighted costs / matrix	Global bio-sequence alignment
Method	Time	Space	Recovers alignment?
Full 2D table	O(m·n)	O(m·n)	Yes
Two-row rolling	O(m·n)	O(min(m,n))	No
Hirschberg	O(m·n)	O(min(m,n))	Yes
Ukkonen band (dist ≤ k)	O(k·min(m,n))	O(k)	Yes (banded)
```

**Code:**

```

// Classic unit-cost Levenshtein distance. Time O(mn), Space O(mn). public int editDistance(String a, String b) { int m = a.length(), n = b.length(); int[][] dp = new int[m + 1][n + 1];

CODE
Copy
for (int i = 0; i <= m; i++) dp[i][0] = i; // i deletions to reach ""
for (int j = 0; j <= n; j++) dp[0][j] = j; // j insertions from ""

for (int i = 1; i <= m; i++) {
    for (int j = 1; j <= n; j++) {
        int cost = (a.charAt(i - 1) == b.charAt(j - 1)) ? 0 : 1;
        dp[i][j] = Math.min(
            dp[i - 1][j - 1] + cost,         // match (0) or substitute (1)
            Math.min(dp[i - 1][j] + 1,       // delete from a
                     dp[i][j - 1] + 1));     // insert into a
    }
}
return dp[m][n];

}

CODE
Copy

```java
// Space-optimized to O(min(m,n)): only the previous & current rows are kept.
// Returns the DISTANCE only (no backtracking possible with rolling rows).
public int editDistanceOpt(String a, String b) {
    if (a.length() < b.length()) { String t = a; a = b; b = t; } // b = shorter
    int m = a.length(), n = b.length();
    int[] prev = new int[n + 1];
    for (int j = 0; j <= n; j++) prev[j] = j;

    for (int i = 1; i <= m; i++) {
        int[] cur = new int[n + 1];
        cur[0] = i;
        for (int j = 1; j <= n; j++) {
            int cost = (a.charAt(i - 1) == b.charAt(j - 1)) ? 0 : 1;
            cur[j] = Math.min(prev[j - 1] + cost,
                     Math.min(prev[j] + 1, cur[j - 1] + 1));
        }
        prev = cur;
    }
    return prev[n];
}
JAVA
Copy
// Reconstruct the edit script by backtracking the full table.
public java.util.List<String> editScript(String a, String b) {
    int m = a.length(), n = b.length();
    int[][] dp = new int[m + 1][n + 1];
    for (int i = 0; i <= m; i++) dp[i][0] = i;
    for (int j = 0; j <= n; j++) dp[0][j] = j;
    for (int i = 1; i <= m; i++)
        for (int j = 1; j <= n; j++) {
            int c = a.charAt(i-1) == b.charAt(j-1) ? 0 : 1;
            dp[i][j] = Math.min(dp[i-1][j-1]+c, Math.min(dp[i-1][j]+1, dp[i][j-1]+1));
        }

    java.util.LinkedList<String> ops = new java.util.LinkedList<>();
    int i = m, j = n;
    while (i > 0 || j > 0) {
        if (i > 0 && j > 0 && a.charAt(i-1) == b.charAt(j-1) && dp[i][j] == dp[i-1][j-1]) {
            i--; j--;                                             // match
        } else if (i > 0 && j > 0 && dp[i][j] == dp[i-1][j-1]+1) {
            ops.addFirst("SUB " + a.charAt(i-1) + "->" + b.charAt(j-1)); i--; j--;
        } else if (i > 0 && dp[i][j] == dp[i-1][j]+1) {
            ops.addFirst("DEL " + a.charAt(i-1)); i--;
        } else {
            ops.addFirst("INS " + b.charAt(j-1)); j--;
        }
    }
    return ops;
}
```

**Example input / output:**

```text
A = "horse", B = "ros"   -> edit distance = 3

DP table (rows = A prefixes "", h, ho, hor, hors, horse;
          cols = B prefixes "", r, o, s):

        ""  r   o   s
   ""   0   1   2   3
   h    1   1   2   3
   o    2   2   1   2
   r    3   2   2   2
   s    4   3   3   2
   e    5   4   4   3   <- dp[5][3] = 3  (answer)

One optimal edit script (from backtracking):
   SUB h->r , DEL r , DEL e     ("horse" -> "rorse" -> "rose" -> "ros")

More examples:
   "intention" -> "execution"  = 5
   "kitten"    -> "sitting"     = 3   (k->s, e->i, +g)
   "abc"       -> "abc"         = 0
   ""          -> "abc"         = 3   (3 inserts)
   "flaw"      -> "lawn"        = 2   (del f, ins n)
```

**Flow:**

1. i==0
2. j==0
3. both 0
4. yes
5. no
6. Compute dp[i][j] for prefixesA[0..i-1], B[0..j-1]
7. i==0 or j==0?
8. dp = j (j inserts)
9. dp = i (i deletes)
10. A[i-1] == B[j-1]?
11. dp = dp[i-1][j-1] (match, free)
12. dp = 1 + min( sub: dp[i-1][j-1],del: dp[i-1][j], ins: dp[i][j-1] )
13. Answer = dp[m][n]

