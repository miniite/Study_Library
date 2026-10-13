---
type: "practice"
difficulty: "Other"
---

# Reordering Tagged Text and Number Records

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Sorting](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Sorting.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given a list of records. Each record is a single space-separated string of tokens. The first token is a tag (its unique label); every token after the tag is the record's content.

Every record is exactly one of two kinds, decided by looking at the tokens after the tag:

Text records: all content tokens consist of lowercase English letters.
Number records: all content tokens consist of digits.

Reorder the records so that:

Every text record comes before every number record.
The text records are ordered by their content, compared lexicographically over the whole content (the part after the tag). If two text records have identical content, break the tie by comparing their tags lexicographically.
The number records keep the same relative order they had in the input (their ordering is left untouched).

Return the reordered list.

Input: a list of strings records. Each string has a tag followed by at least one content token, all separated by single spaces. A record is guaranteed to be entirely a text record or entirely a number record.

Output: the list of records in the reordered arrangement described above.

#### Cons
traints:

1 <= records.length <= 100
3 <= records[i].length <= 100
Tokens within each record are separated by a single space.
Each record has a tag and at least one content token after it.

#### Example
s & test cases
TEXT
Input: records = ["num5 7 4 2 9", "wa3 red fox", "id7 sky blue moon", "num2 3 8", "wb1 red owl"] -> Output: ["wa3 red fox", "wb1 red owl", "id7 sky blue moon", "num5 7 4 2 9", "num2 3 8"]
Why: text contents sort as "red fox" < "red owl" < "sky blue moon"; the two number records keep their input order (num5 before num2).

Input: records = ["t3 go home", "t1 go home", "z9 8 2 5", "m2 fly kite"] -> Output: ["m2 fly kite", "t1 go home", "t3 go home", "z9 8 2 5"]
Why: content "fly kite" < "go home"; the two "go home" contents tie, so the tags decide and "t1" < "t3".

Input: records = ["k1 4 5 6", "k2 9 0"] -> Output: ["k1 4 5 6", "k2 9 0"]
Why: all records are number records, so the output equals the input (relative order preserved).

Input: records = ["p2 zebra", "p1 ant crawls"] -> Output: ["p1 ant crawls", "p2 zebra"]
Why: all text records; "ant crawls" < "zebra", so they sort purely by content.

Input: records = ["x1 hello world"] -> Output: ["x1 hello world"]
Why: a single record is returned unchanged.

Input: records = ["d4 5 5 5", "a2 cat", "d1 5", "b3 cat"] -> Output: ["a2 cat", "b3 cat", "d4 5 5 5", "d1 5"]
Why: text records first with equal content "cat" broken by tag ("a2" < "b3"); number records keep input order (d4 before d1).

#### Approach

Partition, then sort with a custom comparator. First split each record into "text" or "number" by inspecting a single character — the first character right after the first space. Number records are collected in input order and never sorted, which keeps their relative order intact. Text records are sorted by a two-key comparator: primary key is the content (everything after the tag), secondary key is the tag itself for content ties. Finally concatenate the sorted text records followed by the untouched number records. The core pattern is a stable/deterministic multi-key custom sort combined with a stream partition; splitting each record once into (tag, content) avoids recomputing the boundary during every comparison.

Complexity

Time: O(L * n log n), where n = number of records and L = maximum record length — partitioning into the two groups is O(n * L), and sorting the text records does O(n log n) comparisons with each string comparison costing up to O(L). Space: O(n * L) to hold the split groups and the output list (O(n) auxiliary beyond the stored strings).

Python
PYTHON
from typing import List, Tuple

class Solution:
    def reorderLogFiles(self, records: List[str]) -> List[str]:
        number_records: List[str] = []            # keep input order
        text_records: List[Tuple[str, str]] = []  # (content, tag)

        for r in records:
            i = r.index(' ')
            if r[i + 1].isdigit():
                number_records.append(r)
            else:
                text_records.append((r[i + 1:], r[:i]))

        text_records.sort()  # by content, then tag

        return [tag + ' ' + content for content, tag in text_records] + number_records
JavaScript
JAVASCRIPT
function reorderLogFiles(records) {

#### cons
t numberRecords = [];   // keep input order

#### cons
t textRecords = [];     // [tag, content]

    for (const r of records) {

#### cons
t sp = r.indexOf(' ');

#### cons
t content = r.slice(sp + 1);
        if (content[0] >= '0' && content[0] <= '9')
            numberRecords.push(r);
        else
            textRecords.push([r.slice(0, sp), content]);
    }

    textRecords.sort((a, b) => {
        if (a[1] !== b[1]) return a[1] < b[1] ? -1 : 1;     // by content
        return a[0] < b[0] ? -1 : (a[0] > b[0] ? 1 : 0);    // tie: by tag
    });

#### cons
t ans = [];
    for (const [tag, content] of textRecords) ans.push(tag + ' ' + content);
    for (const nr of numberRecords) ans.push(nr);
    return ans;
}
Java
JAVA
import java.util.*;

class Solution {
    public String[] reorderLogFiles(String[] records) {
        List<String> numberRecords = new ArrayList<>();     // keep input order
        List<String[]> textRecords = new ArrayList<>();     // {tag, content}

        for (String r : records) {
            int sp = r.indexOf(' ');
            if (Character.isDigit(r.charAt(sp + 1)))
                numberRecords.add(r);
            else
                textRecords.add(new String[] { r.substring(0, sp), r.substring(sp + 1) });
        }

        textRecords.sort((a, b) ->
            a[1].equals(b[1]) ? a[0].compareTo(b[0]) : a[1].compareTo(b[1]));

        String[] ans = new String[records.length];
        int k = 0;
        for (String[] tr : textRecords) ans[k++] = tr[0] + " " + tr[1];
        for (String nr : numberRecords) ans[k++] = nr;
        return ans;
    }
}
C++
CPP
#include <string>
#include <vector>
#include <cctype>
#include <algorithm>
using namespace std;

class Solution {
public:
    vector<string> reorderLogFiles(vector<string>& records) {
        vector<string> numberRecords;                 // keep input order
        vector<pair<string, string>> textRecords;     // (tag, content)

        for (const string& r : records) {
            size_t sp = r.find(' ');
            if (isdigit((unsigned char)r[sp + 1]))
                numberRecords.push_back(r);
            else
                textRecords.emplace_back(r.substr(0, sp), r.substr(sp + 1));
        }

        stable_sort(textRecords.begin(), textRecords.end(),
                    [] (const pair<string, string>& a, const pair<string, string>& b) {
                        return a.second != b.second ? a.second < b.second
                                                    : a.first  < b.first;
                    });

        vector<string> ans;
        ans.reserve(records.size());
        for (const auto& tr : textRecords)
            ans.push_back(tr.first + ' ' + tr.second);
        for (const string& nr : numberRecords)
            ans.push_back(nr);
        return ans;
    }
};
C
C
#include <stdlib.h>
#include <string.h>

/* Pointer to the content (the part after the first space). Every record has a
 * space per the constraints, so strchr never returns NULL. */
static const char* content_of(const char* s) {
    return strchr(s, ' ') + 1;
}

/* Order two text records: by content, then (on a tie) by tag. Comparing the
 * whole strings after the contents match compares the tags, because ' ' (0x20)
 * sorts before any letter/digit and acts as a natural end-of-tag marker. */
static int cmp_text(const void* a, const void* b) {

#### cons
t char* la = *(const char* const*)a;

#### cons
t char* lb = *(const char* const*)b;
    int r = strcmp(content_of(la), content_of(lb));
    if (r != 0) return r;
    return strcmp(la, lb);
}

char** reorderLogFiles(char** records, int recordsSize, int* returnSize) {
    char** ans     = (char**)malloc(sizeof(char*) * recordsSize);
    char** texts   = (char**)malloc(sizeof(char*) * recordsSize);
    char** numbers = (char**)malloc(sizeof(char*) * recordsSize);
    int nt = 0, nn = 0;

    for (int i = 0; i < recordsSize; ++i) {
        char c = content_of(records[i])[0];
        if (c >= '0' && c <= '9')
            numbers[nn++] = records[i];     /* keep input order */
        else
            texts[nt++] = records[i];
    }

    qsort(texts, nt, sizeof(char*), cmp_text);

    int k = 0;
    for (int i = 0; i < nt; ++i) ans[k++] = texts[i];
    for (int i = 0; i < nn; ++i) ans[k++] = numbers[i];

    free(texts); free(numbers);
    *returnSize = recordsSize;
    return ans;
}

#### Follow-up
questions
What if a tag itself may contain digits — how do you still classify a record correctly? (Look only at the content tokens after the tag.)
Sort the number records by their numeric value instead of preserving input order.
Handle records where content could mix letters and digits, and define a rule for them.
Make the comparator locale- or case-insensitive and discuss how that changes tie-breaking.

