---
type: "practice"
difficulty: "Other"
---

# Fewest Replacements to Make an Array a Consecutive Run

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Sliding Window](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Sliding_Window.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given an array of integers arr. In a single move you may pick any one element of arr and overwrite it with any integer you choose (positive, negative, or zero — there is no restriction on the replacement value).

The array is called compact when both of these hold:

Every element of arr is distinct (no value repeats).
The difference between the largest element and the smallest element equals arr.length - 1.

Equivalently, a compact array is exactly a set of arr.length consecutive integers in some order — for example [8, 6, 9, 7] is compact (it is the consecutive run 6..9 shuffled), while [3, 4, 5, 7] is not (there is a gap at 6).

Return the minimum number of moves required to make arr compact.

Input

An array arr of integers.

Output

A single integer: the least number of element replacements needed.

#### Cons
traints

1 <= arr.length <= 10^5
1 <= arr[i] <= 10^9

#### Example
s & test cases
TEXT
Input: arr = [7,5,8,6] -> Output: 0
Why: The values are distinct and max - min = 8 - 5 = 3 = length - 1, so it is already the consecutive run 5..8; no moves needed.

Input: arr = [6,7,8,10,11] -> Output: 1
Why: Length 5 needs a window of 5 consecutive values. The window [6..10] already contains 6,7,8,10 (four keepers), so only one element (11) must be changed — e.g. to 9.

Input: arr = [5,50,500,5000] -> Output: 3
Why: Any window of 4 consecutive integers can cover at most one of these far-apart values, so three of the four must be replaced.

Input: arr = [4] -> Output: 0
Why: A single element is trivially distinct and max - min = 0 = length - 1, so it is already compact.

Input: arr = [9,9,9] -> Output: 2
Why: Duplicates violate distinctness; keep one 9 and replace the other two (e.g. to 8 and 10 giving 8,9,10).

Input: arr = [1,2,4,4,5] -> Output: 1
Why: Length 5 needs a run of 5. The window [1..5] holds the distinct values 1,2,4,5 (four keepers), so only the extra element (the duplicate 4) must be changed — e.g. to 3.

#### Approach

A compact array of length N is exactly a permutation of some run of N consecutive integers [x, x+N-1]. Elements you keep must already be distinct and fall inside that run; every other slot needs one replacement. So the task reduces to: choose the window [x, x+N-1] that already contains the maximum number of the array's distinct values, and pay N minus that count in replacements. Deduplicate and sort the values, then for each distinct value s[i] treated as the window's left edge, use a two-pointer (sliding window) to count how many distinct values lie in [s[i], s[i] + N - 1]. Because duplicates can never both be kept, working on the unique sorted list automatically prevents overcounting. The best window gives the fewest replacements. Pattern: sort + fixed-width sliding window over unique values.

Complexity

Time: O(N log N), dominated by sorting the N elements (deduplication and the two-pointer sweep are both O(N)). Space: O(N) for the sorted, deduplicated copy of the values. Each pointer moves forward at most N times across the whole scan, so the window sliding is linear.

Python
PYTHON
def min_replacements(arr):
    n = len(arr)
    s = sorted(set(arr))                 # distinct values, ascending
    res = n
    j = 0
    for i in range(len(s)):
        # extend the window as far right as still fits in [s[i], s[i] + n - 1]
        while j < len(s) and s[j] <= s[i] + n - 1:
            j += 1
        res = min(res, n - (j - i))      # elements outside the window must change
    return res
JavaScript
JAVASCRIPT
function minReplacements(arr) {

#### cons
t n = arr.length;

#### cons
t s = [...new Set(arr)].sort((a, b) => a - b);   // distinct, ascending
    let res = n, j = 0;
    for (let i = 0; i < s.length; i++) {
        while (j < s.length && s[j] <= s[i] + n - 1) j++; // widen window while it fits
        res = Math.min(res, n - (j - i));                 // replace what falls outside
    }
    return res;
}
Java
JAVA
import java.util.Arrays;

class Solution {
    public int minReplacements(int[] arr) {
        int n = arr.length;
        int[] s = Arrays.stream(arr).distinct().sorted().toArray(); // unique, sorted
        int m = s.length, res = n, j = 0;
        for (int i = 0; i < m; i++) {
            while (j < m && (long) s[j] <= (long) s[i] + n - 1) j++;
            res = Math.min(res, n - (j - i));           // keep window, replace the rest
        }
        return res;
    }
}
C++
CPP
#include <vector>
#include <algorithm>
using namespace std;

class Solution {
public:
    int minReplacements(vector<int>& arr) {
        int n = (int)arr.size();
        vector<int> s(arr.begin(), arr.end());
        sort(s.begin(), s.end());
        s.erase(unique(s.begin(), s.end()), s.end());   // distinct sorted values
        int m = (int)s.size(), res = n, j = 0;
        for (int i = 0; i < m; ++i) {
            while (j < m && (long long)s[j] <= (long long)s[i] + n - 1) ++j;
            res = min(res, n - (j - i));                // replace everything not kept
        }
        return res;
    }
};
C
C
#include <stdlib.h>

static int intCmp(const void* a, const void* b) {
    int x = *(const int*)a, y = *(const int*)b;
    return (x > y) - (x < y);
}

int minReplacements(int* arr, int arrSize) {
    int n = arrSize;
    int* s = (int*)malloc(sizeof(int) * (size_t)n);
    for (int i = 0; i < n; ++i) s[i] = arr[i];
    qsort(s, n, sizeof(int), intCmp);

    /* deduplicate the sorted values in place */
    int m = 0;
    for (int i = 0; i < n; ++i) {
        if (m == 0 || s[i] != s[m - 1]) s[m++] = s[i];
    }

    int res = n, j = 0;
    for (int i = 0; i < m; ++i) {
        /* slide right end while it fits in the window [s[i], s[i] + n - 1] */
        while (j < m && (long long)s[j] <= (long long)s[i] + n - 1) ++j;
        int keep = j - i;              /* distinct values already inside the window */
        if (n - keep < res) res = n - keep;
    }
    free(s);
    return res;
}

#### Follow-up
questions
What if replacements were restricted to values within a given range, so some windows become infeasible?
Instead of the count of moves, return the actual set of consecutive integers the array should become.
How would you answer many queries, each asking the cost to make a subarray compact?
What changes if "compact" allowed the run to have a fixed step k > 1 (arithmetic progression) instead of consecutive integers?

