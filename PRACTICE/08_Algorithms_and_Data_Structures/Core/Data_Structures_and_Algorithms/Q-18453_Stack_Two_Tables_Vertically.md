---
type: "practice"
difficulty: "Other"
---

# Stack Two Tables Vertically

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Arrays](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Arrays.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given two tables that share the same schema. Every row in either table has three fields:

id — an integer key,
name — a short piece of text,
age — an integer.

Return a single table that contains every row of the first table, in order, immediately followed by every row of the second table, in order. This is a vertical concatenation (an append/stack): rows are not merged, deduplicated, sorted, or otherwise reordered — the second table's rows are simply placed after the first table's rows. If a row appears in both tables it must appear twice in the result.

Input

first — a list of records, each with fields id (int), name (text), age (int).
second — a second list of records with the identical schema.

Output

One list equal to all rows of first (in order) followed by all rows of second (in order).

#### Cons
traints

0 <= len(first) <= 10^4 and 0 <= len(second) <= 10^4.
id is an integer in [0, 10^9]; ids need not be unique or sorted.
name is a non-empty string of at most 50 characters.
0 <= age <= 10^3.
Duplicate rows (within or across the two tables) are kept — do not deduplicate.

#### Example
s & test cases
TEXT
- Input: `first = [(1,"Nadia",8),(2,"Omar",6),(3,"Priya",15),(4,"Quinn",17)]`, `second = [(5,"Ravi",7),(6,"Sara",7)]` -> Output: `[(1,"Nadia",8),(2,"Omar",6),(3,"Priya",15),(4,"Quinn",17),(5,"Ravi",7),(6,"Sara",7)]`
  Why: the two rows of `second` are appended after the four rows of `first`, order preserved (typical case).
- Input: `first = []`, `second = [(9,"Tom",30)]` -> Output: `[(9,"Tom",30)]`
  Why: an empty first table contributes nothing; the result is just `second` (empty-left edge case).
- Input: `first = [(1,"A",5)]`, `second = []` -> Output: `[(1,"A",5)]`
  Why: an empty second table contributes nothing; the result is just `first` (empty-right edge case).
- Input: `first = []`, `second = []` -> Output: `[]`
  Why: both tables empty produces an empty table (both-empty edge case).
- Input: `first = [(1,"X",10)]`, `second = [(1,"X",10)]` -> Output: `[(1,"X",10),(1,"X",10)]`
  Why: an identical row appearing in both tables is kept twice — no deduplication (duplicate-preservation case).

#### Approach

Allocate an output large enough for both tables (m + n rows), copy the first table's rows in order, then copy the second table's rows right after them. This is plain sequence concatenation/append — there is no join, no key matching, no sorting, and deliberately no deduplication, so duplicates and original order are preserved. Pre-sizing the result (reserve/ArrayList(capacity)/single malloc) keeps it a single linear pass with no intermediate reallocations.

Complexity

Time: O(m + n) where m = len(first) and n = len(second) — every row is copied once into the output. Space: O(m + n) for the returned table (no auxiliary structures beyond the result). Justification: concatenation is a straight copy of both inputs end to end; reserving m + n slots up front avoids repeated reallocation.

Python
PYTHON
from typing import List, Dict, Any

class Solution:
    def concatTables(self, first: List[Dict[str, Any]],
                     second: List[Dict[str, Any]]) -> List[Dict[str, Any]]:
        # each record is {"id": int, "name": str, "age": int}
        return list(first) + list(second)
JavaScript
JAVASCRIPT
// each record is { id: number, name: string, age: number }
function concatTables(first, second) {
    return first.concat(second);
}
Java
JAVA
import java.util.ArrayList;
import java.util.List;

class Row {
    int id;
    String name;
    int age;
    Row(int id, String name, int age) {
        this.id = id;
        this.name = name;
        this.age = age;
    }
}

class Solution {
    public List<Row> concatTables(List<Row> first, List<Row> second) {
        List<Row> result = new ArrayList<>(first.size() + second.size());
        result.addAll(first);
        result.addAll(second);
        return result;
    }
}
C++
CPP
#include <string>
#include <vector>
using namespace std;

struct Row {
    int id;
    string name;
    int age;
};

class Solution {
 public:
  vector<Row> concatTables(const vector<Row>& first, const vector<Row>& second) {
    vector<Row> result;
    result.reserve(first.size() + second.size());
    result.insert(result.end(), first.begin(), first.end());
    result.insert(result.end(), second.begin(), second.end());
    return result;
  }
};
C
C
#include <stdlib.h>
#include <stddef.h>

typedef struct {
    int   id;
    char *name;
    int   age;
} Row;

/* Returns a newly allocated array of length (n1 + n2); caller frees it.
   *outLen receives the total length. */
Row *concatTables(const Row *first, size_t n1,

#### cons
t Row *second, size_t n2, size_t *outLen) {
    size_t total = n1 + n2;
    Row *result = (Row *)malloc(total * sizeof(Row));
    if (result == NULL) { *outLen = 0; return NULL; }
    for (size_t i = 0; i < n1; ++i) result[i] = first[i];
    for (size_t j = 0; j < n2; ++j) result[n1 + j] = second[j];
    *outLen = total;
    return result;
}

#### Follow-up
questions
What if the two tables list their columns in a different order or have slightly different schemas — how would you align them before stacking?
How would you concatenate horizontally (pair up rows by position, widening each row) instead of vertically?
What if the result must have duplicate rows removed after stacking — how would that change the approach and complexity?
How would you efficiently concatenate k tables at once rather than just two?

