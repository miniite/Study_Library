---
type: "practice"
difficulty: "Other"
---

# Counting Rotation Sequences That Reach a Target

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ String Matching](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/String_Matching.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given two strings A and B, each of length m, made up of lowercase English letters.

A single move on A works as follows: pick a length L with 0 < L < m, remove the last L characters of A (its suffix of length L), and place those characters — in their original order — at the front of A.

For example, if A = "wxyz" and you choose L = 2, you remove the suffix "yz" and prepend it, turning A into "yzwx". (Every move is therefore a circular rotation of the string by a nonzero, proper amount.)

You are also given an integer r. Count the number of distinct move sequences of length exactly r that transform A into B. Two sequences are considered different if they differ in the chosen L at any step (even if the intermediate strings coincide). Because the answer can be enormous, return it modulo 1000000007.

Input

A, B: strings of equal length m over lowercase English letters.
r: a positive integer, the exact number of moves.

Output

A single integer: the number of valid length-r move sequences, taken modulo 1000000007.

#### Cons
traints

2 <= m <= 5 * 10^5
|A| == |B| == m
1 <= r <= 10^15
A and B consist of lowercase English letters only.

#### Example
s & test cases
TEXT
- Input: `A = "wxyz", B = "yzwx", r = 2` -> Output: `2`
  Why: `B` is `A` rotated by 2. Reaching it in exactly 2 moves can be done two ways (two suffix lengths whose rotations sum to the needed total), so there are 2 sequences.
- Input: `A = "mnmnmn", B = "mnmnmn", r = 1` -> Output: `2`
  Why: `A` equals `B`, and because the string has period 2, rotating by 2 or by 4 leaves it unchanged — two single moves keep it identical.
- Input: `A = "ab", B = "ba", r = 1` -> Output: `1`
  Why: `m = 2`, so the only legal move length is `L = 1`, which swaps the two characters into `"ba"`. Exactly one sequence works.
- Input: `A = "ab", B = "ab", r = 1` -> Output: `0`
  Why: The one legal move (`L = 1`) necessarily changes `"ab"` into `"ba"`; you cannot stay at `"ab"` in exactly one move, so there are no valid sequences.
- Input: `A = "aaa", B = "aaa", r = 3` -> Output: `8`
  Why: Every character is identical, so any rotation still equals `B`. Each of the 3 moves has `m - 1 = 2` choices, giving `2^3 = 8` sequences that all end at `B`.

#### Approach

Each move is a circular rotation of A by a nonzero, proper amount, so after r moves the result depends only on the sum of the chosen rotation amounts modulo m. First, find every offset that maps A onto B: build A+A, run KMP to locate all start positions p in [0, m-1] where the window equals B. The offset p = 0 (meaning A == B) is special. Second, count sequences of r nonzero rotations whose total is a required residue mod m. By symmetry there are only two counts: a_k (sequences summing to residue 0) and b_k (sequences summing to any one fixed nonzero residue), satisfying a_k = (m-1)*b_{k-1} and b_k = a_{k-1} + (m-2)*b_{k-1}. This linear recurrence is evaluated with matrix exponentiation M = [[0, m-1],[1, m-2]] to handle r up to 10^15. The final answer is cnt0 * a_r + cnt_nonzero * b_r, where cnt0 is 1 if A == B and cnt_nonzero is the number of other matching offsets. Pattern: string matching (KMP) on the doubled string + linear-recurrence acceleration by matrix power.

Complexity

Time: O(m + log r). Building the KMP failure function and scanning A+A is O(m); the answer for exactly r moves is obtained by 2x2 matrix exponentiation in O(log r). Space: O(m) for the failure array (the matrices are O(1)).

Python
PYTHON
def count_rotation_sequences(A: str, B: str, r: int) -> int:
    MOD = 10**9 + 7
    n = len(A)

    # KMP failure function of B
    fail = [0] * n
    j = 0
    for i in range(1, n):
        c = B[i]
        while j > 0 and B[j] != c:
            j = fail[j - 1]
        if B[j] == c:
            j += 1
        fail[i] = j

    # Matches of B inside A+A at start positions 0..n-1 = valid rotation offsets
    total = 0
    cnt0 = 0
    j = 0
    for i in range(2 * n):
        c = A[i] if i < n else A[i - n]
        while j > 0 and B[j] != c:
            j = fail[j - 1]
        if B[j] == c:
            j += 1
        if j == n:
            start = i - n + 1
            if start < n:
                total += 1
                if start == 0:
                    cnt0 = 1
            j = fail[j - 1]

    # a_k = #length-r nonzero-rotation sequences summing to 0 (mod n)
    # b_k = same but summing to a fixed nonzero residue
    # [a_k, b_k]^T = M^r [1, 0]^T,  M = [[0, n-1], [1, n-2]]
    def mul(X, Y):
        return [
            [(X[0][0] * Y[0][0] + X[0][1] * Y[1][0]) % MOD,
             (X[0][0] * Y[0][1] + X[0][1] * Y[1][1]) % MOD],
            [(X[1][0] * Y[0][0] + X[1][1] * Y[1][0]) % MOD,
             (X[1][0] * Y[0][1] + X[1][1] * Y[1][1]) % MOD],
        ]

    M = [[0, (n - 1) % MOD], [1, (n - 2) % MOD]]
    R = [[1, 0], [0, 1]]
    e = r
    while e:
        if e & 1:
            R = mul(R, M)
        M = mul(M, M)
        e >>= 1

    a_k = R[0][0] % MOD
    b_k = R[1][0] % MOD
    cnt_nonzero = total - cnt0
    return (cnt0 * a_k + cnt_nonzero * b_k) % MOD
JavaScript
JAVASCRIPT
function countRotationSequences(A, B, r) {

#### cons
t MOD = 1000000007n;

#### cons
t n = A.length;

  // KMP failure function of B

#### cons
t fail = new Int32Array(n);
  let j = 0;
  for (let i = 1; i < n; i++) {

#### cons
t c = B[i];
    while (j > 0 && B[j] !== c) j = fail[j - 1];
    if (B[j] === c) j++;
    fail[i] = j;
  }

  // Matches of B inside A+A at start positions 0..n-1 = valid rotation offsets
  let total = 0, cnt0 = 0;
  j = 0;
  for (let i = 0; i < 2 * n; i++) {

#### cons
t c = i < n ? A[i] : A[i - n];
    while (j > 0 && B[j] !== c) j = fail[j - 1];
    if (B[j] === c) j++;
    if (j === n) {

#### cons
t start = i - n + 1;
      if (start < n) {
        total++;
        if (start === 0) cnt0 = 1;
      }
      j = fail[j - 1];
    }
  }

  // Matrix exponentiation (BigInt): M = [[0, n-1],[1, n-2]], vector [1,0]^T

#### cons
t mul = (X, Y) => [
    [(X[0][0] * Y[0][0] + X[0][1] * Y[1][0]) % MOD,
     (X[0][0] * Y[0][1] + X[0][1] * Y[1][1]) % MOD],
    [(X[1][0] * Y[0][0] + X[1][1] * Y[1][0]) % MOD,
     (X[1][0] * Y[0][1] + X[1][1] * Y[1][1]) % MOD],
  ];

  let M = [[0n, BigInt(n - 1) % MOD], [1n, BigInt(n - 2) % MOD]];
  let R = [[1n, 0n], [0n, 1n]];
  let e = BigInt(r);
  while (e > 0n) {
    if (e & 1n) R = mul(R, M);
    M = mul(M, M);
    e >>= 1n;
  }

#### cons
t aK = R[0][0] % MOD;

#### cons
t bK = R[1][0] % MOD;

#### cons
t cntNonzero = BigInt(total - cnt0);
  return ((BigInt(cnt0) * aK + cntNonzero * bK) % MOD).toString();
}
Java
JAVA
class Solution {
    static final long MOD = 1000000007L;

    public long countRotationSequences(String A, String B, long r) {
        int n = A.length();

        // KMP failure function of B
        int[] fail = new int[n];
        int j = 0;
        for (int i = 1; i < n; i++) {
            char c = B.charAt(i);
            while (j > 0 && B.charAt(j) != c) j = fail[j - 1];
            if (B.charAt(j) == c) j++;
            fail[i] = j;
        }

        // Matches of B in A+A at start positions 0..n-1 = valid rotation offsets
        long total = 0;
        int cnt0 = 0;
        j = 0;
        for (int i = 0; i < 2 * n; i++) {
            char c = (i < n) ? A.charAt(i) : A.charAt(i - n);
            while (j > 0 && B.charAt(j) != c) j = fail[j - 1];
            if (B.charAt(j) == c) j++;
            if (j == n) {
                int start = i - n + 1;
                if (start < n) {
                    total++;
                    if (start == 0) cnt0 = 1;
                }
                j = fail[j - 1];
            }
        }

        // Matrix exponentiation: M = [[0, n-1],[1, n-2]], vector [1,0]^T
        long[][] M = {{0, (n - 1) % MOD}, {1, (n - 2) % MOD}};
        long[][] R = {{1, 0}, {0, 1}};
        long e = r;
        while (e > 0) {
            if ((e & 1) == 1) R = mul(R, M);
            M = mul(M, M);
            e >>= 1;
        }
        long aK = R[0][0] % MOD;
        long bK = R[1][0] % MOD;

        long cntNonzero = total - cnt0;
        return ((cnt0 * aK) % MOD + (cntNonzero % MOD) * bK) % MOD;
    }

    private static long[][] mul(long[][] a, long[][] b) {
        long[][] c = new long[2][2];
        for (int i = 0; i < 2; i++)
            for (int j = 0; j < 2; j++)
                c[i][j] = (a[i][0] * b[0][j] + a[i][1] * b[1][j]) % MOD;
        return c;
    }
}
C++
CPP
#include <string>
#include <vector>
using namespace std;

class Solution {
    static const long long MOD = 1000000007LL;

    static void mul(long long C[2][2], long long A[2][2], long long B[2][2]) {
        long long t[2][2];
        for (int i = 0; i < 2; i++)
            for (int j = 0; j < 2; j++)
                t[i][j] = (A[i][0] * B[0][j] + A[i][1] * B[1][j]) % MOD;
        for (int i = 0; i < 2; i++)
            for (int j = 0; j < 2; j++)
                C[i][j] = t[i][j];
    }

public:
    long long countRotationSequences(const string& A, const string& B, long long r) {
        int n = (int)A.size();

        // KMP failure function of B
        vector<int> fail(n, 0);
        int j = 0;
        for (int i = 1; i < n; i++) {
            char c = B[i];
            while (j > 0 && B[j] != c) j = fail[j - 1];
            if (B[j] == c) j++;
            fail[i] = j;
        }

        // Matches of B inside A+A at start positions 0..n-1 = valid rotation offsets
        long long total = 0;
        int cnt0 = 0;
        j = 0;
        for (int i = 0; i < 2 * n; i++) {
            char c = (i < n) ? A[i] : A[i - n];
            while (j > 0 && B[j] != c) j = fail[j - 1];
            if (B[j] == c) j++;
            if (j == n) {
                int start = i - n + 1;
                if (start < n) {
                    total++;
                    if (start == 0) cnt0 = 1;
                }
                j = fail[j - 1];
            }
        }

        // Matrix exponentiation: M = [[0, n-1],[1, n-2]], vector [1,0]^T
        long long M[2][2] = {{0, (n - 1) % MOD}, {1, (n - 2) % MOD}};
        long long R[2][2] = {{1, 0}, {0, 1}};
        long long e = r;
        while (e) {
            if (e & 1) mul(R, R, M);
            mul(M, M, M);
            e >>= 1;
        }
        long long a_k = R[0][0] % MOD;
        long long b_k = R[1][0] % MOD;

        long long cnt_nonzero = total - cnt0;
        return (((long long)cnt0 * a_k) % MOD + (cnt_nonzero % MOD) * b_k) % MOD;
    }
};
C
C
#include <stdlib.h>
#include <string.h>

#define MODC 1000000007LL

/* 2x2 matrix multiply mod MODC, result into C (temp-safe: C may alias A or B) */
static void mat_mul(long long C[2][2], long long A[2][2], long long B[2][2]) {
    long long t[2][2];
    for (int i = 0; i < 2; i++)
        for (int j = 0; j < 2; j++)
            t[i][j] = (A[i][0] * B[0][j] + A[i][1] * B[1][j]) % MODC;
    memcpy(C, t, sizeof(t));
}

/* Number of length-r nonzero-rotation move sequences turning A into B, mod 1e9+7 */
long long countRotationSequences(const char *A, const char *B, long long r) {
    int n = (int)strlen(A);

    /* KMP failure function of B */
    int *fail = (int *)malloc(sizeof(int) * n);
    fail[0] = 0;
    int j = 0;
    for (int i = 1; i < n; i++) {
        char c = B[i];
        while (j > 0 && B[j] != c) j = fail[j - 1];
        if (B[j] == c) j++;
        fail[i] = j;
    }

    /* Scan A+A; a match starting at p in [0, n-1] means left-rotating A by p equals B */
    long long total = 0;
    int cnt0 = 0;          /* whether the p == 0 (A already equals B) rotation matches */
    j = 0;
    for (int i = 0; i < 2 * n; i++) {
        char c = (i < n) ? A[i] : A[i - n];
        while (j > 0 && B[j] != c) j = fail[j - 1];
        if (B[j] == c) j++;
        if (j == n) {
            int start = i - n + 1;
            if (start < n) {
                total++;
                if (start == 0) cnt0 = 1;
            }
            j = fail[j - 1];
        }
    }
    free(fail);

    /* a_k = #length-r sequences of nonzero rotations summing to 0 (mod n);
       b_k = same but summing to any fixed nonzero residue.
       [a_k, b_k]^T = M^r [1, 0]^T with M = [[0, n-1],[1, n-2]] */
    long long M[2][2] = {{0, (n - 1) % MODC}, {1, (n - 2) % MODC}};
    long long R[2][2] = {{1, 0}, {0, 1}};
    long long e = r;
    while (e) {
        if (e & 1) mat_mul(R, R, M);
        mat_mul(M, M, M);
        e >>= 1;
    }
    long long a_k = R[0][0] % MODC;   /* R * [1,0]^T -> first column */
    long long b_k = R[1][0] % MODC;

    long long cnt_nonzero = total - cnt0;
    return ((long long)cnt0 * a_k + cnt_nonzero * b_k) % MODC;
}

#### Follow-up
questions
How does the answer change if the identity move (L = 0 or L = m) is also allowed at each step?
Instead of exactly r moves, count the ways to reach B in at most r moves.
Why can't you just simulate the recurrence in O(r) time, and how does matrix exponentiation fix the 10^15 bound?
If two matching offsets exist, argue why every nonzero offset yields the same count b_r (symmetry of the residue classes).

