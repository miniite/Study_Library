---
type: "practice"
difficulty: "Other"
---

# Attach Item Labels to Each Sale Record

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Hash Map](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Hash_Map.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You maintain two collections of records for a small retail system and need to combine them into a single readable report.

The first collection, Orders, lists individual sale events. Each order record has the following fields:

order_id — an identifier for the sale event,
item_id — the identifier of the item that was sold,
period — the calendar year in which the sale happened,
units — how many units were sold,
unit_price — the price charged per single unit.

The pair (order_id, period) uniquely identifies a row. item_id refers to an item described in the second collection. The price stored on an order is per unit.

The second collection, Catalog, describes items. Each catalog record has:

item_id — a unique identifier for the item,
item_label — the human-readable name of the item.

Task. For every order in the Orders collection, produce one output row containing three values: the label of the item that was sold, the period of the sale, and the unit price. Every order must contribute exactly one output row. Items in the catalog that were never sold do not appear in the output. The output rows may be returned in any order.

#### Cons
traints.

1 <= number of orders <= 10^5
1 <= number of catalog items <= 10^5
Each order's item_id appears exactly once in the catalog (the reference is always valid).
order_id, item_id, period, and units are integers; unit_price is a non-negative integer.
item_label is a non-empty string (at most 100 characters).
An item may be sold zero, one, or many times, in the same period or across different periods.

#### Example
s & test cases
TEXT
Input: Orders = [(1,100,2019,10,5000), (2,100,2020,12,5000), (7,200,2022,15,9000)], Catalog = {100:"Aurora Phone", 200:"Zephyr Tablet", 300:"Nimbus Watch"} -> Output: [("Aurora Phone",2019,5000), ("Aurora Phone",2020,5000), ("Zephyr Tablet",2022,9000)]
Why: Each order becomes one row; item 300 was never sold so it never appears; item 100 sold twice yields two rows.

Input: Orders = [], Catalog = {100:"Aurora Phone"} -> Output: []
Why: No orders means no rows, regardless of catalog contents.

Input: Orders = [(5,42,2021,3,1200)], Catalog = {42:"Solo Item", 99:"Unsold"} -> Output: [("Solo Item",2021,1200)]
Why: Minimal single-order case; the unsold catalog entry is ignored.

Input: Orders = [(1,7,2020,2,800), (2,7,2020,4,800)], Catalog = {7:"Twin Sale"} -> Output: [("Twin Sale",2020,800), ("Twin Sale",2020,800)]
Why: Two orders of the same item in the same period produce two independent rows (duplicates are kept).

Input: Orders = [(1,1,2000,1,0)], Catalog = {1:"Free Sample"} -> Output: [("Free Sample",2000,0)]
Why: Edge case with a zero unit price; it is a valid non-negative price and is reported as-is.

#### Approach

This is a hash join between two record sets on a shared key (item_id). Build a hash map from item_id to item_label by scanning the smaller/lookup side (Catalog) once. Then scan the driving side (Orders) once; for each order, look up its label in O(1) and emit (label, period, unit_price). Because every order's item_id is guaranteed present in the catalog, the lookup never misses. The pattern is the classic build-then-probe hash join used whenever you must enrich rows from one table with attributes from another instead of doing an O(N*C) nested-loop match. The equivalent set-based (SQL) formulation is an inner join of Orders with Catalog on item_id, projecting label, period, and price — the hash map is exactly what a query planner builds for such a join.

Complexity

Time: O(C + N), where C is the number of catalog items and N is the number of orders — one pass to index the catalog into a hash map, one pass over the orders. Space: O(C) for the item-id -> label map plus O(N) for the output rows. Hash lookups are expected O(1), so the join never degrades to the O(C*N) nested-loop cost.

Python
PYTHON
from typing import Dict, List, Tuple

def report_sales(orders: List[dict], catalog: List[dict]) -> List[Tuple[str, int, int]]:
    label: Dict[int, str] = {item["item_id"]: item["item_label"] for item in catalog}
    return [(label[o["item_id"]], o["period"], o["unit_price"]) for o in orders]

if __name__ == "__main__":
    catalog = [
        {"item_id": 100, "item_label": "Aurora Phone"},
        {"item_id": 200, "item_label": "Zephyr Tablet"},
        {"item_id": 300, "item_label": "Nimbus Watch"},
    ]
    orders = [
        {"order_id": 1, "item_id": 100, "period": 2019, "units": 10, "unit_price": 5000},
        {"order_id": 2, "item_id": 100, "period": 2020, "units": 12, "unit_price": 5000},
        {"order_id": 7, "item_id": 200, "period": 2022, "units": 15, "unit_price": 9000},
    ]
    for row in report_sales(orders, catalog):
        print(*row)
JavaScript
JAVASCRIPT
function reportSales(orders, catalog) {

#### cons
t label = new Map();
  for (const item of catalog) label.set(item.itemId, item.itemLabel);
  return orders.map((o) => [label.get(o.itemId), o.period, o.unitPrice]);
}

// demo

#### cons
t catalog = [
  { itemId: 100, itemLabel: "Aurora Phone" },
  { itemId: 200, itemLabel: "Zephyr Tablet" },
  { itemId: 300, itemLabel: "Nimbus Watch" },
];

#### cons
t orders = [
  { orderId: 1, itemId: 100, period: 2019, units: 10, unitPrice: 5000 },
  { orderId: 2, itemId: 100, period: 2020, units: 12, unitPrice: 5000 },
  { orderId: 7, itemId: 200, period: 2022, units: 15, unitPrice: 9000 },
];
for (const [label, period, price] of reportSales(orders, catalog)) {

#### cons
ole.log(label, period, price);
}
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
    static class Item {
        int itemId; String label;
        Item(int i, String l) { itemId = i; label = l; }
    }
    static class Row {
        String label; int period, unitPrice;
        Row(String l, int p, int pr) { label = l; period = p; unitPrice = pr; }
    }

    static List<Row> reportSales(List<Order> orders, List<Item> catalog) {
        Map<Integer, String> label = new HashMap<>();
        for (Item it : catalog) label.put(it.itemId, it.label);

        List<Row> res = new ArrayList<>(orders.size());
        for (Order o : orders) res.add(new Row(label.get(o.itemId), o.period, o.unitPrice));
        return res;
    }

    public static void main(String[] args) {
        List<Item> catalog = Arrays.asList(
            new Item(100, "Aurora Phone"),
            new Item(200, "Zephyr Tablet"),
            new Item(300, "Nimbus Watch"));
        List<Order> orders = Arrays.asList(
            new Order(1, 100, 2019, 10, 5000),
            new Order(2, 100, 2020, 12, 5000),
            new Order(7, 200, 2022, 15, 9000));
        for (Row r : reportSales(orders, catalog))
            System.out.println(r.label + " " + r.period + " " + r.unitPrice);
    }
}
C++
CPP
#include <bits/stdc++.h>
using namespace std;

