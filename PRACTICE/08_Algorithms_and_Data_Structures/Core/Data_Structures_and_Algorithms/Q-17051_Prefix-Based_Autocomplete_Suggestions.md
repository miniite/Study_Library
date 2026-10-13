---
type: "practice"
difficulty: "Other"
---

# Prefix-Based Autocomplete Suggestions

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Sorting](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Sorting.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are building the autocomplete box for a catalog. You are given an array of distinct strings catalog (the items that can be suggested) and a string query that a user types one character at a time, left to right.

After each character of query is typed, the box must show at most three catalog items whose prefix matches everything typed so far. Concretely, after the first k characters of query have been typed (forming the prefix query[0..k-1]), a catalog item is eligible if it starts with that exact prefix. Among all eligible items, suggest the three lexicographically smallest; if fewer than three are eligible, suggest all of them (which may be zero).

Return a list of lists: the suggestion list produced after each character of query is typed, in order. The returned outer list therefore has exactly query.length inner lists.

Input

catalog: an array of distinct lowercase strings.
query: a non-empty lowercase string, typed character by character.

Output

A list of query.length lists of strings. The k-th inner list (0-indexed) holds up to three lexicographically smallest catalog items that begin with the prefix query[0..k].

#### Cons
traints

1 <= catalog.length <= 1000
1 <= catalog[i].length <= 3000
1 <= sum of all catalog[i].length <= 20000
All strings in catalog are unique.
catalog[i] consists of lowercase English letters.
1 <= query.length <= 1000
query consists of lowercase English letters.

#### Example
s & test cases
TEXT
Input: catalog = ["cabbage","cable","cactus","candle","canvas"], query = "cable" -> Output: [["cabbage","cable","cactus"],["cabbage","cable","cactus"],["cabbage","cable"],["cable"],["cable"]]
Why: After "c" and "ca" all five start with the prefix, so the three lexicographically smallest show; "cab" narrows to two; "cabl" and "cable" narrow to one.

Input: catalog = ["orbit"], query = "orbit" -> Output: [["orbit"],["orbit"],["orbit"],["orbit"],["orbit"]]
Why: A single catalog item always keeps matching every prefix of itself, so it is suggested at every keystroke (single-item edge case).

Input: catalog = ["delta","depth","depot"], query = "deqx" -> Output: [["delta","depot","depth"],["delta","depot","depth"],[],[]]
Why: "d" and "de" match all three (shown lexicographically), but no item has 'q' at index 2, so "deq" and "deqx" yield empty suggestion lists.

Input: catalog = ["ant","anchor","angle","android","ankle"], query = "an" -> Output: [["anchor","android","angle"],["anchor","android","angle"]]
Why: More than three items share the prefix, so only the three lexicographically smallest are returned, capping the list at three.

Input: catalog = ["zebra","zero","zenith"], query = "ze" -> Output: [["zebra","zenith","zero"],["zebra","zenith","zero"]]
Why: All match on "z" and "ze"; sorted order (zebra < zenith < zero) determines the three shown, confirming lexicographic tie-breaking.

#### Approach

Sort the catalog once. In sorted order, every set of strings sharing a common prefix forms a contiguous window, and the three lexicographically smallest matches are simply the first three entries of that window. As the user types, the prefix only grows, so the matching window can only shrink from both ends and never move backward. Maintain two pointers left and right bounding the current window. At character index i with typed char ch, advance left forward while catalog[left] is too short or mismatches at index i, and pull right backward similarly; then emit up to three items starting at left. Because the pointers only move inward across the whole process, the total pointer movement is linear. Pattern: sort + shrinking two-pointer window over lexicographically ordered strings (a Trie or per-prefix binary search are equivalent alternatives).

Complexity

Time: O(L log n + n + m) where n = catalog.length, L = total length of all catalog strings, and m = query.length. Sorting dominates at O(L log n) (string comparisons cost up to their length). Space: O(1) auxiliary beyond the output (in-place sort plus two pointers), or O(L) if the sort copies. Once sorted, each keystroke only advances two pointers monotonically, so the scan across all keystrokes is amortized O(n + m).

Python
PYTHON
from typing import List

def suggest_matches(catalog: List[str], query: str) -> List[List[str]]:
    catalog.sort()
    result = []
    left, right = 0, len(catalog) - 1

    for i, ch in enumerate(query):
        while left <= right and (len(catalog[left]) <= i or catalog[left][i] != ch):
            left += 1
        while left <= right and (len(catalog[right]) <= i or catalog[right][i] != ch):
            right -= 1
        result.append(catalog[left:min(right + 1, left + 3)])

    return result
JavaScript
JAVASCRIPT
function suggestMatches(catalog, query) {
    catalog.sort();

#### cons
t result = [];
    let left = 0, right = catalog.length - 1;

    for (let i = 0; i < query.length; i++) {

#### cons
t ch = query[i];
        while (left <= right &&
               (catalog[left].length <= i || catalog[left][i] !== ch))
            left++;
        while (left <= right &&
               (catalog[right].length <= i || catalog[right][i] !== ch))
            right--;

#### cons
t row = [];
        for (let j = left; j <= right && j < left + 3; j++)
            row.push(catalog[j]);
        result.push(row);
    }
    return result;
}
Java
JAVA
import java.util.*;

class Solution {
    public List<List<String>> suggestMatches(String[] catalog, String query) {
        Arrays.sort(catalog);
        List<List<String>> result = new ArrayList<>();
        int left = 0, right = catalog.length - 1;

        for (int i = 0; i < query.length(); i++) {
            char ch = query.charAt(i);
            while (left <= right &&
                   (catalog[left].length() <= i || catalog[left].charAt(i) != ch))
                left++;
            while (left <= right &&
                   (catalog[right].length() <= i || catalog[right].charAt(i) != ch))
                right--;

            List<String> row = new ArrayList<>();
            for (int j = left; j <= right && j < left + 3; j++)
                row.add(catalog[j]);
            result.add(row);
        }
        return result;
    }
}
C++
CPP
#include <vector>
#include <string>
#include <algorithm>
using namespace std;

class Solution {
public:
    vector<vector<string>> suggestMatches(vector<string>& catalog, string query) {
        sort(catalog.begin(), catalog.end());
        vector<vector<string>> result;
        int left = 0, right = (int)catalog.size() - 1;

        for (int i = 0; i < (int)query.size(); i++) {
            char ch = query[i];
            while (left <= right &&
                   ((int)catalog[left].size() <= i || catalog[left][i] != ch))
                left++;
            while (left <= right &&
                   ((int)catalog[right].size() <= i || catalog[right][i] != ch))
                right--;

            vector<string> row;
            for (int j = left; j <= right && j < left + 3; j++)
                row.push_back(catalog[j]);
            result.push_back(row);
        }
        return result;
    }
};
C
C
#include <stdlib.h>
#include <string.h>

static int cmpStr(const void* a, const void* b) {
    return strcmp(*(const char* const*)a, *(const char* const*)b);
}

/* Returns query-length rows; each row holds up to 3 strings.
   *returnSize is set to strlen(query); (*returnColSizes)[i] is the count in row i. */
char*** suggestMatches(char** catalog, int catalogSize, char* query,
                       int* returnSize, int** returnColSizes) {
    qsort(catalog, catalogSize, sizeof(char*), cmpStr);
    int qlen = (int)strlen(query);
    *returnSize = qlen;
    *returnColSizes = (int*)malloc(sizeof(int) * qlen);
    char*** result = (char***)malloc(sizeof(char**) * qlen);

    int left = 0, right = catalogSize - 1;
    for (int i = 0; i < qlen; i++) {
        char ch = query[i];
        while (left <= right &&
               ((int)strlen(catalog[left]) <= i || catalog[left][i] != ch))
            left++;
        while (left <= right &&
               ((int)strlen(catalog[right]) <= i || catalog[right][i] != ch))
            right--;

        int cnt = 0;
        char** row = (char**)malloc(sizeof(char*) * 3);
        for (int j = left; j <= right && j < left + 3; j++)
            row[cnt++] = catalog[j];
        (*returnColSizes)[i] = cnt;
        result[i] = row;
    }
    return result;
}

#### Follow-up
questions
The catalog is fixed but many different queries stream in — build a Trie once so each keystroke answers in O(1) amortized without re-sorting.
Support hundreds of thousands of catalog items where suggestions must be ranked by popularity score, not lexicographically — how do you keep the top three per prefix?
Allow the query to backspace (delete a character) as well as type — how do you restore the previous window efficiently?
Return suggestions for a prefix appearing anywhere in the string (substring match) rather than only at the start — what data structure fits?

