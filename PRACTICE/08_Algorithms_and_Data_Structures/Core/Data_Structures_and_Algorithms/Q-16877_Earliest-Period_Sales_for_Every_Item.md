---
type: "practice"
difficulty: "Other"
---

# Earliest-Period Sales for Every Item

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Hash Map](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Hash_Map.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given one collection of sale records, Orders. Each record describes a single sale event and has the following fields:

order_id — an identifier for the sale event,
item_id — the identifier of the item that was sold,
period — the calendar year in which the sale happened,
units — how many units were sold,
unit_price — the price charged per single unit.

The pair (order_id, period) uniquely identifies a row. The same item may be sold multiple times, in the same period or across different periods.

Task. For each distinct item, determine the earliest period in which that item was ever sold — call it the item's first year. Then, for every sale that occurred in that item's first year, output a row containing four values: the item_id, the first year, the units of that sale, and the unit_price of that sale.

If an item was sold more than once during its first year, output each of those sales as a separate row. Sales that happened after an item's first year are excluded. The output rows may be returned in any order.

#### Cons
traints.

1 <= number of orders <= 10^5
order_id, item_id, period, and units are integers; unit_price is a non-negative integer.
1 <= period <= 3000
Every item appears in at least one order (there are no items with zero sales in the input).
Multiple orders may share the same item_id and the same period.

#### Example
s & test cases
TEXT
Input: Orders = [(1,100,2019,10,5000), (2,100,2020,12,5000), (7,200,2022,15,9000)] -> Output: [(100,2019,10,5000), (200,2022,15,9000)]
Why: Item 100's earliest period is 2019, so only its 2019 sale is kept (the 2020 sale is dropped); item 200 has a single sale in 2022.

Input: Orders = [(1,100,2019,10,5000), (2,100,2019,8,4800), (3,100,2020,5,5200)] -> Output: [(100,2019,10,5000), (100,2019,8,4800)]
Why: The item was sold twice in its first year (2019); both sales are returned, while the later 2020 sale is excluded.

Input: Orders = [(9,42,2021,3,1200)] -> Output: [(42,2021,3,1200)]
Why: Minimal single-order case; that sole sale is trivially in the item's first year.

Input: Orders = [(1,5,2015,2,300), (2,5,2015,7,300), (3,6,2010,1,900)] -> Output: [(5,2015,2,300), (5,2015,7,300), (6,2010,1,900)]
Why: Item 5 has two sales, both in its first year 2015, so both are kept; item 6 has its own earlier first year 2010 independent of item 5.

Input: Orders = [(1,8,2005,4,10), (2,8,2003,6,12), (3,8,2004,5,11)] -> Output: [(8,2003,6,12)]
Why: The first year is the minimum period 2003 even though it does not appear first in the input; only the 2003 sale qualifies.

#### Approach

The pattern is group-by-key aggregation (a min per group) followed by a filter/self-join against that aggregate. First pass: scan all orders and maintain a hash map from item_id to the smallest period seen so far for that item; this yields each item's first year in linear time. Second pass: scan the orders again and keep every order whose period equals its item's recorded first year, emitting (item_id, first_year, units, unit_price). Because ties are kept, an item sold several times in its first year contributes several rows. This two-pass hash aggregation avoids sorting (which would be O(N log N)) and avoids the O(N^2) nested comparison of every row against every other row of the same item. The equivalent set-based (SQL) formulation groups Orders by item to get MIN(period), then joins that back to Orders on both item and period.

Complexity

Time: O(N), where N is the number of orders — one pass to compute the minimum period per item and one pass to collect the qualifying rows, each with expected O(1) hash operations. Space: O(K) for the item -> first-year map (K distinct items, K <= N) plus O(N) worst case for the output when every sale happens in its item's first year.

Python
PYTHON
from typing import Dict, List, Tuple

def first_year_sales(orders: List[dict]) -> List[Tuple[int, int, int, int]]:
    first_year: Dict[int, int] = {}
    for o in orders:
        item = o["item_id"]
        if item not in first_year or o["period"] < first_year[item]:
            first_year[item] = o["period"]

    return [
        (o["item_id"], o["period"], o["units"], o["unit_price"])
        for o in orders
        if o["period"] == first_year[o["item_id"]]
    ]

if __name__ == "__main__":
    orders = [
        {"order_id": 1, "item_id": 100, "period": 2019, "units": 10, "unit_price": 5000},
        {"order_id": 2, "item_id": 100, "period": 2020, "units": 12, "unit_price": 5000},
        {"order_id": 7, "item_id": 200, "period": 2022, "units": 15, "unit_price": 9000},
    ]
    for row in first_year_sales(orders):
        print(*row)
JavaScript
JAVASCRIPT
function firstYearSales(orders) {

#### cons
t firstYear = new Map();
  for (const o of orders) {

#### cons
t cur = firstYear.get(o.itemId);
    if (cur === undefined || o.period < cur) firstYear.set(o.itemId, o.period);
  }
  return orders
    .filter((o) => o.period === firstYear.get(o.itemId))
    .map((o) => [o.itemId, o.period, o.units, o.unitPrice]);
}

// demo

#### cons
t orders = [
  { orderId: 1, itemId: 100, period: 2019, units: 10, unitPrice: 5000 },
  { orderId: 2, itemId: 100, period: 2020, units: 12, unitPrice: 5000 },
  { orderId: 7, itemId: 200, period: 2022, units: 15, unitPrice: 9000 },
];
for (const row of firstYearSales(orders)) console.log(...row);
Java
JAVA
import java.util.*;

public class Solution {
    static class Order {
        int orderId, itemId, period, units, unitPrice;
        Order(int o, int i, int p, int u, int pr) {
            orderId = o; itemId = i; period = p; units = u; unitPrice = pr;
        }
    }
    static class Row {
        int itemId, firstYear, units, unitPrice;
        Row(int i, int y, int u, int pr) { itemId = i; firstYear = y; units = u; unitPrice = pr; }
    }

    static List<Row> firstYearSales(List<Order> orders) {
        Map<Integer, Integer> firstYear = new HashMap<>();
        for (Order o : orders) firstYear.merge(o.itemId, o.period, Math::min);

        List<Row> res = new ArrayList<>();
        for (Order o : orders)
            if (o.period == firstYear.get(o.itemId))
                res.add(new Row(o.itemId, o.period, o.units, o.unitPrice));
        return res;
    }

    public static void main(String[] args) {
        List<Order> orders = Arrays.asList(
            new Order(1, 100, 2019, 10, 5000),
            new Order(2, 100, 2020, 12, 5000),
            new Order(7, 200, 2022, 15, 9000));
        for (Row r : firstYearSales(orders))
            System.out.println(r.itemId + " " + r.firstYear + " " + r.units + " " + r.unitPrice);
    }
}
C++
CPP
#include <bits/stdc++.h>
using namespace std;

struct Order { int orderId, itemId, period, units, unitPrice; };
struct Row   { int itemId, firstYear, units, unitPrice; };

vector<Row> firstYearSales(const vector<Order>& orders) {
    unordered_map<int, int> firstYear;
    firstYear.reserve(orders.size() * 2);
    for (const auto& o : orders) {
        auto it = firstYear.find(o.itemId);
        if (it == firstYear.end() || o.period < it->second) firstYear[o.itemId] = o.period;
    }

    vector<Row> res;
    for (const auto& o : orders)
        if (o.period == firstYear[o.itemId])
            res.push_back({o.itemId, o.period, o.units, o.unitPrice});
    return res;
}

int main() {
    vector<Order> orders = {{1,100,2019,10,5000},{2,100,2020,12,5000},{7,200,2022,15,9000}};
    for (const auto& r : firstYearSales(orders))
        cout << r.itemId << ' ' << r.firstYear << ' ' << r.units << ' ' << r.unitPrice << '\n';
    return 0;
}
C
C
#include <stdio.h>
#include <stdlib.h>

typedef struct { int orderId, itemId, period, units, unitPrice; } Order;
typedef struct { int itemId, firstYear, units, unitPrice; } Row;

/* open-addressing hash map: itemId -> minimum period seen */
typedef struct { int key, val, used; } Slot;
static int gCap;
static Slot *gTable;

static void mapInit(int n) {
    gCap = 1;
    while (gCap < (n + 1) * 2) gCap <<= 1;
    gTable = calloc(gCap, sizeof(Slot));
}
static unsigned hashKey(int k) { return (unsigned)k * 2654435761u; }
static void mapMin(int k, int v) {
    unsigned i = hashKey(k) & (gCap - 1);
    while (gTable[i].used) {
        if (gTable[i].key == k) { if (v < gTable[i].val) gTable[i].val = v; return; }
        i = (i + 1) & (gCap - 1);
    }
    gTable[i].used = 1; gTable[i].key = k; gTable[i].val = v;
}
static int mapGet(int k) {
    unsigned i = hashKey(k) & (gCap - 1);
    while (gTable[i].used) {
        if (gTable[i].key == k) return gTable[i].val;
        i = (i + 1) & (gCap - 1);
    }
    return -1;
}

Row *firstYearSales(Order *orders, int n, int *outLen) {
    mapInit(n);
    for (int i = 0; i < n; i++) mapMin(orders[i].itemId, orders[i].period);

    Row *res = malloc(sizeof(Row) * (n > 0 ? n : 1));
    int m = 0;
    for (int i = 0; i < n; i++) {
        if (orders[i].period == mapGet(orders[i].itemId)) {
            res[m].itemId    = orders[i].itemId;
            res[m].firstYear = orders[i].period;
            res[m].units     = orders[i].units;
            res[m].unitPrice = orders[i].unitPrice;
            m++;
        }
    }
    *outLen = m;
    free(gTable);
    return res;
}

int main(void) {
    Order orders[] = {{1,100,2019,10,5000},{2,100,2020,12,5000},{7,200,2022,15,9000}};
    int outLen;
    Row *res = firstYearSales(orders, 3, &outLen);
    for (int i = 0; i < outLen; i++)
        printf("%d %d %d %d\n", res[i].itemId, res[i].firstYear, res[i].units, res[i].unitPrice);
    free(res);
    return 0;
}

#### Follow-up
questions
What if ties should collapse to a single row per item — say, the first-year sale with the most units, or with the highest total revenue (units * unit_price)?
How would you also report, in the same passes, each item's total units sold across its first year?
Could you produce the result in a single pass over the input, and what state would that require?
If orders arrive as a stream and you cannot store them all, how would you keep only what is needed to answer this query (and what makes it hard when a brand-new earlier period appears late)?

