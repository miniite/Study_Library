---
type: "practice"
difficulty: "Other"
---

# Total On-Site Minutes per Person per Day

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Database](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Database.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

A building's access system records one row every time a person taps in and later taps out. Each record has four fields:

personId — an integer identifying the person,
logDate — the calendar day the session happened (a date string like "2023-03-05"),
tapIn — the minute of the day the person entered (an integer),
tapOut — the minute of the day the person left (an integer).

Within a single day a person may enter and leave more than once, producing several records for the same (logDate, personId) pair. The time spent on-site for one record is tapOut - tapIn.

Compute, for every (logDate, personId) combination that appears in the data, the total minutes that person spent on-site that day (summed across all of their sessions that day).

Input

A collection of records, each [personId, logDate, tapIn, tapOut].

Output

A collection of rows [logDate, personId, totalMinutes], one per distinct (logDate, personId) pair, where totalMinutes is the sum of tapOut - tapIn over that pair's records. The rows may be returned in any order.

#### Cons
traints

0 <= number of records <= 10^5
1 <= tapIn < tapOut <= 1440 (minutes within a single day)
For any given day, a person's sessions never overlap in time.
personId fits in a 32-bit integer.

#### Example
s & test cases
TEXT
Input: logs = [[7,"2023-03-05",10,50],[7,"2023-03-05",90,130],[7,"2023-03-06",5,60],[4,"2023-03-05",20,35]] -> Output: [["2023-03-05",7,80],["2023-03-06",7,55],["2023-03-05",4,15]]
Why: Person 7 has two sessions on 2023-03-05, (50-10)+(130-90)=80; a separate day 2023-03-06 gives 55; person 4's single session gives 15. (Any row order is accepted.)

Input: logs = [[3,"2024-01-01",1,1440]] -> Output: [["2024-01-01",3,1439]]
Why: A single full-day session using the minimum tapIn and maximum tapOut, 1440-1=1439.

Input: logs = [[1,"2022-06-10",100,120],[2,"2022-06-10",100,140]] -> Output: [["2022-06-10",1,20],["2022-06-10",2,40]]
Why: Two different people on the same day stay in separate groups; grouping is by the pair, not by date alone.

Input: logs = [] -> Output: []
Why: With no records there are no groups, so the result is empty.

#### Approach

This is a group-and-aggregate problem — the imperative analogue of a SQL GROUP BY event_day, emp_id with SUM(out_time - in_time). The composite grouping key is the pair (logDate, personId); grouping by date alone or person alone would be wrong. Walk the records once, and for each one add its tapOut - tapIn duration into a hash-map bucket for its key. Because non-overlapping sessions are guaranteed, simple summation gives the correct total with no interval merging. Finally emit one row per bucket. Preserving first-seen order (LinkedHashMap / ordered list) is optional since any output order is accepted.

Complexity

Time: O(n) to scan the n records once, using a hash map keyed on (logDate, personId) to accumulate durations (string-key building is O(key length), effectively constant here). Space: O(g) where g is the number of distinct (day, person) groups (at most n). The equivalent SQL is a single GROUP BY aggregate, also linear in the number of rows.

Python
PYTHON
from collections import defaultdict

def total_minutes_per_day(logs):
    """logs: list of [personId, logDate, tapIn, tapOut].
    Returns list of [logDate, personId, totalMinutes]."""
    totals = defaultdict(int)
    order = []
    for person_id, log_date, tap_in, tap_out in logs:
        key = (log_date, person_id)
        if key not in totals:
            order.append(key)
        totals[key] += tap_out - tap_in
    return [[log_date, person_id, totals[(log_date, person_id)]]
            for (log_date, person_id) in order]

if __name__ == "__main__":
    logs = [[7, "2023-03-05", 10, 50], [7, "2023-03-05", 90, 130],
            [7, "2023-03-06", 5, 60],  [4, "2023-03-05", 20, 35]]
    for row in total_minutes_per_day(logs):
        print(*row)
JavaScript
JAVASCRIPT
// logs: array of [personId, logDate, tapIn, tapOut]
// returns array of [logDate, personId, totalMinutes]
function totalMinutesPerDay(logs) {

#### cons
t totals = new Map(); // key = logDate + "#" + personId
    for (const [personId, logDate, tapIn, tapOut] of logs) {

#### cons
t key = `${logDate}#${personId}`;
        totals.set(key, (totals.get(key) || 0) + (tapOut - tapIn));
    }

#### cons
t result = [];
    for (const [key, minutes] of totals) {

#### cons
t idx = key.indexOf('#');

#### cons
t logDate = key.slice(0, idx);

#### cons
t personId = Number(key.slice(idx + 1));
        result.push([logDate, personId, minutes]);
    }
    return result;
}

#### cons
t logs = [[7, "2023-03-05", 10, 50], [7, "2023-03-05", 90, 130],
              [7, "2023-03-06", 5, 60],  [4, "2023-03-05", 20, 35]];

#### cons
ole.log(totalMinutesPerDay(logs));
Java
JAVA
import java.util.*;

public class Solution {
    // Parallel arrays keep the input simple; groups by (logDate, personId).
    public static List<String[]> totalMinutesPerDay(int[] personId, String[] logDate,
                                                    int[] tapIn, int[] tapOut) {
        Map<String, Integer> totals = new LinkedHashMap<>(); // first-seen order
        for (int i = 0; i < personId.length; i++) {
            String key = logDate[i] + "#" + personId[i];
            totals.merge(key, tapOut[i] - tapIn[i], Integer::sum);
        }
        List<String[]> result = new ArrayList<>();
        for (Map.Entry<String, Integer> e : totals.entrySet()) {
            String[] parts = e.getKey().split("#");
            result.add(new String[]{parts[0], parts[1], String.valueOf(e.getValue())});
        }
        return result;
    }

    public static void main(String[] args) {
        int[] ids   = {7, 7, 7, 4};
        String[] dt = {"2023-03-05", "2023-03-05", "2023-03-06", "2023-03-05"};
        int[] in    = {10, 90, 5, 20};
        int[] out   = {50, 130, 60, 35};
        for (String[] row : totalMinutesPerDay(ids, dt, in, out))
            System.out.println(row[0] + " " + row[1] + " " + row[2]);
    }
}
C++
CPP
#include <bits/stdc++.h>
using namespace std;

struct Entry { int personId; string logDate; int tapIn; int tapOut; };

// Groups by (logDate, personId) and sums durations; O(n) with hashing.
vector<tuple<string,int,int>> totalMinutesPerDay(const vector<Entry>& logs) {
    unordered_map<string, int> totals;                 // key = logDate + "#" + personId
    vector<pair<string,int>> order;                    // preserve first-seen order
    for (const auto& e : logs) {
        string key = e.logDate + "#" + to_string(e.personId);
        if (!totals.count(key)) order.push_back({e.logDate, e.personId});
        totals[key] += e.tapOut - e.tapIn;
    }
    vector<tuple<string,int,int>> result;
    for (const auto& [date, id] : order)
        result.push_back({date, id, totals[date + "#" + to_string(id)]});
    return result;
}

int main() {
    vector<Entry> logs = {
        {7,"2023-03-05",10,50}, {7,"2023-03-05",90,130},
        {7,"2023-03-06",5,60},  {4,"2023-03-05",20,35}
    };
    for (auto& [d, id, mins] : totalMinutesPerDay(logs))
        cout << d << " " << id << " " << mins << "\n";
}
C
C
#include <stdio.h>
#include <string.h>

typedef struct { int personId; char logDate[16]; int tapIn; int tapOut; } Entry;
typedef struct { char logDate[16]; int personId; int minutes; } Result;

/* Fills out[] (caller allocates >= n rows) and returns the number of rows. */
int totalMinutesPerDay(const Entry *logs, int n, Result *out) {
    int m = 0;
    for (int i = 0; i < n; i++) {
        int dur = logs[i].tapOut - logs[i].tapIn;
        int j;
        for (j = 0; j < m; j++) {
            if (out[j].personId == logs[i].personId &&
                strcmp(out[j].logDate, logs[i].logDate) == 0) {
                out[j].minutes += dur;
                break;
            }
        }
        if (j == m) {
            strcpy(out[m].logDate, logs[i].logDate);
            out[m].personId = logs[i].personId;
            out[m].minutes = dur;
            m++;
        }
    }
    return m;
}

int main(void) {
    Entry logs[] = {
        {7, "2023-03-05", 10, 50}, {7, "2023-03-05", 90, 130},
        {7, "2023-03-06", 5, 60},  {4, "2023-03-05", 20, 35}
    };
    Result out[4];
    int m = totalMinutesPerDay(logs, 4, out);
    for (int i = 0; i < m; i++)
        printf("%s %d %d\n", out[i].logDate, out[i].personId, out[i].minutes);
    return 0;
}

#### Follow-up
questions
Also produce each person's grand total across all days (group by person only), or the busiest single day per person.
If the no-overlap guarantee is dropped, merge overlapping sessions before summing so shared minutes are not double-counted.
Format the totals as HH:MM instead of raw minutes.
Express the same result as a single SQL query and reason about the index that makes the GROUP BY efficient.

