---
type: "practice"
difficulty: "Other"
---

# Maximum Minimum-Weighted Subarray Score

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Monotonic Stack](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Monotonic_Stack.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

For any non-empty array of integers, define its score as the minimum element of that array multiplied by the sum of all its elements.

For example, the array [4, 3, 6] has minimum value 3 and sum 13, so its score is 3 * 13 = 39.

You are given an integer array values. Consider every non-empty contiguous subarray of values. Return the maximum score achievable over all such subarrays.

Because the answer can be very large, return it modulo 10^9 + 7. Note that the score must be maximized before applying the modulo. The test data is chosen so that the maximum score before the modulo fits in a 64-bit signed integer.

A subarray is a contiguous portion of the array.

Input

values: an array of positive integers.

Output

A single integer: the maximum subarray score, taken modulo 10^9 + 7.

#### Cons
traints

1 <= values.length <= 10^5
1 <= values[i] <= 10^7

#### Example
s & test cases
TEXT
Input: values = [2,4,3] -> Output: 21
Why: The subarray [4,3] has minimum 3 and sum 7, giving 3 * 7 = 21, which beats [4] (16) and the whole array [2,4,3] (2 * 9 = 18).

Input: values = [3,4,4,2,3] -> Output: 33
Why: The subarray [3,4,4] has minimum 3 and sum 11, giving 3 * 11 = 33, larger than [4,4] (4 * 8 = 32).

Input: values = [7] -> Output: 49
Why: A single-element array's only subarray is itself: minimum 7 times sum 7 equals 49 (minimum-size input).

Input: values = [5,5,5,5] -> Output: 100
Why: With all elements equal, the widest subarray wins: minimum 5 times sum 20 equals 100 (duplicates handled correctly).

Input: values = [10,1,10] -> Output: 100
Why: A single large element [10] scores 10 * 10 = 100; widening to include the 1 drags the minimum to 1, so the narrow subarray is best.

#### Approach

The optimal score is achieved by treating each element as the minimum of some maximal subarray. For a fixed element values[j] acting as the minimum, the best subarray is the widest one in which it stays the minimum: it extends left until a strictly smaller element and right until a smaller-or-equal element. Multiplying values[j] by the sum of that widest window can only increase the score compared with any narrower window (all elements are positive), so this window is the candidate for that element. To find these boundaries in linear time, sweep left to right with a monotonic (non-decreasing) stack of indices. When the incoming element is not larger than the stack top, the top's right boundary is here, so pop it; the new stack top is its left boundary. Precomputed prefix sums turn each window's sum into an O(1) subtraction. Track the maximum product across all pops (using 64-bit / BigInt arithmetic to avoid overflow), and apply the modulo only at the very end. Pattern: monotonic stack for previous/next smaller element combined with prefix sums.

Complexity

Time: O(n) — each index is pushed onto and popped from the monotonic stack exactly once, and prefix sums are built in one pass. Space: O(n) for the prefix-sum array and the stack.

Python
PYTHON
def max_subarray_score(values):
    MOD = 1_000_000_007
    n = len(values)
    prefix = [0] * (n + 1)
    for i in range(n):
        prefix[i + 1] = prefix[i] + values[i]

    stack = []
    best = 0
    for i in range(n + 1):
        # i == n is a sentinel smaller than every element, flushing the stack
        while stack and (i == n or values[stack[-1]] >= values[i]):
            j = stack.pop()
            left = stack[-1] if stack else -1
            total = prefix[i] - prefix[left + 1]   # values[j] is the min of this range
            best = max(best, values[j] * total)
        if i < n:
            stack.append(i)
    return best % MOD
JavaScript
JAVASCRIPT
function maxSubarrayScore(values) {

#### cons
t MOD = 1000000007n;

#### cons
t n = values.length;

#### cons
t prefix = new Array(n + 1);
    prefix[0] = 0n;
    for (let i = 0; i < n; i++) prefix[i + 1] = prefix[i] + BigInt(values[i]);

#### cons
t stack = [];
    let best = 0n;
    for (let i = 0; i <= n; i++) {
        while (stack.length && (i === n || values[stack[stack.length - 1]] >= values[i])) {

#### cons
t j = stack.pop();

#### cons
t left = stack.length ? stack[stack.length - 1] : -1;

#### cons
t total = prefix[i] - prefix[left + 1];   // values[j] is the min of this range

#### cons
t product = BigInt(values[j]) * total;
            if (product > best) best = product;
        }
        if (i < n) stack.push(i);
    }
    return Number(best % MOD);
}
Java
JAVA
import java.util.ArrayDeque;
import java.util.Deque;

class Solution {
    public int maxSubarrayScore(int[] values) {
        final long MOD = 1_000_000_007L;
        int n = values.length;
        long[] prefix = new long[n + 1];
        for (int i = 0; i < n; i++) prefix[i + 1] = prefix[i] + values[i];

        Deque<Integer> stack = new ArrayDeque<>();
        long best = 0;
        for (int i = 0; i <= n; i++) {
            while (!stack.isEmpty() && (i == n || values[stack.peek()] >= values[i])) {
                int j = stack.pop();
                int left = stack.isEmpty() ? -1 : stack.peek();
                long sum = prefix[i] - prefix[left + 1];      // values[j] is the min of this range
                best = Math.max(best, (long) values[j] * sum);
            }
            if (i < n) stack.push(i);
        }
        return (int) (best % MOD);
    }
}
C++
CPP
#include <vector>
#include <algorithm>
using namespace std;

class Solution {
public:
    int maxSubarrayScore(vector<int>& values) {

#### cons
t long long MOD = 1000000007LL;
        int n = (int)values.size();
        vector<long long> prefix(n + 1, 0);
        for (int i = 0; i < n; i++) prefix[i + 1] = prefix[i] + values[i];

        vector<int> stk;
        long long best = 0;
        for (int i = 0; i <= n; i++) {
            while (!stk.empty() && (i == n || values[stk.back()] >= values[i])) {
                int j = stk.back(); stk.pop_back();
                int left = stk.empty() ? -1 : stk.back();
                long long sum = prefix[i] - prefix[left + 1];   // values[j] is the min of this range
                best = max(best, (long long)values[j] * sum);
            }
            if (i < n) stk.push_back(i);
        }
        return (int)(best % MOD);
    }
};
C
C
#include <stdlib.h>

long long maxSubarrayScore(int* values, int valuesSize) {

#### cons
t long long MOD = 1000000007LL;
    int n = valuesSize;
    long long* prefix = (long long*)malloc((n + 1) * sizeof(long long));
    prefix[0] = 0;
    for (int i = 0; i < n; i++) prefix[i + 1] = prefix[i] + values[i];

    int* stack = (int*)malloc(n * sizeof(int));
    int top = -1;
    long long best = 0;

    for (int i = 0; i <= n; i++) {
        /* i == n acts as a sentinel smaller than everything, flushing the stack */
        while (top >= 0 && (i == n || values[stack[top]] >= values[i])) {
            int j = stack[top--];
            int left = (top >= 0) ? stack[top] : -1;
            long long sum = prefix[i] - prefix[left + 1];   /* range (left, i-1] where values[j] is min */
            long long product = (long long)values[j] * sum;
            if (product > best) best = product;
        }
        if (i < n) stack[++top] = i;
    }
    free(prefix);
    free(stack);
    return best % MOD;
}

#### Follow-up
questions
Return the actual start and end indices of the subarray that achieves the maximum score, not just the value.
Why does building the widest window where an element is the minimum guarantee the optimal score for that element? Prove it depends on all values being positive.
Explain why the monotonic-stack sweep is O(n) even though it contains a nested while loop.
How would the approach change if the array could contain negative numbers or zeros?

