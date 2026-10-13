---
type: "practice"
difficulty: "Other"
---

# XOR Signature over All Ordered Index Triples

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Bit Manipulation](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Bit_Manipulation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given a 0-indexed integer array arr of length n.

For any ordered triple of indices (i, j, k) with 0 <= i, j, k < n, define its combined value as:

CODE
(arr[i] | arr[j]) & arr[k]

where | is bitwise OR and & is bitwise AND.

The signature of the array is the bitwise XOR of the combined values of all n^3 ordered triples (i, j, k) (indices may repeat; i, j, and k range independently over all positions).

Return the signature of arr.

#### Cons
traints:

1 <= n <= 10^5
1 <= arr[i] <= 10^9

Note that n^3 can be as large as 10^15, so an approach that literally enumerates every triple is far too slow; you are expected to find the underlying structure.

#### Example
s & test cases
TEXT
- Input: `arr = [2, 5]` -> Output: `7`
  Why: the eight triples give combined values 2, 0, 2, 5, 2, 5, 0, 5; XOR of all of them is 7, which also equals `2 ^ 5`.
- Input: `arr = [7, 12, 9, 3, 20]` -> Output: `21`
  Why: normal mixed case — the signature equals `7 ^ 12 ^ 9 ^ 3 ^ 20 = 21`.
- Input: `arr = [10]` -> Output: `10`
  Why: single element — the only triple is (0,0,0) with combined value `(10 | 10) & 10 = 10`.
- Input: `arr = [6, 6]` -> Output: `0`
  Why: duplicates — every element cancels in XOR (`6 ^ 6 = 0`), so the signature is 0.
- Input: `arr = [1, 2, 4, 8]` -> Output: `15`
  Why: disjoint bits — `1 ^ 2 ^ 4 ^ 8 = 15`, confirming the reduction to a plain XOR of all elements.

#### Approach

Work one bit at a time. Fix a bit position b and let c be the number of array elements that have bit b set. The combined value of a triple (i, j, k) has bit b set exactly when arr[k] has bit b (that is the AND part) AND at least one of arr[i], arr[j] has bit b (that is the OR part). The number of such triples is c * (n^2 - (n - c)^2), because k has c valid choices and the number of (i, j) pairs with at least one bit set is n^2 - (n - c)^2. Bit b of the final XOR is the parity of that count. Simplify: c * (n^2 - (n-c)^2) = c * (2nc - c^2) = 2n*c^2 - c^3. Modulo 2, the 2n*c^2 term vanishes, leaving c^3 mod 2 = c mod 2. So bit b of the signature equals c mod 2 — exactly the parity of how many elements have bit b, which is precisely bit b of the XOR of all elements. Since this holds for every bit, the whole signature collapses to arr[0] ^ arr[1] ^ ... ^ arr[n-1], computable in one linear pass.

Complexity

Time: O(n) — a single pass XORing every element. Space: O(1) — only the running accumulator is kept. This replaces the impossible O(n^3) brute-force enumeration of all triples.

Python
PYTHON
from typing import List

def array_signature(arr: List[int]) -> int:
    result = 0
    for x in arr:
        result ^= x          # signature equals XOR of all elements
    return result
JavaScript
JAVASCRIPT
function arraySignature(arr) {
    let result = 0;
    for (const x of arr) result ^= x;   // signature == XOR of all elements
    return result;
}
Java
JAVA
class Solution {
    public int arraySignature(int[] arr) {
        int result = 0;
        for (int x : arr) result ^= x;   // signature reduces to XOR of all elements
        return result;
    }
}
C++
CPP
#include <vector>
using namespace std;

int arraySignature(vector<int>& arr) {
    int result = 0;
    for (int x : arr) result ^= x;   // signature == XOR of all elements
    return result;
}
C
C
int arraySignature(int* arr, int arrSize) {
    int result = 0;
    for (int i = 0; i < arrSize; i++) {
        result ^= arr[i];   /* signature collapses to XOR of all elements */
    }
    return result;
}

#### Follow-up
questions
Prove rigorously why the answer reduces to the XOR of all elements (the per-bit parity argument above).
How does the result change if the combined value is (arr[i] & arr[j]) | arr[k] instead? Redo the parity counting for the new operation.
What if only unordered distinct triples i < j < k are considered? The counting per bit changes and the clean collapse no longer holds.
Can you compute the signature for a streaming feed of values where n is unknown in advance? (Yes — maintain a single running XOR.)

