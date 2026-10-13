---
type: "practice"
difficulty: "Other"
---

# Monthly Payment Summary by Region

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Hash Map](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Hash_Map.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given a list of payment records. Each record describes one incoming payment and has four fields:

region — a short text code for where the payment came from (e.g. "US", "DE").
status — either "settled" (the payment went through) or "rejected" (the payment was refused).
amount — a non-negative integer, the value of the payment.
date — a calendar date written as "YYYY-MM-DD".

Group the records by the pair (month, region), where month is the "YYYY-MM" prefix of date. For every group that appears in the data, compute:

count — the total number of payment records in that (month, region) group.
settled_count — how many of those records have status "settled".
total_amount — the sum of amount over all records in the group.
settled_amount — the sum of amount over only the "settled" records in the group.

Return one row per (month, region) group, each row containing [month, region, count, settled_count, total_amount, settled_amount]. Only groups that actually occur in the input should appear. The rows may be returned in any order.

Input

A list of records, each a {region, status, amount, date} object/struct.

Output

A list of summary rows, one per distinct (month, region) pair.

#### Cons
traints

0 <= number of records <= 10^5
region is a non-empty string of at most 32 characters.
status is exactly one of "settled" or "rejected".
0 <= amount <= 10^9, and a group's total can exceed 32-bit range, so accumulate sums in 64-bit integers.
date is a valid "YYYY-MM-DD" string.

#### Example
s & test cases
TEXT
Input: records = [
  {US, settled, 1000, 2018-12-18},
  {US, rejected, 2000, 2018-12-19},
  {US, settled, 2000, 2019-01-01},
  {DE, settled, 2000, 2019-01-07}
] -> Output (any order):
  [2018-12, US, 2, 1, 3000, 1000]
  [2019-01, US, 1, 1, 2000, 2000]
  [2019-01, DE, 1, 1, 2000, 2000]
Why: The two December US records merge into one group (count 2, one settled worth 1000, total 3000). January splits into US and DE, each a single settled payment.

Input: records = [] -> Output: []
Why: No records means no groups to summarize.

Input: records = [{IN, rejected, 500, 2020-03-05}] -> Output: [2020-03, IN, 1, 0, 500, 0]
Why: A lone rejected payment still forms a group; total_amount counts it (500) but settled_count and settled_amount stay 0.

Input: records = [
  {UK, settled, 300, 2021-06-10},
  {UK, settled, 700, 2021-06-28},
  {UK, rejected, 400, 2021-07-02}
] -> Output (any order):
  [2021-06, UK, 2, 2, 1000, 1000]
  [2021-07, UK, 1, 0, 400, 0]
Why: Same region splits by month; June has two settled payments summing to 1000, July has one rejected payment.

Input: records = [
  {US, settled, 1000000000, 2022-01-15},
  {US, settled, 1000000000, 2022-01-20},
  {US, settled, 1000000000, 2022-01-25}
] -> Output: [2022-01, US, 3, 3, 3000000000, 3000000000]
Why: Three large settled payments push the group total past the 32-bit signed limit, confirming 64-bit accumulation is required.

#### Approach

This is a single-pass grouped aggregation (a hash-map GROUP BY). Derive each record's month as the first 7 characters of its date, form the composite key (month, region), and keep a running tally per key: increment count and add amount to total_amount for every record, and additionally increment settled_count / add to settled_amount only when the status is "settled" — this "count/sum with a condition" is exactly a conditional aggregate. Sums are kept in 64-bit integers because a group total can exceed the 32-bit range. In SQL this is the same idea:

SQL
SELECT SUBSTRING(date, 1, 7) AS month,
       region,
       COUNT(*)                                        AS count,
       SUM(status = 'settled')                         AS settled_count,
       SUM(amount)                                     AS total_amount,
       SUM(IF(status = 'settled', amount, 0))          AS settled_amount
FROM payments
GROUP BY month, region;

The imperative solutions mirror that GROUP BY with a hash map instead of the database engine's grouping.

Complexity

Time: O(n) — every record is processed once, and each hash-map insert/update is amortized O(1) (the date prefix and region form a bounded-length key). Space: O(g) for the g distinct (month, region) groups, at most O(n).

Python
PYTHON
from typing import List, Dict, Tuple

def monthly_totals(records: List[dict]) -> List[dict]:
    agg: Dict[Tuple[str, str], dict] = {}
    for r in records:
        month = r["date"][:7]                     # "YYYY-MM"
        key = (month, r["region"])
        row = agg.get(key)
        if row is None:
            row = {
                "month": month,
                "region": r["region"],
                "count": 0,
                "settled_count": 0,
                "total_amount": 0,
                "settled_amount": 0,
            }
            agg[key] = row
        row["count"] += 1
        row["total_amount"] += r["amount"]
        if r["status"] == "settled":
            row["settled_count"] += 1
            row["settled_amount"] += r["amount"]
    return list(agg.values())
JavaScript
JAVASCRIPT
function monthlyTotals(records) {

#### cons
t agg = new Map();   // "month|region" -> row
    for (const r of records) {

#### cons
t month = r.date.slice(0, 7);          // "YYYY-MM"

#### cons
t key = month + "|" + r.region;
        let row = agg.get(key);
        if (!row) {
            row = {
                month,
                region: r.region,
                count: 0,
                settledCount: 0,
                totalAmount: 0,
                settledAmount: 0,
            };
            agg.set(key, row);
        }
        row.count++;
        row.totalAmount += r.amount;
        if (r.status === "settled") {
            row.settledCount++;
            row.settledAmount += r.amount;
        }
    }
    return [...agg.values()];
}
Java
JAVA
import java.util.*;

class Solution {
    static class Record {
        String region, status, date;
        int amount;
        Record(String region, String status, int amount, String date) {
            this.region = region; this.status = status; this.amount = amount; this.date = date;
        }
    }
    static class Row {
        String month, region;
        int count, settledCount;
        long totalAmount, settledAmount;
    }

    public List<Row> monthlyTotals(List<Record> recs) {
        Map<String, Row> agg = new LinkedHashMap<>();   // "month|region" -> Row
        for (Record r : recs) {
            String month = r.date.substring(0, 7);      // "YYYY-MM"
            String key = month + "|" + r.region;
            Row row = agg.computeIfAbsent(key, k -> {
                Row nr = new Row();
                nr.month = month;
                nr.region = r.region;
                return nr;
            });
            row.count++;
            row.totalAmount += r.amount;
            if (r.status.equals("settled")) {
                row.settledCount++;
                row.settledAmount += r.amount;
            }
        }
        return new ArrayList<>(agg.values());
    }
}
C++
CPP
#include <string>
#include <vector>
#include <map>
using namespace std;

struct Record { string region, status; int amount; string date; };
struct Row {
    string month, region;
    int count = 0, settledCount = 0;
    long long totalAmount = 0, settledAmount = 0;
};

vector<Row> monthlyTotals(const vector<Record>& recs) {
    map<pair<string, string>, Row> agg;   // (month, region) -> Row
    for (const auto& r : recs) {
        string month = r.date.substr(0, 7);   // "YYYY-MM"
        Row& row = agg[{month, r.region}];
        row.month = month;
        row.region = r.region;
        row.count++;
        row.totalAmount += r.amount;
        if (r.status == "settled") {
            row.settledCount++;
            row.settledAmount += r.amount;
        }
    }
    vector<Row> res;
    res.reserve(agg.size());
    for (auto& kv : agg) res.push_back(kv.second);
    return res;
}
C
C
#include <stdlib.h>
#include <string.h>
#include <stdio.h>

typedef struct {

#### cons
t char* region;   /* e.g. "US" */

#### cons
t char* status;   /* "settled" or "rejected" */
    int amount;

#### cons
t char* date;     /* "YYYY-MM-DD" */
} Record;

typedef struct {
    char month[8];        /* "YYYY-MM" */
    char region[33];
    int count;
    int settledCount;
    long long totalAmount;
    long long settledAmount;
} Row;

/* keys[i] = "YYYY-MM|region" used to sort record indices into groups. */
static char (*g_keys)[48];
static int cmpIdx(const void* pa, const void* pb) {
    return strcmp(g_keys[*(const int*)pa], g_keys[*(const int*)pb]);
}

/* Returns an array of grouped Rows; *outCount = number of groups. Caller frees. */
Row* monthlyTotals(const Record* recs, int n, int* outCount) {
    if (n == 0) { *outCount = 0; return NULL; }

    char (*keys)[48] = malloc((size_t)n * sizeof(*keys));
    int* idx = malloc((size_t)n * sizeof(int));
    for (int i = 0; i < n; i++) {
        char month[8];
        memcpy(month, recs[i].date, 7); month[7] = '\0';   /* "YYYY-MM" */
        snprintf(keys[i], 48, "%s|%s", month, recs[i].region);
        idx[i] = i;
    }
    g_keys = keys;
    qsort(idx, n, sizeof(int), cmpIdx);

    Row* rows = malloc((size_t)n * sizeof(Row));
    int rc = 0, i = 0;
    while (i < n) {
        Row r; memset(&r, 0, sizeof(r));

#### cons
t Record* first = &recs[idx[i]];
        memcpy(r.month, first->date, 7); r.month[7] = '\0';
        strncpy(r.region, first->region, 32); r.region[32] = '\0';
        int j = i;
        while (j < n && strcmp(keys[idx[j]], keys[idx[i]]) == 0) {

#### cons
t Record* rec = &recs[idx[j]];
            r.count++;
            r.totalAmount += rec->amount;
            if (strcmp(rec->status, "settled") == 0) {
                r.settledCount++;
                r.settledAmount += rec->amount;
            }
            j++;
        }
        rows[rc++] = r;
        i = j;
    }

    free(keys); free(idx);
    *outCount = rc;
    return rows;
}

#### Follow-up
questions
Also report the rejected_count / rejected_amount and the average settled amount per group.
Return only the groups whose settled_amount exceeds a given threshold, or the top-k groups by total amount.
Produce the output sorted by month ascending and, within a month, region descending.
Express the solution as a SQL query, then discuss how you would keep the running summary up to date as new payments stream in (incremental aggregation).

