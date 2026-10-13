---
type: "practice"
difficulty: "Other"
---

# Detecting a Run of Three Odd Readings

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Array](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Array.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

A device logs a stream of non-negative integer readings into an array readings. You need to decide whether the log ever contains three odd numbers in a row — that is, three consecutive positions in the array whose values are all odd.

Return true if such a triple of adjacent odd readings exists anywhere in the array, and false otherwise.

A number is odd when it is not divisible by 2. The three odd values must occupy three consecutive indices i, i + 1, and i + 2; they do not need to be equal to one another.

#### Cons
traints:

1 <= readings.length <= 1000
1 <= readings[i] <= 1000

#### Example
s & test cases
TEXT
Input: readings = [2,6,4,1] -> Output: false
Why: only one odd value (1) appears and it is isolated, so no run of three odds exists.

Input: readings = [1,2,34,3,4,5,7,23,12] -> Output: true
Why: the values 5, 7, 23 sit at consecutive indices and are all odd.

Input: readings = [3,5,7] -> Output: true
Why: the whole array is exactly three consecutive odd numbers.

Input: readings = [1,1] -> Output: false
Why: there are only two elements, so a run of three positions is impossible.

Input: readings = [9] -> Output: false
Why: a single element cannot form three consecutive odds.

Input: readings = [1,3,2,5,7,9,4] -> Output: true
Why: 5, 7, 9 are three consecutive odds even though an earlier pair 1,3 was broken by the even value 2.

#### Approach

Sweep the array once while maintaining a counter of how many odd values have appeared consecutively up to the current index. Each time an odd value is seen, increment the counter; the moment it reaches 3 you have found three adjacent odds and can return true immediately. Whenever an even value is seen, the run is broken, so reset the counter to 0. If the scan finishes without the counter ever reaching 3, no such triple exists. This is the classic running-streak / sliding-window-of-fixed-length pattern collapsed into a single counter.

Complexity

Time: O(n) — a single pass over the array. Space: O(1) — only a running counter is kept regardless of input size.

Python
PYTHON
from typing import List

class Solution:
    def threeConsecutiveOdds(self, readings: List[int]) -> bool:
        run = 0
        for x in readings:
            if x % 2 != 0:
                run += 1
                if run == 3:
                    return True
            else:
                run = 0
        return False
JavaScript
JAVASCRIPT
function threeConsecutiveOdds(readings) {
    let run = 0;
    for (const x of readings) {
        if (x % 2 !== 0) {
            if (++run === 3) return true;
        } else {
            run = 0;
        }
    }
    return false;
}
Java
JAVA
class Solution {
    public boolean threeConsecutiveOdds(int[] readings) {
        int run = 0;
        for (int x : readings) {
            if (x % 2 != 0) {
                if (++run == 3) return true;
            } else {
                run = 0;
            }
        }
        return false;
    }
}
C++
CPP
#include <vector>
using namespace std;

class Solution {
public:
    bool threeConsecutiveOdds(vector<int>& readings) {
        int run = 0;
        for (int x : readings) {
            if (x % 2 != 0) {
                if (++run == 3) return true;
            } else {
                run = 0;
            }
        }
        return false;
    }
};
C
C
#include <stdbool.h>

bool threeConsecutiveOdds(int* readings, int readingsSize) {
    int run = 0;
    for (int i = 0; i < readingsSize; i++) {
        if (readings[i] % 2 != 0) {
            run++;
            if (run == 3) return true;
        } else {
            run = 0;
        }
    }
    return false;
}

#### Follow-up
questions
Generalize to detecting k consecutive odd numbers instead of exactly three.
Return the starting index of the first qualifying run, or all such starting indices, instead of a boolean.
Handle a streaming input where readings arrive one at a time and you must answer as soon as the condition is met without storing the whole array.
What if you needed three consecutive numbers of the same parity (all odd or all even)? How does the counter logic change?

