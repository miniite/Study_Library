---
type: "practice"
difficulty: "Other"
---

# Creators Who Viewed Their Own Content

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Database](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Database.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

A platform records a log of content-view events. Each event stores four fields:

post_id — the identifier of the post that was opened.
creator_id — the identifier of the person who created that post.
reader_id — the identifier of the person who opened the post.
access_date — the calendar date on which the post was opened.

A single person may both create posts and read posts. Two identifiers refer to the same person exactly when they are equal, so a creator opened one of their own posts whenever an event has creator_id == reader_id. The log has no unique key and may contain fully duplicate rows.

Return the identifiers of every creator who has viewed at least one of their own posts. The returned identifiers must be distinct and returned in ascending order.

Input

A collection of view events, each described by post_id, creator_id, reader_id, and access_date. Equivalently, in a programming setting, two parallel integer arrays creator_id[i] and reader_id[i] describing event i.

Output

A sorted, ascending, duplicate-free list of every creator_id for which some event satisfies creator_id == reader_id.

#### Cons
traints

0 <= number of events <= 10^5
1 <= post_id, creator_id, reader_id <= 10^9
access_date is a valid calendar date.
The log may contain duplicate rows.
The result must contain no duplicate identifiers and must be sorted in ascending order.

#### Example
s & test cases
TEXT
Input: events = [(10,2,9,"2021-03-01"), (10,2,8,"2021-03-02"), (20,6,6,"2021-03-01"), (20,6,8,"2021-03-02"), (40,6,3,"2021-02-20"), (30,5,5,"2021-02-18"), (30,5,5,"2021-02-18")] -> Output: [5, 6]
Why: creator 6 opened their own post 20 (reader 6 == creator 6) and creator 5 opened their own post 30 (5 == 5); the duplicate row for creator 5 collapses to a single entry, and results are sorted.

Input: events = [] -> Output: []
Why: an empty log yields no self-viewers (min-size edge case).

Input: events = [(1,42,42,"2020-01-01")] -> Output: [42]
Why: the single event has creator == reader, so 42 is a self-viewer.

Input: events = [(1,3,5,"2020-01-01"), (2,7,8,"2020-01-02")] -> Output: []
Why: no event has creator == reader, so no creator viewed their own content.

Input: events = [(1,9,9,"2020-05-01"), (2,2,2,"2020-05-01"), (3,9,9,"2020-05-02")] -> Output: [2, 9]
Why: creators 9 and 2 both viewed their own posts; the two self-view rows for creator 9 dedupe to one entry, and output is ascending (2 before 9).

#### Approach

The condition "a creator viewed their own post" is simply creator_id == reader_id on a single row, because equal identifiers denote the same person. So the task reduces to: scan every event, keep the creator_id of each row where creator equals reader, remove duplicates, and sort ascending. A hash-set (or sorted set) handles the distinctness; a final sort (or an inherently ordered set) gives ascending order. In its native SQL form the query is:

SQL
SELECT DISTINCT creator_id AS id
FROM Views
WHERE creator_id = reader_id
ORDER BY id;

The pattern is a self-join-free row filter plus distinct-and-order — a filter/dedup/sort pipeline.

Complexity

Time: O(n log n) — one linear pass over the n events to find self-views, then de-duplication/sorting of at most n ids. Space: O(k) for the set of k distinct self-viewers (O(n) worst case). If expressed as SQL, the engine does the same work as SELECT DISTINCT ... ORDER BY.

Python
PYTHON
from typing import List

def self_viewers(creator_id: List[int], reader_id: List[int]) -> List[int]:
    # Collect distinct creators who opened their own post, then sort ascending.
    seen = {c for c, r in zip(creator_id, reader_id) if c == r}
    return sorted(seen)
JavaScript
JAVASCRIPT
function selfViewers(creatorId, readerId) {

#### cons
t seen = new Set();
    for (let i = 0; i < creatorId.length; i++) {
        if (creatorId[i] === readerId[i]) seen.add(creatorId[i]);
    }
    // Numeric ascending sort of the distinct ids.
    return [...seen].sort((a, b) => a - b);
}
Java
JAVA
import java.util.*;

class Solution {
    // TreeSet gives distinct ids in ascending order.
    public List<Integer> selfViewers(int[] creatorId, int[] readerId) {
        TreeSet<Integer> uniq = new TreeSet<>();
        for (int i = 0; i < creatorId.length; i++) {
            if (creatorId[i] == readerId[i]) uniq.add(creatorId[i]);
        }
        return new ArrayList<>(uniq);
    }
}
C++
CPP
#include <vector>
#include <set>
using namespace std;

// A std::set keeps the ids both unique and sorted ascending automatically.
vector<int> selfViewers(const vector<int>& creatorId, const vector<int>& readerId) {
    set<int> uniq;
    for (size_t i = 0; i < creatorId.size(); ++i) {
        if (creatorId[i] == readerId[i]) uniq.insert(creatorId[i]);
    }
    return vector<int>(uniq.begin(), uniq.end());
}
C
C
#include <stdlib.h>

static int cmpInt(const void *a, const void *b) {
    int x = *(const int *)a, y = *(const int *)b;
    return (x > y) - (x < y);
}

/* Returns a malloc'd, ascending, de-duplicated array of creators who viewed
   their own post. *returnSize receives the number of elements. */
int *selfViewers(const int *creatorId, const int *readerId, int n, int *returnSize) {
    int *tmp = (int *)malloc(sizeof(int) * (n > 0 ? n : 1));
    int m = 0;
    for (int i = 0; i < n; i++) {
        if (creatorId[i] == readerId[i]) tmp[m++] = creatorId[i];
    }
    qsort(tmp, m, sizeof(int), cmpInt);
    int k = 0;
    for (int i = 0; i < m; i++) {
        if (i == 0 || tmp[i] != tmp[i - 1]) tmp[k++] = tmp[i];
    }
    *returnSize = k;
    return tmp;
}

#### Follow-up
questions
Also return, for each self-viewer, how many times they viewed their own posts.
Return only creators who viewed every post they created, not just at least one.
Handle a stream/log too large to fit in memory (external dedup or hashing by shard).
Additionally report each self-viewer's most recent access_date.

