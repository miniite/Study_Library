---
type: "practice"
difficulty: "Other"
---

# Per-User Buyer Order Count for a Given Year

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Database](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Database.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You operate an online marketplace where every member can both buy items from and sell items to other members. You are given two datasets.

members — one record per member:

member_id (unique integer)
join_date (calendar date, ISO string YYYY-MM-DD)
favorite_label (a text tag; not needed for the answer but present in the data)

transactions — one record per completed order:

transaction_id (unique integer)
transaction_date (ISO date YYYY-MM-DD)
listing_id (integer, the item that was sold)
buyer_id (the member who bought — references member_id)
seller_id (the member who sold — references member_id)

Given a target calendar year Y, produce, for every member, three values:

the member's member_id,
their join_date,
the number of orders they placed as a buyer during year Y.

A member who made no purchases in year Y must still appear in the result, with a count of 0. The result rows may be returned in any order.

Input / Output

Input: the members list, the transactions list, and the integer year Y.
Output: a collection of (member_id, join_date, orders_in_year) rows, one per member.

#### Cons
traints

1 <= number of members <= 10^5
0 <= number of transactions <= 10^5
All member_id values are distinct positive integers; every buyer_id/seller_id refers to an existing member.
Dates are valid ISO YYYY-MM-DD strings; Y is a 4-digit year.
A member may appear as buyer, seller, both, or neither.

#### Example
s & test cases
TEXT
Input:
members = [(11,"2020-03-04","Aurora"), (12,"2020-06-15","Bluewave"), (13,"2020-01-27","Cedar"), (14,"2020-09-08","Delta")]
transactions = [(501,"2022-04-10",14,11,12), (502,"2021-04-11",12,11,13), (503,"2022-04-12",13,12,13), (504,"2021-04-13",11,14,12), (505,"2021-04-13",11,13,14), (506,"2022-04-14",12,12,14)]
Y = 2022
-> Output: (11,"2020-03-04",1), (12,"2020-06-15",2), (13,"2020-01-27",0), (14,"2020-09-08",0)
Why: member 11 bought once in 2022 (txn 501); member 12 bought twice in 2022 (txns 503, 506); members 13 and 14 made no 2022 purchases but still appear with 0.

Input: same data, Y = 2021
-> Output: (11,"2020-03-04",1), (12,"2020-06-15",0), (13,"2020-01-27",1), (14,"2020-09-08",1)
Why: in 2021 member 11 bought once (txn 502), member 13 once (txn 505), member 14 once (txn 504); member 12 bought nothing in 2021.

Input: members = [(7,"2019-05-05","Solo")], transactions = [], Y = 2023
-> Output: (7,"2019-05-05",0)
Why: no transactions at all, yet the single member still appears with count 0 (empty-transactions edge case).

Input: members = [(1,"2018-02-02","X"), (2,"2018-03-03","Y")], transactions = [(90,"2020-12-31",5,2,1), (91,"2021-01-01",5,2,1)], Y = 2020
-> Output: (1,"2018-02-02",0), (2,"2018-03-03",1)
Why: only txn 90 is in 2020 and member 2 is its buyer; the year boundary (Dec 31 vs Jan 1) is handled by year comparison, and seller-side member 1 gets 0.

#### Approach

This is a group-and-count / left-join problem. The canonical relational query left-joins members to transactions on the buyer key while filtering to the target year, then aggregates:

SQL
SELECT m.member_id,
       m.join_date,
       COUNT(t.transaction_id) AS orders_in_year
FROM   members m
LEFT JOIN transactions t
       ON t.buyer_id = m.member_id
      AND YEAR(t.transaction_date) = 2022
GROUP BY m.member_id, m.join_date;

The LEFT JOIN is essential: it keeps members who bought nothing in the year, and COUNT(t.transaction_id) yields 0 for them (COUNT ignores NULLs) rather than dropping the row.

The in-memory equivalent (implemented above in all five languages) mirrors that plan: make one pass over the transactions, and for every transaction whose date falls in the target year, increment a hash-map counter keyed by buyer_id. Then make one pass over the members, emitting (member_id, join_date, count) where the count defaults to 0 when the member is absent from the map. Comparing only the 4-character year prefix of the ISO date avoids any date-parsing cost and correctly handles year boundaries (Dec 31 vs Jan 1).

Complexity

Time: O(m + t), where m is the number of members and t is the number of transactions. We scan the transactions once to build a buyer -> count hash map (each date year-check is O(1) on a 4-char prefix), then scan members once to emit each row with an O(1) map lookup. Space: O(u) for the map, where u is the number of distinct buyers in the target year (at most min(m, t)).

Python
PYTHON
from collections import defaultdict

def yearly_purchases(members, transactions, year):
    # members:      list of (member_id, join_date, favorite_label)
    # transactions: list of (txn_id, txn_date, listing_id, buyer_id, seller_id)
    y = str(year)
    bought = defaultdict(int)
    for _txn_id, txn_date, _listing, buyer_id, _seller in transactions:
        if txn_date[:4] == y:          # compare the year portion of YYYY-MM-DD
            bought[buyer_id] += 1
    return [(member_id, join_date, bought.get(member_id, 0))
            for member_id, join_date, _label in members]

if __name__ == "__main__":
    members = [
        (11, "2020-03-04", "Aurora"),
        (12, "2020-06-15", "Bluewave"),
        (13, "2020-01-27", "Cedar"),
        (14, "2020-09-08", "Delta"),
    ]
    transactions = [
        (501, "2022-04-10", 14, 11, 12),
        (502, "2021-04-11", 12, 11, 13),
        (503, "2022-04-12", 13, 12, 13),
        (504, "2021-04-13", 11, 14, 12),
        (505, "2021-04-13", 11, 13, 14),
        (506, "2022-04-14", 12, 12, 14),
    ]
    for row in yearly_purchases(members, transactions, 2022):
        print(row)
JavaScript
JAVASCRIPT
function yearlyPurchases(members, transactions, year) {

#### cons
t y = String(year);

#### cons
t bought = new Map();
  for (const t of transactions) {
    if (t.txnDate.slice(0, 4) === y) {          // year prefix of YYYY-MM-DD
      bought.set(t.buyerId, (bought.get(t.buyerId) || 0) + 1);
    }
  }
  return members.map((m) => ({
    memberId: m.memberId,
    joinDate: m.joinDate,
    ordersInYear: bought.get(m.memberId) || 0,
  }));
}

#### cons
t members = [
  { memberId: 11, joinDate: "2020-03-04", favoriteLabel: "Aurora" },
  { memberId: 12, joinDate: "2020-06-15", favoriteLabel: "Bluewave" },
  { memberId: 13, joinDate: "2020-01-27", favoriteLabel: "Cedar" },
  { memberId: 14, joinDate: "2020-09-08", favoriteLabel: "Delta" },
];

#### cons
t transactions = [
  { txnId: 501, txnDate: "2022-04-10", listingId: 14, buyerId: 11, sellerId: 12 },
  { txnId: 502, txnDate: "2021-04-11", listingId: 12, buyerId: 11, sellerId: 13 },
  { txnId: 503, txnDate: "2022-04-12", listingId: 13, buyerId: 12, sellerId: 13 },
  { txnId: 504, txnDate: "2021-04-13", listingId: 11, buyerId: 14, sellerId: 12 },
  { txnId: 505, txnDate: "2021-04-13", listingId: 11, buyerId: 13, sellerId: 14 },
  { txnId: 506, txnDate: "2022-04-14", listingId: 12, buyerId: 12, sellerId: 14 },
];

#### cons
ole.log(yearlyPurchases(members, transactions, 2022));
Java
JAVA
import java.util.*;

public class YearlyPurchases {
    static class Member {
        int id; String joinDate, favoriteLabel;
        Member(int id, String joinDate, String favoriteLabel) {
            this.id = id; this.joinDate = joinDate; this.favoriteLabel = favoriteLabel;
        }
    }
    static class Txn {
        int id; String date; int listingId, buyerId, sellerId;
        Txn(int id, String date, int listingId, int buyerId, int sellerId) {
            this.id = id; this.date = date; this.listingId = listingId;
            this.buyerId = buyerId; this.sellerId = sellerId;
        }
    }
    static class Row {
        int memberId; String joinDate; int ordersInYear;
        Row(int m, String j, int o) { memberId = m; joinDate = j; ordersInYear = o; }
        public String toString() {
            return "(" + memberId + ", " + joinDate + ", " + ordersInYear + ")";
        }
    }

    public static List<Row> yearlyPurchases(List<Member> members, List<Txn> txns, int year) {
        String y = String.valueOf(year);
        Map<Integer, Integer> bought = new HashMap<>();
        for (Txn t : txns)
            if (t.date.startsWith(y))
                bought.merge(t.buyerId, 1, Integer::sum);
        List<Row> res = new ArrayList<>();
        for (Member m : members)
            res.add(new Row(m.id, m.joinDate, bought.getOrDefault(m.id, 0)));
        return res;
    }

    public static void main(String[] args) {
        List<Member> members = Arrays.asList(
            new Member(11, "2020-03-04", "Aurora"),
            new Member(12, "2020-06-15", "Bluewave"),
            new Member(13, "2020-01-27", "Cedar"),
            new Member(14, "2020-09-08", "Delta"));
        List<Txn> txns = Arrays.asList(
            new Txn(501, "2022-04-10", 14, 11, 12),
            new Txn(502, "2021-04-11", 12, 11, 13),
            new Txn(503, "2022-04-12", 13, 12, 13),
            new Txn(504, "2021-04-13", 11, 14, 12),
            new Txn(505, "2021-04-13", 11, 13, 14),
            new Txn(506, "2022-04-14", 12, 12, 14));
        for (Row r : yearlyPurchases(members, txns, 2022))
            System.out.println(r);
    }
}
C++
CPP
#include <bits/stdc++.h>
using namespace std;

struct Member { int id; string joinDate; string favoriteLabel; };
struct Txn    { int id; string date; int listingId, buyerId, sellerId; };
struct Row    { int memberId; string joinDate; int ordersInYear; };

vector<Row> yearlyPurchases(const vector<Member>& members,

#### cons
t vector<Txn>& txns, int year) {
    string y = to_string(year);
    unordered_map<int, int> bought;
    for (const auto& t : txns)
        if (t.date.compare(0, 4, y) == 0)   // compare year prefix
            bought[t.buyerId]++;
    vector<Row> res;
    res.reserve(members.size());
    for (const auto& m : members) {
        auto it = bought.find(m.id);
        res.push_back({m.id, m.joinDate, it == bought.end() ? 0 : it->second});
    }
    return res;
}

int main() {
    vector<Member> members = {
        {11,"2020-03-04","Aurora"}, {12,"2020-06-15","Bluewave"},
        {13,"2020-01-27","Cedar"},  {14,"2020-09-08","Delta"}};
    vector<Txn> txns = {
        {501,"2022-04-10",14,11,12}, {502,"2021-04-11",12,11,13},
        {503,"2022-04-12",13,12,13}, {504,"2021-04-13",11,14,12},
        {505,"2021-04-13",11,13,14}, {506,"2022-04-14",12,12,14}};
    for (const auto& r : yearlyPurchases(members, txns, 2022))
        cout << "(" << r.memberId << ", " << r.joinDate << ", "
             << r.ordersInYear << ")\n";
    return 0;
}
C
C
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

typedef struct { int id; const char *joinDate; const char *favoriteLabel; } Member;
typedef struct { int id; const char *date; int listingId, buyerId, sellerId; } Txn;

/* minimal open-addressing hash map: buyerId -> count */
typedef struct { int *keys; int *vals; int cap; } Map;

static Map mapNew(int cap) {
    if (cap < 8) cap = 8;
    Map m; m.cap = cap;
    m.keys = (int *)malloc(sizeof(int) * cap);
    m.vals = (int *)calloc(cap, sizeof(int));
    for (int i = 0; i < cap; i++) m.keys[i] = -1;
    return m;
}
static int mapSlot(Map *m, int key) {
    unsigned h = (unsigned)key * 2654435761u % (unsigned)m->cap;
    while (m->keys[h] != -1 && m->keys[h] != key) h = (h + 1) % m->cap;
    return (int)h;
}
static void mapInc(Map *m, int key) {
    int s = mapSlot(m, key);
    m->keys[s] = key;
    m->vals[s] += 1;
}
static int mapGet(Map *m, int key) {
    int s = mapSlot(m, key);
    return m->keys[s] == key ? m->vals[s] : 0;
}

/* prints (member_id, join_date, buyer-side orders in `year`) for every member */
void yearlyPurchases(Member *members, int mCount, Txn *txns, int tCount, int year) {
    char y[8];
    sprintf(y, "%d", year);
    Map bought = mapNew(tCount * 2 + 8);
    for (int j = 0; j < tCount; j++)
        if (strncmp(txns[j].date, y, 4) == 0)
            mapInc(&bought, txns[j].buyerId);
    for (int i = 0; i < mCount; i++)
        printf("(%d, %s, %d)\n", members[i].id, members[i].joinDate,
               mapGet(&bought, members[i].id));
    free(bought.keys); free(bought.vals);
}

int main(void) {
    Member members[] = {
        {11,"2020-03-04","Aurora"}, {12,"2020-06-15","Bluewave"},
        {13,"2020-01-27","Cedar"},  {14,"2020-09-08","Delta"}};
    Txn txns[] = {
        {501,"2022-04-10",14,11,12}, {502,"2021-04-11",12,11,13},
        {503,"2022-04-12",13,12,13}, {504,"2021-04-13",11,14,12},
        {505,"2021-04-13",11,13,14}, {506,"2022-04-14",12,12,14}};
    yearlyPurchases(members, 4, txns, 6, 2022);
    return 0;
}

#### Follow-up
questions
Also report the count of orders each member made as a seller in the same year (a second aggregation on seller_id).
Generalize from a single year to an arbitrary date range, or produce a per-year breakdown for every member.
Only count purchases where the item's brand equals the member's favorite_label (requires joining the items/listings table too).
How would the SQL change on a database without a YEAR() function -- use a half-open range predicate on transaction_date so an index can be used.

