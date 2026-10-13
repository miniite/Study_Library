---
type: "practice"
difficulty: "Other"
---

# Build a Sequence by Positional Insertion

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Array](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Array.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given two integer arrays, values and slots, both of the same length n. They describe how to assemble a list one element at a time.

Start with an empty list called result. Then process the two arrays together from left to right. For each step i (from 0 up to n - 1):

Take the value values[i].
Insert it into result at index slots[i] (0-indexed). Inserting at index k means the new value ends up at position k, and every element that was previously at position k or later is shifted one place to the right.

After all n insertions have been performed, return the final result list.

It is guaranteed that every insertion is valid — that is, slots[i] is always a position that already exists in result or is exactly one past its current end, so no insertion ever leaves a gap.

Input

values: an array of n integers.
slots: an array of n integers, where slots[i] is the target insertion index for values[i].

Output

The single list obtained after performing all insertions in order.

#### Cons
traints

1 <= n <= 100
values.length == slots.length == n
0 <= values[i] <= 100
0 <= slots[i] <= i (so the insertion position is always valid at step i)

#### Example
s & test cases
TEXT
Input: values = [7,9,4,2,5], slots = [0,1,2,2,1] -> Output: [7,5,9,2,4]
Why: normal case with a mid-list insertion. Steps: [7] -> [7,9] -> [7,9,4] -> [7,9,2,4] (2 pushes 4 right) -> [7,5,9,2,4] (5 pushes the tail right).

Input: values = [3,6,8,1,0], slots = [0,1,2,3,0] -> Output: [0,3,6,8,1]
Why: every insertion except the last appends to the end; the last inserts at the front, shifting everything right.

Input: values = [42], slots = [0] -> Output: [42]
Why: minimum size (n = 1); the single value lands in an empty list.

Input: values = [5,5,5], slots = [0,0,0] -> Output: [5,5,5]
Why: duplicates and repeated front insertions; each new value is placed at index 0, pushing prior ones right.

Input: values = [1,2,3,4], slots = [0,0,0,0] -> Output: [4,3,2,1]
Why: always inserting at the front reverses the order of `values`.

Input: values = [10,20,30], slots = [0,1,1] -> Output: [10,30,20]
Why: the third value is inserted before the second, demonstrating a shift of a single trailing element.

#### Approach

Direct simulation. Maintain a growing list and, for each step, insert the current value at the requested index; the list data structure (or manual shifting in C) takes care of pushing later elements to the right. Because the problem guarantees every slots[i] <= i, the target index always exists within the partially built list, so no bounds handling is needed. The underlying pattern is array/list simulation; an insertion at index k costs O(k) due to shifting, giving the quadratic bound. For much larger inputs one could replace the list with an order-statistics balanced BST or a Fenwick-tree-indexed structure to reach O(n log n), but for the given constraints plain simulation is optimal and simplest.

Complexity

Time: O(n^2) in the worst case — each of the n insertions may shift up to O(n) trailing elements one position to the right. Space: O(n) for the result list (O(1) extra beyond the required output).

Python
PYTHON
from typing import List

class Solution:
    def buildSequence(self, values: List[int], slots: List[int]) -> List[int]:
        result: List[int] = []
        for value, slot in zip(values, slots):
            result.insert(slot, value)  # shifts existing elements at slot.. right
        return result
JavaScript
JAVASCRIPT
function buildSequence(values, slots) {

#### cons
t result = [];
    for (let i = 0; i < values.length; i++) {
        result.splice(slots[i], 0, values[i]); // insert without removing
    }
    return result;
}
Java
JAVA
import java.util.ArrayList;
import java.util.List;

class Solution {
    public int[] buildSequence(int[] values, int[] slots) {
        List<Integer> result = new ArrayList<>();
        for (int i = 0; i < values.length; i++) {
            result.add(slots[i], values[i]); // insert value at index slots[i]
        }
        int[] out = new int[result.size()];
        for (int i = 0; i < out.length; i++) {
            out[i] = result.get(i);
        }
        return out;
    }
}
C++
CPP
#include <vector>
using namespace std;

class Solution {
public:
    vector<int> buildSequence(vector<int>& values, vector<int>& slots) {
        vector<int> result;
        result.reserve(values.size());
        for (size_t i = 0; i < values.size(); ++i) {
            result.insert(result.begin() + slots[i], values[i]);
        }
        return result;
    }
};
C
C
#include <stdlib.h>

/* Returns a newly allocated array of length *returnSize. Caller frees it. */
int* buildSequence(int* values, int valuesSize, int* slots, int slotsSize, int* returnSize) {
    int* result = (int*)malloc(sizeof(int) * (valuesSize > 0 ? valuesSize : 1));
    int len = 0;
    for (int i = 0; i < valuesSize; ++i) {
        int pos = slots[i];
        /* shift elements [pos, len) one step right to open a hole at pos */
        for (int j = len; j > pos; --j) {
            result[j] = result[j - 1];
        }
        result[pos] = values[i];
        ++len;
    }
    *returnSize = len;
    return result;
}

#### Follow-up
questions
Can you achieve better than O(n^2) when n is large (e.g., 10^5)? (Use an order-statistics balanced BST or a Fenwick tree over positions to insert in O(log n).)
What if an insertion index could be out of range — how would you validate and handle it gracefully?
Suppose you must also support deletions interleaved with insertions and report the final list; how does the data-structure choice change?
Can you reconstruct the original slots array if you are given only the final list and the insertion order of values?

