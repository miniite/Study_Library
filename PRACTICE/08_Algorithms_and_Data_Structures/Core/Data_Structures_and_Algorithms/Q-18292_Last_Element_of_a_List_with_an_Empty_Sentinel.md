---
type: "practice"
difficulty: "Other"
---

# Last Element of a List with an Empty Sentinel

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Arrays](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Arrays.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Given a list of integers, return its last element. If the list is empty (contains no elements), return -1 as a sentinel value indicating "no last element."

Note that -1 is used purely as the empty-list signal. If the list is non-empty and its last element happens to be -1, you still return -1 — that is simply the correct last element.

Input

nums: an array (list) of integers, possibly empty.

Output

The last element of nums, or -1 when nums is empty.

#### Cons
traints

0 <= nums.length <= 1000
Each element of nums is an integer that fits in a 32-bit signed integer.

#### Example
s & test cases
TEXT
- Input: nums=[4, 9, 2] -> Output: 2
  Why: the final element of a normal multi-element list is returned directly.
- Input: nums=[] -> Output: -1
  Why: the list is empty, so the sentinel -1 is returned (edge case).
- Input: nums=[7] -> Output: 7
  Why: a single-element list; the only element is also the last one.
- Input: nums=[-3, -8, -1] -> Output: -1
  Why: the actual last element is -1, so it is returned as-is (it is a real value, not the empty signal).
- Input: nums=[0, 0, 5, 0] -> Output: 0
  Why: duplicates and zeros are fine; only the final position matters, which is 0.

#### Approach

Direct constant-time index access. The last element of a contiguous list sits at position length - 1, so there is no need to iterate. The only special case is an empty list, where position -1/length - 1 is invalid; guard for length == 0 first and return the agreed sentinel -1. The pattern is "index from the end with a defined default," a common building block for safe accessors. Because -1 doubles as both a valid stored value and the empty signal, the contract explicitly resolves the ambiguity by defining the empty case, keeping the accessor total (defined for every input).

Complexity

Time: O(1) — a single length check and one direct index into the last position, independent of list size. Space: O(1) — no auxiliary storage is used.

Python
PYTHON
from typing import List

def last_element(nums: List[int]) -> int:
    """Return the final element, or -1 if the list is empty."""
    if not nums:
        return -1
    return nums[-1]
JavaScript
JS
/**
 * @param {number[]} nums
 * @return {number} the last element, or -1 if nums is empty.
 */
function lastElement(nums) {
    return nums.length === 0 ? -1 : nums[nums.length - 1];
}
Java
JAVA
class Solution {
    public int lastElement(int[] nums) {
        if (nums.length == 0) {
            return -1;
        }
        return nums[nums.length - 1];
    }
}
C++
CPP
#include <vector>
using namespace std;

class Solution {
public:
    int lastElement(const vector<int>& nums) {
        if (nums.empty()) {
            return -1;
        }
        return nums.back();
    }
};
C
C
/* Returns the last element of nums, or -1 when the array is empty. */
int lastElement(const int* nums, int numsSize) {
    if (numsSize == 0) {
        return -1;
    }
    return nums[numsSize - 1];
}

#### Follow-up
questions
Generalize to a kthFromEnd(nums, k) accessor that returns the k-th element counted from the end; how do you keep it total for out-of-range k?
If you exposed this by augmenting the array/list type globally with a .last() method, what pitfalls arise (enumeration pollution, collisions with other libraries) and how do you make the added property non-enumerable?
How would you add a symmetric first() accessor, and how do you keep the empty-list contract consistent between the two?
What if elements can be arbitrary values (objects, nulls) rather than integers — how would you choose an unambiguous "empty" sentinel then?

