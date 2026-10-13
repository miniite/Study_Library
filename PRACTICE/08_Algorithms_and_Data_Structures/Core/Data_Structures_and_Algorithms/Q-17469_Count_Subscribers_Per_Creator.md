---
type: "practice"
difficulty: "Other"
---

# Count Subscribers Per Creator

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Database](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Database.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

A media platform records, for every subscription relationship, which account subscribes to which account. You are given a dataset subscriptions, where each record is a pair (creator_id, subscriber_id) meaning the account subscriber_id subscribes to (follows) the account creator_id.

Each (creator_id, subscriber_id) pair is unique — the combination is a primary key, so the same subscriber never appears twice for the same creator.

For every creator that has at least one subscriber, report how many distinct subscribers that creator has. A creator appears in the output exactly once, and only creators that show up in the creator_id position of some record are included.

Return the result as a list of (creator_id, subscribers_count) rows, ordered by creator_id in ascending order.

Input: a list of (creator_id, subscriber_id) integer pairs.

Output: a list of (creator_id, subscribers_count) rows, sorted ascending by creator_id.

#### Cons
traints:

The number of records n satisfies 1 <= n <= 10^5.
1 <= creator_id, subscriber_id <= 10^9.
Each (creator_id, subscriber_id) pair is distinct.
A given account id may appear as a creator, as a subscriber, as both, or as neither in different records.

#### Example
s & test cases
TEXT
Input: subscriptions = [(10,20), (10,30), (20,10), (30,10), (30,20), (30,40)] -> Output: [(10,2), (20,1), (30,3)]
Why: creator 10 is followed by {20,30} (2); creator 20 by {10} (1); creator 30 by {10,20,40} (3); rows are sorted by creator_id.

Input: subscriptions = [(5,9)] -> Output: [(5,1)]
Why: a single relationship — creator 5 has exactly one subscriber.

Input: subscriptions = [(7,1), (7,2), (7,3), (7,4)] -> Output: [(7,4)]
Why: all records share the same creator, so only creator 7 appears, with a count of 4.

Input: subscriptions = [(3,1), (1,3), (2,3), (1,2)] -> Output: [(1,2), (2,1), (3,1)]
Why: mutual/cross follows still count once per pair; creator 1 has {3,2}, creator 2 has {3}, creator 3 has {1}; output is ascending by creator_id.

Input: subscriptions = [(100,1), (2,50), (100,2), (2,60), (2,70)] -> Output: [(2,3), (100,2)]
Why: even though creator 100's records come first in the input, the output is sorted by creator_id, so creator 2 (count 3) is listed before creator 100 (count 2).

#### Approach

This is a group-and-count aggregation. Because each (creator_id, subscriber_id) pair is guaranteed unique, every record contributes exactly one distinct subscriber to its creator, so we do not need to de-duplicate — a plain frequency count of the creator_id column gives each creator's subscriber total. Make one pass over the records incrementing a hash-map counter keyed by creator_id, then emit the (creator_id, count) rows sorted ascending by creator_id.

The canonical relational form of the same plan is:

SQL
SELECT creator_id,
       COUNT(subscriber_id) AS subscribers_count
FROM   subscriptions
GROUP BY creator_id
ORDER BY creator_id;

The GROUP BY creator_id buckets records per creator, COUNT(...) sizes each bucket, and ORDER BY creator_id gives the required ascending ordering. Only creators present in the data appear, which matches the "at least one subscriber" requirement.

Complexity

Time: O(n + u log u), where n is the number of records and u is the number of distinct creators. We scan every record once to build the per-creator counts in a hash map (O(n)), then sort the u distinct creator ids for the ascending output (O(u log u)); using a tree/ordered map folds the sort into the insertions. Space: O(u) for the counts map and the result.

Python
PYTHON
from collections import Counter

def followers_count(subscriptions):
    # subscriptions: list of (creator_id, subscriber_id) pairs
    counts = Counter(creator_id for creator_id, _subscriber in subscriptions)
    return [(creator_id, counts[creator_id]) for creator_id in sorted(counts)]

if __name__ == "__main__":
    subscriptions = [(10, 20), (10, 30), (20, 10), (30, 10), (30, 20), (30, 40)]
    for row in followers_count(subscriptions):
        print(row)
JavaScript
JAVASCRIPT
function followersCount(subscriptions) {
  // subscriptions: array of [creatorId, subscriberId] pairs

#### cons
t counts = new Map();
  for (const [creatorId] of subscriptions) {
    counts.set(creatorId, (counts.get(creatorId) || 0) + 1);
  }
  return [...counts.keys()]
    .sort((a, b) => a - b)                       // ascending by creatorId
    .map((creatorId) => [creatorId, counts.get(creatorId)]);
}

#### cons
t subscriptions = [[10, 20], [10, 30], [20, 10], [30, 10], [30, 20], [30, 40]];

#### cons
ole.log(followersCount(subscriptions));
Java
JAVA
import java.util.*;

public class SubscriberCounts {
    // returns rows {creatorId, subscribersCount} sorted ascending by creatorId
    public static List<int[]> followersCount(int[][] subscriptions) {
        Map<Integer, Integer> cnt = new TreeMap<>();     // TreeMap keeps keys sorted
        for (int[] p : subscriptions)
            cnt.merge(p[0], 1, Integer::sum);            // p[0] = creator_id

        List<int[]> res = new ArrayList<>();
        for (Map.Entry<Integer, Integer> e : cnt.entrySet())
            res.add(new int[]{e.getKey(), e.getValue()});
        return res;
    }

    public static void main(String[] args) {
        int[][] subs = {{10,20},{10,30},{20,10},{30,10},{30,20},{30,40}};
        for (int[] r : followersCount(subs))
            System.out.println("(" + r[0] + ", " + r[1] + ")");
    }
}
C++
CPP
#include <bits/stdc++.h>
using namespace std;

// each pair is (creator_id, subscriber_id)
vector<pair<int,int>> followersCount(const vector<pair<int,int>>& subscriptions) {
    unordered_map<int,int> cnt;
    for (const auto& p : subscriptions) cnt[p.first]++;   // count per creator

    vector<pair<int,int>> res(cnt.begin(), cnt.end());
    sort(res.begin(), res.end());                          // ascending by creator_id
    return res;
}

int main() {
    vector<pair<int,int>> subs = {{10,20},{10,30},{20,10},{30,10},{30,20},{30,40}};
    for (const auto& r : followersCount(subs))
        cout << "(" << r.first << ", " << r.second << ")\n";
    return 0;
}
C
C
#include <stdio.h>
#include <stdlib.h>

/* open-addressing hash map: creator_id -> subscriber count */
typedef struct { int *keys; int *vals; int cap, size; } Map;

static Map mapNew(int cap) {
    if (cap < 8) cap = 8;
    Map m; m.cap = cap; m.size = 0;
    m.keys = (int *)malloc(sizeof(int) * cap);
    m.vals = (int *)calloc(cap, sizeof(int));
    for (int i = 0; i < cap; i++) m.keys[i] = -1;
    return m;
}
static int mapSlot(Map *m, int key) {
    unsigned h = ((unsigned)key * 2654435761u) % (unsigned)m->cap;
    while (m->keys[h] != -1 && m->keys[h] != key) h = (h + 1) % m->cap;
    return (int)h;
}
static void mapInc(Map *m, int key) {
    int s = mapSlot(m, key);
    if (m->keys[s] == -1) { m->keys[s] = key; m->size++; }
    m->vals[s] += 1;
}

typedef struct { int creator; int count; } Row;
static int cmpRow(const void *a, const void *b) {
    int x = ((const Row *)a)->creator, y = ((const Row *)b)->creator;
    return (x > y) - (x < y);
}

/* pairs[i][0] = creator_id, pairs[i][1] = subscriber_id */
Row *followersCount(int pairs[][2], int n, int *returnSize) {
    Map m = mapNew(n * 2 + 8);
    for (int i = 0; i < n; i++) mapInc(&m, pairs[i][0]);

    Row *res = (Row *)malloc(sizeof(Row) * (m.size ? m.size : 1));
    int k = 0;
    for (int i = 0; i < m.cap; i++)
        if (m.keys[i] != -1) { res[k].creator = m.keys[i]; res[k].count = m.vals[i]; k++; }
    qsort(res, k, sizeof(Row), cmpRow);       /* ascending by creator_id */

    free(m.keys); free(m.vals);
    *returnSize = k;
    return res;
}

int main(void) {
    int pairs[][2] = {{10,20},{10,30},{20,10},{30,10},{30,20},{30,40}};
    int sz;
    Row *res = followersCount(pairs, 6, &sz);
    for (int i = 0; i < sz; i++) printf("(%d, %d)\n", res[i].creator, res[i].count);
    free(res);
    return 0;
}

#### Follow-up
questions
Include creators that have zero subscribers (requires the full account list and a left join / a count that defaults to 0).
Return only the top-k most-followed creators instead of every creator's count.
For each creator, also report how many of their subscribers they follow back (mutual relationships).
Drop the uniqueness guarantee on the pairs — count only distinct subscribers per creator (COUNT DISTINCT / a per-creator set).

