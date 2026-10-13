---
type: "practice"
difficulty: "Other"
---

# Conditional Reward Per Staff Record

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Array](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Array.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given a collection of staff records. Each record contains three fields:

staff_id — a unique positive integer identifier for the person.
full_name — the person's name (a non-empty string, unless stated otherwise).
wage — a non-negative integer base wage.

For each record you must compute a reward according to this rule:

The reward equals 100% of the person's wage if both of the following hold:
staff_id is an odd number, and
full_name does not begin with the capital letter 'K'.
Otherwise, the reward is 0.

Return one result row per staff record, each containing staff_id and its computed reward, sorted by staff_id in ascending order.

#### Cons
traints:

1 <= number of records <= 10^5
1 <= staff_id <= 10^6, all staff_id values are distinct.
0 <= wage <= 10^6
full_name consists of printable characters; its first character determines the letter test (comparison is case-sensitive, so only an uppercase 'K' disqualifies a name).

#### Example
s & test cases
TEXT
- Input: `records = [(4, "Priya", 5200), (5, "Kevin", 4800), (6, "Rohan", 6000), (11, "Aditi", 7100), (13, "Karan", 6600), (15, "Neha", 5400)]`
  -> Output: `[(4, 0), (5, 0), (6, 0), (11, 7100), (13, 0), (15, 5400)]`
  Why: id 4 and 6 are even → 0; id 5 ("Kevin") and id 13 ("Karan") are odd but start with 'K' → 0; id 11 ("Aditi") and id 15 ("Neha") are odd and do not start with 'K' → full wage.
- Input: `records = [(3, "Karim", 9000)]`
  -> Output: `[(3, 0)]`
  Why: odd id but the name begins with 'K', so the reward is zeroed out.
- Input: `records = [(7, "Sara", 0)]`
  -> Output: `[(7, 0)]`
  Why: qualifies for the reward, but the base wage is 0 so 100% of it is still 0 (edge: zero wage).
- Input: `records = [(20, "Kabir", 8000), (9, "Amit", 4500), (2, "Kiara", 3000)]`
  -> Output: `[(2, 0), (9, 4500), (20, 0)]`
  Why: output must be re-sorted by staff_id; only id 9 ("Amit", odd, not 'K') earns the wage.
- Input: `records = [(1, "kavya", 5000)]`
  -> Output: `[(1, 5000)]`
  Why: case-sensitivity edge — the name starts with a lowercase 'k', which does NOT match the uppercase 'K' disqualifier, so the odd id still earns the full wage.

#### Approach

This is a straightforward per-record classification followed by an ordering. For every record evaluate a two-part boolean predicate — staff_id is odd (staff_id % 2 != 0) and the name's first character is not the uppercase letger 'K' — and assign the full wage when both are true, otherwise 0. Collect (staff_id, reward) pairs and sort ascending by staff_id for the required output order. The natural relational equivalent is a single conditional projection:

SQL
SELECT staff_id,
       CASE WHEN staff_id % 2 = 1 AND SUBSTRING(full_name, 1, 1) <> 'K'
            THEN wage ELSE 0 END AS reward
FROM Staff
ORDER BY staff_id;

Pattern: conditional aggregation / row-wise transformation with a sort — no auxiliary data structure beyond the result list is needed.

Complexity

Time: O(n log n) dominated by the final sort by staff_id (the per-record predicate check is O(1), or O(L) for name length L but only the first character is inspected). Space: O(n) for the output rows (excluding input). If the records already arrive sorted by id, the pass alone is O(n).

Python
PYTHON
from typing import List, Tuple

def conditional_reward(records: List[Tuple[int, str, int]]) -> List[Tuple[int, int]]:
    out = []
    for staff_id, full_name, wage in records:
        is_odd = staff_id % 2 != 0
        not_k = not full_name.startswith('K')   # case-sensitive first-char test
        out.append((staff_id, wage if (is_odd and not_k) else 0))
    out.sort(key=lambda row: row[0])
    return out
JavaScript
JAVASCRIPT
/**
 * @param {{staffId:number, fullName:string, wage:number}[]} records
 * @return {[number, number][]} rows [staffId, reward] sorted by staffId asc
 */
function conditionalReward(records) {

#### cons
t out = records.map(({ staffId, fullName, wage }) => {

#### cons
t isOdd = staffId % 2 !== 0;

#### cons
t notK = !fullName.startsWith('K');
        return [staffId, isOdd && notK ? wage : 0];
    });
    out.sort((a, b) => a[0] - b[0]);
    return out;
}
Java
JAVA
import java.util.*;

class Solution {
    // Returns rows [staff_id, reward] sorted ascending by staff_id.
    public long[][] conditionalReward(int[] staffId, String[] fullName, long[] wage) {
        int n = staffId.length;
        long[][] out = new long[n][2];
        for (int i = 0; i < n; i++) {
            out[i][0] = staffId[i];
            boolean odd  = (staffId[i] % 2) != 0;
            boolean notK = fullName[i].isEmpty() || fullName[i].charAt(0) != 'K';
            out[i][1] = (odd && notK) ? wage[i] : 0L;
        }
        Arrays.sort(out, (a, b) -> Long.compare(a[0], b[0]));
        return out;
    }
}
C++
CPP
#include <vector>
#include <string>
#include <algorithm>
using namespace std;

struct Staff { int staff_id; string full_name; long long wage; };

vector<pair<int, long long>> conditionalReward(const vector<Staff>& staff) {
    vector<pair<int, long long>> out;
    out.reserve(staff.size());
    for (const auto& s : staff) {
        bool odd  = (s.staff_id % 2) != 0;
        bool notK = s.full_name.empty() || s.full_name[0] != 'K';
        out.emplace_back(s.staff_id, (odd && notK) ? s.wage : 0LL);
    }
    sort(out.begin(), out.end(),
         [] (const auto& a, const auto& b) { return a.first < b.first; });
    return out;
}
C
C
#include <stdlib.h>

typedef struct { int staff_id; const char* full_name; int wage; } Staff;
typedef struct { int staff_id; long long reward; } Result;

static int cmpById(const void* a, const void* b) {
    int x = ((const Result*)a)->staff_id;
    int y = ((const Result*)b)->staff_id;
    return (x > y) - (x < y);
}

/* Fills out[] (size n) with {staff_id, reward} sorted by staff_id ascending. */
void conditionalReward(Staff* staff, int n, Result* out) {
    for (int i = 0; i < n; ++i) {
        out[i].staff_id = staff[i].staff_id;
        int odd  = (staff[i].staff_id % 2) != 0;
        int notK = staff[i].full_name[0] != 'K';   /* first char, case-sensitive */
        out[i].reward = (odd && notK) ? staff[i].wage : 0;
    }
    qsort(out, n, sizeof(Result), cmpById);
}

#### Follow-up
questions
Make the leading-letter test case-insensitive, so both 'K' and 'k' disqualify a name — how does the predicate change?
Award a partial reward (say 50% of wage) to the records that currently receive 0, instead of a flat zero.
Return only the rows whose reward is non-zero, still ordered by id — how does that affect the output size and query?
Express the entire computation in pure SQL for a table of 10 million rows and reason about whether an index on staff_id helps the ordering.