struct Order { int orderId, itemId, period, units, unitPrice; };
struct Item  { int itemId; string label; };
struct Row   { string label; int period, unitPrice; };

vector<Row> reportSales(const vector<Order>& orders, const vector<Item>& catalog) {
    unordered_map<int, string> label;
    label.reserve(catalog.size() * 2);
    for (const auto& it : catalog) label[it.itemId] = it.label;

    vector<Row> res;
    res.reserve(orders.size());
    for (const auto& o : orders)
        res.push_back({label[o.itemId], o.period, o.unitPrice});
    return res;
}

int main() {
    vector<Item> catalog = {{100,"Aurora Phone"},{200,"Zephyr Tablet"},{300,"Nimbus Watch"}};
    vector<Order> orders = {{1,100,2019,10,5000},{2,100,2020,12,5000},{7,200,2022,15,9000}};
    for (const auto& r : reportSales(orders, catalog))
        cout << r.label << ' ' << r.period << ' ' << r.unitPrice << '\n';
    return 0;
}
C
C
#include <stdio.h>
#include <stdlib.h>

typedef struct { int orderId, itemId, period, units, unitPrice; } Order;
typedef struct { int itemId; const char *label; } Item;
typedef struct { const char *label; int period; int unitPrice; } Row;

/* open-addressing hash map: itemId -> label */
typedef struct { int key; const char *val; int used; } Slot;
static int gCap;
static Slot *gTable;

static void mapInit(int n) {
    gCap = 1;
    while (gCap < (n + 1) * 2) gCap <<= 1;
    gTable = calloc(gCap, sizeof(Slot));
}
static unsigned hashKey(int k) { return (unsigned)k * 2654435761u; }
static void mapPut(int k, const char *v) {
    unsigned i = hashKey(k) & (gCap - 1);
    while (gTable[i].used) {
        if (gTable[i].key == k) { gTable[i].val = v; return; }
        i = (i + 1) & (gCap - 1);
    }
    gTable[i].used = 1; gTable[i].key = k; gTable[i].val = v;
}
static const char *mapGet(int k) {
    unsigned i = hashKey(k) & (gCap - 1);
    while (gTable[i].used) {
        if (gTable[i].key == k) return gTable[i].val;
        i = (i + 1) & (gCap - 1);
    }
    return NULL;
}

Row *reportSales(Order *orders, int nOrders, Item *catalog, int nItems, int *outLen) {
    mapInit(nItems);
    for (int i = 0; i < nItems; i++) mapPut(catalog[i].itemId, catalog[i].label);
    Row *res = malloc(sizeof(Row) * (nOrders > 0 ? nOrders : 1));
    for (int i = 0; i < nOrders; i++) {
        res[i].label = mapGet(orders[i].itemId);
        res[i].period = orders[i].period;
        res[i].unitPrice = orders[i].unitPrice;
    }
    *outLen = nOrders;
    free(gTable);
    return res;
}

int main(void) {
    Item catalog[] = {{100,"Aurora Phone"},{200,"Zephyr Tablet"},{300,"Nimbus Watch"}};
    Order orders[] = {{1,100,2019,10,5000},{2,100,2020,12,5000},{7,200,2022,15,9000}};
    int outLen;
    Row *res = reportSales(orders, 3, catalog, 3, &outLen);
    for (int i = 0; i < outLen; i++)
        printf("%s %d %d\n", res[i].label, res[i].period, res[i].unitPrice);
    free(res);
    return 0;
}

#### Follow-up
questions
What if an order's item_id might be missing from the catalog? Distinguish inner-join behavior (drop the row) from left-join behavior (keep it with a null/placeholder label).
What if the catalog could contain duplicate item_id entries? Decide which label wins (first, last, or reject as invalid input).
How would you also list catalog items that were never sold, with empty sale fields (a full/left outer view)?
If both collections are far larger than memory, how would you perform the join (external hash partitioning or a sort-merge join)?

