---
type: "practice"
difficulty: "Other"
---

# Keep the First Record for Each Key

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Hash Set](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Hash_Set.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given a table of records that were collected in a fixed order. Every record has exactly three fields:

id — a 32-bit integer identifier,
label — a short text field (a display name),
key — a short text field used to detect repeats (for example, a normalized contact string).

Some records repeat the same key. A record is considered a duplicate if a record with the same key appeared earlier in the table.

Write a function that removes every duplicate record, keeping only the first record for each distinct key. The surviving records must be returned in their original relative order. Key comparison is exact and case-sensitive (two keys are equal only if they are identical character-for-character).

Input: the list of records in processing order. Output: the filtered list of records — the first record seen for each distinct key, in original order.

#### Cons
traints

0 <= number of records <= 10^5
each id fits in a signed 32-bit integer (ids are not guaranteed unique and are irrelevant to deduplication)
1 <= len(label) <= 100, 1 <= len(key) <= 100
label and key contain printable characters; keys are compared as exact byte strings.

#### Example
s & test cases
TEXT
- Input: `[(1,"Ivy","k-emily"),(2,"Dan","k-mike"),(3,"Zoe","k-sara"),(4,"Ada","k-john"),(5,"Finn","k-john"),(6,"Val","k-ada")]` -> Output: `[(1,"Ivy","k-emily"),(2,"Dan","k-mike"),(3,"Zoe","k-sara"),(4,"Ada","k-john"),(6,"Val","k-ada")]`
  Why: record 5 repeats `k-john` first seen at record 4, so it is dropped; everything else is a first occurrence. (normal case)
- Input: `[(10,"A","x"),(11,"B","y"),(12,"C","z")]` -> Output: `[(10,"A","x"),(11,"B","y"),(12,"C","z")]`
  Why: all keys are distinct, so nothing is removed. (no-duplicate case)
- Input: `[]` -> Output: `[]`
  Why: an empty table produces an empty result. (empty edge case)
- Input: `[(1,"P","same"),(2,"Q","same"),(3,"R","same"),(4,"S","same")]` -> Output: `[(1,"P","same")]`
  Why: every record shares one key, so only the very first survives. (all-duplicate edge case)
- Input: `[(1,"A","a"),(2,"B","b"),(3,"C","a"),(4,"D","b"),(5,"E","c")]` -> Output: `[(1,"A","a"),(2,"B","b"),(5,"E","c")]`
  Why: keys `a` and `b` first appear at records 1 and 2; later repeats (3,4) are dropped; `c` is new. (interleaved duplicates)
- Input: `[(1,"A","Key"),(2,"B","key")]` -> Output: `[(1,"A","Key"),(2,"B","key")]`
  Why: comparison is case-sensitive, so `Key` and `key` are different keys. (case-sensitivity edge case)

#### Approach

Deduplication by first occurrence is a single-pass hashing problem. Maintain a hash set of the keys already emitted. Walk the records in order; for each record, attempt to insert its key into the set — if the insert succeeds (the key was not present) it is the first occurrence, so append the record to the output; if it fails, the key was seen earlier, so skip the record. Because we scan left to right and only ever emit on the first sighting, original order is preserved automatically and the "keep first" rule is satisfied without any sorting.

Complexity

Time: O(n) average — one linear pass, with O(1) average-time hash-set membership/insertion per record (n = number of records). Space: O(k) for the set of distinct keys plus O(n) for the output (k <= n). Worst-case hashing is O(n) per op, but with a well-sized table the expected total is O(n).

Python
PYTHON
def keep_first_per_key(rows):
    """rows: list of (id, label, key). Returns rows with only the first
    occurrence of each key, in original order."""
    seen = set()
    out = []
    for row in rows:
        key = row[2]
        if key not in seen:
            seen.add(key)
            out.append(row)
    return out

if __name__ == "__main__":
    data = [
        (1, "Ivy", "k-emily"), (2, "Dan", "k-mike"), (3, "Zoe", "k-sara"),
        (4, "Ada", "k-john"),  (5, "Finn", "k-john"), (6, "Val", "k-ada"),
    ]
    for r in keep_first_per_key(data):
        print(r[0], r[1], r[2])
JavaScript
JAVASCRIPT
// rows: array of { id, label, key }. Returns rows with only the first
// occurrence of each key, in original order.
function keepFirstPerKey(rows) {

#### cons
t seen = new Set();

#### cons
t out = [];
  for (const row of rows) {
    if (!seen.has(row.key)) {
      seen.add(row.key);
      out.push(row);
    }
  }
  return out;
}

#### cons
t data = [
  { id: 1, label: "Ivy", key: "k-emily" },
  { id: 2, label: "Dan", key: "k-mike" },
  { id: 3, label: "Zoe", key: "k-sara" },
  { id: 4, label: "Ada", key: "k-john" },
  { id: 5, label: "Finn", key: "k-john" },
  { id: 6, label: "Val", key: "k-ada" },
];
for (const r of keepFirstPerKey(data)) {

#### cons
ole.log(r.id, r.label, r.key);
}
Java
JAVA
import java.util.*;

public class Solution {
    public static class Row {
        public int id;
        public String label, key;
        public Row(int id, String label, String key) {
            this.id = id; this.label = label; this.key = key;
        }
    }

    // Keeps only the first row per distinct key, preserving original order.
    public static List<Row> keepFirstPerKey(List<Row> rows) {
        Set<String> seen = new HashSet<>();
        List<Row> out = new ArrayList<>();
        for (Row r : rows) {
            if (seen.add(r.key)) out.add(r);   // add() returns false if already present
        }
        return out;
    }

    public static void main(String[] args) {
        List<Row> rows = Arrays.asList(
            new Row(1, "Ivy", "k-emily"), new Row(2, "Dan", "k-mike"),
            new Row(3, "Zoe", "k-sara"),  new Row(4, "Ada", "k-john"),
            new Row(5, "Finn", "k-john"), new Row(6, "Val", "k-ada")
        );
        for (Row r : keepFirstPerKey(rows))
            System.out.println(r.id + " " + r.label + " " + r.key);
    }
}
C++
CPP
#include <bits/stdc++.h>
using namespace std;

struct Row {
    int    id;
    string label;
    string key;
};

// Keeps only the first row seen for each distinct key, in original order.
vector<Row> keepFirstPerKey(const vector<Row>& rows) {
    unordered_set<string> seen;
    seen.reserve(rows.size() * 2);
    vector<Row> out;
    out.reserve(rows.size());
    for (const auto& r : rows) {
        if (seen.insert(r.key).second)   // true only when newly inserted
            out.push_back(r);
    }
    return out;
}

int main() {
    vector<Row> rows = {
        {1, "Ivy", "k-emily"}, {2, "Dan", "k-mike"}, {3, "Zoe", "k-sara"},
        {4, "Ada", "k-john"},  {5, "Finn", "k-john"}, {6, "Val", "k-ada"}
    };
    for (const auto& r : keepFirstPerKey(rows))
        cout << r.id << ' ' << r.label << ' ' << r.key << '\n';
    return 0;
}
C
C
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

typedef struct {
    int   id;
    char *label;
    char *key;
} Row;

/* Open-addressing (linear-probe) hash set of string keys. */
typedef struct {
    char **slots;
    int    cap;
} StrSet;

static unsigned long hash_str(const char *s) {
    unsigned long h = 1469598103934665603UL;      /* FNV-1a */
    while (*s) { h ^= (unsigned char)*s++; h *= 1099511628211UL; }
    return h;
}

/* Returns 1 if key was newly inserted, 0 if it was already present. */
static int set_add(StrSet *set, char *key) {
    unsigned long h = hash_str(key) % (unsigned long)set->cap;
    while (set->slots[h] != NULL) {
        if (strcmp(set->slots[h], key) == 0) return 0;
        h = (h + 1) % set->cap;
    }
    set->slots[h] = key;
    return 1;
}

/* Keeps the first row per key; writes survivors into out (size >= n); returns count. */
int keep_first_per_key(const Row *rows, int n, Row *out) {
    StrSet set;
    set.cap   = n * 2 + 1;
    set.slots = calloc(set.cap, sizeof(char *));
    int m = 0;
    for (int i = 0; i < n; i++) {
        if (set_add(&set, rows[i].key)) out[m++] = rows[i];
    }
    free(set.slots);
    return m;
}

int main(void) {
    Row rows[] = {
        {1, "Ivy", "k-emily"}, {2, "Dan", "k-mike"}, {3, "Zoe", "k-sara"},
        {4, "Ada", "k-john"},  {5, "Finn", "k-john"}, {6, "Val", "k-ada"}
    };
    int n = 6;
    Row out[6];
    int m = keep_first_per_key(rows, n, out);
    for (int i = 0; i < m; i++)
        printf("%d %s %s\n", out[i].id, out[i].label, out[i].key);
    return 0;
}

#### Follow-up
questions
Keep the last occurrence of each key instead of the first (hint: reverse-scan, or overwrite positions and compact).
Deduplicate on a composite key (multiple fields) or on a case-insensitive / whitespace-normalized key.
Do it as a streaming operation over data too large for memory (bounded memory, approximate with a Bloom filter, or external sort).
Return the count of duplicates removed per key in addition to the filtered rows.

