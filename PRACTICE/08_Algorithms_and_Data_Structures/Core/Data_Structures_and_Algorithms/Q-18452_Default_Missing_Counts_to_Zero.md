---
type: "practice"
difficulty: "Other"
---

# Default Missing Counts to Zero

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Arrays](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Arrays.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given a table of inventory records. Each record has three fields:

label — a short piece of text naming the item,
count — an integer giving how many units are on hand, which may be missing/unknown for some records,
value — an integer giving the item's unit value.

Some records do not record a count (that field is absent/null). Return the same table, in the same order, with every missing count replaced by 0. Every other field, and every count that is already present (including a legitimately recorded 0), must be left exactly as it is. No rows are added, removed, or reordered.

Input

A list of n records; each record has label (text), count (an integer or a "missing" marker), and value (an integer).

Output

The list of records in the original order, with each missing count set to 0.

#### Cons
traints

0 <= n <= 10^4
label is a non-empty string of at most 50 characters.
When present, 0 <= count <= 10^6.
0 <= value <= 10^6.
The relative order of the records must be preserved; a present count of 0 is a real value, not "missing".

#### Example
s & test cases
TEXT
- Input: `[("Wristwatch", missing, 135), ("WirelessEarbuds", missing, 821), ("GolfClubs", 779, 9319), ("Printer", 849, 3051)]` -> Output: `[("Wristwatch", 0, 135), ("WirelessEarbuds", 0, 821), ("GolfClubs", 779, 9319), ("Printer", 849, 3051)]`
  Why: the two records with a missing count get `0`; the two present counts stay untouched (typical case).
- Input: `[]` -> Output: `[]`
  Why: an empty table has nothing to fill and is returned unchanged (empty edge case).
- Input: `[("Lamp", 0, 500)]` -> Output: `[("Lamp", 0, 500)]`
  Why: a recorded `0` is a real value, not a missing marker, so it is left as-is (distinguishes `0` from "missing").
- Input: `[("Cable", missing, 0)]` -> Output: `[("Cable", 0, 0)]`
  Why: the missing count is filled even when the item's value happens to be `0`.
- Input: `[("A", 5, 10), ("B", 12, 20), ("C", 7, 30)]` -> Output: `[("A", 5, 10), ("B", 12, 20), ("C", 7, 30)]`
  Why: no counts are missing, so the table comes back identical (no-op case).

#### Approach

Scan the records once. For each row, test whether its nullable count field is absent (the "missing" marker: None/null/nullopt/null Integer/a has_count == false flag depending on the language) and, if so, assign the fixed default 0; otherwise leave it. This is the classic "coalesce a nullable column to a default value" data-cleaning pattern — preserve order and all other fields, and treat an explicitly recorded 0 as a real value rather than as missing. The only per-language subtlety is how "missing" is represented: optional/nullable wrapper/sentinel flag.

Complexity

Time: O(n) — a single pass inspects each of the n records exactly once. Space: O(1) extra — the fill is done in place; only a constant amount of bookkeeping is used (returning a fresh copy instead would cost O(n) output space). Justification: replacing a missing marker with 0 is constant work per record.

Python
PYTHON
from typing import List, Optional, Dict, Any

class Solution:
    def fillMissingCounts(self, records: List[Dict[str, Any]]) -> List[Dict[str, Any]]:
        # each record is {"label": str, "count": Optional[int], "value": int}
        for r in records:
            if r["count"] is None:      # None marks a missing count
                r["count"] = 0
        return records
JavaScript
JAVASCRIPT
// each record is { label: string, count: number|null, value: number }
function fillMissingCounts(records) {
    for (const r of records) {
        if (r.count === null || r.count === undefined) {
            r.count = 0;
        }
    }
    return records;
}
Java
JAVA
import java.util.List;

class Item {
    String label;
    Integer count;   // null == missing
    int value;
    Item(String label, Integer count, int value) {
        this.label = label;
        this.count = count;
        this.value = value;
    }
}

class Solution {
    public List<Item> fillMissingCounts(List<Item> records) {
        for (Item r : records) {
            if (r.count == null) {
                r.count = 0;
            }
        }
        return records;
    }
}
C++
CPP
#include <optional>
#include <string>
#include <vector>
using namespace std;

struct Item {
    string label;
    optional<int> count;  // nullopt == missing
    int value;
};

class Solution {
 public:
  vector<Item> fillMissingCounts(vector<Item> records) {
    for (auto &r : records) {
      if (!r.count.has_value()) {
        r.count = 0;
      }
    }
    return records;
  }
};
C
C
#include <stdbool.h>
#include <stddef.h>

typedef struct {
    char *label;
    int   count;      /* meaningful only when has_count is true */
    bool  has_count;  /* false == the count is missing            */
    int   value;
} Item;

/* Replaces every missing count with 0, in place, preserving order. */
void fillMissingCounts(Item *records, size_t n) {
    for (size_t i = 0; i < n; ++i) {
        if (!records[i].has_count) {
            records[i].count = 0;
            records[i].has_count = true;
        }
    }
}

#### Follow-up
questions
What if several columns can be missing, each with its own default value — how do you generalize the pass?
What if you must not mutate the input and must return a new copy while keeping the original intact?
How would you fill missing counts in a table too large to hold in memory (streamed row by row)?
What if "missing" is encoded as an empty string or NaN instead of null — how does the detection change?

