---
type: "practice"
difficulty: "Other"
---

# Counting Doubly-Symmetric Squares in a Range

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ String](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/String.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

A positive integer is called doubly symmetric when it satisfies BOTH of the following conditions:

The integer itself is a palindrome — its decimal representation reads the same left-to-right and right-to-left (for example, 1, 9, 484, 12321).
The integer is the exact square of some palindrome — there exists a palindrome p such that p * p equals the integer.

For instance, 484 is doubly symmetric because 484 is a palindrome and 484 = 22 * 22, where 22 is itself a palindrome. In contrast, 676 is a palindrome but is not doubly symmetric: 676 = 26 * 26, and 26 is not a palindrome, and no palindromic number squares to 676.

You are given two positive integers low and high, each provided as a decimal string (because they can be very large). Return the count of doubly-symmetric integers x such that low <= x <= high, inclusive.

Input
low: a string of digits with no leading zeros, representing the lower bound.
high: a string of digits with no leading zeros, representing the upper bound.
Output
A single integer: the number of doubly-symmetric values in the inclusive range [low, high].

#### Cons
traints
1 <= low.length, high.length <= 18
low and high consist only of the characters '0'–'9'.
Neither low nor high has a leading zero.
Both strings represent integers in the range [1, 10^18 - 1].
The value of low is less than or equal to the value of high.

#### Example
s & test cases
TEXT
Input: low = "1", high = "50" -> Output: 3
  Why: 1 (=1^2), 4 (=2^2), 9 (=3^2) qualify; the next candidate 121 (=11^2) exceeds 50.

Input: low = "10", high = "150" -> Output: 1
  Why: only 121 (=11^2) is in range; 1, 4, 9 are below 10 and 484 is above 150.

Input: low = "1", high = "500" -> Output: 5
  Why: 1, 4, 9, 121, and 484 (=22^2) all qualify; 676 does not because 26 is not a palindrome.

Input: low = "8", high = "700" -> Output: 3
  Why: 9, 121, and 484 fall in [8, 700]; 1 and 4 are below the lower bound of 8.

Input: low = "1", high = "1000000" -> Output: 10
  Why: 1, 4, 9, 121, 484, 10201 (=101^2), 12321 (=111^2), 14641 (=121^2), 40804 (=202^2), 44944 (=212^2); the next one, 1002001 (=1001^2), exceeds 1000000.

Input: low = "1", high = "1" -> Output: 1
  Why: minimal range containing exactly the value 1, which is doubly symmetric.

#### Approach

The key observation is the double square-root shrink. A doubly-symmetric value x <= R has a root p with p*p = x, so p <= sqrt(R) <= 10^9, and since p must itself be a palindrome, p is fully determined by its front half, which has at most ~5 digits (about 10^5 possibilities). So instead of scanning numbers, we ENUMERATE palindromic roots directly: for each front half k, mirror it two ways (odd-length by skipping the middle digit, even-length by mirroring all digits) to build a palindrome p, square it, and check that the square lies in [low, high] and is itself a palindrome. Because generated palindromes increase monotonically with k, we can break early once the square exceeds high (and cap the root below 10^9 to avoid 64-bit overflow). This is the classic "generate-and-test from the half" enumeration pattern for constrained palindromes.

Complexity

Time: O(R^{1/4} * log R) where R is the upper bound. Because a valid root palindrome is below sqrt(R) <= 10^9, and each palindrome is generated from a front half of at most ~10^5 candidates (that is R^{1/4}); each candidate does O(log R) digit work to build and palindrome-check the square. Space: O(log R) for the string buffers holding at most 18–19 digits.

Python
PYTHON
class Solution:
    def countDoublySymmetric(self, low: str, high: str) -> int:
        lo, hi = int(low), int(high)
        LIMIT = 1_000_000_000  # roots beyond this square past 10**18

        def is_palindrome(num: int) -> bool:
            s = str(num)
            return s == s[::-1]

        def build(k: int, even: bool) -> int:
            head = str(k)
            tail = head[::-1] if even else head[-2::-1]
            return int(head + tail)

        count = 0
        for even in (False, True):
            k = 1
            while k < 100_000:
                root = build(k, even)
                if root > LIMIT:
                    break                 # palindromes are monotone in k
                sq = root * root
                if sq > hi:
                    break
                if sq >= lo and is_palindrome(sq):
                    count += 1
                k += 1
        return count
JavaScript
JAVASCRIPT
/**
 * @param {string} low
 * @param {string} high
 * @return {number}
 */
var countDoublySymmetric = function (low, high) {

#### cons
t lo = BigInt(low);

#### cons
t hi = BigInt(high);

#### cons
t LIMIT = 1000000000n; // roots beyond this square past 10^18

#### cons
t isPalindrome = (num) => {

#### cons
t s = num.toString();
        for (let l = 0, r = s.length - 1; l < r; l++, r--)
            if (s[l] !== s[r]) return false;
        return true;
    };

#### cons
t build = (k, even) => {

#### cons
t head = k.toString();
        let s = head;

#### cons
t start = even ? head.length - 1 : head.length - 2;
        for (let i = start; i >= 0; i--) s += head[i];
        return BigInt(s);
    };

    let count = 0;
    for (const even of [false, true]) {
        for (let k = 1; k < 100000; k++) {

#### cons
t root = build(k, even);
            if (root > LIMIT) break;          // monotone in k

#### cons
t sq = root * root;
            if (sq > hi) break;
            if (sq >= lo && isPalindrome(sq)) count++;
        }
    }
    return count;
};
Java
JAVA
class Solution {
    public int countDoublySymmetric(String low, String high) {
        long lo = Long.parseLong(low);
        long hi = Long.parseLong(high);
        final long LIMIT = 1_000_000_000L; // roots beyond this overflow when squared
        int count = 0;
        for (int even = 0; even <= 1; ++even) {
            for (long k = 1; k < 100000L; ++k) {
                long root = buildPalindrome(k, even);
                if (root > LIMIT) break;          // palindromes increase with k
                long sq = root * root;
                if (sq > hi) break;
                if (sq >= lo && isPalindrome(sq)) ++count;
            }
        }
        return count;
    }

    private long buildPalindrome(long k, int even) {
        String head = Long.toString(k);
        StringBuilder sb = new StringBuilder(head);
        int start = (even == 1) ? head.length() - 1 : head.length() - 2;
        for (int i = start; i >= 0; --i) sb.append(head.charAt(i));
        return Long.parseLong(sb.toString());
    }

    private boolean isPalindrome(long num) {
        String s = Long.toString(num);
        int l = 0, r = s.length() - 1;
        while (l < r)
            if (s.charAt(l++) != s.charAt(r--)) return false;
        return true;
    }
}
C++
CPP
#include <string>
#include <cmath>
using namespace std;

class Solution {
public:
    int countDoublySymmetric(const string& low, const string& high) {
        long long lo = stoll(low);
        long long hi = stoll(high);

#### cons
t long long LIMIT = 1000000000LL; // roots beyond this overflow when squared
        int count = 0;
        for (int even = 0; even <= 1; ++even) {
            for (long long k = 1; k < 100000LL; ++k) {
                long long root = buildPalindrome(k, even);
                if (root > LIMIT) break;          // monotone in k
                long long sq = root * root;
                if (sq > hi) break;
                if (sq >= lo && isPalindrome(sq)) ++count;
            }
        }
        return count;
    }

private:
    long long buildPalindrome(long long k, int even) {
        string head = to_string(k);
        string s = head;
        int start = even ? (int)head.size() - 1 : (int)head.size() - 2;
        for (int i = start; i >= 0; --i) s += head[i];
        return stoll(s);
    }

    bool isPalindrome(long long num) {
        string s = to_string(num);
        int l = 0, r = (int)s.size() - 1;
        while (l < r)
            if (s[l++] != s[r--]) return false;
        return true;
    }
};
C
C
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

static int isPalindrome(long long num) {
    char s[24];
    sprintf(s, "%lld", num);
    int l = 0, r = (int)strlen(s) - 1;
    while (l < r) {
        if (s[l] != s[r]) return 0;
        l++; r--;
    }
    return 1;
}

/* Build a palindrome from its front half `k`; `even` decides parity of length. */
static long long buildPalindrome(long long k, int even) {
    char head[16];
    sprintf(head, "%lld", k);
    int hlen = (int)strlen(head);
    char buf[32];
    strcpy(buf, head);
    int pos = hlen;
    /* mirror: even mirrors all digits, odd skips the last (middle) digit */
    int start = even ? hlen - 1 : hlen - 2;
    for (int i = start; i >= 0; i--) buf[pos++] = head[i];
    buf[pos] = '\0';
    return atoll(buf);
}

int countDoublySymmetric(const char *low, const char *high) {
    long long lo = atoll(low);
    long long hi = atoll(high);

#### cons
t long long LIMIT = 1000000000LL; /* floor(sqrt(10^18)) bound; larger roots overflow */
    int count = 0;
    for (int even = 0; even <= 1; even++) {
        for (long long k = 1; k < 100000LL; k++) {
            long long root = buildPalindrome(k, even);
            if (root > LIMIT) break;      /* palindromes grow monotonically in k */
            long long sq = root * root;
            if (sq > hi) break;
            if (sq >= lo && isPalindrome(sq)) count++;
        }
    }
    return count;
}

int main(void) {
    printf("%d\n", countDoublySymmetric("1", "50"));       /* 3 */
    printf("%d\n", countDoublySymmetric("1", "500"));      /* 5 */
    printf("%d\n", countDoublySymmetric("1", "1000000"));  /* 10 */
    return 0;
}

#### Follow-up
questions
How does the running time change if the upper bound R can be as large as 10^36 (arbitrary precision)? Which parts still hold and which need big-integer support?
Can you return the actual list of doubly-symmetric numbers rather than just the count, without materially increasing the time complexity?
How would you extend this to a different base b instead of base 10 — what changes in the half-mirroring construction?
Prove why enumerating fronts up to about 10^5 is sufficient and cannot miss any valid root.

