---
type: "practice"
difficulty: "Other"
---

# High-Volume Products Within a Target Month

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Hash Map](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Hash_Map.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

An online store keeps two collections of records.

The first collection, catalog, lists every product. Each product record has three fields: a unique integer id, a text name, and a text category. No two products share the same id.

The second collection, sales, is a log of order lines. Each order line has three fields: product_id (which refers to a product id in the catalog), order_date (a calendar date written as the string "YYYY-MM-DD"), and units (a positive integer, the number of items sold on that line). The same product may appear in many order lines, and the log may contain duplicate rows.

Given a target year, a target month, and an integer threshold, report every product whose total units sold during that single target month is at least threshold. For each qualifying product, report its name together with that monthly total. Products with no sales in the target month, or whose total falls below the threshold, are excluded. The results may be returned in any order.

Input / Output

Input: the catalog list, the sales list, and the integers year, month, threshold.
Output: a list of (name, total_units) pairs — one per qualifying product.

#### Cons
traints

0 <= number of products <= 10^5
0 <= number of order lines <= 10^5
Each order_date is a valid date string of the form "YYYY-MM-DD".
1 <= units <= 10^4 per order line; a product's monthly total fits in a 64-bit integer.
1 <= month <= 12, 1 <= threshold <= 10^9.
Every product_id in sales refers to some id in catalog.

#### Example
s & test cases
TEXT
Input:
catalog = [(1,"Wireless Mouse","Electronics"), (2,"Yoga Mat","Fitness"), (3,"Ceramic Mug","Kitchen"), (4,"Desk Lamp","Electronics"), (5,"Water Bottle","Fitness")]
sales = [(1,"2021-06-03",60), (1,"2021-06-19",70), (2,"2021-05-22",90), (2,"2021-06-14",40), (3,"2021-06-02",5), (3,"2021-06-27",8), (4,"2021-07-01",200), (5,"2021-06-10",55), (5,"2021-06-28",45), (5,"2021-07-02",50)]
year = 2021, month = 6, threshold = 100
-> Output: [("Wireless Mouse", 130), ("Water Bottle", 100)]
Why: Product 1 sells 60+70=130 in June, product 5 sells 55+45=100 (its July line is excluded). Product 2's June total is only 40, product 3's is 13, product 4 sold nothing in June.

Input:
catalog = [(7,"Notebook","Office")]
sales = []
year = 2021, month = 6, threshold = 1
-> Output: []
Why: There are no sales at all, so nothing meets the threshold.

Input:
catalog = [(1,"Gadget","Tech")]
sales = [(1,"2020-02-29",100)]
year = 2020, month = 2, threshold = 100
-> Output: [("Gadget", 100)]
Why: A single leap-day order of exactly 100 units meets the threshold (boundary case: total == threshold qualifies).

Input:
catalog = [(1,"Alpha","X"), (2,"Beta","X")]
sales = [(1,"2022-01-15",60), (2,"2022-01-31",99), (2,"2022-02-01",5)]
year = 2022, month = 1, threshold = 100
-> Output: []
Why: In January, product 1 totals 60 and product 2 totals 99; the February line for product 2 is a different month and is not counted, so neither reaches 100.

#### Approach

This is a filter-then-group-then-join task. First filter the order log to just the lines whose order_date falls in the requested year and month (compare the parsed YYYY and MM fields directly — no need to build real date objects). While filtering, aggregate units per product_id into a hash map, giving each product's total for that month in one linear pass. Then walk the catalog once; for each product look up its total (defaulting to 0 when absent) and emit (name, total) whenever it meets or exceeds the threshold. Using a hash map keeps the whole thing O(P + S) instead of the O(P * S) nested scan. In SQL this is the classic SELECT p.name, SUM(s.units) FROM catalog p JOIN sales s ON p.id = s.product_id WHERE YEAR(s.order_date) = @year AND MONTH(s.order_date) = @month GROUP BY p.id, p.name HAVING SUM(s.units) >= @threshold.

Complexity

Time: O(P + S), where P is the number of products and S the number of order lines. Each order line is examined once to accumulate its product's monthly total in a hash map, and each product is examined once to test the threshold. Space: O(K) for the hash map, where K is the number of distinct products with sales in the target month (at most min(P, S)).

Python
PYTHON
from typing import Dict, List, Tuple

def high_volume_products(catalog: List[Tuple[int, str]],
                         sales: List[Tuple[int, str, int]],
                         year: int, month: int, threshold: int) -> List[Tuple[str, int]]:
    # catalog: (id, name); sales: (product_id, order_date "YYYY-MM-DD", units)
    totals: Dict[int, int] = {}
    for product_id, order_date, units in sales:
        oy, om = int(order_date[:4]), int(order_date[5:7])
        if oy == year and om == month:
            totals[product_id] = totals.get(product_id, 0) + units

    result: List[Tuple[str, int]] = []
    for product_id, name in catalog:
        total = totals.get(product_id, 0)
        if total >= threshold:
            result.append((name, total))
    return result
JavaScript
JAVASCRIPT
function highVolumeProducts(catalog, sales, year, month, threshold) {
    // catalog: [{id, name}]; sales: [{productId, orderDate: "YYYY-MM-DD", units}]

#### cons
t totals = new Map();                       // product id -> units in target month
    for (const s of sales) {

#### cons
t oy = parseInt(s.orderDate.slice(0, 4), 10);

#### cons
t om = parseInt(s.orderDate.slice(5, 7), 10);
        if (oy === year && om === month) {
            totals.set(s.productId, (totals.get(s.productId) || 0) + s.units);
        }
    }

#### cons
t result = [];
    for (const p of catalog) {

#### cons
t total = totals.get(p.id) || 0;
        if (total >= threshold) result.push([p.name, total]);
    }
    return result;
}
Java
JAVA
import java.util.*;

class Solution {
    static class Product { int id; String name; Product(int i, String n){ id=i; name=n; } }
    static class Sale { int productId; String orderDate; int units; Sale(int p,String d,int u){ productId=p; orderDate=d; units=u; } }

    public List<Object[]> highVolumeProducts(List<Product> catalog, List<Sale> sales,
                                             int year, int month, long threshold) {
        Map<Integer, Long> totals = new HashMap<>();   // product id -> units in target month
        for (Sale s : sales) {
            int oy = Integer.parseInt(s.orderDate.substring(0, 4));
            int om = Integer.parseInt(s.orderDate.substring(5, 7));
            if (oy == year && om == month)
                totals.merge(s.productId, (long) s.units, Long::sum);
        }

        List<Object[]> result = new ArrayList<>();
        for (Product p : catalog) {
            long t = totals.getOrDefault(p.id, 0L);
            if (t >= threshold) result.add(new Object[]{ p.name, t });
        }
        return result;
    }
}
C++
CPP
#include <string>
#include <vector>
#include <unordered_map>
using namespace std;

struct Product { int id; string name; };
struct Sale { int productId; string orderDate; int units; };

vector<pair<string, long long>> highVolumeProducts(

#### cons
t vector<Product>& catalog, const vector<Sale>& sales,
        int year, int month, long long threshold) {
    auto inTargetMonth = [&] (const string& d) {
        int oy = stoi(d.substr(0, 4));
        int om = stoi(d.substr(5, 2));
        return oy == year && om == month;
    };

    unordered_map<int, long long> totals;         // product id -> units in target month
    for (const auto& s : sales)
        if (inTargetMonth(s.orderDate)) totals[s.productId] += s.units;

    vector<pair<string, long long>> result;
    for (const auto& p : catalog) {
        auto it = totals.find(p.id);
        long long t = (it == totals.end()) ? 0 : it->second;
        if (t >= threshold) result.push_back({p.name, t});
    }
    return result;
}
C
C
#include <stdio.h>
#include <stdlib.h>

typedef struct { int id; const char *name; } Product;
typedef struct { int productId; const char *orderDate; int units; } Sale;

static int inTargetMonth(const char *d, int year, int month) {
    int oy = (d[0]-'0')*1000 + (d[1]-'0')*100 + (d[2]-'0')*10 + (d[3]-'0');
    int om = (d[5]-'0')*10 + (d[6]-'0');
    return oy == year && om == month;
}

typedef struct { int key; long long val; int used; } Slot;

/* Fills outNames[]/outUnits[] and returns the number of qualifying products.
   Uses an open-addressing hash keyed by product id -> summed units. O(P + O). */
int highVolumeProducts(Product *catalog, int pCount,
                       Sale *sales, int sCount,
                       int year, int month, long long threshold,

#### cons
t char **outNames, long long *outUnits) {
    int cap = 1;
    while (cap < (sCount + 1) * 2) cap <<= 1;
    Slot *table = (Slot*)calloc(cap, sizeof(Slot));

    for (int j = 0; j < sCount; j++) {
        if (!inTargetMonth(sales[j].orderDate, year, month)) continue;
        int key = sales[j].productId;
        unsigned h = (unsigned)key & (cap - 1);
        while (table[h].used && table[h].key != key) h = (h + 1) & (cap - 1);
        table[h].used = 1;
        table[h].key = key;
        table[h].val += sales[j].units;
    }

    int found = 0;
    for (int i = 0; i < pCount; i++) {
        int key = catalog[i].id;
        unsigned h = (unsigned)key & (cap - 1);
        long long total = 0;
        while (table[h].used) {
            if (table[h].key == key) { total = table[h].val; break; }
            h = (h + 1) & (cap - 1);
        }
        if (total >= threshold) {
            outNames[found] = catalog[i].name;
            outUnits[found] = total;
            found++;
        }
    }
    free(table);
    return found;
}

#### Follow-up
questions
Support an arbitrary date range (a start and end date) rather than a single calendar month.
Also return products with zero qualifying sales (left-join semantics) with a total of 0.
Sort the output by total units descending, breaking ties alphabetically by product name.
The sales log is too large to fit in memory and arrives as a stream — compute the per-product totals in a single streaming pass with bounded extra state.

