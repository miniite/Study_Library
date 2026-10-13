---
type: "practice"
difficulty: "Other"
---

# Minimum Adjacent-Free Swaps to Match an Anagram

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Breadth-First Search](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Breadth-First_Search.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given two strings source and goal of equal length. They are guaranteed to be anagrams of each other, meaning goal can be obtained from source by rearranging its characters.

In a single move you may pick any two positions of source and swap the characters sitting at those two positions. (The two positions do not have to be adjacent — any pair of indices is allowed.)

Return the minimum number of moves needed to transform source into goal.

Input
source: a string of lowercase letters.
goal: a string of lowercase letters, an anagram of source and the same length.
Output
A single integer: the least number of position swaps required so that source becomes exactly equal to goal.

#### Cons
traints
1 <= source.length <= 20
goal.length == source.length
source and goal consist only of lowercase letters drawn from a small alphabet of at most 6 distinct letters.
goal is guaranteed to be an anagram of source, so a valid answer always exists.

#### Example
s & test cases
TEXT
- Input: `source = "xy", goal = "yx"` -> Output: `1`
  Why: One swap of the two positions turns "xy" into "yx".
- Input: `source = "pqr", goal = "qrp"` -> Output: `2`
  Why: This is a single 3-cycle; a k-cycle needs k-1 swaps, so 2 swaps ("pqr" -> "qpr" -> "qrp").
- Input: `source = "code", goal = "code"` -> Output: `0`
  Why: The strings are already identical (trivial anagram), so zero swaps are needed. Edge case: already-matching input.
- Input: `source = "aabb", goal = "bbaa"` -> Output: `2`
  Why: Two independent swaps fix both halves ("aabb" -> "baab" -> "bbaa"); duplicate-character edge case.
- Input: `source = "abcd", goal = "bcda"` -> Output: `3`
  Why: A single 4-cycle rotation needs 4-1 = 3 swaps; greedy fixing the first wrong slot each time reaches the optimum.

#### Approach

Model each intermediate string as a node in an implicit graph; an edge connects two strings that differ by exactly one swap. The minimum number of swaps is the shortest path from source to goal, so run BFS. The key optimization that keeps BFS tractable is greedy child generation: locate the first index i where the current string differs from goal, and only try swaps that place the correct character goal[i] into position i (swap it with some later j holding that character). Any optimal swap sequence can be reordered to always repair the leftmost incorrect position first, so restricting expansion to those swaps preserves optimality while cutting the branching factor. The BFS level index at which goal is dequeued is the answer.

Complexity

Time: O(n^2 * B) where n is the string length and B is the number of distinct states explored by BFS (bounded because at every level we only ever fix the first mismatched slot, which prunes the search tree dramatically). Each state costs O(n) to find the first mismatch and O(n) per generated child. Space: O(n * B) for the queue and the visited set. With n <= 20 and the greedy "fix leftmost wrong position" branching, the explored space stays small in practice.

Python
PYTHON
from collections import deque

def min_swaps(source: str, goal: str) -> int:
    if source == goal:
        return 0

    queue = deque([source])
    seen = {source}
    steps = 0

    while queue:
        for _ in range(len(queue)):
            cur = queue.popleft()
            if cur == goal:
                return steps

            i = 0
            while cur[i] == goal[i]:      # first position that is wrong
                i += 1

            chars = list(cur)
            for j in range(i + 1, len(chars)):
                if chars[j] == goal[i]:   # swapping j into i fixes position i
                    chars[i], chars[j] = chars[j], chars[i]
                    child = ''.join(chars)
                    chars[i], chars[j] = chars[j], chars[i]
                    if child not in seen:
                        seen.add(child)
                        queue.append(child)
        steps += 1

    return -1
JavaScript
JAVASCRIPT
function minSwaps(source, goal) {
    if (source === goal) return 0;

    let level = [source];

#### cons
t seen = new Set([source]);
    let steps = 0;

    while (level.length) {

#### cons
t next = [];
        for (const cur of level) {
            if (cur === goal) return steps;

            let i = 0;
            while (cur[i] === goal[i]) i++;       // first mismatched index

#### cons
t chars = cur.split('');
            for (let j = i + 1; j < chars.length; j++) {
                if (chars[j] === goal[i]) {       // swap brings goal[i] to i
                    [chars[i], chars[j]] = [chars[j], chars[i]];

#### cons
t child = chars.join('');
                    [chars[i], chars[j]] = [chars[j], chars[i]];
                    if (!seen.has(child)) {
                        seen.add(child);
                        next.push(child);
                    }
                }
            }
        }
        level = next;
        steps++;
    }

    return -1;
}
Java
JAVA
import java.util.*;

class Solution {
    public int minSwaps(String source, String goal) {
        if (source.equals(goal)) return 0;
        Queue<String> queue = new ArrayDeque<>();
        Set<String> seen = new HashSet<>();
        queue.offer(source);
        seen.add(source);
        int steps = 0;
        while (!queue.isEmpty()) {
            for (int sz = queue.size(); sz > 0; sz--) {
                String cur = queue.poll();
                if (cur.equals(goal)) return steps;
                int i = 0;
                while (cur.charAt(i) == goal.charAt(i)) i++;   // first mismatch
                char[] chars = cur.toCharArray();
                for (int j = i + 1; j < chars.length; j++) {
                    if (chars[j] == goal.charAt(i)) {          // swap fixes index i
                        swap(chars, i, j);
                        String child = new String(chars);
                        swap(chars, i, j);
                        if (!seen.contains(child)) {
                            seen.add(child);
                            queue.offer(child);
                        }
                    }
                }
            }
            steps++;
        }
        return -1;
    }

    private void swap(char[] a, int i, int j) {
        char t = a[i]; a[i] = a[j]; a[j] = t;
    }
}
C++
CPP
#include <string>
#include <queue>
#include <unordered_set>
using namespace std;

class Solution {
public:
    int minSwaps(string source, string goal) {
        if (source == goal) return 0;
        queue<string> q;
        unordered_set<string> seen;
        q.push(source);
        seen.insert(source);
        int steps = 0;
        while (!q.empty()) {
            for (int sz = (int)q.size(); sz > 0; --sz) {
                string cur = q.front();
                q.pop();
                if (cur == goal) return steps;
                int i = 0;
                while (cur[i] == goal[i]) ++i;          // first mismatched index
                for (int j = i + 1; j < (int)cur.size(); ++j) {
                    if (cur[j] == goal[i]) {            // swap fixes position i
                        swap(cur[i], cur[j]);
                        if (!seen.count(cur)) {
                            seen.insert(cur);
                            q.push(cur);
                        }
                        swap(cur[i], cur[j]);          // undo for next candidate
                    }
                }
            }
            ++steps;
        }
        return -1;
    }
};
C
C
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

/* ---- open-addressing hash set of strings (for visited states) ---- */
typedef struct { char **keys; int cap; int size; } StrSet;

static unsigned long hashStr(const char *s) {
    unsigned long h = 1469598103934665603UL; /* FNV-1a */
    for (; *s; ++s) { h ^= (unsigned char)*s; h *= 1099511628211UL; }
    return h;
}
static void setInit(StrSet *set, int cap) {
    set->cap = cap; set->size = 0;
    set->keys = calloc(cap, sizeof(char *));
}
static int setContains(StrSet *set, const char *s) {
    unsigned long idx = hashStr(s) % set->cap;
    while (set->keys[idx]) {
        if (strcmp(set->keys[idx], s) == 0) return 1;
        idx = (idx + 1) % set->cap;
    }
    return 0;
}
static void setRehash(StrSet *set, int newcap) {
    char **old = set->keys; int oldcap = set->cap;
    set->keys = calloc(newcap, sizeof(char *));
    set->cap = newcap; set->size = 0;
    for (int i = 0; i < oldcap; ++i)
        if (old[i]) {
            unsigned long idx = hashStr(old[i]) % set->cap;
            while (set->keys[idx]) idx = (idx + 1) % set->cap;
            set->keys[idx] = old[i]; set->size++;
        }
    free(old);
}
static void setAdd(StrSet *set, const char *s) {
    if ((set->size + 1) * 2 >= set->cap) setRehash(set, set->cap * 2);
    unsigned long idx = hashStr(s) % set->cap;
    while (set->keys[idx]) {
        if (strcmp(set->keys[idx], s) == 0) return;
        idx = (idx + 1) % set->cap;
    }
    set->keys[idx] = strdup(s); set->size++;
}
static void setFree(StrSet *set) {
    for (int i = 0; i < set->cap; ++i) free(set->keys[i]);
    free(set->keys);
}

int minSwaps(const char *source, const char *goal) {
    int n = (int)strlen(source);
    if (strcmp(source, goal) == 0) return 0;

    StrSet seen; setInit(&seen, 1024);
    setAdd(&seen, source);

    int qcap = 1024, qhead = 0, qtail = 0;
    char **queue = malloc(qcap * sizeof(char *));
    queue[qtail++] = strdup(source);

    int steps = 0, answer = -1;
    char *buf = malloc(n + 1);

    while (qhead < qtail && answer < 0) {
        int levelEnd = qtail;
        while (qhead < levelEnd) {
            char *cur = queue[qhead++];
            if (strcmp(cur, goal) == 0) { answer = steps; break; }
            int i = 0;
            while (cur[i] == goal[i]) i++;      /* first mismatched index */
            for (int j = i + 1; j < n; ++j) {
                if (cur[j] == goal[i]) {         /* swap brings goal[i] into place i */
                    strcpy(buf, cur);
                    char t = buf[i]; buf[i] = buf[j]; buf[j] = t;
                    if (!setContains(&seen, buf)) {
                        setAdd(&seen, buf);
                        if (qtail == qcap) { qcap *= 2; queue = realloc(queue, qcap * sizeof(char *)); }
                        queue[qtail++] = strdup(buf);
                    }
                }
            }
        }
        if (answer < 0) steps++;
    }

    free(buf);
    for (int i = 0; i < qtail; ++i) free(queue[i]);
    free(queue);
    setFree(&seen);
    return answer;
}

int main(void) {
    printf("%d\n", minSwaps("xy", "yx"));     /* 1 */
    printf("%d\n", minSwaps("pqr", "qrp"));   /* 2 */
    printf("%d\n", minSwaps("aabb", "bbaa")); /* 2 */
    printf("%d\n", minSwaps("abcd", "bcda")); /* 3 */
    return 0;
}

#### Follow-up
questions
How would you reconstruct and print the actual sequence of swaps, not just the count?
Prove that always repairing the leftmost mismatched position first still yields a globally minimum number of swaps.
If characters were arbitrary (large alphabet, no 6-letter limit), how does the state space grow and can you bound the answer using cycle decomposition of the permutation?
Can you speed this up with bidirectional BFS or A* using an admissible heuristic (e.g. number of mismatched positions / 2)?

