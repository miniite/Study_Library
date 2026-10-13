---
type: "practice"
difficulty: "Other"
---

# Recover a Sequence from Its Consecutive XOR Encoding

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Bit Manipulation](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Bit_Manipulation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

A hidden sequence original of n non-negative integers was compressed into a shorter encoding. The encoding is an array code of length n - 1 where each entry is the bitwise XOR of two neighbouring elements of the original sequence:

CODE
code[i] = original[i] XOR original[i + 1]   for every valid i

For example, if original = [3, 3, 5] then code = [3 XOR 3, 3 XOR 5] = [0, 6].

You are given the code array together with a single integer head, which is the very first element of the original sequence (head = original[0]). Reconstruct and return the entire original sequence original.

It is guaranteed that a valid original sequence exists and is unique.

Recall that the bitwise XOR (^) of two numbers compares them bit by bit, producing 1 where the bits differ and 0 where they match, and that XOR is its own inverse: (x XOR y) XOR y == x.

Input

code: an array of n - 1 non-negative integers (the consecutive XORs).
head: a non-negative integer, the first element of the original sequence.

Output

The original array original of length n (n = code.length + 1).

#### Cons
traints

2 <= n <= 10^4
code.length == n - 1
0 <= code[i] <= 10^5
0 <= head <= 10^5

#### Example
s & test cases
TEXT
- Input: `code = [5,1,6], head = 2` -> Output: `[2,7,6,0]`
  Why: `2^5=7`, `7^1=6`, `6^6=0`; rolling XOR forward from the head reconstructs the sequence.
- Input: `code = [3,8], head = 9` -> Output: `[9,10,2]`
  Why: `9^3=10`, `10^8=2`; a short normal case with distinct values.
- Input: `code = [0], head = 5` -> Output: `[5,5]`
  Why: minimum length (`n = 2`); a zero XOR means the two adjacent elements are equal.
- Input: `code = [10], head = 0` -> Output: `[0,10]`
  Why: the first element is `0`, so the next element is just `code[0]` itself (`0^10=10`).
- Input: `code = [7,7], head = 0` -> Output: `[0,7,0]`
  Why: `0^7=7`, `7^7=0`; repeated encoding values return the sequence to an earlier value (edge case with repeats).

#### Approach

Use the defining property of XOR: it is its own inverse, so a XOR b == c implies a == c XOR b and b == c XOR a. Since code[i] = original[i] XOR original[i+1], XORing both sides by original[i] gives original[i+1] = original[i] XOR code[i]. Starting from the known first element head = original[0], we roll this recurrence forward: each next element is the running value XORed with the current encoded entry. This is a straightforward linear scan (a prefix/running-XOR technique) — no candidate guessing is needed because the head pins the whole sequence, guaranteeing the unique answer.

Complexity

Time: O(n) — a single pass over the n - 1 encoded values, doing one XOR each. Space: O(n) for the output array and O(1) auxiliary beyond it. Justification: each original element is produced exactly once from the previous element and the corresponding encoded value.

Python
PYTHON
from typing import List

class Solution:
    def decodeSequence(self, code: List[int], head: int) -> List[int]:
        original = [head]
        for c in code:
            original.append(original[-1] ^ c)   # original[i+1] = original[i] XOR code[i]
        return original
JavaScript
JAVASCRIPT
function decodeSequence(code, head) {

#### cons
t original = new Array(code.length + 1);
    original[0] = head;
    for (let i = 0; i < code.length; i++)
        original[i + 1] = original[i] ^ code[i];   // roll the running XOR forward
    return original;
}
Java
JAVA
class Solution {
    public int[] decodeSequence(int[] code, int head) {
        int n = code.length + 1;
        int[] original = new int[n];
        original[0] = head;
        for (int i = 0; i < code.length; i++)
            original[i + 1] = original[i] ^ code[i];   // XOR self-inverse recovers each element
        return original;
    }
}
C++
CPP
#include <vector>
using namespace std;

class Solution {
 public:
  vector<int> decodeSequence(vector<int>& code, int head) {
    vector<int> original(code.size() + 1);
    original[0] = head;
    for (size_t i = 0; i < code.size(); ++i)
      original[i + 1] = original[i] ^ code[i];   // roll the running XOR forward
    return original;
  }
};
C
C
#include <stdlib.h>

/* Returns a newly allocated array of length codeSize + 1; caller frees it. */
int* decodeSequence(int* code, int codeSize, int head, int* returnSize) {
    int n = codeSize + 1;
    int* original = (int*)malloc(sizeof(int) * n);
    original[0] = head;
    for (int i = 0; i < codeSize; ++i)
        original[i + 1] = original[i] ^ code[i];   /* XOR is self-inverse: peel off each step */
    *returnSize = n;
    return original;
}

#### Follow-up
questions
What if only the last element were given instead of the first — how would you reconstruct the sequence?
Given the full original sequence, produce the encoded array (the inverse direction).
If a companion problem gives you the XOR of an odd-length permutation-based array, how do you recover one missing element in O(1) space using the same self-inverse property?
How would you verify a reconstructed sequence is consistent with the encoding in a single pass?

