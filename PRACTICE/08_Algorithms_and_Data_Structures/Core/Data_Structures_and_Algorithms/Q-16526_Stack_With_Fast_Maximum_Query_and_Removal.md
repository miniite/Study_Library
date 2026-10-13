---
type: "practice"
difficulty: "Other"
---

# Stack With Fast Maximum Query and Removal

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Stack](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Stack.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Design a stack-like container that stores integer tokens and, in addition to the usual stack behaviour, lets you look at and remove the largest value it currently holds.

Implement a class MaxTrackingStack supporting the following operations:

MaxTrackingStack() — create an empty container.
void push(int x) — place token x on top of the container.
int pop() — remove the token currently on top and return its value.
int top() — return the value of the token on top without removing it.
int getMax() — return the largest value currently stored without removing anything.
int extractMax() — remove and return the largest value currently stored. If several tokens share the largest value, remove only the top-most one (the one closest to the top of the container).

You should aim for O(1) time for every top call and O(log n) time for each of the other operations, where n is the number of tokens currently held.

#### Example

#### Cons
ider the operation sequence: push(7), push(3), push(7), top(), extractMax(), top(), getMax(), pop(), top()

push(7) → container (bottom→top): [7]
push(3) → [7, 3]
push(7) → [7, 3, 7]; the top-most token and the maximum are both the last 7
top() → returns 7, container unchanged
extractMax() → returns 7; removes the top-most 7, leaving [7, 3]
top() → returns 3
getMax() → returns 7; container unchanged [7, 3]
pop() → returns 3, leaving [7]
top() → returns 7

#### Cons
traints

-10^7 <= x <= 10^7
At most 10^5 total calls will be made across push, pop, top, getMax, and extractMax.
The container is guaranteed to be non-empty whenever pop, top, getMax, or extractMax is called.

#### Example
s & test cases
TEXT
Input: push(7), push(3), push(7), top(), extractMax(), top(), getMax(), pop(), top()
-> Output: (push/push/push return nothing), 7, 7, 3, 7, 3, 7
Why: With two tokens tied at the maximum (7), extractMax removes only the top-most one; getMax then still reports 7 while leaving the container untouched.

Input: push(-5), top(), getMax(), pop()
-> Output: (push returns nothing), -5, -5, -5
Why: Single negative token; the maximum is that same token, confirming negatives and a size-one container are handled.

Input: push(2), push(2), extractMax(), top(), pop()
-> Output: (pushes return nothing), 2, 2, 2
Why: All values equal; extractMax must remove the top-most duplicate, after which top and pop still see the remaining 2.

Input: push(1), push(5), push(9), extractMax(), extractMax(), top()
-> Output: (pushes return nothing), 9, 5, 1
Why: Repeated extractMax peels off maxima from largest downward, exercising removals from the top region of the container.

Input: push(8), push(2), getMax(), top(), extractMax(), top()
-> Output: (pushes return nothing), 8, 2, 8, 2
Why: The maximum (8) sits below the top (2); extractMax must reach into the middle and unlink it while top and order are preserved.

#### Approach

Keep the stack order in a doubly linked list so the top is reachable in O(1) and any node can be unlinked in O(1) once located. Keep a max-heap keyed by (value, insertionId), where insertionId increases with every push. Because still-present tokens preserve push order in the list, the present token with the largest id is always the top-most, so ordering the heap by value then id makes extractMax return exactly the top-most maximum. pop and extractMax only mark nodes as removed and unlink them from the list; stale heap entries are discarded lazily the next time the heap top is inspected. This "linked list + heap with lazy deletion" augmentation is the standard way to get both stack order and fast max access/removal.

Complexity

Time: top is O(1) (read the tail sentinel's predecessor). push, pop, getMax, and extractMax are O(log n) amortized — each is one heap insertion/removal plus lazy cleanup, and every token enters and leaves the heap at most once. Space: O(n) for the doubly linked list and the heap.

Python
PYTHON
import heapq

class _Node:
    __slots__ = ("val", "nid", "prev", "next", "removed")

    def __init__(self, val, nid):
        self.val = val
        self.nid = nid
        self.prev = None
        self.next = None
        self.removed = False

class MaxTrackingStack:
    def __init__(self):
        self.head = _Node(0, -1)
        self.tail = _Node(0, -1)          # tail side = top of stack
        self.head.next = self.tail
        self.tail.prev = self.head
        self.heap = []                    # entries (-val, -nid, node)
        self.counter = 0

    def push(self, x):
        node = _Node(x, self.counter)
        self.counter += 1
        p = self.tail.prev
        p.next = node
        node.prev = p
        node.next = self.tail
        self.tail.prev = node
        # negate so heapq (a min-heap) yields the largest value, then the
        # largest id -> the top-most token among equal maxima.
        heapq.heappush(self.heap, (-x, -node.nid, node))

    def _unlink(self, node):
        node.prev.next = node.next
        node.next.prev = node.prev
        node.removed = True

    def pop(self):
        node = self.tail.prev
        self._unlink(node)
        return node.val

    def top(self):
        return self.tail.prev.val

    def _clean(self):
        while self.heap and self.heap[0][2].removed:
            heapq.heappop(self.heap)

    def getMax(self):
        self._clean()
        return -self.heap[0][0]

    def extractMax(self):
        self._clean()
        node = self.heap[0][2]
        heapq.heappop(self.heap)
        self._unlink(node)
        return node.val
JavaScript
JAVASCRIPT
class _Node {

#### cons
tructor(val, id) {
        this.val = val;
        this.id = id;
        this.prev = null;
        this.next = null;
        this.removed = false;
    }
}

class MaxHeap {

#### cons
tructor() { this.a = []; }
    size() { return this.a.length; }
    peek() { return this.a[0]; }
    // x has higher priority than y: larger value, then larger id.
    _higher(x, y) {
        if (x.val !== y.val) return x.val > y.val;
        return x.id > y.id;
    }
    push(node) {

#### cons
t a = this.a;
        a.push(node);
        let i = a.length - 1;
        while (i > 0) {

#### cons
t p = (i - 1) >> 1;
            if (this._higher(a[i], a[p])) { [a[i], a[p]] = [a[p], a[i]]; i = p; }
            else break;
        }
    }
    pop() {

#### cons
t a = this.a;

#### cons
t top = a[0];

#### cons
t last = a.pop();
        if (a.length) { a[0] = last; this._down(0); }
        return top;
    }
    _down(i) {

#### cons
t a = this.a, n = a.length;
        while (true) {
            let l = 2 * i + 1, r = 2 * i + 2, best = i;
            if (l < n && this._higher(a[l], a[best])) best = l;
            if (r < n && this._higher(a[r], a[best])) best = r;
            if (best === i) break;
            [a[i], a[best]] = [a[best], a[i]];
            i = best;
        }
    }
}

class MaxTrackingStack {

#### cons
tructor() {
        this.head = new _Node(0, -1);
        this.tail = new _Node(0, -1);          // tail side = top of stack
        this.head.next = this.tail;
        this.tail.prev = this.head;
        this.heap = new MaxHeap();
        this.counter = 0;
    }
    push(x) {

#### cons
t node = new _Node(x, this.counter++);

#### cons
t p = this.tail.prev;
        p.next = node; node.prev = p;
        node.next = this.tail; this.tail.prev = node;
        this.heap.push(node);
    }
    _unlink(node) {
        node.prev.next = node.next;
        node.next.prev = node.prev;
        node.removed = true;
    }
    pop() {

#### cons
t node = this.tail.prev;
        this._unlink(node);
        return node.val;
    }
    top() { return this.tail.prev.val; }
    _clean() {
        while (this.heap.size() && this.heap.peek().removed) this.heap.pop();
    }
    getMax() { this._clean(); return this.heap.peek().val; }
    extractMax() {
        this._clean();

#### cons
t node = this.heap.pop();
        this._unlink(node);
        return node.val;
    }
}
Java
JAVA
import java.util.*;

class MaxTrackingStack {
    private static class Node {
        int val, id;
        Node prev, next;
        boolean removed;
        Node(int val, int id) { this.val = val; this.id = id; }
    }

    private final Node head = new Node(0, -1);
    private final Node tail = new Node(0, -1);   // tail side = top of stack
    private final PriorityQueue<Node> heap;
    private int counter = 0;

    public MaxTrackingStack() {
        head.next = tail;
        tail.prev = head;
        // Max on value, then on id (later push => top-most equal maximum).
        heap = new PriorityQueue<>((a, b) ->
            a.val != b.val ? Integer.compare(b.val, a.val)
                           : Integer.compare(b.id, a.id));
    }

    public void push(int x) {
        Node node = new Node(x, counter++);
        Node p = tail.prev;
        p.next = node; node.prev = p;
        node.next = tail; tail.prev = node;
        heap.offer(node);
    }

    public int pop() {
        Node node = tail.prev;
        unlink(node);
        return node.val;
    }

    public int top() { return tail.prev.val; }

    public int getMax() { clean(); return heap.peek().val; }

    public int extractMax() {
        clean();
        Node node = heap.poll();
        unlink(node);
        return node.val;
    }

    private void unlink(Node node) {
        node.prev.next = node.next;
        node.next.prev = node.prev;
        node.removed = true;
    }

    private void clean() {
        while (!heap.isEmpty() && heap.peek().removed) heap.poll();
    }
}
C++
CPP
#include <list>
#include <queue>
#include <unordered_map>
#include <unordered_set>
using namespace std;

class MaxTrackingStack {
public:
    MaxTrackingStack() : counter(0) {}

    void push(int x) {
        int id = counter++;
        dll.push_back({x, id});                 // back of the list = top
        pos[id] = prev(dll.end());
        pq.push({x, id});                        // max-heap by (value, id)
    }

    int pop() {
        auto it = prev(dll.end());
        int val = it->first, id = it->second;
        dll.pop_back();
        pos.erase(id);
        removed.insert(id);
        return val;
    }

    int top() { return dll.back().first; }

    int getMax() { clean(); return pq.top().first; }

    int extractMax() {
        clean();
        auto [val, id] = pq.top(); pq.pop();
        dll.erase(pos[id]);
        pos.erase(id);
        removed.insert(id);
        return val;
    }

private:
    // (value, id); back = top. pair ordering makes pq a max-heap on
    // value then id, so the top-most equal maximum comes out first.
    list<pair<int,int>> dll;
    priority_queue<pair<int,int>> pq;
    unordered_map<int, list<pair<int,int>>::iterator> pos;
    unordered_set<int> removed;
    int counter;

    void clean() {
        while (!pq.empty() && removed.count(pq.top().second)) pq.pop();
    }
};
C
C
#include <stdlib.h>
#include <stdbool.h>

typedef struct Node {
    int val, id;
    struct Node *prev, *next;
    bool removed;
} Node;

typedef struct {
    Node head, tail;      /* sentinels; tail side is the top of the stack */
    Node **heap;          /* array-based max-heap of Node* by (val, id)   */
    int heapSize, heapCap;
    int counter;          /* monotonically increasing insertion id        */
} MaxTrackingStack;

/* x outranks y if it has a larger value, or an equal value pushed later
   (larger id => closer to the top => the top-most among equal maxima). */
static bool higher(Node *x, Node *y) {
    if (x->val != y->val) return x->val > y->val;
    return x->id > y->id;
}

static void heapPush(MaxTrackingStack *s, Node *node) {
    if (s->heapSize == s->heapCap) {
        s->heapCap = s->heapCap ? s->heapCap * 2 : 16;
        s->heap = (Node **)realloc(s->heap, s->heapCap * sizeof(Node *));
    }
    int i = s->heapSize++;
    s->heap[i] = node;
    while (i > 0) {
        int p = (i - 1) / 2;
        if (higher(s->heap[i], s->heap[p])) {
            Node *t = s->heap[i]; s->heap[i] = s->heap[p]; s->heap[p] = t; i = p;
        } else break;
    }
}

static Node *heapPop(MaxTrackingStack *s) {
    Node *top = s->heap[0];
    s->heap[0] = s->heap[--s->heapSize];
    int i = 0, n = s->heapSize;
    while (1) {
        int l = 2 * i + 1, r = 2 * i + 2, best = i;
        if (l < n && higher(s->heap[l], s->heap[best])) best = l;
        if (r < n && higher(s->heap[r], s->heap[best])) best = r;
        if (best == i) break;
        Node *t = s->heap[i]; s->heap[i] = s->heap[best]; s->heap[best] = t; i = best;
    }
    return top;
}

/* Discard heap entries that were already unlinked from the stack. */
static void clean(MaxTrackingStack *s) {
    while (s->heapSize && s->heap[0]->removed) free(heapPop(s));
}

MaxTrackingStack *mtsCreate(void) {
    MaxTrackingStack *s = (MaxTrackingStack *)calloc(1, sizeof(MaxTrackingStack));
    s->head.next = &s->tail;
    s->tail.prev = &s->head;
    return s;
}

void mtsPush(MaxTrackingStack *s, int x) {
    Node *node = (Node *)calloc(1, sizeof(Node));
    node->val = x; node->id = s->counter++;
    Node *p = s->tail.prev;
    p->next = node; node->prev = p;
    node->next = &s->tail; s->tail.prev = node;
    heapPush(s, node);
}

/* Only mark as removed; the node stays alive until it leaves the heap. */
static void unlink(Node *node) {
    node->prev->next = node->next;
    node->next->prev = node->prev;
    node->removed = true;
}

int mtsPop(MaxTrackingStack *s) {
    Node *node = s->tail.prev;
    int v = node->val;
    unlink(node);
    return v;
}

int mtsTop(MaxTrackingStack *s) { return s->tail.prev->val; }

int mtsGetMax(MaxTrackingStack *s) { clean(s); return s->heap[0]->val; }

int mtsExtractMax(MaxTrackingStack *s) {
    clean(s);
    Node *node = heapPop(s);
    unlink(node);
    int v = node->val;
    free(node);
    return v;
}

void mtsFree(MaxTrackingStack *s) {
    for (int i = 0; i < s->heapSize; i++) free(s->heap[i]);
    free(s->heap);
    free(s);
}

#### Follow-up
questions
Add a symmetric extractMin operation while keeping the same complexity bounds.
Change the tie rule so extractMax removes the bottom-most maximum instead of the top-most — what in the ordering must change?
If only push, pop, top, and getMax are required (no arbitrary removal), how would you achieve O(1) for all of them with a paired "current max" stack?
With up to 10^5 operations and heavy duplicate values, how do you keep memory bounded and avoid unbounded growth of stale heap entries?

