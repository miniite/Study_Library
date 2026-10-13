---
type: "practice"
difficulty: "Other"
---

# Alternating Iterator Over Two Sequences

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Queue](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Queue.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given two lists of integers, first and second. Build an iterator that returns their elements in alternating order: one element from first, then one element from second, then the next element from first, and so on. When one of the lists runs out of elements, the iterator keeps returning the remaining elements of the other list in their original order.

Implement a class (call it AlternatingIterator) exposing three operations:

AlternatingIterator(first, second) — initialize the iterator with the two integer lists.
hasNext() — return true if the iterator still has at least one element to produce, false otherwise.
next() — return the current element and advance the iterator to the following element. It is only called when hasNext() is true.

Calling next() repeatedly until hasNext() becomes false must reproduce the full alternating sequence described above.

Input: two integer lists first and second. The driver then makes a series of hasNext() / next() calls.

Output: the sequence of values produced by successive next() calls.

#### Cons
traints:

0 <= first.length, second.length <= 1000
1 <= first.length + second.length <= 2000
-2^31 <= first[i], second[i] <= 2^31 - 1
next() is invoked only when hasNext() returns true.

#### Follow-up
What if you are given k lists instead of two? Design the iterator so it extends cleanly to k lists, cycling through them in round-robin (cyclic) order and skipping any list that has been exhausted.

#### Example
s & test cases
TEXT
Input: first = [10, 20], second = [30, 40, 50, 60] -> Output: [10, 30, 20, 40, 50, 60]
Why: normal case — alternate until the shorter list is exhausted, then drain the longer one in order.

Input: first = [7], second = [] -> Output: [7]
Why: one list is empty; the iterator yields only the non-empty list's element.

Input: first = [], second = [9] -> Output: [9]
Why: the first list is empty from the start; the iterator falls straight through to the second list.

Input: first = [1, 2, 3], second = [4, 5] -> Output: [1, 4, 2, 5, 3]
Why: uneven lengths — after the second list is spent, the leftover tail of the first list is returned.

Input: first = [-5], second = [-6, -7] -> Output: [-5, -6, -7]
Why: negative values are handled like any other integers; alternation then drain applies.

Input: first = [100], second = [200] -> Output: [100, 200]
Why: single element each — exactly one alternation, minimal non-trivial case.

#### Approach

Keep a queue of "cursors," one per still-active source list, where a cursor remembers its list and current position (or, equivalently, a live language iterator). To produce the next value, pop the cursor at the front, read its current element, advance it, and if it is not yet exhausted push it to the back of the queue; otherwise drop it. Because a cursor always goes to the back after being used, the queue naturally rotates through the sources in round-robin order, which is exactly the alternating pattern for two lists and generalizes to cyclic order for k lists. Empty lists are never enqueued, so exhausted sources are skipped automatically. This is the queue-of-iterators design pattern for merging/interleaving multiple sequences.

Complexity

Time: construction is O(k) where k is the number of source lists (k = 2 here); each hasNext() is O(1) and each next() is O(1) worst case (one dequeue, one optional enqueue). Producing all n elements is therefore O(n). Space: O(k) extra for the queue of cursors — it stores only references/indices into the original lists, not copies of the data, so it is O(1) for the two-list case and independent of n.

Python
PYTHON
from collections import deque
from typing import List

class AlternatingIterator:
    def __init__(self, first: List[int], second: List[int]):
        # Queue of (list, index) cursors; requeuing the front gives round-robin order.
        self.q = deque()
        for seq in (first, second):
            if seq:
                self.q.append((seq, 0))

    def hasNext(self) -> bool:
        return len(self.q) > 0

    def next(self) -> int:
        seq, i = self.q.popleft()
        if i + 1 < len(seq):            # still elements left -> requeue advanced cursor
            self.q.append((seq, i + 1))
        return seq[i]
JavaScript
JAVASCRIPT
class AlternatingIterator {

#### cons
tructor(first, second) {
        // Queue of { seq, i } cursors; requeuing the front gives round-robin order.
        this.q = [];
        if (first.length)  this.q.push({ seq: first,  i: 0 });
        if (second.length) this.q.push({ seq: second, i: 0 });
    }

    hasNext() {
        return this.q.length > 0;
    }

    next() {

#### cons
t cur = this.q.shift();          // front source in round-robin order

#### cons
t value = cur.seq[cur.i];
        if (cur.i + 1 < cur.seq.length)      // still elements left -> send to the back
            this.q.push({ seq: cur.seq, i: cur.i + 1 });
        return value;
    }
}
Java
JAVA
import java.util.*;

class AlternatingIterator {
    private final Queue<Iterator<Integer>> q = new ArrayDeque<>();

    public AlternatingIterator(List<Integer> first, List<Integer> second) {
        if (!first.isEmpty())  q.offer(first.iterator());
        if (!second.isEmpty()) q.offer(second.iterator());
    }

    public boolean hasNext() {
        return !q.isEmpty();
    }

    public int next() {
        Iterator<Integer> it = q.poll();     // front source in round-robin order
        int value = it.next();
        if (it.hasNext())                    // still elements left -> send to the back
            q.offer(it);
        return value;
    }
}
C++
CPP
#include <queue>
#include <vector>
using namespace std;

class AlternatingIterator {
public:
    AlternatingIterator(vector<int>& first, vector<int>& second) {
        if (!first.empty())  q.push({first.begin(),  first.end()});
        if (!second.empty()) q.push({second.begin(), second.end()});
    }

    bool hasNext() { return !q.empty(); }

    int next() {
        auto [cur, end] = q.front();
        q.pop();
        if (cur + 1 != end)               // still elements left -> requeue advanced cursor
            q.push({cur + 1, end});
        return *cur;
    }

private:
    // Each entry is {current iterator, end iterator}; the queue gives round-robin order.
    queue<pair<vector<int>::iterator, vector<int>::iterator>> q;
};
C
C
#include <stdbool.h>
#include <stdlib.h>

/* A cursor over one source list: the data pointer, current index, and length. */
typedef struct {

#### cons
t int *data;
    int idx;
    int len;
} Cursor;

/* Circular queue of active (non-exhausted) cursors. For k sources use cap = k. */
typedef struct {
    Cursor *q;
    int cap;
    int head;
    int size;
} AlternatingIterator;

AlternatingIterator *aiCreate(const int *first, int firstLen,

#### cons
t int *second, int secondLen) {
    AlternatingIterator *it = (AlternatingIterator *)malloc(sizeof(AlternatingIterator));
    it->cap = 2;                       /* two sources; grow to k for the follow-up */
    it->q = (Cursor *)malloc(sizeof(Cursor) * it->cap);
    it->head = 0;
    it->size = 0;
    if (firstLen  > 0) it->q[it->size++] = (Cursor){first,  0, firstLen};
    if (secondLen > 0) it->q[it->size++] = (Cursor){second, 0, secondLen};
    return it;
}

bool aiHasNext(AlternatingIterator *it) {
    return it->size > 0;
}

int aiNext(AlternatingIterator *it) {
    Cursor c = it->q[it->head];              /* take the cursor at the front */
    it->head = (it->head + 1) % it->cap;
    it->size--;
    int value = c.data[c.idx++];
    if (c.idx < c.len) {                      /* still has elements -> requeue at back */
        int tail = (it->head + it->size) % it->cap;
        it->q[tail] = c;
        it->size++;
    }
    return value;
}

void aiFree(AlternatingIterator *it) {
    free(it->q);
    free(it);
}

#### Follow-up
questions
Generalize to k lists so elements come out in cyclic round-robin order; show the queue-of-cursors design already handles this with no algorithmic change.
Support a remove()/reset() operation, or make the iterator resettable to replay from the start.
What changes if the sources are lazy or potentially infinite streams rather than finite in-memory lists?
Make the iterator thread-safe for concurrent hasNext()/next() callers, or discuss why that is expensive.

