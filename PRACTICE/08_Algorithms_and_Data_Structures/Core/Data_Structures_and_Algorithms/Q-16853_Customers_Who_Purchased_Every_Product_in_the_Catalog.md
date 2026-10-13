---
type: "practice"
difficulty: "Other"
---

# Customers Who Purchased Every Product in the Catalog

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Hash Map](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Hash_Map.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

A store keeps a log of purchases and a catalog of products.

The purchase log is a list of records, each recording that one customer bought one product. It is given as a list of (customerId, productId) pairs. The log may contain duplicate records (a customer can buy the same product more than once, and each purchase is logged separately).
The catalog is the list of all productId values the store sells, given as a list of distinct product ids.

Report the id of every customer who has bought all of the products in the catalog (each catalog product at least once). A customer's repeated purchases of the same product still count as buying that one product a single time toward this requirement.

Return the qualifying customer ids in any order. If no customer bought the whole catalog, return an empty list.

Input
purchases: a list of (customerId, productId) integer pairs; may contain duplicates. Every productId here appears in the catalog.
catalog: a list of distinct integer product ids (the complete set of products).
Output
A list of the customer ids that purchased every product in the catalog.

#### Cons
traints
0 <= purchases.length <= 10^5
1 <= catalog.length <= 10^4
All product ids in purchases also appear in catalog.
Customer ids and product ids are positive integers.

#### Example
s & test cases
TEXT
- Input: `purchases = [(100,10),(100,20),(100,30),(200,10),(200,20),(300,30),(300,10),(300,20),(300,10)], catalog = [10,20,30]` -> Output: `[100, 300]`
  Why: 100 bought all three; 300 bought all three (the duplicate `(300,10)` counts once); 200 bought only 2 of 3. Normal case with duplicates.
- Input: `purchases = [(1,5),(2,5)], catalog = [5]` -> Output: `[1, 2]`
  Why: with a single-product catalog, every buyer of that product qualifies. Single-product edge case.
- Input: `purchases = [(7,10),(7,10)], catalog = [10,20]` -> Output: `[]`
  Why: customer 7 bought product 10 twice but never bought 20, so no one bought the whole catalog. Duplicates-but-incomplete edge case.
- Input: `purchases = [], catalog = [1,2,3]` -> Output: `[]`
  Why: no purchases at all means nobody bought everything. Empty-log edge case.
- Input: `purchases = [(9,1),(9,2),(9,3),(9,4)], catalog = [1,2,3,4]` -> Output: `[9]`
  Why: one customer covering the full catalog exactly. Single-qualifier case.

#### Approach

This is a relational division: find every customer whose set of purchased products contains the entire catalog. Because all logged products are guaranteed to be catalog products, a customer bought the whole catalog exactly when the number of distinct products they bought equals the catalog size. So group the purchase log by customer, collapse each customer's products into a set (which automatically discards duplicate purchases), and keep the customers whose set size matches the total product count. A hash map from customer id to a hash set of product ids does the grouping and de-duplication in one linear pass; comparing each set's size against the catalog size selects the answers. The underlying pattern is hash-based grouping and aggregation with set-membership counting.

Complexity

Time: O(P + C), where P = purchases.length and C = catalog.length. Building the catalog set is O(C); scanning every purchase and inserting into its customer's product set is O(P) expected with hashing; the final sweep over customers is O(number of distinct (customer, product) pairs) <= O(P). Space: O(C + D), where D is the number of distinct (customer, product) pairs stored across all customers, bounded by P. (The C-language variant sorts instead of hashing, giving O(P log P + C log C) time.)

Python
PYTHON
from collections import defaultdict
from typing import List, Tuple

class Solution:
    def customersWhoBoughtAll(self, purchases: List[Tuple[int, int]],
                              catalog: List[int]) -> List[int]:
        cat = set(catalog)
        bought = defaultdict(set)
        for cust, prod in purchases:
            if prod in cat:
                bought[cust].add(prod)
        need = len(cat)
        return [cust for cust, prods in bought.items() if len(prods) == need]
JavaScript
JAVASCRIPT
function customersWhoBoughtAll(purchases, catalog) {

#### cons
t cat = new Set(catalog);

#### cons
t bought = new Map();
    for (const [cust, prod] of purchases) {
        if (!cat.has(prod)) continue;
        if (!bought.has(cust)) bought.set(cust, new Set());
        bought.get(cust).add(prod);
    }

#### cons
t need = cat.size;

#### cons
t res = [];
    for (const [cust, prods] of bought)
        if (prods.size === need) res.push(cust);
    return res;
}
Java
JAVA
import java.util.*;

class Solution {
    public List<Integer> customersWhoBoughtAll(int[][] purchases, int[] catalog) {
        Set<Integer> cat = new HashSet<>();
        for (int c : catalog) cat.add(c);

        Map<Integer, Set<Integer>> bought = new HashMap<>();
        for (int[] pr : purchases)
            if (cat.contains(pr[1]))
                bought.computeIfAbsent(pr[0], x -> new HashSet<>()).add(pr[1]);

        List<Integer> res = new ArrayList<>();
        for (Map.Entry<Integer, Set<Integer>> e : bought.entrySet())
            if (e.getValue().size() == cat.size()) res.add(e.getKey());
        return res;
    }
}
C++
CPP
#include <vector>
#include <utility>
#include <unordered_map>
#include <unordered_set>
using namespace std;

class Solution {
public:
    vector<int> customersWhoBoughtAll(const vector<pair<int,int>>& purchases,

#### cons
t vector<int>& catalog) {
        unordered_set<int> cat(catalog.begin(), catalog.end());
        unordered_map<int, unordered_set<int>> bought;
        for (const auto& pr : purchases)
            if (cat.count(pr.second)) bought[pr.first].insert(pr.second);

        size_t need = cat.size();
        vector<int> res;
        for (const auto& kv : bought)
            if (kv.second.size() == need) res.push_back(kv.first);
        return res;
    }
};
C
C
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

typedef struct { int cust; int prod; } Pair;

static int cmpInt(const void *a, const void *b) {
    return (*(const int *)a > *(const int *)b) - (*(const int *)a < *(const int *)b);
}
static int cmpPair(const void *a, const void *b) {

#### cons
t Pair *x = (const Pair *)a, *y = (const Pair *)b;
    if (x->cust != y->cust) return (x->cust > y->cust) - (x->cust < y->cust);
    return (x->prod > y->prod) - (x->prod < y->prod);
}
static int inCatalog(const int *cat, int m, int key) {
    int lo = 0, hi = m - 1;
    while (lo <= hi) {
        int mid = (lo + hi) / 2;
        if (cat[mid] == key) return 1;
        if (cat[mid] < key) lo = mid + 1; else hi = mid - 1;
    }
    return 0;
}

/* Caller frees the returned array; *returnSize holds its length. */
int *customersWhoBoughtAll(const int *custId, const int *prodId, int purchaseCount,

#### cons
t int *catalog, int catalogSize, int *returnSize) {
    int *cat = (int *)malloc(sizeof(int) * (catalogSize > 0 ? catalogSize : 1));
    memcpy(cat, catalog, sizeof(int) * catalogSize);
    qsort(cat, catalogSize, sizeof(int), cmpInt);

    Pair *p = (Pair *)malloc(sizeof(Pair) * (purchaseCount > 0 ? purchaseCount : 1));
    int cntP = 0;
    for (int i = 0; i < purchaseCount; i++)
        if (inCatalog(cat, catalogSize, prodId[i])) {
            p[cntP].cust = custId[i];
            p[cntP].prod = prodId[i];
            cntP++;
        }
    qsort(p, cntP, sizeof(Pair), cmpPair);

    int *out = (int *)malloc(sizeof(int) * (cntP > 0 ? cntP : 1));
    int cnt = 0, i = 0;
    while (i < cntP) {
        int j = i, cust = p[i].cust, distinct = 0;
        while (j < cntP && p[j].cust == cust) {
            if (j == i || p[j].prod != p[j - 1].prod) distinct++;
            j++;
        }
        if (distinct == catalogSize) out[cnt++] = cust;
        i = j;
    }
    free(cat);
    free(p);
    *returnSize = cnt;
    return out;
}

#### Follow-up
questions
Instead of customers who bought everything, return those who bought at least K distinct catalog products.
The catalog has 10^7 products but each fits in a compact id range — how would per-customer bitsets cut memory versus hash sets?
For each qualifying-or-not customer, also report exactly which catalog products they are missing.
The purchase log arrives as an unbounded stream — how do you maintain the answer incrementally as records flow in?

