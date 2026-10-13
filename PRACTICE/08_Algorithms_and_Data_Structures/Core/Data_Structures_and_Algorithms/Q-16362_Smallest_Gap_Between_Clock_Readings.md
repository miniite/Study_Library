---
type: "practice"
difficulty: "Other"
---

# Smallest Gap Between Clock Readings

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ String](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/String.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given a list of clock readings taken during a single day, each written as a string in 24-hour "HH:MM" format (from "00:00" up to "23:59"). Think of them as timestamps stamped on a circular 24-hour dial.

Return the smallest number of minutes separating any two readings in the list. Because the dial is circular, the distance between two readings is measured the shorter way around the clock: for example "23:59" and "00:00" are only 1 minute apart, not 1439. If two readings are identical, their gap is 0.

Input: an array of strings, each a valid "HH:MM" time. Readings may repeat, and the same minute may appear more than once.

Output: a single integer — the minimum minute gap between the two closest readings, measured around the circular clock.

#### Cons
traints:

2 <= number of readings <= 2 * 10^4.
Every reading is a valid 24-hour time in "HH:MM" form, i.e. hours 00–23 and minutes 00–59.
Readings are not necessarily sorted and may contain duplicates.

Note: there are only 1440 distinct minutes in a day, so once the list holds more than 1440 readings two of them must share a minute and the answer is 0.

#### Example
s & test cases
TEXT
Input: readings = ["22:30", "06:15", "22:31"] -> Output: 1
Why: 22:30 and 22:31 are one minute apart; every other pair is farther.

Input: readings = ["12:00", "12:00"] -> Output: 0
Why: two identical readings map to the same minute, so the smallest gap is 0.

Input: readings = ["00:05", "23:55"] -> Output: 10
Why: going forward is 1430 minutes, but wrapping backward past midnight is only 1440 - 1430 = 10.

Input: readings = ["01:00", "13:00"] -> Output: 720
Why: the two readings are exactly 12 hours apart either way around the dial, so the gap is 720.

Input: readings = ["23:59", "00:00", "12:00"] -> Output: 1
Why: 23:59 and 00:00 straddle midnight and are 1 minute apart, beating the 12-hour gaps to noon.

Input: readings = ["00:00", "04:00", "22:00"] -> Output: 120
Why: the closest pair is 22:00 and 00:00, which wraps across midnight for a 120-minute gap.

#### Approach

Bucketing plus the pigeonhole principle. Map each "HH:MM" reading to its minute-of-day in [0, 1439] and mark that slot in a fixed 1440-element boolean array. If a slot is already occupied, two readings coincide and the answer is immediately 0 (and since only 1440 minutes exist, any list longer than 1440 is guaranteed such a collision). Marking slots also sorts the distinct minutes for free: walk the occupied slots in increasing order and track the smallest difference between consecutive marks. Finally, close the loop — the clock is circular — by comparing against the wrap gap 1440 - lastMark + firstMark, which measures the distance across midnight from the latest reading back to the earliest. An O(n log n) alternative sorts the minute values and does the same adjacent scan; the bucket version trades a constant amount of space for linear time.

Complexity

Time: O(n + 1440) = O(n) — one pass parses and buckets the readings, then a fixed 1440-slot scan finds the closest pair. Space: O(1440) = O(1) — the bucket size is a constant independent of the input.

Python
PYTHON
from typing import List

class Solution:
    def smallestGap(self, readings: List[str]) -> int:
        seen = [False] * 1440
        for r in readings:
            minute = int(r[:2]) * 60 + int(r[3:5])
            if seen[minute]:            # duplicate minute -> gap 0
                return 0
            seen[minute] = True

        marks = [i for i in range(1440) if seen[i]]
        # circular wrap between last and first mark, across midnight
        ans = 1440 - marks[-1] + marks[0]
        for i in range(1, len(marks)):
            ans = min(ans, marks[i] - marks[i - 1])
        return ans
JavaScript
JAVASCRIPT
function smallestGap(readings) {

#### cons
t seen = new Array(1440).fill(false);
    for (const r of readings) {

#### cons
t minute = parseInt(r.slice(0, 2), 10) * 60 + parseInt(r.slice(3, 5), 10);
        if (seen[minute]) return 0;     // duplicate minute -> gap 0
        seen[minute] = true;
    }

#### cons
t marks = [];
    for (let i = 0; i < 1440; i++) if (seen[i]) marks.push(i);

    // circular wrap between last and first mark, across midnight
    let ans = 1440 - marks[marks.length - 1] + marks[0];
    for (let i = 1; i < marks.length; i++) {
        ans = Math.min(ans, marks[i] - marks[i - 1]);
    }
    return ans;
}
Java
JAVA
import java.util.List;

class Solution {
    public int smallestGap(List<String> readings) {
        boolean[] seen = new boolean[1440];
        for (String r : readings) {
            int minute = Integer.parseInt(r.substring(0, 2)) * 60
                       + Integer.parseInt(r.substring(3, 5));
            if (seen[minute])          // duplicate minute -> gap 0
                return 0;
            seen[minute] = true;
        }

        int prev = -1, first = -1, ans = 1440;
        for (int i = 0; i < 1440; i++) {
            if (!seen[i]) continue;
            if (first == -1) first = i;
            else ans = Math.min(ans, i - prev);
            prev = i;
        }
        // circular wrap between last and first mark, across midnight
        return Math.min(ans, 1440 - prev + first);
    }
}
C++
CPP
#include <string>
#include <vector>
#include <algorithm>
using namespace std;

class Solution {
public:
    int smallestGap(vector<string>& readings) {
        vector<bool> seen(1440, false);
        for (const string& r : readings) {
            int minute = stoi(r.substr(0, 2)) * 60 + stoi(r.substr(3, 2));
            if (seen[minute])          // duplicate minute -> gap 0
                return 0;
            seen[minute] = true;
        }

        int prev = -1, first = -1, ans = 1440;
        for (int i = 0; i < 1440; ++i) {
            if (!seen[i]) continue;
            if (first == -1) first = i;
            else ans = min(ans, i - prev);
            prev = i;
        }
        // circular wrap between last and first mark, across midnight
        return min(ans, 1440 - prev + first);
    }
};
C
C
#include <stdlib.h>
#include <stdbool.h>

int smallestGap(char** readings, int readingsSize) {
    bool seen[1440] = { false };
    for (int k = 0; k < readingsSize; ++k) {

#### cons
t char* r = readings[k];
        int minute = ((r[0] - '0') * 10 + (r[1] - '0')) * 60
                   +  (r[3] - '0') * 10 + (r[4] - '0');
        if (seen[minute])          /* duplicate minute -> gap 0 */
            return 0;
        seen[minute] = true;
    }

    int prev = -1, first = -1, ans = 1440;
    for (int i = 0; i < 1440; ++i) {
        if (!seen[i]) continue;
        if (first == -1) first = i;
        else if (i - prev < ans) ans = i - prev;
        prev = i;
    }
    /* circular wrap between last and first mark, across midnight */
    if (1440 - prev + first < ans) ans = 1440 - prev + first;
    return ans;
}

#### Follow-up
questions
What if readings could span multiple days with a date attached — how does the circular trick change?
Return the actual pair of readings that achieves the minimum gap, not just the number of minutes.
If readings arrive as a stream, maintain the smallest gap online as each new reading comes in.
Generalize to seconds precision ("HH:MM:SS") — what bucket size and constant factors change?

