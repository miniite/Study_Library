---
type: "practice"
difficulty: "Other"
---

# Positions With Enough Smaller Elements on Both Sides

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Segment Tree](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Segment_Tree.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given a 0-indexed integer array arr and a positive integer k.

Call an index i balanced when it satisfies both of these conditions:

There are at least k distinct indices p with p < i and arr[p] < arr[i] (at least k strictly smaller values lie to its left).
There are at least k distinct indices q with q > i and arr[q] < arr[i] (at least k strictly smaller values lie to its right).

Return the number of balanced indices in arr.

Note that only strictly smaller values count; equal values on either side do not contribute. An index near the ends of the array can still be balanced as long as enough strictly smaller elements exist on both sides.

Input

arr: a 0-indexed array of integers.
k: a positive integer, the required count of strictly smaller elements on each side.

Output

The number of balanced indices.

#### Cons
traints

1 <= arr.length <= 10^5
1 <= k <= arr.length
1 <= arr[i] <= 10^9

#### Example
s & test cases
TEXT
- Input: `arr = [5,7,9,4,7], k = 1` -> Output: `2`
  Why: index `1` (value `7`) has smaller value `5` on the left and `4` on the right; index `2` (value `9`) has `{5,7}` left and `{4,7}` right. Both meet `k = 1` (a normal case).
- Input: `arr = [1,5,2,5,3,5,1], k = 2` -> Output: `1`
  Why: only index `3` (value `5`) has at least two strictly smaller values on each side (`{1,2}` left, `{3,1}` right); the other positions fall short (duplicates present).
- Input: `arr = [4,4,4], k = 1` -> Output: `0`
  Why: every value equals its neighbors, so no index has any strictly smaller element on either side (all-equal edge).
- Input: `arr = [1,2,3,4,5], k = 2` -> Output: `0`
  Why: the array strictly increases, so no index has any strictly smaller element to its right (monotonic edge).
- Input: `arr = [6,3,5], k = 2` -> Output: `0`
  Why: the array is too short for any index to have two strictly smaller elements on both sides at once (small-array edge).

#### Approach

For each index we need two order-statistics: left[i] = how many earlier elements are strictly smaller, and right[i] = how many later elements are strictly smaller; index i is balanced iff left[i] >= k and right[i] >= k. Naively comparing all pairs is O(n^2); instead use a Fenwick (Binary Indexed) tree as a frequency table over values. First compress the values to ranks 1..m (so the tree stays O(n) regardless of value magnitude). Sweep left to right: before inserting arr[i], query the prefix count of ranks < rank(arr[i]) already inserted — that is exactly the number of strictly smaller elements seen so far on the left; then insert arr[i]. Reset the tree and sweep right to left the same way to get right[i]. Finally count indices where both counts reach k. The pattern is "count of smaller elements to one side," a classic Fenwick / order-statistics technique (a merge-sort inversion-style count is an equivalent alternative). Note: a shortcut that only measures the length of consecutive increasing/decreasing runs does NOT solve this — smaller elements may be non-adjacent — so a true rank counter is required.

Complexity

Time: O(n log n) — coordinate-compressing the values costs one O(n log n) sort, and each of the two directional sweeps performs n Fenwick-tree queries/updates at O(log n) each. Space: O(n) — the compressed value list, the left/right counts, and the Fenwick array are all O(n). Justification: with n up to 10^5, n log n is a few million operations.

Python
PYTHON
from bisect import bisect_left

class Solution:
    def countBalancedIndices(self, arr, k):
        n = len(arr)
        uniq = sorted(set(arr))
        m = len(uniq)
        bit = [0] * (m + 1)

        def update(i):
            while i <= m:
                bit[i] += 1
                i += i & (-i)

        def query(i):
            s = 0
            while i > 0:
                s += bit[i]
                i -= i & (-i)
            return s

        left = [0] * n
        for i in range(n):                          # smaller values already on the left
            r = bisect_left(uniq, arr[i]) + 1       # 1-based rank
            left[i] = query(r - 1)
            update(r)

        bit[:] = [0] * (m + 1)                      # reset the same Fenwick array
        right = [0] * n
        for i in range(n - 1, -1, -1):              # smaller values already on the right
            r = bisect_left(uniq, arr[i]) + 1
            right[i] = query(r - 1)
            update(r)

        return sum(1 for i in range(n) if left[i] >= k and right[i] >= k)
JavaScript
JAVASCRIPT
function countBalancedIndices(arr, k) {

#### cons
t n = arr.length;

#### cons
t uniq = Array.from(new Set(arr)).sort((a, b) => a - b);

#### cons
t m = uniq.length;

#### cons
t bit = new Array(m + 1).fill(0);

#### cons
t update = (i) => { for (; i <= m; i += i & (-i)) bit[i]++; };

#### cons
t query = (i) => { let s = 0; for (; i > 0; i -= i & (-i)) s += bit[i]; return s; };

#### cons
t rankOf = (x) => {                         // 1-based rank via binary search
        let lo = 0, hi = m - 1, pos = 0;
        while (lo <= hi) {

#### cons
t mid = (lo + hi) >> 1;
            if (uniq[mid] < x) lo = mid + 1;
            else { pos = mid; hi = mid - 1; }
        }
        return pos + 1;
    };

#### cons
t left = new Array(n).fill(0);
    for (let i = 0; i < n; i++) {                    // smaller values already on the left

#### cons
t r = rankOf(arr[i]);
        left[i] = query(r - 1);
        update(r);
    }
    bit.fill(0);

#### cons
t right = new Array(n).fill(0);
    for (let i = n - 1; i >= 0; i--) {               // smaller values already on the right

#### cons
t r = rankOf(arr[i]);
        right[i] = query(r - 1);
        update(r);
    }

    let count = 0;
    for (let i = 0; i < n; i++)
        if (left[i] >= k && right[i] >= k) count++;
    return count;
}
Java
JAVA
import java.util.Arrays;

class Solution {
  private int[] bit;
  private int m;

  private void update(int i) { for (; i <= m; i += i & (-i)) bit[i]++; }
  private int query(int i) { int s = 0; for (; i > 0; i -= i & (-i)) s += bit[i]; return s; }

  private int rankOf(int[] uniq, int len, int x) {   // 1-based rank via binary search
    int lo = 0, hi = len - 1, pos = 0;
    while (lo <= hi) {
      int mid = (lo + hi) >>> 1;
      if (uniq[mid] < x) lo = mid + 1;
      else { pos = mid; hi = mid - 1; }
    }
    return pos + 1;
  }

  public int countBalancedIndices(int[] arr, int k) {
    int n = arr.length;
    int[] uniq = arr.clone();
    Arrays.sort(uniq);
    int u = 0;
    for (int i = 0; i < n; ++i)
      if (i == 0 || uniq[i] != uniq[i - 1]) uniq[u++] = uniq[i];
    m = u;

    int[] left = new int[n], right = new int[n];
    bit = new int[m + 1];
    for (int i = 0; i < n; ++i) {                 // smaller values already on the left
      int r = rankOf(uniq, u, arr[i]);
      left[i] = query(r - 1);
      update(r);
    }
    Arrays.fill(bit, 0);
    for (int i = n - 1; i >= 0; --i) {            // smaller values already on the right
      int r = rankOf(uniq, u, arr[i]);
      right[i] = query(r - 1);
      update(r);
    }

    int count = 0;
    for (int i = 0; i < n; ++i)
      if (left[i] >= k && right[i] >= k) ++count;
    return count;
  }
}
C++
CPP
class Solution {
  vector<int> bit;
  int m = 0;
  void update(int i) { for (; i <= m; i += i & (-i)) bit[i]++; }
  int query(int i) { int s = 0; for (; i > 0; i -= i & (-i)) s += bit[i]; return s; }

 public:
  int countBalancedIndices(vector<int>& arr, int k) {
    int n = (int)arr.size();
    vector<int> uniq(arr);
    sort(uniq.begin(), uniq.end());
    uniq.erase(unique(uniq.begin(), uniq.end()), uniq.end());
    m = (int)uniq.size();
    auto rankOf = [&] (int x) {
      return (int)(lower_bound(uniq.begin(), uniq.end(), x) - uniq.begin()) + 1;
    };

    vector<int> left(n), right(n);
    bit.assign(m + 1, 0);
    for (int i = 0; i < n; ++i) {                 // smaller values already on the left
      int r = rankOf(arr[i]);
      left[i] = query(r - 1);
      update(r);
    }
    fill(bit.begin(), bit.end(), 0);
    for (int i = n - 1; i >= 0; --i) {            // smaller values already on the right
      int r = rankOf(arr[i]);
      right[i] = query(r - 1);
      update(r);
    }

    int count = 0;
    for (int i = 0; i < n; ++i)
      if (left[i] >= k && right[i] >= k) ++count;
    return count;
  }
};
C
C
#include <stdlib.h>
#include <string.h>

static int cmpInt(const void* a, const void* b) {
    int x = *(const int*)a, y = *(const int*)b;
    return (x > y) - (x < y);
}

/* Fenwick (Binary Indexed) tree over compressed ranks 1..m */
static void bitUpdate(int* bit, int m, int i) {
    for (; i <= m; i += i & (-i)) bit[i]++;
}
static int bitQuery(int* bit, int i) {
    int s = 0;
    for (; i > 0; i -= i & (-i)) s += bit[i];
    return s;
}
/* 1-based rank of x among the unique sorted values */
static int rankOf(int* uniq, int m, int x) {
    int lo = 0, hi = m - 1, pos = 0;
    while (lo <= hi) {
        int mid = (lo + hi) / 2;
        if (uniq[mid] < x) lo = mid + 1;
        else { pos = mid; hi = mid - 1; }
    }
    return pos + 1;
}

int countBalancedIndices(int* arr, int n, int k) {
    if (n == 0) return 0;
    int* uniq = (int*)malloc(n * sizeof(int));
    memcpy(uniq, arr, n * sizeof(int));
    qsort(uniq, n, sizeof(int), cmpInt);
    int m = 0;
    for (int i = 0; i < n; ++i)
        if (i == 0 || uniq[i] != uniq[i - 1]) uniq[m++] = uniq[i];

    int* left = (int*)calloc(n, sizeof(int));
    int* right = (int*)calloc(n, sizeof(int));
    int* bit = (int*)calloc(m + 1, sizeof(int));

    for (int i = 0; i < n; ++i) {                 /* smaller values already on the left */
        int r = rankOf(uniq, m, arr[i]);
        left[i] = bitQuery(bit, r - 1);
        bitUpdate(bit, m, r);
    }
    memset(bit, 0, (m + 1) * sizeof(int));
    for (int i = n - 1; i >= 0; --i) {            /* smaller values already on the right */
        int r = rankOf(uniq, m, arr[i]);
        right[i] = bitQuery(bit, r - 1);
        bitUpdate(bit, m, r);
    }

    int count = 0;
    for (int i = 0; i < n; ++i)
        if (left[i] >= k && right[i] >= k) ++count;

    free(uniq); free(left); free(right); free(bit);
    return count;
}

#### Follow-up
questions
How would you adapt the solution to count indices with at least k strictly larger elements on both sides instead of smaller?
What if the required counts differ for the two sides (say kLeft on the left and kRight on the right)?
Solve the same "count of smaller elements on each side" using a merge-sort inversion count rather than a Fenwick tree.
If elements arrived as a stream and queries came online, which structure (e.g., an order-statistics balanced BST) would you use?

