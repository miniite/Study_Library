---
type: "practice"
difficulty: "Other"
---

# Detect a Contiguous Block That Repeats Back-to-Back

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Sliding Window](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Sliding_Window.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given an array of positive integers values. Your task is to decide whether some fixed-length block of consecutive elements repeats itself, immediately and without gaps, at least a required number of times.

Define a block as a contiguous run of exactly blockLen elements. The block is said to repeat reps times when that exact same sequence of blockLen values appears reps times in a row — each copy directly following the previous one, with no gaps between copies and no overlap.

Return true if there exists any block of length blockLen that appears reps or more consecutive times somewhere inside values; otherwise return false.

Notes:

The repetition must be at least reps copies back-to-back. Fewer copies than reps do not count.
The copies must be adjacent and must not overlap one another.

Input

values: an array of positive integers.
blockLen: the length of the block to look for.
reps: the minimum number of consecutive repetitions required.

Output

A boolean: true if such a repeated block exists, false otherwise.

#### Cons
traints

2 <= values.length <= 100
1 <= values[i] <= 100
1 <= blockLen <= 100
2 <= reps <= 100

#### Example
s & test cases
TEXT
Input: values = [7,3,9,9,9,9], blockLen = 1, reps = 3 -> Output: true
Why: the block (9) of length 1 appears 4 times in a row, which is at least the required 3.

Input: values = [5,6,5,6,5,5,5,2], blockLen = 2, reps = 2 -> Output: true
Why: the block (5,6) of length 2 appears twice back-to-back at the start, satisfying reps = 2.

Input: values = [5,6,5,6,5,2], blockLen = 2, reps = 3 -> Output: false
Why: the block (5,6) repeats only 2 consecutive times, one short of the required 3, and no other length-2 block does better.

Input: values = [4,4], blockLen = 1, reps = 2 -> Output: true
Why: the smallest edge case — the block (4) appears exactly twice in a row.

Input: values = [1,2,3,4], blockLen = 2, reps = 2 -> Output: false
Why: no length-2 block is immediately followed by an identical copy of itself.

Input: values = [8,8,8], blockLen = 2, reps = 2 -> Output: false
Why: two copies of a length-2 block would need 4 elements, but the array has only 3, so it cannot fit.

#### Approach

A block of length blockLen repeated reps times back-to-back is exactly a window whose values are periodic with period blockLen. Periodicity means values[i] == values[i + blockLen] for every index inside the overlapping region, and that region spans (reps - 1) * blockLen consecutive indices. So scan the array once, maintaining a streak counter of how many consecutive positions satisfy values[i] == values[i + blockLen]; reset it to zero on any mismatch. The moment the streak reaches (reps - 1) * blockLen, a qualifying repeated block exists, so return true. If the array is too short (fewer than blockLen * reps elements) the streak can never reach the target and the answer is naturally false. Pattern: sliding periodicity / consecutive-run counting.

Complexity

Time: O(n), where n is the length of values — a single linear pass comparing each element with the one blockLen positions ahead. Space: O(1) — only a running counter is kept.

Python
PYTHON
def has_repeated_block(values, block_len, reps):
    n = len(values)
    # A block of length block_len repeated reps times has period block_len:
    # values[i] == values[i + block_len] must hold for (reps-1)*block_len
    # consecutive indices.
    target = (reps - 1) * block_len
    count = 0
    for i in range(n - block_len):
        if values[i] == values[i + block_len]:
            count += 1
            if count == target:
                return True
        else:
            count = 0
    return False
JavaScript
JAVASCRIPT
function hasRepeatedBlock(values, blockLen, reps) {

#### cons
t n = values.length;
    // Need (reps-1)*blockLen consecutive matches of values[i] === values[i+blockLen].

#### cons
t target = (reps - 1) * blockLen;
    let count = 0;
    for (let i = 0; i + blockLen < n; i++) {
        if (values[i] === values[i + blockLen]) {
            if (++count === target) return true;
        } else {
            count = 0;
        }
    }
    return false;
}
Java
JAVA
class Solution {
    public boolean hasRepeatedBlock(int[] values, int blockLen, int reps) {
        int n = values.length;
        // Need (reps-1)*blockLen consecutive matches of values[i] == values[i+blockLen].
        int target = (reps - 1) * blockLen;
        int count = 0;
        for (int i = 0; i + blockLen < n; i++) {
            if (values[i] == values[i + blockLen]) {
                if (++count == target) return true;
            } else {
                count = 0;
            }
        }
        return false;
    }
}
C++
CPP
#include <vector>
using namespace std;

bool hasRepeatedBlock(vector<int>& values, int blockLen, int reps) {
    int n = (int)values.size();
    // Need (reps-1)*blockLen consecutive matches of values[i] == values[i+blockLen].
    int target = (reps - 1) * blockLen;
    int count = 0;
    for (int i = 0; i + blockLen < n; i++) {
        if (values[i] == values[i + blockLen]) {
            if (++count == target) return true;
        } else {
            count = 0;
        }
    }
    return false;
}
C
C
#include <stdbool.h>

bool hasRepeatedBlock(int *values, int valuesSize, int blockLen, int reps) {
    /* A length-blockLen block repeated reps times has period blockLen, i.e.
       values[i] == values[i + blockLen] must hold for (reps-1)*blockLen

#### cons
ecutive positions. Count that streak. */
    int target = (reps - 1) * blockLen;
    int count = 0;
    for (int i = 0; i + blockLen < valuesSize; i++) {
        if (values[i] == values[i + blockLen]) {
            if (++count == target) return true;
        } else {
            count = 0;
        }
    }
    return false;
}

#### Follow-up
questions
Instead of a boolean, return the actual repeated block (or its starting index).
Count how many distinct length-blockLen blocks repeat at least reps times.
What changes if overlapping repetitions are allowed to count?
Generalize to detect the longest block that repeats at least reps times.

