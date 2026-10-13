---
type: "practice"
difficulty: "Other"
---

# Count Contiguous Segments Containing Every Distinct Value

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Sliding Window](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Sliding_Window.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given an array values consisting of positive integers.

Let D be the number of distinct integers that appear anywhere in values.

We call a contiguous, non-empty segment (subarray) of values full if it contains exactly D distinct integers — that is, the segment includes at least one occurrence of every value that appears in the entire array.

Return the number of full segments.

A segment (subarray) is a contiguous non-empty portion of the array.

Input

values: an array of positive integers.

Output

A single integer: the count of contiguous segments whose set of distinct values equals the set of distinct values of the whole array.

#### Cons
traints

1 <= values.length <= 1000
1 <= values[i] <= 2000

#### Example
s & test cases
TEXT
Input: values = [1,3,1,2,2] -> Output: 4
Why: the whole array has 3 distinct values {1,2,3}. The full segments are [1,3,1,2], [1,3,1,2,2], [3,1,2], and [3,1,2,2].

Input: values = [5,5,5,5] -> Output: 10
Why: only one distinct value exists, so every one of the 4+3+2+1 = 10 non-empty segments is full (all-duplicates edge case).

Input: values = [7] -> Output: 1
Why: a single element; the whole array has 1 distinct value and the only segment [7] contains it (min-size input).

Input: values = [1,2,3] -> Output: 1
Why: 3 distinct values, so only the entire array [1,2,3] contains all of them.

Input: values = [2,1,2,1,2] -> Output: 6
Why: 2 distinct values {1,2}. Every segment that includes at least one 1 and one 2 is full; there are 6 such segments.

#### Approach

First compute D, the total number of distinct values in the array. The key observation: if a window [left, right] already contains all D distinct values, then extending its right endpoint to any position >= right keeps it full — so it contributes (n - right) full segments for the current left. Use a sliding window with a frequency map tracking how many distinct values the window currently holds. Expand right; whenever the window becomes full (windowDistinct == totalDistinct), add (n - right) to the answer and shrink from the left to look for the next, smaller full window at this right boundary. This "count all valid right-extensions, then shrink left" technique counts every full subarray in a single linear pass. (Equivalent alternative: for each left, binary-search / two-pointer the minimal right that makes it full, then add n - right.)

Complexity

Time: O(n) — each of the two pointers left and right advances at most n times, and hash operations are O(1) amortized (a single pass counts total distinct values first). Space: O(D) — the frequency map holds at most D distinct keys, where D <= n.

Python
PYTHON
from typing import List
from collections import defaultdict

class Solution:
    def countFullSegments(self, values: List[int]) -> int:
        total_distinct = len(set(values))
        n = len(values)

        freq = defaultdict(int)
        window_distinct = 0
        answer = 0
        left = 0
        for right in range(n):
            if freq[values[right]] == 0:
                window_distinct += 1
            freq[values[right]] += 1
            while window_distinct == total_distinct:
                answer += n - right
                freq[values[left]] -= 1
                if freq[values[left]] == 0:
                    window_distinct -= 1
                left += 1
        return answer
JavaScript
JAVASCRIPT
/**
 * @param {number[]} values
 * @return {number}
 */
var countFullSegments = function(values) {

#### cons
t totalDistinct = new Set(values).size;

#### cons
t n = values.length;

#### cons
t freq = new Map();
    let windowDistinct = 0;
    let answer = 0;
    let left = 0;
    for (let right = 0; right < n; right++) {

#### cons
t cur = values[right];

#### cons
t c = freq.get(cur) || 0;
        freq.set(cur, c + 1);
        if (c === 0) windowDistinct++;
        while (windowDistinct === totalDistinct) {
            answer += (n - right);

#### cons
t lv = values[left];

#### cons
t lc = freq.get(lv);
            freq.set(lv, lc - 1);
            if (lc - 1 === 0) windowDistinct--;
            left++;
        }
    }
    return answer;
};
Java
JAVA
import java.util.HashMap;
import java.util.HashSet;
import java.util.Map;
import java.util.Set;

class Solution {
    public int countFullSegments(int[] values) {
        Set<Integer> distinct = new HashSet<>();
        for (int v : values) distinct.add(v);
        int totalDistinct = distinct.size();
        int n = values.length;

        Map<Integer, Integer> freq = new HashMap<>();
        int windowDistinct = 0;
        long answer = 0;
        int left = 0;
        for (int right = 0; right < n; right++) {
            int c = freq.getOrDefault(values[right], 0);
            freq.put(values[right], c + 1);
            if (c == 0) windowDistinct++;
            while (windowDistinct == totalDistinct) {
                answer += (n - right);
                int lc = freq.get(values[left]);
                freq.put(values[left], lc - 1);
                if (lc - 1 == 0) windowDistinct--;
                left++;
            }
        }
        return (int) answer;
    }
}
C++
CPP
#include <vector>
#include <unordered_set>
#include <unordered_map>
using namespace std;

class Solution {
public:
    int countFullSegments(vector<int>& values) {
        unordered_set<int> distinct(values.begin(), values.end());
        int totalDistinct = (int)distinct.size();
        int n = (int)values.size();

        unordered_map<int, int> freq;
        int windowDistinct = 0;
        long long answer = 0;
        int left = 0;
        for (int right = 0; right < n; right++) {
            if (freq[values[right]]++ == 0) windowDistinct++;
            while (windowDistinct == totalDistinct) {
                answer += (n - right);
                if (--freq[values[left]] == 0) windowDistinct--;
                left++;
            }
        }
        return (int)answer;
    }
};
C
C
#include <stddef.h>

// Counts full segments using a sliding window over value range [1, 2000].
int countFullSegments(int* values, int valuesSize) {
    int seen[2001] = {0};
    int totalDistinct = 0;
    for (int i = 0; i < valuesSize; i++) {
        if (seen[values[i]] == 0) {
            totalDistinct++;
            seen[values[i]] = 1;
        }
    }

    int freq[2001] = {0};
    int windowDistinct = 0;
    long long answer = 0;
    int left = 0;
    for (int right = 0; right < valuesSize; right++) {
        if (freq[values[right]] == 0) windowDistinct++;
        freq[values[right]]++;
        // Once [left..right] holds all distinct values, every prefix start
        // from left..(shrunk boundary) keeps it full; shrink and count.
        while (windowDistinct == totalDistinct) {
            answer += (valuesSize - right); // all right-extensions stay full
            freq[values[left]]--;
            if (freq[values[left]] == 0) windowDistinct--;
            left++;
        }
    }
    return (int)answer;
}

#### Follow-up
questions
Can you instead count subarrays with exactly k distinct values? (atMost(k) - atMost(k-1) trick.)
Return the shortest full segment (its length or its start/end indices) rather than the count.
How does the approach change if values can be negative or very large (no fixed [1, 2000] bound)? (Use a hash map instead of a fixed array.)
Handle very large arrays where the input arrives as a stream — can you keep the window incrementally?

