---
type: "practice"
difficulty: "Other"
---

# Reporting Runs of Three or More Consecutive High-Traffic Log Entries

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Sorting](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Sorting.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

A venue keeps a log of gate readings. Each reading is a record with three fields:

entryId — a unique, strictly increasing integer identifier for the reading. Ids increase with time, but the sequence of ids may skip values (readings can be missing), so consecutive-in-time readings are not always consecutive-in-id.
logDate — the calendar date of the reading (a string, e.g. "2021-03-04"). Because ids increase with time, sorting by entryId is the same as sorting by logDate.
headcount — the number of people counted during that reading.

You are given the full list of records. Report every record that belongs to a block of three or more records with consecutive entryId values (i.e. ids that form a run k, k+1, k+2, ... with no gaps) in which every record of the block has headcount >= 100.

A block breaks whenever the next id is not exactly one greater than the current id, or a record's headcount drops below 100. Only maximal runs of length >= 3 qualify, and when a run qualifies, all of its records are reported.

Return the qualifying records ordered by logDate in ascending order (equivalently, by entryId ascending).

Input

A list of records, each (entryId, logDate, headcount). Ids are unique; the list is not necessarily pre-sorted.

Output

The list of qualifying records (entryId, logDate, headcount), sorted ascending by entryId / logDate.

#### Cons
traints

0 <= number of records <= 10^5
1 <= entryId <= 10^9, all entryId distinct
0 <= headcount <= 10^9
Dates are valid YYYY-MM-DD strings and increase together with entryId.

#### Example
s & test cases
TEXT
- Input: `[(1,"2021-03-01",30), (2,"2021-03-02",118), (3,"2021-03-03",142), (4,"2021-03-04",77), (5,"2021-03-05",160), (6,"2021-03-06",1320), (7,"2021-03-07",205), (8,"2021-03-09",191)]`
  -> Output: `[(5,"2021-03-05",160), (6,"2021-03-06",1320), (7,"2021-03-07",205), (8,"2021-03-09",191)]`
  Why: ids 5-8 are consecutive and all `>= 100` (length 4). Ids 2-3 are only length 2, so excluded. Note id 8 qualifies despite a date gap — consecutiveness is on id, not date.
- Input: `[]` -> Output: `[]`
  Why: Empty log, nothing to report (empty-input edge case).
- Input: `[(10,"2022-01-01",500), (11,"2022-01-02",500)]` -> Output: `[]`
  Why: Only two consecutive high-traffic records; a run of length 2 fails the "three or more" rule.
- Input: `[(4,"2022-05-04",120), (5,"2022-05-05",130), (6,"2022-05-06",140)]` -> Output: `[(4,"2022-05-04",120), (5,"2022-05-05",130), (6,"2022-05-06",140)]`
  Why: Minimum qualifying run: exactly three consecutive ids all `>= 100`.
- Input: `[(1,"2020-01-01",200), (2,"2020-01-02",200), (4,"2020-01-04",200), (5,"2020-01-05",200), (6,"2020-01-06",200)]`
  -> Output: `[(4,"2020-01-04",200), (5,"2020-01-05",200), (6,"2020-01-06",200)]`
  Why: The id gap between 2 and 4 splits the data into runs {1,2} (length 2, rejected) and {4,5,6} (length 3, accepted).
- Input: `[(7,"2019-07-07",300), (8,"2019-07-08",99), (9,"2019-07-09",300), (10,"2019-07-10",300), (11,"2019-07-11",300)]`
  -> Output: `[(9,"2019-07-09",300), (10,"2019-07-10",300), (11,"2019-07-11",300)]`
  Why: The `99` at id 8 breaks the run (below threshold), so {7} alone fails and {9,10,11} qualifies.

#### Approach

Reframe the task as finding maximal runs of consecutive integer keys that all satisfy a threshold — a grouping / streak-detection pattern. First sort records by entryId (which also gives date order, since ids increase with time). Then sweep left to right: whenever a record meets the headcount >= 100 threshold, extend a window forward as long as the next record's id is exactly one greater and it also meets the threshold. When the window can no longer grow, if its length is at least 3, emit every record inside it; otherwise discard the whole window. Skip below-threshold records individually. Two conditions independently break a run — an id gap or a sub-threshold headcount — and both are checked while extending. This is the imperative equivalent of the SQL solution that assigns each record a "group key" of id minus a dense rank over the qualifying rows, keeps groups whose size is >= 3, and orders by date; the linear scan achieves the same grouping without window functions.

Complexity

Time: O(n log n), dominated by the initial sort by id; the streak scan is a single O(n) pass because each record is examined a constant number of times. If ids arrive already sorted, the scan alone is O(n). Space: O(1) extra beyond the output list (in-place scan; the output holds at most n records).

Python
PYTHON
from typing import List, Tuple

# Each record is (entry_id, log_date, headcount).
Record = Tuple[int, str, int]

def high_traffic_runs(records: List[Record]) -> List[Record]:
    """Records in a run of >= 3 consecutive ids where every headcount >= 100."""
    recs = sorted(records, key=lambda r: r[0])   # by id == by date
    n = len(recs)
    out: List[Record] = []
    i = 0
    while i < n:
        if recs[i][2] >= 100:
            j = i
            while (j + 1 < n
                   and recs[j + 1][0] == recs[j][0] + 1
                   and recs[j + 1][2] >= 100):
                j += 1
            if j - i + 1 >= 3:
                out.extend(recs[i:j + 1])
            i = j + 1
        else:
            i += 1
    return out

if __name__ == "__main__":
    data = [
        (1, "2021-03-01", 30),  (2, "2021-03-02", 118), (3, "2021-03-03", 142),
        (4, "2021-03-04", 77),  (5, "2021-03-05", 160), (6, "2021-03-06", 1320),
        (7, "2021-03-07", 205), (8, "2021-03-09", 191),
    ]
    for row in high_traffic_runs(data):
        print(row)
JavaScript
JAVASCRIPT
/**
 * Return records in a run of >= 3 consecutive entryIds, each headcount >= 100.
 * @param {{id:number, date:string, headcount:number}[]} records
 * @return {{id:number, date:string, headcount:number}[]}
 */
function highTrafficRuns(records) {

#### cons
t recs = records.slice().sort((a, b) => a.id - b.id); // by id == by date

#### cons
t out = [];

#### cons
t n = recs.length;
    let i = 0;
    while (i < n) {
        if (recs[i].headcount >= 100) {
            let j = i;
            while (j + 1 < n &&
                   recs[j + 1].id === recs[j].id + 1 &&
                   recs[j + 1].headcount >= 100) {
                j++;
            }
            if (j - i + 1 >= 3) {
                for (let k = i; k <= j; k++) out.push(recs[k]);
            }
            i = j + 1;
        } else {
            i++;
        }
    }
    return out;
}
Java
JAVA
import java.util.*;

class Solution {
    static class Record {
        int id;
        String date;
        long headcount;
        Record(int id, String date, long headcount) {
            this.id = id; this.date = date; this.headcount = headcount;
        }
    }

    // Return records in a run of >= 3 consecutive ids, each headcount >= 100.
    public List<Record> highTrafficRuns(List<Record> recs) {
        recs.sort(Comparator.comparingInt(r -> r.id));
        List<Record> out = new ArrayList<>();
        int n = recs.size();
        int i = 0;
        while (i < n) {
            if (recs.get(i).headcount >= 100) {
                int j = i;
                while (j + 1 < n
                        && recs.get(j + 1).id == recs.get(j).id + 1
                        && recs.get(j + 1).headcount >= 100) {
                    j++;
                }
                if (j - i + 1 >= 3) {
                    for (int k = i; k <= j; k++) out.add(recs.get(k));
                }
                i = j + 1;
            } else {
                i++;
            }
        }
        return out;   // ascending by id == ascending by date
    }
}
C++
CPP
#include <vector>
#include <string>
#include <algorithm>
#include <tuple>

struct Record {
    int id;
    std::string date;
    long long headcount;
};

class Solution {
public:
    // Return records that belong to a run of >= 3 consecutive ids, each headcount >= 100.
    std::vector<Record> highTrafficRuns(std::vector<Record> recs) {
        std::sort(recs.begin(), recs.end(),
                  [] (const Record &a, const Record &b) { return a.id < b.id; });

        std::vector<Record> out;
        int n = static_cast<int>(recs.size());
        int i = 0;
        while (i < n) {
            if (recs[i].headcount >= 100) {
                int j = i;
                while (j + 1 < n &&
                       recs[j + 1].id == recs[j].id + 1 &&
                       recs[j + 1].headcount >= 100) {
                    ++j;
                }
                if (j - i + 1 >= 3) {
                    for (int k = i; k <= j; ++k) out.push_back(recs[k]);
                }
                i = j + 1;
            } else {
                ++i;
            }
        }
        return out;   // already sorted ascending by id (== by date)
    }
};
C
C
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

typedef struct {
    int id;
    char date[16];
    long long headcount;
} Record;

static int cmpById(const void *a, const void *b) {
    long long d = ((const Record *)a)->id - (long long)((const Record *)b)->id;
    return (d > 0) - (d < 0);
}

/* Writes qualifying records into out[] (caller allocates >= n slots).
   Returns the number of qualifying records. */
int highTrafficRuns(Record *recs, int n, Record *out) {
    if (n == 0) return 0;
    qsort(recs, n, sizeof(Record), cmpById);

    int outCount = 0;
    int i = 0;
    while (i < n) {
        if (recs[i].headcount >= 100) {
            int j = i;
            while (j + 1 < n &&
                   recs[j + 1].id == recs[j].id + 1 &&
                   recs[j + 1].headcount >= 100) {
                j++;
            }
            if (j - i + 1 >= 3) {
                for (int k = i; k <= j; k++)
                    out[outCount++] = recs[k];
            }
            i = j + 1;
        } else {
            i++;
        }
    }
    return outCount;
}

int main(void) {
    Record recs[] = {
        {1,"2021-03-01",30},  {2,"2021-03-02",118}, {3,"2021-03-03",142},
        {4,"2021-03-04",77},  {5,"2021-03-05",160}, {6,"2021-03-06",1320},
        {7,"2021-03-07",205}, {8,"2021-03-09",191}
    };
    int n = 8;
    Record *out = malloc(n * sizeof(Record));
    int m = highTrafficRuns(recs, n, out);
    for (int i = 0; i < m; i++)
        printf("%d %s %lld\n", out[i].id, out[i].date, out[i].headcount);
    free(out);
    return 0;
}

#### Follow-up
questions
Make the run-length threshold (3) and the headcount threshold (100) parameters and support arbitrary values.
Return only the single longest qualifying run rather than all of them (and break ties by earliest id).
Count how many distinct qualifying runs exist instead of returning their records.
Solve it in SQL using window functions (dense rank grouping) and compare against the linear scan; then adapt the scan to a streaming input where records arrive in id order one at a time.

