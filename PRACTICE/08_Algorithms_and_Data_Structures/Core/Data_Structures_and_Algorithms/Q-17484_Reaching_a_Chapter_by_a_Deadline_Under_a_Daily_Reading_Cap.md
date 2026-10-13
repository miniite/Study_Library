---
type: "practice"
difficulty: "Other"
---

# Reaching a Chapter by a Deadline Under a Daily Reading Cap

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Prefix Sum](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Prefix_Sum.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are reading a book that is split into chapters. You are given a 0-indexed array chapterPages of positive integers, where chapterPages[i] is the number of pages in chapter i.

You read according to these rules:

You start reading on day 0.
You may not read any page of chapter i until you have read every page of chapter i - 1 (chapters must be finished strictly in order).
You must read at least one page every day until the whole book is finished.

You are also given a list queries, where queries[j] = [chapter_j, day_j, limit_j]. For each query, decide whether it is possible to read a page belonging to chapter chapter_j on day day_j, under the extra restriction that you never read more than limit_j pages on any single day. You are free to read pages from more than one chapter on the same day, as long as the ordering rule above is respected.

Build and return a boolean array answer with answer.length == queries.length, where answer[j] is true if the j-th query is achievable and false otherwise. Each query is judged independently.

Input

chapterPages: array of positive integers (pages per chapter, read in order).
queries: list of triples [chapter, day, limit].

Output

A boolean array, one entry per query.

#### Cons
traints

1 <= chapterPages.length <= 10^5
1 <= chapterPages[i] <= 10^5
1 <= queries.length <= 10^5
queries[j].length == 3
0 <= chapter_j < chapterPages.length
0 <= day_j <= 10^9
1 <= limit_j <= 10^9

#### Example
s & test cases
TEXT
Input: chapterPages = [4,2,5,3], queries = [[0,1,2],[3,4,100],[2,2,1]] -> Output: [true,true,false]
Why: prefix = [0,4,6,11,14]. Chapter 0 spans positions 1..4; by end of day 1 you have read between 2 and 4 pages, overlapping 1..4, so true. Chapter 3 spans 12..14; by day 4 you read between 5 and 500 pages, overlapping 12..14, so true. Chapter 2 spans 7..11; cap 1 forces exactly 1 page/day, so by day 2 only 3 pages are read, short of position 7, so false.

Input: chapterPages = [10], queries = [[0,0,1]] -> Output: [true]
Why: Single chapter occupies positions 1..10; on day 0 you read exactly 1 page (at least 1, at most 1) which lands in 1..10, so true.

Input: chapterPages = [3,3], queries = [[1,5,1],[0,5,1]] -> Output: [true,false]
Why: prefix = [0,3,6]; chapter 0 spans 1..3 and chapter 1 spans 4..6. With cap 1 you read exactly 1 page/day, so by end of day 5 you have read 6 pages (position 6). Position 6 is the last page of chapter 1, so [1,5,1] is true; but chapter 0 ended at position 3, long finished by day 5, so [0,5,1] is false.

Input: chapterPages = [7,4,5,3,8], queries = [[0,2,2],[4,2,4],[2,13,1000000000]] -> Output: [true,false,true]
Why: prefix = [0,7,11,16,19,27]. Chapter 0 spans 1..7; by day 2 with cap 2 you read 3..6 pages, overlapping 1..7, so true. Chapter 4 spans 20..27; by day 2 with cap 4 you read 3..12 pages, all below 20, so false. Chapter 2 spans 12..16; by day 13 with a huge cap you read at least 14 pages and at most a billion, overlapping 12..16, so true.

#### Approach

Precompute prefix sums so that prefix[c] is the total pages in all chapters before chapter c. In 1-indexed reading order, the pages of chapter c occupy positions prefix[c] + 1 through prefix[c + 1]. Reading at least 1 and at most limit pages per day, by the end of day d (0-indexed) the total pages read lies somewhere in the interval [d + 1, (d + 1) * limit]. You can read a page of chapter c on day d exactly when that reachable interval overlaps the chapter's position range, i.e. d + 1 <= prefix[c + 1] (the slow reader has not blown past the chapter) AND (d + 1) * limit >= prefix[c] + 1 (the fast reader has at least reached its first page). Use 64-bit arithmetic because (d + 1) * limit can be as large as about 10^18. The technique is prefix sums combined with interval-overlap reasoning.

Complexity

Time: O(n + q) where n = number of chapters and q = number of queries — one pass builds the prefix sums, then each query is answered in O(1). Space: O(n) for the prefix-sum array (plus O(q) for the output).

Python
PYTHON
from itertools import accumulate
from typing import List

class Solution:
    def canReachChapter(self, chapterPages: List[int],
                        queries: List[List[int]]) -> List[bool]:
        # prefix[i] = total pages before chapter i
        prefix = [0] + list(accumulate(chapterPages))

        ans = []
        for chapter, day, limit in queries:
            min_read = day + 1
            max_read = (day + 1) * limit
            # Chapter occupies positions prefix[chapter]+1 .. prefix[chapter+1]
            ok = (min_read <= prefix[chapter + 1]) and (max_read >= prefix[chapter] + 1)
            ans.append(ok)
        return ans
JavaScript
JAVASCRIPT
function canReachChapter(chapterPages, queries) {

#### cons
t n = chapterPages.length;
    // prefix uses BigInt-safe numbers; values stay well within 2^53.

#### cons
t prefix = new Array(n + 1);
    prefix[0] = 0;
    for (let i = 0; i < n; i++) prefix[i + 1] = prefix[i] + chapterPages[i];

#### cons
t ans = [];
    for (const [chapter, day, limit] of queries) {

#### cons
t minRead = day + 1;

#### cons
t maxRead = (day + 1) * limit; // <= (1e9+1)*1e9 < 2^60, but < 2^53? use care
        // (day+1)*limit can exceed 2^53; compare via the two clean inequalities.

#### cons
t reachStart = maxRead >= prefix[chapter] + 1;

#### cons
t reachEnd = minRead <= prefix[chapter + 1];
        ans.push(reachStart && reachEnd);
    }
    return ans;
}
Java
JAVA
class Solution {
    public boolean[] canReachChapter(int[] chapterPages, int[][] queries) {
        int n = chapterPages.length;
        long[] prefix = new long[n + 1];
        for (int i = 0; i < n; i++)
            prefix[i + 1] = prefix[i] + chapterPages[i];

        boolean[] ans = new boolean[queries.length];
        for (int j = 0; j < queries.length; j++) {
            long chapter = queries[j][0];
            long day = queries[j][1];
            long limit = queries[j][2];
            long minRead = day + 1;
            long maxRead = (day + 1) * limit;
            // Chapter occupies positions prefix[chapter]+1 .. prefix[chapter+1].
            ans[j] = (minRead <= prefix[(int) chapter + 1]) &&
                     (maxRead >= prefix[(int) chapter] + 1);
        }
        return ans;
    }
}
C++
CPP
#include <vector>
using namespace std;

class Solution {
public:
    vector<bool> canReachChapter(vector<int>& chapterPages,
                                 vector<vector<int>>& queries) {
        int n = (int)chapterPages.size();
        vector<long long> prefix(n + 1, 0);
        for (int i = 0; i < n; i++)
            prefix[i + 1] = prefix[i] + chapterPages[i];

        vector<bool> ans;
        ans.reserve(queries.size());
        for (auto& q : queries) {
            long long chapter = q[0], day = q[1], limit = q[2];
            long long minRead = day + 1;
            long long maxRead = (day + 1) * limit;
            // Chapter occupies positions prefix[chapter]+1 .. prefix[chapter+1].
            bool ok = (minRead <= prefix[chapter + 1]) &&
                      (maxRead >= prefix[chapter] + 1);
            ans.push_back(ok);
        }
        return ans;
    }
};
C
C
#include <stdlib.h>
#include <stdbool.h>

/* prefix[i] = total pages before chapter i.
   Chapter c occupies 1-indexed reading positions prefix[c]+1 .. prefix[c+1].
   By end of day d (0-indexed) pages read is in [d+1, (d+1)*limit].
   Reachable iff (d+1) <= prefix[c+1]  AND  (long)(d+1)*limit >= prefix[c]+1. */
bool* canReachChapter(int* chapterPages, int chapterPagesSize,
                      int** queries, int queriesSize, int* queriesColSize,
                      int* returnSize) {
    long long* prefix = (long long*)malloc((chapterPagesSize + 1) * sizeof(long long));
    prefix[0] = 0;
    for (int i = 0; i < chapterPagesSize; i++)
        prefix[i + 1] = prefix[i] + chapterPages[i];

    bool* ans = (bool*)malloc(queriesSize * sizeof(bool));
    for (int j = 0; j < queriesSize; j++) {
        long long chapter = queries[j][0];
        long long day = queries[j][1];
        long long limit = queries[j][2];
        long long minEaten = day + 1;
        long long maxEaten = (day + 1) * limit;
        ans[j] = (minEaten <= prefix[chapter + 1]) &&
                 (maxEaten >= prefix[chapter] + 1);
    }

    free(prefix);
    *returnSize = queriesSize;
    return ans;
}

#### Follow-up
questions
Why is 64-bit arithmetic necessary here, and which single subexpression overflows a 32-bit integer?
Reframe the two inequalities as an interval-overlap test — what are the two intervals, and why does overlap capture "can I be reading this chapter on this day"?
How would the answer change if you were allowed to read zero pages on some days (not "at least one")?
If instead you had to read exactly the same number of pages every day, how would each query be decided?

