---
type: "practice"
difficulty: "Other"
---

# Flag Structural Markers in Nucleotide Strands

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ String Processing](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/String_Processing.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given a collection of laboratory records. Each record holds a unique integer id, a strand — a non-empty string made only of the uppercase letters A, T, G, C — and an origin label (a text tag describing where the strand came from).

For every record you must report four independent boolean markers about its strand, each written as the integer 1 (present) or 0 (absent):

marker_open — 1 if the strand begins with the exact prefix ATG.
marker_close — 1 if the strand ends with any one of the three suffixes TAA, TAG, or TGA.
marker_repeat — 1 if the strand contains the substring ATAT anywhere inside it.
marker_run — 1 if the strand contains a run of at least three consecutive G characters (i.e. it contains the substring GGG; a longer run such as GGGG also counts).

The four checks are completely independent — a single strand may set any combination of them, from none to all four.

Return one output row per input record, preserving the original id, strand, and origin, and appending the four marker columns. Order the output rows by id in ascending order.

Input / Output

Input: a list of records, each (id, strand, origin).
Output: the same records annotated with marker_open, marker_close, marker_repeat, marker_run, sorted ascending by id.

#### Cons
traints

1 <= number of records <= 10^5.
Every id is unique; 1 <= id <= 10^9.
1 <= len(strand) <= 1000; each character of strand is one of A, T, G, C.
1 <= len(origin) <= 50.
A prefix/suffix marker is 0 when the strand is shorter than the pattern being tested (e.g. a length-2 strand can never open with ATG).

#### Example
s & test cases
TEXT
Input: id=10, strand="ATGCCGCGCTAA", origin="Feline" -> Output: marker_open=1, marker_close=1, marker_repeat=0, marker_run=0
  Why: begins with ATG and ends with stop suffix TAA; no ATAT substring and no run of 3 G's.

Input: id=11, strand="GGGGATCACT", origin="Feline" -> Output: 0, 0, 0, 1
  Why: only the leading run of four G's matches (GGGG contains GGG); it neither opens with ATG nor ends with a stop suffix.

Input: id=12, strand="ATATATCCGT", origin="Feline" -> Output: 0, 0, 1, 0
  Why: contains the ATAT motif, but starts with ATA (not ATG) and ends in GT.

Input: id=13, strand="ATGGGGCATTGA", origin="Canine" -> Output: 1, 1, 0, 1
  Why: starts ATG, ends TGA, and has a GGGG run — three markers set at once.

Input: id=14, strand="CACTCACTCA", origin="Canine" -> Output: 0, 0, 0, 0
  Why: matches none of the four markers (all zero) — the common "no signal" case.

Input: id=15, strand="AT", origin="Avian" -> Output: 0, 0, 0, 0
  Why: edge case — strand shorter than every pattern, so no prefix/suffix/substring can match.

#### Approach

Each of the four markers is an independent, constant-pattern string test, so no clever algorithm is needed — just direct prefix/suffix/substring matching. marker_open is a startsWith("ATG"); marker_close is a logical OR of three endsWith tests; marker_repeat is a contains("ATAT"); and "three or more consecutive G" reduces to contains("GGG") because any longer run necessarily includes GGG as a substring. Compute the flags per row, then sort the rows by id for deterministic ascending output. The underlying pattern is straightforward string-matching / linear scanning with fixed patterns.

Complexity

Time: O(n log n + n·L) — sorting n records dominates with O(n log n), and each strand of length up to L is scanned a constant number of times for the prefix, suffix, and two substring searches. Space: O(1) auxiliary beyond the output list (in-place checks; sorting is O(n) or O(log n) depending on the library).

Python
PYTHON
def annotate_sequences(records):
    """
    records: iterable of (id, strand, origin).
    Returns rows (id, strand, origin, marker_open, marker_close,
    marker_repeat, marker_run) sorted ascending by id.
    """
    stops = ("TAA", "TAG", "TGA")
    result = []
    for rid, strand, origin in sorted(records, key=lambda r: r[0]):
        marker_open   = 1 if strand.startswith("ATG") else 0
        marker_close  = 1 if strand.endswith(stops) else 0   # tuple suffix test
        marker_repeat = 1 if "ATAT" in strand else 0
        marker_run    = 1 if "GGG" in strand else 0
        result.append((rid, strand, origin,
                       marker_open, marker_close, marker_repeat, marker_run))
    return result
JavaScript
JAVASCRIPT
/**
 * records: array of { id, strand, origin }.
 * Returns the same rows annotated with four 0/1 markers, sorted asc by id.
 */
function annotateSequences(records) {

#### cons
t stops = ["TAA", "TAG", "TGA"];
  return records
    .slice()
    .sort((a, b) => (a.id < b.id ? -1 : a.id > b.id ? 1 : 0))
    .map(({ id, strand, origin }) => ({
      id,
      strand,
      origin,
      markerOpen:   strand.startsWith("ATG") ? 1 : 0,
      markerClose:  stops.some((s) => strand.endsWith(s)) ? 1 : 0,
      markerRepeat: strand.includes("ATAT") ? 1 : 0,
      markerRun:    strand.includes("GGG") ? 1 : 0,
    }));
}
Java
JAVA
import java.util.*;

public class Solution {

    public static class Row {
        long id;
        String strand, origin;
        int markerOpen, markerClose, markerRepeat, markerRun;

        public Row(long id, String strand, String origin) {
            this.id = id;
            this.strand = strand;
            this.origin = origin;
        }
    }

    // Annotates and returns the rows sorted ascending by id.
    public List<Row> annotateSequences(List<Row> records) {
        String[] stops = {"TAA", "TAG", "TGA"};
        List<Row> out = new ArrayList<>(records);
        out.sort(Comparator.comparingLong(r -> r.id));
        for (Row r : out) {
            r.markerOpen = r.strand.startsWith("ATG") ? 1 : 0;
            r.markerClose = 0;
            for (String s : stops) {
                if (r.strand.endsWith(s)) { r.markerClose = 1; break; }
            }
            r.markerRepeat = r.strand.contains("ATAT") ? 1 : 0;
            r.markerRun = r.strand.contains("GGG") ? 1 : 0;
        }
        return out;
    }
}
C++
CPP
#include <bits/stdc++.h>
using namespace std;

struct Row {
    long long id;
    string strand, origin;
    int markerOpen = 0, markerClose = 0, markerRepeat = 0, markerRun = 0;
};

static bool endsWith(const string &s, const string &suf) {
    return s.size() >= suf.size() &&
           s.compare(s.size() - suf.size(), suf.size(), suf) == 0;
}

// Annotates and returns the rows sorted ascending by id.
vector<Row> annotateSequences(vector<Row> rows) {
    sort(rows.begin(), rows.end(),
         [] (const Row &a, const Row &b) { return a.id < b.id; });

#### cons
t vector<string> stops = {"TAA", "TAG", "TGA"};
    for (Row &r : rows) {
        r.markerOpen   = r.strand.rfind("ATG", 0) == 0 ? 1 : 0;   // prefix at pos 0
        r.markerClose  = 0;
        for (const string &s : stops)
            if (endsWith(r.strand, s)) { r.markerClose = 1; break; }
        r.markerRepeat = r.strand.find("ATAT") != string::npos ? 1 : 0;
        r.markerRun    = r.strand.find("GGG")  != string::npos ? 1 : 0;
    }
    return rows;
}
C
C
#include <stdio.h>
#include <string.h>
#include <stdlib.h>

typedef struct {
    long long id;
    char strand[1024];
    char origin[64];
    int markerOpen, markerClose, markerRepeat, markerRun;
} Row;

static int starts_with(const char *s, const char *p) {
    return strncmp(s, p, strlen(p)) == 0;
}

static int ends_with(const char *s, const char *suf) {
    size_t ls = strlen(s), lp = strlen(suf);
    return ls >= lp && strcmp(s + ls - lp, suf) == 0;
}

static int cmp_id(const void *a, const void *b) {
    long long d = ((const Row *)a)->id - ((const Row *)b)->id;
    return (d > 0) - (d < 0);
}

/* Annotates rows in place and sorts them ascending by id. */
void annotateSequences(Row *rows, int n) {

#### cons
t char *stops[3] = {"TAA", "TAG", "TGA"};
    qsort(rows, n, sizeof(Row), cmp_id);
    for (int i = 0; i < n; i++) {
        Row *r = &rows[i];
        r->markerOpen   = starts_with(r->strand, "ATG");
        r->markerClose  = 0;
        for (int k = 0; k < 3; k++)
            if (ends_with(r->strand, stops[k])) { r->markerClose = 1; break; }
        r->markerRepeat = strstr(r->strand, "ATAT") != NULL ? 1 : 0;
        r->markerRun    = strstr(r->strand, "GGG")  != NULL ? 1 : 0;
    }
}

#### Follow-up
questions
Generalize marker_run to "at least k consecutive of a given character" and support an arbitrary threshold k.
Support case-insensitive strands or an extended alphabet (e.g. an ambiguity code N) without breaking the fixed patterns.
Return the count and start positions of every ATAT occurrence (including overlaps) instead of a single boolean.
Stream millions of strands that do not fit in memory: keep only the annotated flags and process rows in a single pass without a full in-memory sort.

