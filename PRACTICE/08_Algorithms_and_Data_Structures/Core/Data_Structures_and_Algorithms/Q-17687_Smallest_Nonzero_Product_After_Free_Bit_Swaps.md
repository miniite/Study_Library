---
type: "practice"
difficulty: "Other"
---

# Smallest Nonzero Product After Free Bit Swaps

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Greedy](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Greedy.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Fix a positive integer k. Build a collection S that contains every integer from 1 to 2^k - 1 inclusive, each written using exactly k binary digits (leading zeros allowed). You may repeat the following operation as many times as you wish:

Pick any two numbers currently in the collection, and pick a bit position j (with 0 <= j < k). Exchange the bit at position j in the first number with the bit at the same position j in the second number.

Because every operation only moves bits around within one column across two numbers, the total count of set bits in each column is preserved no matter how many operations you perform.

Among all collections reachable this way, find the smallest possible product of all the numbers that is still nonzero — a product counts as nonzero only if none of the elements has become 0. Since this product can be astronomically large, return it modulo 1_000_000_007.

Note: the minimization is over the true product; the modulo is applied only to the final answer, never during the comparison of candidate products.

Input: a single integer k. Output: the minimum nonzero product, taken modulo 1_000_000_007.

#### Cons
traints:

1 <= k <= 60

#### Example
s & test cases
TEXT
Input: k = 1 -> Output: 1
Why: S = {1}; a single element, so the product is just 1. Smallest input.

Input: k = 2 -> Output: 6
Why: S = {01, 10, 11}; any bit swap either drives an element to 0 or leaves the product unchanged, so 1*2*3 = 6 is already minimal.

Input: k = 3 -> Output: 1512
Why: the best reachable multiset gives 1*6*1*6*1*6*7 = 1512 (three pairs each pushed to (1, 6), plus the fixed 7).

Input: k = 4 -> Output: 581202553
Why: x = 15, y = 14, number of pairs = 2^3 - 1 = 7; the true product is 15 * 14^7 = 1581202560, and 1581202560 mod 1_000_000_007 = 581202553. Exercises modular reduction.

Input: k = 60 -> Output: 813987236
Why: upper bound; exponent is 2^59 - 1 and x = 2^60 - 1, so the solution must use 64-bit / big-integer arithmetic and fast modular exponentiation without overflow.

#### Approach

Since each swap only rearranges bits inside a single column across two numbers, the number of set bits in every column is invariant. That means the all-ones value x = 2^k - 1 can never be broken up or exceeded and is always present in the collection. The remaining 2^k - 2 numbers can be paired off. For two numbers whose combined column bits are fixed, their sum is fixed, and a product a * (sum - a) is minimized by making the pair as lopsided as possible — pushing one member down to 1 and the other up to 2^k - 2. There are (2^k - 2) / 2 = 2^(k-1) - 1 such pairs, each contributing a factor of 1 * (2^k - 2). Hence the minimum nonzero product is the closed form (2^k - 1) * (2^k - 2)^(2^(k-1) - 1), computed with modular fast exponentiation. Pattern: greedy pairing argument + modular exponentiation.

Complexity

Time: O(k) — the exponent has O(k) bits, so fast exponentiation performs O(k) modular multiplications. Space: O(1). Justification: the answer is a closed form evaluated with binary exponentiation.

Python
PYTHON
def min_non_zero_product(k):
    MOD = 1_000_000_007
    x = (1 << k) - 1          # 2^k - 1 : the all-ones value, always present
    y = x - 1                 # 2^k - 2 : the large partner in each pair
    exp = (1 << (k - 1)) - 1  # 2^(k-1) - 1 : number of pairs
    return (x % MOD) * pow(y % MOD, exp, MOD) % MOD
JavaScript
JAVASCRIPT
function modPow(base, exp, mod) {
    let result = 1n;
    base %= mod;
    while (exp > 0n) {
        if (exp & 1n) result = result * base % mod;
        base = base * base % mod;
        exp >>= 1n;
    }
    return result;
}

function minNonZeroProduct(k) {

#### cons
t MOD = 1000000007n;

#### cons
t x = (1n << BigInt(k)) - 1n;        // 2^k - 1

#### cons
t y = x - 1n;                        // 2^k - 2

#### cons
t exp = (1n << BigInt(k - 1)) - 1n;  // 2^(k-1) - 1 pairs
    return Number((x % MOD) * modPow(y % MOD, exp, MOD) % MOD);
}
Java
JAVA
class Solution {
    private static final long MOD = 1_000_000_007L;

    private long modPow(long base, long exp) {
        long result = 1;
        base %= MOD;
        while (exp > 0) {
            if ((exp & 1L) == 1L) result = result * base % MOD;
            base = base * base % MOD;
            exp >>= 1;
        }
        return result;
    }

    public int minNonZeroProduct(int k) {
        long x   = (1L << k) - 1;           // 2^k - 1  (fits in a signed 64-bit long)
        long y   = x - 1;                   // 2^k - 2
        long exp = (1L << (k - 1)) - 1;     // 2^(k-1) - 1 pairs
        return (int)((x % MOD) * modPow(y % MOD, exp) % MOD);
    }
}
C++
CPP
#include <cstdint>
using namespace std;

class Solution {
    static const uint64_t MOD = 1000000007ULL;

    uint64_t modPow(uint64_t base, uint64_t exp) {
        uint64_t result = 1;
        base %= MOD;
        while (exp) {
            if (exp & 1ULL) result = (__uint128_t)result * base % MOD;
            base = (__uint128_t)base * base % MOD;
            exp >>= 1;
        }
        return result;
    }

public:
    int minNonZeroProduct(int k) {
        uint64_t x   = (1ULL << k) - 1;           // 2^k - 1
        uint64_t y   = x - 1;                      // 2^k - 2
        uint64_t exp = (1ULL << (k - 1)) - 1;      // 2^(k-1) - 1 pairs
        return (int)((x % MOD) * modPow(y % MOD, exp) % MOD);
    }
};
C
C
#include <stdint.h>

/* fast modular exponentiation: base^exp mod m */
static uint64_t mod_pow(uint64_t base, uint64_t exp, uint64_t m) {
    uint64_t result = 1 % m;
    base %= m;
    while (exp > 0) {
        if (exp & 1ULL) result = (unsigned __int128)result * base % m;
        base = (unsigned __int128)base * base % m;
        exp >>= 1;
    }
    return result;
}

long long minNonZeroProduct(int k) {

#### cons
t uint64_t MOD = 1000000007ULL;
    uint64_t x   = ((uint64_t)1 << k) - 1;         /* 2^k - 1 : the all-ones value, always kept */
    uint64_t y   = x - 1;                          /* 2^k - 2 : the "large" partner in each pair */
    uint64_t exp = ((uint64_t)1 << (k - 1)) - 1;   /* 2^(k-1) - 1 : number of pairs             */
    uint64_t ans = (x % MOD) * mod_pow(y % MOD, exp, MOD) % MOD;
    return (long long)ans;
}

#### Follow-up
questions
Prove that splitting each pair into (1, 2^k - 2) is optimal, rather than any other split with the same sum.
Why must the minimum be determined on the true product before applying the modulo? Show how comparing values already reduced mod 1e9+7 could pick the wrong "minimum".
How large does the exponent get, and why is fast exponentiation required instead of a linear loop of multiplications?
If bit swaps were restricted to numbers that are adjacent in value, would the reachable set — and therefore the answer — change?

