---
type: "practice"
difficulty: "Other"
---

# Distance to the Next Greater Value

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Monotonic Stack](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Monotonic_Stack.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given an array readings of integers. For each position i, you want to know how many positions you must move to the right before you encounter the first element that is strictly greater than readings[i].

Return an array answer of the same length where answer[i] is that distance. If no element strictly greater than readings[i] exists anywhere to the right of position i, set answer[i] = 0.

Input: an array readings of integers. Output: an array answer where answer[i] is the number of steps to the right to reach the first strictly larger element, or 0 if there is none.

#### Cons
traints:

1 <= readings.length <= 10^5
Each element of readings fits in a signed 32-bit integer.
The comparison is strict: an equal value does not count as "greater".

#### Example
s & test cases
TEXT
Input: readings = [40,50,60,20,10,30,70,40] -> Output: [1,1,4,2,1,1,0,0]
Why: Mixed case — e.g. from index 2 (value 60) the next strictly larger value is 70 at index 6, a distance of 4; the last two elements have nothing larger ahead, so they are 0.

Input: readings = [10,20,30,40] -> Output: [1,1,1,0]
Why: Strictly increasing, so every element's next larger value is its immediate neighbor except the last, which has none.

Input: readings = [9,8,7,6] -> Output: [0,0,0,0]
Why: Strictly decreasing; no element ever has a larger value to its right.

Input: readings = [5,5,5] -> Output: [0,0,0]
Why: All equal — since the comparison is strict, no element is greater than another, so every answer is 0.

Input: readings = [3] -> Output: [0]
Why: Single-element edge case; there is nothing to the right.

#### Approach

Monotonic stack. Keep a stack of indices whose corresponding values are strictly decreasing from bottom to top. Walk the array left to right; when the current element is strictly greater than the value at the index on top of the stack, that top index has just found its next greater element, so pop it and record the distance current_index - popped_index. Repeat the popping until the stack is empty or its top value is at least the current one, then push the current index. Any indices still on the stack at the end never found a larger element to their right, so their answers stay at the initialized 0. Because each index enters and leaves the stack at most once, the whole scan is linear — far better than the O(n^2) brute force of looking ahead from every position.

Complexity

Time: O(n) — each index is pushed onto the stack exactly once and popped at most once, so the total work is linear. Space: O(n) for the index stack (plus the O(n) output array).

Python
PYTHON
from typing import List

def next_greater_distances(readings: List[int]) -> List[int]:
    answer = [0] * len(readings)
    stack = []  # indices with strictly decreasing values

    for i, value in enumerate(readings):
        while stack and value > readings[stack[-1]]:
            idx = stack.pop()
            answer[idx] = i - idx
        stack.append(i)

    return answer
JavaScript
JAVASCRIPT
function nextGreaterDistances(readings) {

#### cons
t answer = new Array(readings.length).fill(0);

#### cons
t stack = []; // indices with strictly decreasing values

    for (let i = 0; i < readings.length; i++) {
        while (stack.length > 0 && readings[i] > readings[stack[stack.length - 1]]) {

#### cons
t idx = stack.pop();
            answer[idx] = i - idx;
        }
        stack.push(i);
    }

    return answer;
}
Java
JAVA
import java.util.ArrayDeque;
import java.util.Deque;

class Solution {
    public int[] nextGreaterDistances(int[] readings) {
        int n = readings.length;
        int[] answer = new int[n];
        Deque<Integer> stack = new ArrayDeque<>();  // indices with decreasing values

        for (int i = 0; i < n; ++i) {
            while (!stack.isEmpty() && readings[i] > readings[stack.peek()]) {
                int idx = stack.pop();
                answer[idx] = i - idx;
            }
            stack.push(i);
        }

        return answer;
    }
}
C++
CPP
#include <vector>
using namespace std;

vector<int> nextGreaterDistances(vector<int>& readings) {
    int n = (int)readings.size();
    vector<int> answer(n, 0);
    vector<int> stack;               // indices with strictly decreasing values

    for (int i = 0; i < n; ++i) {
        while (!stack.empty() && readings[i] > readings[stack.back()]) {
            int idx = stack.back();
            stack.pop_back();
            answer[idx] = i - idx;
        }
        stack.push_back(i);
    }

    return answer;
}
C
C
#include <stdlib.h>

int* nextGreaterDistances(int* readings, int readingsSize, int* returnSize) {
    int* answer = (int*)calloc(readingsSize, sizeof(int));
    int* stack = (int*)malloc(readingsSize * sizeof(int)); /* indices, decreasing values */
    int top = -1;

    for (int i = 0; i < readingsSize; ++i) {
        while (top >= 0 && readings[i] > readings[stack[top]]) {
            int idx = stack[top--];
            answer[idx] = i - idx;
        }
        stack[++top] = i;
    }

    free(stack);
    *returnSize = readingsSize;
    return answer;
}

#### Follow-up
questions
Return the actual next greater value at each position instead of the distance to it.
Find the distance to the next strictly smaller element instead of the next greater.
Solve it for a circular array where the search wraps around past the end.
Support a streaming variant where readings arrive one at a time and answers must be emitted as soon as they are resolved.

