---
type: "practice"
difficulty: "Other"
---

# Longest Contiguous Block of Servers Within a Power Budget

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Sliding Window](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Sliding_Window.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You manage a data center that has n servers standing in a single row, numbered from 0 to n - 1. You are given two integer arrays, spinUpTime and hourlyRate, both of length n. Powering on server i requires spinUpTime[i] units of energy to start it, and once running it draws hourlyRate[i] units per hour.

You may only switch on a contiguous block of servers (a run of consecutive positions in the row). If a chosen block contains k servers, its total power expense is defined as:

CODE
max(spinUpTime over the block) + k * sum(hourlyRate over the block)

That is, the single largest spin-up cost in the block, plus k times the sum of the hourly rates of every server in the block.

You are given an integer budget. Return the maximum number of consecutive servers you can power on so that the total power expense of that block does not exceed budget. If no single server can be powered on within the budget, return 0.

#### Cons
traints:

spinUpTime.length == hourlyRate.length == n
1 <= n <= 5 * 10^4
1 <= spinUpTime[i], hourlyRate[i] <= 10^5
1 <= budget <= 10^15

Note that because budget and the intermediate expense can be as large as roughly 2.5 * 10^14, you must use 64-bit arithmetic to avoid overflow.

#### Example
s & test cases
TEXT
Input: spinUpTime = [4,7,2,4,5], hourlyRate = [3,2,4,5,6], budget = 40 -> Output: 3
Why: The block [0,1,2] costs max(4,7,2) + 3*(3+2+4) = 7 + 27 = 34 <= 40, and [1,2,3] costs 7 + 3*11 = 40. No length-4 block fits, so the answer is 3.

Input: spinUpTime = [12,13,20], hourlyRate = [11,9,8], budget = 20 -> Output: 0
Why: Every single server already exceeds the budget (12+11=23, 13+9=22, 20+8=28), so no block can be powered on.

Input: spinUpTime = [5], hourlyRate = [5], budget = 10 -> Output: 1
Why: The lone server costs 5 + 1*5 = 10, exactly meeting the budget (edge: expense equal to budget is allowed).

Input: spinUpTime = [5,5,5], hourlyRate = [1,1,1], budget = 20 -> Output: 3
Why: Equal spin-up values stress the max-tracking; the full block costs 5 + 3*3 = 14 <= 20.

Input: spinUpTime = [1,2,3], hourlyRate = [1,1,1], budget = 1000000000000000 -> Output: 3
Why: A huge budget lets every server run, exercising 64-bit budget handling.

Input: spinUpTime = [100000,1], hourlyRate = [1,1], budget = 100002 -> Output: 1
Why: The pair costs 100000 + 2*2 = 100004 > budget, but each single server fits, so the longest valid block is length 1.

#### Approach

This is a sliding-window problem where the window's "cost" is monotonic in a useful way: for a fixed right end, extending the window only raises (or keeps) the running-rate sum and the block length, so once a window is too expensive it stays too expensive until the left end advances. Slide a window with two pointers. Maintain the sum of hourlyRate inside the window incrementally, and maintain the maximum spinUpTime inside the window using a monotonic (decreasing) deque of indices so the front always gives the current window max in O(1) amortized time. For each new right end, add its rate, push it onto the deque (popping smaller/equal tails first), then shrink from the left while windowMax + rateSum * windowLength exceeds the budget, dropping the leftmost index from the deque when it leaves the window. Track the largest valid window length seen. The pattern is "sliding window + monotonic deque for range maximum."

Complexity

Time: O(n). Each server index enters and leaves the monotonic deque at most once, and both the left and right pointers only move forward, so total work is linear. Space: O(n) for the deque of indices (O(1) auxiliary beyond that).

Python
PYTHON
from collections import deque

def longest_server_block(spin_up_time, hourly_rate, budget):
    n = len(spin_up_time)
    dq = deque()          # indices with strictly decreasing spin_up_time
    running = 0           # sum of hourly_rate in the current window
    left = 0
    ans = 0
    for right in range(n):
        running += hourly_rate[right]
        while dq and spin_up_time[dq[-1]] <= spin_up_time[right]:
            dq.pop()
        dq.append(right)
        while left <= right and \
                spin_up_time[dq[0]] + running * (right - left + 1) > budget:
            if dq[0] == left:
                dq.popleft()
            running -= hourly_rate[left]
            left += 1
        ans = max(ans, right - left + 1)
    return ans
JavaScript
JAVASCRIPT
function longestServerBlock(spinUpTime, hourlyRate, budget) {

#### cons
t n = spinUpTime.length;

#### cons
t dq = [];        // indices with decreasing spinUpTime; dq[head] holds the window max
    let head = 0;
    let running = 0;      // products stay below 2^53, so plain numbers are safe here
    let left = 0, ans = 0;
    for (let right = 0; right < n; right++) {
        running += hourlyRate[right];
        while (dq.length > head && spinUpTime[dq[dq.length - 1]] <= spinUpTime[right]) dq.pop();
        dq.push(right);
        while (left <= right &&
               spinUpTime[dq[head]] + running * (right - left + 1) > budget) {
            if (dq[head] === left) head++;
            running -= hourlyRate[left];
            left++;
        }
        ans = Math.max(ans, right - left + 1);
    }
    return ans;
}
Java
JAVA
import java.util.ArrayDeque;
import java.util.Deque;

class Solution {
    public int longestServerBlock(int[] spinUpTime, int[] hourlyRate, long budget) {
        int n = spinUpTime.length;
        Deque<Integer> dq = new ArrayDeque<>(); // indices, spinUpTime decreasing
        long sum = 0;
        int left = 0, ans = 0;
        for (int right = 0; right < n; right++) {
            sum += hourlyRate[right];
            while (!dq.isEmpty() && spinUpTime[dq.peekLast()] <= spinUpTime[right]) dq.pollLast();
            dq.offerLast(right);
            while (left <= right &&
                   (long) spinUpTime[dq.peekFirst()] + sum * (long) (right - left + 1) > budget) {
                if (dq.peekFirst() == left) dq.pollFirst();
                sum -= hourlyRate[left];
                left++;
            }
            ans = Math.max(ans, right - left + 1);
        }
        return ans;
    }
}
C++
CPP
#include <vector>
#include <deque>
#include <algorithm>
using namespace std;

int longestServerBlock(vector<int>& spinUpTime, vector<int>& hourlyRate, long long budget) {
    int n = spinUpTime.size();
    deque<int> dq;              // indices, spinUpTime strictly decreasing front -> back
    long long sum = 0;
    int left = 0, ans = 0;
    for (int right = 0; right < n; right++) {
        sum += hourlyRate[right];
        while (!dq.empty() && spinUpTime[dq.back()] <= spinUpTime[right]) dq.pop_back();
        dq.push_back(right);
        while (left <= right &&
               (long long)spinUpTime[dq.front()] + sum * (long long)(right - left + 1) > budget) {
            if (dq.front() == left) dq.pop_front();
            sum -= hourlyRate[left];
            left++;
        }
        ans = max(ans, right - left + 1);
    }
    return ans;
}
C
C
#include <stdlib.h>

int longestServerBlock(int* spinUpTime, int spinUpTimeSize,
                       int* hourlyRate, int hourlyRateSize, long long budget) {
    int n = spinUpTimeSize;
    if (n == 0) return 0;
    int* dq = (int*)malloc(sizeof(int) * n); /* monotonic decreasing deque of indices */
    int head = 0, tail = 0;                  /* valid range [head, tail) */
    long long sum = 0;
    int left = 0, ans = 0;
    for (int right = 0; right < n; right++) {
        sum += hourlyRate[right];
        while (tail > head && spinUpTime[dq[tail - 1]] <= spinUpTime[right]) tail--;
        dq[tail++] = right;
        while (left <= right &&
               (long long)spinUpTime[dq[head]] + sum * (long long)(right - left + 1) > budget) {
            if (dq[head] == left) head++;
            sum -= hourlyRate[left];
            left++;
        }
        if (right - left + 1 > ans) ans = right - left + 1;
    }
    free(dq);
    return ans;
}

#### Follow-up
questions
What if the block need not be contiguous — you may pick any k servers to maximize k within budget? (Different structure: sort by spin-up time, then use a heap over rates.)
Can you solve it by binary searching on the answer length and using a sparse table for range maximum, and how does that complexity compare?
What if you also must return the actual starting index of one longest valid block, not just its length?
How would the algorithm change if the formula used max(spinUpTime) * k + sum(hourlyRate) instead?

