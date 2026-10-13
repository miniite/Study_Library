---
type: "practice"
difficulty: "Other"
---

# Daily Cancellation Rate of Service Requests (Unblocked Parties Only)

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Database](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Database.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You operate an on-demand service platform. Two tables record its activity.

Table: Requests

Column Name	Type
id	int
requester_id	int
agent_id	int
region_id	int
status	enum
request_date	varchar

id is the primary key. Each row is one service request. requester_id and agent_id reference account_id in the Accounts table. status is one of 'completed', 'cancelled_by_agent', 'cancelled_by_requester'. request_date is a date string like '2021-03-01'.

Table: Accounts

Column Name	Type
account_id	int
blocked	enum
role	enum

account_id is the primary key. blocked is 'Yes' or 'No'. role is one of 'requester', 'agent', 'partner'.

The cancellation rate for a given day is the number of cancelled requests (cancelled by either party) divided by the total number of requests on that day — but counting only requests in which neither the requester nor the agent is blocked.

Write a solution that returns the cancellation rate of requests involving only unblocked parties for each day between '2021-03-01' and '2021-03-03' (inclusive) that has at least one such request. Round the cancellation rate to two decimal places. Days with no qualifying request must not appear. The result may be returned in any order, with columns Day and Cancellation Rate.

#### Cons
traints

Both endpoints of the date window are inclusive.
A request counts toward a day only if BOTH the requester and the agent are unblocked.
Rate = (qualifying cancelled requests) / (qualifying total requests), rounded to 2 decimals.
A day is omitted entirely if it has zero qualifying requests (never divide by zero).
Row counts are modest (well within a few thousand); an account id can appear as both requester and agent across different rows.

#### Example
s & test cases
TEXT
Input:
Requests:
| id | requester_id | agent_id | region_id | status                 | request_date |
| 1  | 1  | 10 | 1  | completed              | 2021-03-01 |
| 2  | 2  | 11 | 1  | cancelled_by_agent     | 2021-03-01 |
| 3  | 3  | 12 | 6  | completed              | 2021-03-01 |
| 4  | 4  | 13 | 6  | cancelled_by_requester | 2021-03-01 |
| 5  | 1  | 10 | 1  | completed              | 2021-03-02 |
| 6  | 2  | 11 | 6  | completed              | 2021-03-02 |
| 7  | 3  | 12 | 6  | completed              | 2021-03-02 |
| 8  | 2  | 12 | 12 | completed              | 2021-03-03 |
| 9  | 3  | 10 | 12 | completed              | 2021-03-03 |
| 10 | 4  | 13 | 12 | cancelled_by_agent     | 2021-03-03 |
Accounts:
| account_id | blocked | role      |
| 1  | No  | requester |
| 2  | Yes | requester |
| 3  | No  | requester |
| 4  | No  | requester |
| 10 | No  | agent     |
| 11 | No  | agent     |
| 12 | No  | agent     |
| 13 | No  | agent     |
-> Output:
| Day        | Cancellation Rate |
| 2021-03-01 | 0.33 |
| 2021-03-02 | 0.00 |
| 2021-03-03 | 0.50 |
Why: On 03-01 request id2 is dropped (requester 2 is blocked); 3 qualify, 1 cancelled -> 1/3=0.33. On 03-02 id6 is dropped; 2 qualify, 0 cancelled -> 0.00. On 03-03 id8 is dropped; 2 qualify, 1 cancelled -> 0.50.

Input: A single request on 2021-03-02 by an unblocked requester+agent with status cancelled_by_requester -> Output: | 2021-03-02 | 1.00 |
Why: 1 qualifying request, 1 cancelled -> 1/1 = 1.00. Only that day appears.

Input: Every request on 2021-03-01 involves a blocked account (requester or agent) -> Output: (empty for that day)
Why: Zero qualifying requests that day, so the day is omitted (no division by zero).

Input: A request dated 2021-03-05 (outside the window) with unblocked parties -> Output: (row ignored)
Why: Only days within 2021-03-01..2021-03-03 are considered.

Input: A request whose requester is unblocked but whose agent is blocked -> Output: (row ignored for its day)
Why: BOTH parties must be unblocked; a blocked agent disqualifies the request.

#### Approach

The core pattern is a filtered, conditional aggregation grouped by day. First identify blocked accounts once (a set membership test). A request qualifies only if its date is inside the window and both its requester and its agent are unblocked. For every qualifying request bump that day's total, and additionally bump that day's cancelled count when the status begins with "cancelled". Finally, for each day that has at least one qualifying request, output cancelled/total rounded to two decimals — days with no qualifying request never enter the map, so division by zero cannot occur. In SQL this is one query:

SQL
SELECT t.request_date AS Day,
       ROUND(SUM(t.status <> 'completed') / COUNT(*), 2) AS "Cancellation Rate"
FROM Requests t
JOIN Accounts r ON t.requester_id = r.account_id AND r.blocked = 'No'
JOIN Accounts a ON t.agent_id     = a.account_id AND a.blocked = 'No'
WHERE t.request_date BETWEEN '2021-03-01' AND '2021-03-03'
GROUP BY t.request_date;

The two joins onto Accounts (once as requester, once as agent, each requiring blocked = 'No') enforce the "both parties unblocked" rule; SUM(status <> 'completed') is the conditional count of cancellations; GROUP BY request_date produces the per-day rate, and only days with surviving rows appear.

Complexity

Time: O(A + R) where A = number of accounts and R = number of requests — one pass to collect blocked ids into a hash set, one pass over requests doing O(1) lookups and aggregation, then O(D log D) to emit D distinct days (D <= 3 here). Space: O(A + D) for the blocked set and per-day counters. The canonical SQL solution is a single grouped join, which the engine executes in comparable linear-plus-grouping time.

Python
PYTHON
from typing import List, Tuple

def cancellation_rates(requests: List[dict], accounts: List[dict]) -> List[Tuple[str, str]]:
    START, END = "2021-03-01", "2021-03-03"
    blocked = {a["account_id"] for a in accounts if a["blocked"] == "Yes"}

    total: dict = {}
    cancelled: dict = {}
    for r in requests:
        day = r["request_date"]
        if day < START or day > END:
            continue
        if r["requester_id"] in blocked or r["agent_id"] in blocked:
            continue
        total[day] = total.get(day, 0) + 1
        if r["status"].startswith("cancelled"):
            cancelled[day] = cancelled.get(day, 0) + 1

    result = []
    for day in sorted(total):
        rate = cancelled.get(day, 0) / total[day]
        result.append((day, f"{rate:.2f}"))
    return result
JavaScript
JAVASCRIPT
function cancellationRates(requests, accounts) {

#### cons
t START = "2021-03-01", END = "2021-03-03";

#### cons
t blocked = new Set(
        accounts.filter(a => a.blocked === "Yes").map(a => a.account_id)
    );

#### cons
t total = new Map();

#### cons
t cancelled = new Map();
    for (const r of requests) {

#### cons
t day = r.request_date;
        if (day < START || day > END) continue;
        if (blocked.has(r.requester_id) || blocked.has(r.agent_id)) continue;
        total.set(day, (total.get(day) || 0) + 1);
        if (r.status.startsWith("cancelled")) {
            cancelled.set(day, (cancelled.get(day) || 0) + 1);
        }
    }

#### cons
t result = [];
    for (const day of [...total.keys()].sort()) {

#### cons
t rate = (cancelled.get(day) || 0) / total.get(day);
        result.push([day, rate.toFixed(2)]);
    }
    return result;
}
Java
JAVA
import java.util.*;

class Solution {
    static class Request {
        int requesterId, agentId; String status, requestDate;
        Request(int requesterId, int agentId, String status, String requestDate) {
            this.requesterId = requesterId; this.agentId = agentId;
            this.status = status; this.requestDate = requestDate;
        }
    }
    static class Account {
        int accountId; String blocked;
        Account(int accountId, String blocked) { this.accountId = accountId; this.blocked = blocked; }
    }

    public List<String[]> cancellationRates(List<Request> reqs, List<Account> accts) {
        final String START = "2021-03-01", END = "2021-03-03";
        Set<Integer> blocked = new HashSet<>();
        for (Account a : accts) if (a.blocked.equals("Yes")) blocked.add(a.accountId);

        Map<String,Integer> total = new TreeMap<>();  // sorted by day
        Map<String,Integer> cancelled = new HashMap<>();
        for (Request r : reqs) {
            if (r.requestDate.compareTo(START) < 0 || r.requestDate.compareTo(END) > 0) continue;
            if (blocked.contains(r.requesterId) || blocked.contains(r.agentId)) continue;
            total.merge(r.requestDate, 1, Integer::sum);
            if (r.status.startsWith("cancelled")) cancelled.merge(r.requestDate, 1, Integer::sum);
        }

        List<String[]> result = new ArrayList<>();
        for (Map.Entry<String,Integer> e : total.entrySet()) {
            double rate = (double) cancelled.getOrDefault(e.getKey(), 0) / e.getValue();
            result.add(new String[]{ e.getKey(), String.format("%.2f", rate) });
        }
        return result;
    }
}
C++
CPP
#include <bits/stdc++.h>
using namespace std;

struct Request { int requesterId, agentId; string status, requestDate; };
struct Account { int accountId; string blocked; };

// Returns (day, rate-as-"0.33") sorted by day, for days with >=1 qualifying request.
vector<pair<string,string>> cancellationRates(const vector<Request>& reqs,

#### cons
t vector<Account>& accts) {

#### cons
t string START = "2021-03-01", END = "2021-03-03";
    unordered_set<int> blocked;
    for (const auto& a : accts) if (a.blocked == "Yes") blocked.insert(a.accountId);

    map<string,int> total, cancelled; // map keeps days sorted
    for (const auto& r : reqs) {
        if (r.requestDate < START || r.requestDate > END) continue;
        if (blocked.count(r.requesterId) || blocked.count(r.agentId)) continue;
        total[r.requestDate]++;
        if (r.status.rfind("cancelled", 0) == 0) cancelled[r.requestDate]++;
    }

    vector<pair<string,string>> result;
    for (const auto& [day, tot] : total) {
        double rate = (double)cancelled[day] / tot;
        char buf[16];
        snprintf(buf, sizeof(buf), "%.2f", rate);
        result.emplace_back(day, string(buf));
    }
    return result;
}
C
C
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <stdbool.h>

typedef struct { int requesterId, agentId; char status[32]; char requestDate[16]; } Request;
typedef struct { int accountId; char blocked[4]; } Account;   /* blocked = "Yes"/"No" */
typedef struct { char day[16]; char rate[8]; } DayRate;       /* rate formatted "0.33" */

/* Fills out[] (sorted by day) and returns the number of result rows. */
int cancellationRates(Request* reqs, int reqCount,
                      Account* accts, int accCount, DayRate* out) {

#### cons
t char* START = "2021-03-01";

#### cons
t char* END   = "2021-03-03";

    int maxId = 0;
    for (int i = 0; i < accCount; i++)
        if (accts[i].accountId > maxId) maxId = accts[i].accountId;
    bool* blocked = (bool*)calloc(maxId + 1, sizeof(bool));
    for (int i = 0; i < accCount; i++)
        if (strcmp(accts[i].blocked, "Yes") == 0) blocked[accts[i].accountId] = true;

    char days[512][16];
    int total[512] = {0}, cancelled[512] = {0};
    int dayCount = 0;

    for (int i = 0; i < reqCount; i++) {

#### cons
t char* d = reqs[i].requestDate;
        if (strcmp(d, START) < 0 || strcmp(d, END) > 0) continue;
        int rq = reqs[i].requesterId, ag = reqs[i].agentId;
        if ((rq <= maxId && blocked[rq]) || (ag <= maxId && blocked[ag])) continue;

        int idx = -1;
        for (int k = 0; k < dayCount; k++)
            if (strcmp(days[k], d) == 0) { idx = k; break; }
        if (idx < 0) { idx = dayCount++; strcpy(days[idx], d); }

        total[idx]++;
        if (strncmp(reqs[i].status, "cancelled", 9) == 0) cancelled[idx]++;
    }

    /* insertion-sort day indices for deterministic ascending output */
    int order[512];
    for (int i = 0; i < dayCount; i++) order[i] = i;
    for (int i = 1; i < dayCount; i++) {
        int cur = order[i], j = i - 1;
        while (j >= 0 && strcmp(days[order[j]], days[cur]) > 0) { order[j+1] = order[j]; j--; }
        order[j+1] = cur;
    }
    for (int i = 0; i < dayCount; i++) {
        int k = order[i];
        double rate = (double)cancelled[k] / total[k];
        strcpy(out[i].day, days[k]);
        snprintf(out[i].rate, sizeof(out[i].rate), "%.2f", rate);
    }

    free(blocked);
    return dayCount;
}

#### Follow-up
questions
Extend the report to break the rate down per region as well as per day.
Generalize the date window to arbitrary caller-supplied start and end dates.
How would you handle rows where status may be NULL, or a fourth cancellation category is added later?
If either table grows to hundreds of millions of rows, which indexes (on request_date, account_id, blocked) make the grouped join efficient?

