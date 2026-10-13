---
type: "practice"
difficulty: "Other"
---

# Identifying Values That Occur Exactly Twice Using the Array Itself

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Hash Table](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Hash_Table.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are handed a list of n locker tags. Every tag is an integer somewhere in the inclusive range [1, n], where n is the length of the list. Because of a duplication glitch, some tags were printed twice while the rest were printed once — no tag appears three or more times, and no value outside [1, n] ever appears.

Return a list of every tag value that appears exactly twice. The order of the returned values does not matter.

You must solve this in O(n) time and use only constant extra space, not counting the space needed for the output list. (You are permitted to modify the input list in place while computing the answer.)

Input

An integer array tags of length n, with 1 <= tags[i] <= n for every index i, and each value occurring once or twice.

Output

An array containing each value that occurs twice, in any order. If no value repeats, return an empty array.

#### Cons
traints

n == tags.length
1 <= n <= 10^5
1 <= tags[i] <= n
Each element appears at most twice.
Required: O(n) time and O(1) auxiliary space (output not counted).

#### Example
s & test cases
TEXT
Input: tags = [4,3,2,7,8,2,3,1] -> Output: [2,3]
Why: The values 2 and 3 each appear twice; every other value appears once. Typical mixed case.

Input: tags = [1,1,2] -> Output: [1]
Why: Only the value 1 is repeated; 2 appears once. Duplicate is the smallest value.

Input: tags = [5] -> Output: []
Why: A single element cannot repeat, so the answer is empty. Minimum-length case.

Input: tags = [2,2,3,3,1,1] -> Output: [2,3,1]
Why: Every distinct value repeats; the result contains all of them (order need not match). Stress case with all duplicates.

Input: tags = [1,2,3,4,5] -> Output: []
Why: A perfect permutation with no repeats returns an empty list. No-duplicate case.

Input: tags = [3,1,3,4,2] -> Output: [3]
Why: Only 3 repeats; verifies the marking works regardless of position order.

#### Approach

Because every value lies in [1, n], each value v has a natural "home slot" at index v - 1. We walk the array and, for the value at the current position, look at its home slot. The sign of the number stored there is used as a one-bit visited flag: the first time we reach a value we negate the number in its home slot; if we reach that same value again and find the slot already negative, it must be its second occurrence, so we record it. Using abs() when reading a value lets us recover the original magnitude even after slots have been flipped. This "index-as-hash / in-place sign marking" pattern turns the input array into an O(1)-space hash set, meeting the O(n)-time, constant-space requirement. (If mutating the input were disallowed, a length-(n+1) counting array would give the same answer in O(n) time with O(n) space.)

Complexity

Time: O(n) — a single pass over the array, each step doing O(1) work. Space: O(1) auxiliary — the array is reused as its own visited-marker table by flipping signs in place; only the required output list is allocated separately.

Python
PYTHON
from typing import List

class Solution:
    def findTwiceValues(self, tags: List[int]) -> List[int]:
        ans = []
        for t in tags:
            idx = abs(t) - 1              # value maps to an index in [0, n-1]
            if tags[idx] < 0:
                ans.append(idx + 1)       # slot already flipped -> value seen before
            else:
                tags[idx] = -tags[idx]    # mark this value's slot as visited
        return ans
JavaScript
JAVASCRIPT
function findTwiceValues(tags) {

#### cons
t ans = [];
    for (const t of tags) {

#### cons
t idx = Math.abs(t) - 1;     // value maps to an index in [0, n-1]
        if (tags[idx] < 0) {
            ans.push(idx + 1);           // slot already flipped -> value seen before
        } else {
            tags[idx] = -tags[idx];      // mark this value's slot as visited
        }
    }
    return ans;
}
Java
JAVA
import java.util.ArrayList;
import java.util.List;

class Solution {
    public List<Integer> findTwiceValues(int[] tags) {
        List<Integer> ans = new ArrayList<>();
        for (int t : tags) {
            int idx = Math.abs(t) - 1;   // value maps to an index in [0, n-1]
            if (tags[idx] < 0) {
                ans.add(idx + 1);        // slot already flipped -> value seen before
            } else {
                tags[idx] = -tags[idx];  // mark this value's slot as visited
            }
        }
        return ans;
    }
}
C++
CPP
#include <vector>
#include <cstdlib>
using namespace std;

class Solution {
public:
    vector<int> findTwiceValues(vector<int>& tags) {
        vector<int> ans;
        for (int t : tags) {
            int idx = abs(t) - 1;        // value maps to an index in [0, n-1]
            if (tags[idx] < 0) {
                ans.push_back(idx + 1);  // slot already flipped -> value seen before
            } else {
                tags[idx] = -tags[idx];  // mark this value's slot as visited
            }
        }
        return ans;
    }
};
C
C
#include <stdlib.h>

/* Caller frees the returned buffer. *returnSize gets the number of duplicates. */
int* findTwiceValues(int* tags, int tagsSize, int* returnSize) {
    int* ans = (int*)malloc(sizeof(int) * tagsSize);
    int cnt = 0;
    for (int i = 0; i < tagsSize; i++) {
        int idx = abs(tags[i]) - 1;      // value maps to an index in [0, n-1]
        if (tags[idx] < 0) {
            ans[cnt++] = idx + 1;        // slot already flipped -> value seen before
        } else {
            tags[idx] = -tags[idx];      // mark this value's slot as visited
        }
    }
    *returnSize = cnt;
    return ans;
}

#### Follow-up
questions
Instead of the values that appear twice, return the values in [1, n] that are missing entirely (never appear) using the same in-place technique.
What changes if a value may appear up to three times, and you must report those that appear an even vs. odd number of times?
Solve it without modifying the input array — what is the resulting space complexity, and what is the trade-off?
Explain why the sign trick fails if the values could be zero or negative, and how you would remap the range to make it work.

