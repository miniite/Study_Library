---
type: "practice"
difficulty: "Other"
---

# Books Read by Members at an Exact Network Distance

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Breadth-First Search](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Breadth-First_Search.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

A reading community has n members, numbered from 0 to n - 1. You are given two arrays:

readBooks, where readBooks[i] is the list of book titles that member i has read.
connections, where connections[i] is the list of members who are directly connected to member i. Connections are mutual (undirected): if j appears in connections[i], then i appears in connections[j].

Define the network distance between two members as the number of edges on the shortest connection path between them.

Given your own member id start and a target distance distance, consider every member whose shortest network distance from start is exactly distance. Collect all book titles read by those members (a title read by two such members is counted twice, once per member).

Return the list of these book titles ordered by how many times they were counted, from least frequent to most frequent. Ties (titles with the same count) must be broken by ordering the titles alphabetically from smallest to largest.

The member start itself is at distance 0, their direct connections are at distance 1, the connections of those connections are at distance 2, and so on. Only members at distance exactly equal to distance contribute.

#### Cons
traints

n == readBooks.length == connections.length
2 <= n <= 100
1 <= readBooks[i].length <= 100
1 <= readBooks[i][j].length <= 8, each title consists of lowercase/uppercase letters.
0 <= connections[i].length < n
0 <= connections[i][j] < n, and the graph contains no self-loops or duplicate edges.
0 <= start < n
1 <= distance < n
The connection graph is undirected: if connections[i] contains j, then connections[j] contains i.

#### Example
s & test cases
TEXT
Input: readBooks = [["Atlas","Beacon"],["Cipher"],["Beacon","Cipher"],["Delta"]], connections = [[1,2],[0,3],[0,3],[1,2]], start = 0, distance = 1 -> Output: ["Beacon","Cipher"]
Why: Members at distance 1 from 0 are {1,2}. Titles: Cipher (from 1), Beacon+Cipher (from 2) -> Beacon:1, Cipher:2. Sorted by count ascending gives Beacon then Cipher.

Input: readBooks = [["Atlas","Beacon"],["Cipher"],["Beacon","Cipher"],["Delta"]], connections = [[1,2],[0,3],[0,3],[1,2]], start = 0, distance = 2 -> Output: ["Delta"]
Why: The only member at distance 2 is member 3, whose single title Delta has count 1.

Input: readBooks = [["one"],["two"],["one"]], connections = [[1],[0,2],[1]], start = 0, distance = 2 -> Output: ["one"]
Why: Member 2 is the only one at distance 2; "one" appears once.

Input: readBooks = [["aa","bb"],["bb"],["aa"]], connections = [[1,2],[0],[0]], start = 0, distance = 1 -> Output: ["aa","bb"]
Why: Members 1 and 2 are at distance 1. Counts: aa:1, bb:1. Equal counts, so alphabetical order aa then bb.

Input: readBooks = [["x"],["y"],["z"]], connections = [[1],[0],[]], start = 0, distance = 2 -> Output: []
Why: No member is reachable at distance exactly 2 from 0 (member 2 is disconnected), so the result is empty.

#### Approach

This is a shortest-path-by-hops problem on an unweighted undirected graph, so a single breadth-first search from start labels every member with its exact distance. Sweep the members whose distance equals the target, tally each book title in a hash map to get frequencies, then sort the distinct titles with a composite key: primary key is the frequency ascending, secondary key is the title alphabetically ascending. The core pattern is BFS layering combined with frequency counting and multi-key sorting.

Complexity

Time: O(V + E + B + D log D), where V members and E connection edges are visited once by BFS, B is the total number of book entries collected at the target distance, and D is the number of distinct titles that are sorted. Space: O(V + D) for the distance/queue arrays plus the frequency map.

Python
PYTHON
from collections import deque, Counter

def books_at_distance(read_books, connections, start, distance):
    n = len(read_books)
    dist = [-1] * n
    dist[start] = 0
    q = deque([start])
    while q:
        u = q.popleft()
        for v in connections[u]:
            if dist[v] == -1:
                dist[v] = dist[u] + 1
                q.append(v)

    freq = Counter()
    for i in range(n):
        if dist[i] == distance:
            for b in read_books[i]:
                freq[b] += 1

    return sorted(freq, key=lambda name: (freq[name], name))
JavaScript
JAVASCRIPT
function booksAtDistance(readBooks, connections, start, distance) {

#### cons
t n = readBooks.length;

#### cons
t dist = new Array(n).fill(-1);
    dist[start] = 0;

#### cons
t queue = [start];
    for (let head = 0; head < queue.length; head++) {

#### cons
t u = queue[head];
        for (const v of connections[u]) {
            if (dist[v] === -1) {
                dist[v] = dist[u] + 1;
                queue.push(v);
            }
        }
    }

#### cons
t freq = new Map();
    for (let i = 0; i < n; i++) {
        if (dist[i] !== distance) continue;
        for (const b of readBooks[i]) {
            freq.set(b, (freq.get(b) || 0) + 1);
        }
    }

    return [...freq.keys()].sort((a, b) => {

#### cons
t fa = freq.get(a), fb = freq.get(b);
        if (fa !== fb) return fa - fb;
        return a < b ? -1 : a > b ? 1 : 0;
    });
}
Java
JAVA
import java.util.*;

class Solution {
    public List<String> booksAtDistance(List<List<String>> readBooks,
                                        int[][] connections,
                                        int start, int distance) {
        int n = readBooks.size();
        int[] dist = new int[n];
        Arrays.fill(dist, -1);
        Deque<Integer> q = new ArrayDeque<>();
        dist[start] = 0;
        q.offer(start);
        while (!q.isEmpty()) {
            int u = q.poll();
            for (int v : connections[u]) {
                if (dist[v] == -1) {
                    dist[v] = dist[u] + 1;
                    q.offer(v);
                }
            }
        }

        Map<String, Integer> freq = new HashMap<>();
        for (int i = 0; i < n; i++) {
            if (dist[i] != distance) continue;
            for (String b : readBooks.get(i)) freq.merge(b, 1, Integer::sum);
        }

        List<String> res = new ArrayList<>(freq.keySet());
        res.sort((a, b) -> {
            int fa = freq.get(a), fb = freq.get(b);
            if (fa != fb) return fa - fb;
            return a.compareTo(b);
        });
        return res;
    }
}
C++
CPP
#include <vector>
#include <string>
#include <queue>
#include <unordered_map>
#include <algorithm>
using namespace std;

class Solution {
public:
    vector<string> booksAtDistance(vector<vector<string>>& readBooks,
                                   vector<vector<int>>& connections,
                                   int start, int distance) {
        int n = readBooks.size();
        vector<int> dist(n, -1);
        queue<int> q;
        dist[start] = 0;
        q.push(start);
        while (!q.empty()) {
            int u = q.front(); q.pop();
            for (int v : connections[u]) {
                if (dist[v] == -1) {
                    dist[v] = dist[u] + 1;
                    q.push(v);
                }
            }
        }

        unordered_map<string, int> freq;
        for (int i = 0; i < n; i++) {
            if (dist[i] != distance) continue;
            for (const string& b : readBooks[i]) freq[b]++;
        }

        vector<pair<string, int>> items(freq.begin(), freq.end());
        sort(items.begin(), items.end(),
             [] (const pair<string, int>& a, const pair<string, int>& b) {
                 if (a.second != b.second) return a.second < b.second;
                 return a.first < b.first;
             });

        vector<string> res;
        res.reserve(items.size());
        for (auto& p : items) res.push_back(p.first);
        return res;
    }
};
C
C
#include <stdlib.h>
#include <string.h>

typedef struct { char* name; int count; } Entry;

static int cmpEntry(const void* a, const void* b) {

#### cons
t Entry* x = (const Entry*)a;

#### cons
t Entry* y = (const Entry*)b;
    if (x->count != y->count) return x->count - y->count;
    return strcmp(x->name, y->name);
}

/* Returns a newly allocated array of title strings; count written to *returnSize. */
char** booksAtDistance(char*** readBooks, int readBooksSize, int* readBooksColSize,
                       int** connections, int connectionsSize, int* connectionsColSize,
                       int start, int distance, int* returnSize) {
    int n = readBooksSize;
    int* dist = malloc(sizeof(int) * n);
    for (int i = 0; i < n; i++) dist[i] = -1;
    int* queue = malloc(sizeof(int) * n);
    int head = 0, tail = 0;
    dist[start] = 0;
    queue[tail++] = start;
    while (head < tail) {
        int u = queue[head++];
        for (int j = 0; j < connectionsColSize[u]; j++) {
            int v = connections[u][j];
            if (dist[v] == -1) {
                dist[v] = dist[u] + 1;
                queue[tail++] = v;
            }
        }
    }

    Entry* table = NULL;
    int tableSize = 0, tableCap = 0;
    for (int i = 0; i < n; i++) {
        if (dist[i] != distance) continue;
        for (int j = 0; j < readBooksColSize[i]; j++) {
            char* b = readBooks[i][j];
            int found = -1;
            for (int t = 0; t < tableSize; t++)
                if (strcmp(table[t].name, b) == 0) { found = t; break; }
            if (found >= 0) {
                table[found].count++;
            } else {
                if (tableSize == tableCap) {
                    tableCap = tableCap ? tableCap * 2 : 16;
                    table = realloc(table, sizeof(Entry) * tableCap);
                }
                table[tableSize].name = b;
                table[tableSize].count = 1;
                tableSize++;
            }
        }
    }

    qsort(table, tableSize, sizeof(Entry), cmpEntry);
    char** result = malloc(sizeof(char*) * (tableSize > 0 ? tableSize : 1));
    for (int t = 0; t < tableSize; t++) {
        result[t] = malloc(strlen(table[t].name) + 1);
        strcpy(result[t], table[t].name);
    }
    *returnSize = tableSize;
    free(dist);
    free(queue);
    free(table);
    return result;
}

#### Follow-up
questions
What changes if you must return all titles read by members at distance at most distance, not exactly?
How would you adapt the solution if connections were directed (asymmetric)?
If the same member could appear via multiple equal-length paths, does the count change (it should not — why)?
Reverse the tie-breaking: order by frequency descending, then title descending.

