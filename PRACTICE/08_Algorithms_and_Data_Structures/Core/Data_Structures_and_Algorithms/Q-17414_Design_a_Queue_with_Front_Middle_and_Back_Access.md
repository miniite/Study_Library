---
type: "practice"
difficulty: "Other"
---

# Design a Queue with Front, Middle, and Back Access

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Queue](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Queue.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Design a data structure that behaves like a queue but allows insertion and removal at the front, the middle, and the back.

Implement a class MiddleAccessQueue with the following operations:

MiddleAccessQueue() — create an empty queue.
void pushFront(int val) — insert val at the front.
void pushMiddle(int val) — insert val at the middle.
void pushBack(int val) — insert val at the back.
int popFront() — remove and return the front element; return -1 if the queue is empty.
int popMiddle() — remove and return the middle element; return -1 if the queue is empty.
int popBack() — remove and return the back element; return -1 if the queue is empty.

Middle rule (tie-breaking). When the queue currently holds k elements, there are two cases:

For a push to the middle, the new element is placed at index k / 2 (integer division, 0-based). So pushing into a queue of length 5 places the new value at index 2; pushing into a queue of length 4 places it at index 2.
For a pop from the middle, when there are two equally central elements, the frontmost of the two is removed. Concretely, the removed index is (k - 1) / 2 (integer division, 0-based).

For example, inserting X into the middle of [a, b, c, d, e] gives [a, b, X, c, d, e], and popping the middle of [a, b, c, d, e, f] returns c and leaves [a, b, d, e, f].

#### Cons
traints
1 <= val <= 10^9
At most 1000 total calls will be made across pushFront, pushMiddle, pushBack, popFront, popMiddle, and popBack.

#### Example
s & test cases
TEXT
Input: ops = ["MiddleAccessQueue","pushFront","pushBack","pushMiddle","popMiddle","popBack","popFront","popFront"], args = [[],[8],[9],[7],[],[],[],[]] -> Output: [null,null,null,null,7,9,8,-1]
Why: The queue becomes [8,7,9]; the middle pop takes the central 7, then back returns 9, front returns 8, and the final pop on an empty queue returns -1.

Input: ops = ["MiddleAccessQueue","pushBack","pushBack","pushMiddle","popFront","popMiddle","popMiddle"], args = [[],[5],[6],[9],[],[],[]] -> Output: [null,null,null,null,5,9,6]
Why: Middle-inserting 9 into [5,6] lands at index 1 giving [5,9,6]; verifies middle placement when the length is even before the insert.

Input: ops = ["MiddleAccessQueue","popFront","popMiddle","popBack"], args = [[],[],[],[]] -> Output: [null,-1,-1,-1]
Why: Every removal on an empty queue must return -1.

Input: ops = ["MiddleAccessQueue","pushMiddle","popBack","pushMiddle","popMiddle"], args = [[],[42],[],[100],[]] -> Output: [null,null,42,null,100]
Why: With a single element, front, middle, and back all refer to the same value, so it can be removed by any of the three pops.

Input: ops = ["MiddleAccessQueue","pushBack","pushBack","pushBack","pushBack","pushBack","pushBack","popMiddle"], args = [[],[11],[22],[33],[44],[55],[66],[]] -> Output: [null,null,null,null,null,null,null,33]
Why: The queue is [11,22,33,44,55,66]; the two central elements are 33 and 44, and popMiddle removes the frontmost of the pair, 33.

#### Approach

Split the sequence across two deques, left (front half) and right (back half), maintaining the invariant size(left) <= size(right) <= size(left) + 1. This invariant pins the middle to a fixed, cheap-to-reach spot: when the total count is odd the middle element is right.front(), and when it is even the two central elements are left.back() (frontmost) and right.front(). Every mutation is done at the natural end of one deque, after which a single balance() step restores the invariant by shifting at most one element between the two halves. Push-middle inserts at right.front() when the halves are equal, otherwise at left.back(); pop-middle removes left.back() when the halves are equal (even length -> frontmost of the pair) and right.front() otherwise (odd length). The pattern is the classic "two heaps / two deques straddling the median" balancing technique applied to positional (not value-ordered) access.

Complexity

Time: O(1) per operation. Each push or pop touches only the ends of two deques and performs at most one rebalancing transfer, all constant-time on a linked-list-backed deque (collections.deque, ArrayDeque, std::deque, or the explicit C/JS doubly linked list). Space: O(n) to store the n elements currently held, split across the two halves.

Python
PYTHON
from collections import deque

class MiddleAccessQueue:
    def __init__(self):
        # invariant: len(left) <= len(right) <= len(left) + 1
        self.left = deque()
        self.right = deque()

    def _balance(self):
        if len(self.left) > len(self.right):
            self.right.appendleft(self.left.pop())
        elif len(self.right) > len(self.left) + 1:
            self.left.append(self.right.popleft())

    def pushFront(self, val):
        self.left.appendleft(val)
        self._balance()

    def pushMiddle(self, val):
        if len(self.left) == len(self.right):
            self.right.appendleft(val)
        else:
            self.left.append(val)
        self._balance()

    def pushBack(self, val):
        self.right.append(val)
        self._balance()

    def popFront(self):
        if not self.left and not self.right:
            return -1
        val = self.left.popleft() if self.left else self.right.popleft()
        self._balance()
        return val

    def popMiddle(self):
        if not self.left and not self.right:
            return -1
        if len(self.left) == len(self.right):
            val = self.left.pop()
        else:
            val = self.right.popleft()
        self._balance()
        return val

    def popBack(self):
        if not self.left and not self.right:
            return -1
        val = self.right.pop()
        self._balance()
        return val
JavaScript
JAVASCRIPT
class Deque {

#### cons
tructor() {
        this.head = null;
        this.tail = null;
        this.size = 0;
    }
    pushFront(val) {

#### cons
t node = { val, prev: null, next: this.head };
        if (this.head) this.head.prev = node; else this.tail = node;
        this.head = node;
        this.size++;
    }
    pushBack(val) {

#### cons
t node = { val, prev: this.tail, next: null };
        if (this.tail) this.tail.next = node; else this.head = node;
        this.tail = node;
        this.size++;
    }
    popFront() {

#### cons
t node = this.head;
        this.head = node.next;
        if (this.head) this.head.prev = null; else this.tail = null;
        this.size--;
        return node.val;
    }
    popBack() {

#### cons
t node = this.tail;
        this.tail = node.prev;
        if (this.tail) this.tail.next = null; else this.head = null;
        this.size--;
        return node.val;
    }
}

class MiddleAccessQueue {

#### cons
tructor() {
        // invariant: left.size <= right.size <= left.size + 1
        this.left = new Deque();
        this.right = new Deque();
    }

    _balance() {
        if (this.left.size > this.right.size) {
            this.right.pushFront(this.left.popBack());
        } else if (this.right.size > this.left.size + 1) {
            this.left.pushBack(this.right.popFront());
        }
    }

    pushFront(val) {
        this.left.pushFront(val);
        this._balance();
    }

    pushMiddle(val) {
        if (this.left.size === this.right.size) this.right.pushFront(val);
        else this.left.pushBack(val);
        this._balance();
    }

    pushBack(val) {
        this.right.pushBack(val);
        this._balance();
    }

    popFront() {
        if (this.left.size === 0 && this.right.size === 0) return -1;

#### cons
t val = this.left.size > 0 ? this.left.popFront() : this.right.popFront();
        this._balance();
        return val;
    }

    popMiddle() {
        if (this.left.size === 0 && this.right.size === 0) return -1;

#### cons
t val = this.left.size === this.right.size
            ? this.left.popBack()
            : this.right.popFront();
        this._balance();
        return val;
    }

    popBack() {
        if (this.left.size === 0 && this.right.size === 0) return -1;

#### cons
t val = this.right.popBack();
        this._balance();
        return val;
    }
}
Java
JAVA
import java.util.ArrayDeque;
import java.util.Deque;

class MiddleAccessQueue {
    // invariant: left.size() <= right.size() <= left.size() + 1
    private final Deque<Integer> left = new ArrayDeque<>();
    private final Deque<Integer> right = new ArrayDeque<>();

    public MiddleAccessQueue() {}

    private void balance() {
        if (left.size() > right.size()) {
            right.addFirst(left.pollLast());
        } else if (right.size() > left.size() + 1) {
            left.addLast(right.pollFirst());
        }
    }

    public void pushFront(int val) {
        left.addFirst(val);
        balance();
    }

    public void pushMiddle(int val) {
        if (left.size() == right.size()) right.addFirst(val);
        else left.addLast(val);
        balance();
    }

    public void pushBack(int val) {
        right.addLast(val);
        balance();
    }

    public int popFront() {
        if (left.isEmpty() && right.isEmpty()) return -1;
        int val = left.isEmpty() ? right.pollFirst() : left.pollFirst();
        balance();
        return val;
    }

    public int popMiddle() {
        if (left.isEmpty() && right.isEmpty()) return -1;
        int val = (left.size() == right.size()) ? left.pollLast() : right.pollFirst();
        balance();
        return val;
    }

    public int popBack() {
        if (left.isEmpty() && right.isEmpty()) return -1;
        int val = right.pollLast();
        balance();
        return val;
    }
}
C++
CPP
#include <deque>
using namespace std;

class MiddleAccessQueue {
    deque<int> left, right;   // invariant: left.size() <= right.size() <= left.size()+1

    void balance() {
        if (left.size() > right.size()) {
            right.push_front(left.back());
            left.pop_back();
        } else if (right.size() > left.size() + 1) {
            left.push_back(right.front());
            right.pop_front();
        }
    }

public:
    MiddleAccessQueue() {}

    void pushFront(int val) { left.push_front(val); balance(); }

    void pushMiddle(int val) {
        if (left.size() == right.size()) right.push_front(val);
        else left.push_back(val);
        balance();
    }

    void pushBack(int val) { right.push_back(val); balance(); }

    int popFront() {
        if (left.empty() && right.empty()) return -1;
        int val;
        if (!left.empty()) { val = left.front(); left.pop_front(); }
        else { val = right.front(); right.pop_front(); }
        balance();
        return val;
    }

    int popMiddle() {
        if (left.empty() && right.empty()) return -1;
        int val;
        if (left.size() == right.size()) { val = left.back(); left.pop_back(); }
        else { val = right.front(); right.pop_front(); }
        balance();
        return val;
    }

    int popBack() {
        if (left.empty() && right.empty()) return -1;
        int val = right.back();
        right.pop_back();
        balance();
        return val;
    }
};
C
C
#include <stdlib.h>

typedef struct DNode {
    int val;
    struct DNode* prev;
    struct DNode* next;
} DNode;

typedef struct {
    DNode* head;
    DNode* tail;
    int size;
} Deque;

static Deque* dequeCreate(void) {
    Deque* d = (Deque*)malloc(sizeof(Deque));
    d->head = d->tail = NULL;
    d->size = 0;
    return d;
}

static void dequePushFront(Deque* d, int val) {
    DNode* node = (DNode*)malloc(sizeof(DNode));
    node->val = val;
    node->prev = NULL;
    node->next = d->head;
    if (d->head) d->head->prev = node; else d->tail = node;
    d->head = node;
    d->size++;
}

static void dequePushBack(Deque* d, int val) {
    DNode* node = (DNode*)malloc(sizeof(DNode));
    node->val = val;
    node->next = NULL;
    node->prev = d->tail;
    if (d->tail) d->tail->next = node; else d->head = node;
    d->tail = node;
    d->size++;
}

static int dequePopFront(Deque* d) {
    DNode* node = d->head;
    int val = node->val;
    d->head = node->next;
    if (d->head) d->head->prev = NULL; else d->tail = NULL;
    free(node);
    d->size--;
    return val;
}

static int dequePopBack(Deque* d) {
    DNode* node = d->tail;
    int val = node->val;
    d->tail = node->prev;
    if (d->tail) d->tail->next = NULL; else d->head = NULL;
    free(node);
    d->size--;
    return val;
}

typedef struct {
    Deque* left;
    Deque* right;
} MiddleAccessQueue;

/* Invariant: size(left) <= size(right) <= size(left) + 1. */
static void mqBalance(MiddleAccessQueue* q) {
    if (q->left->size > q->right->size) {
        dequePushFront(q->right, dequePopBack(q->left));
    } else if (q->right->size > q->left->size + 1) {
        dequePushBack(q->left, dequePopFront(q->right));
    }
}

MiddleAccessQueue* middleAccessQueueCreate(void) {
    MiddleAccessQueue* q = (MiddleAccessQueue*)malloc(sizeof(MiddleAccessQueue));
    q->left = dequeCreate();
    q->right = dequeCreate();
    return q;
}

void pushFront(MiddleAccessQueue* q, int val) {
    dequePushFront(q->left, val);
    mqBalance(q);
}

void pushMiddle(MiddleAccessQueue* q, int val) {
    if (q->left->size == q->right->size)
        dequePushFront(q->right, val);
    else
        dequePushBack(q->left, val);
    mqBalance(q);
}

void pushBack(MiddleAccessQueue* q, int val) {
    dequePushBack(q->right, val);
    mqBalance(q);
}

int popFront(MiddleAccessQueue* q) {
    if (q->left->size == 0 && q->right->size == 0) return -1;
    int val = (q->left->size > 0) ? dequePopFront(q->left) : dequePopFront(q->right);
    mqBalance(q);
    return val;
}

int popMiddle(MiddleAccessQueue* q) {
    if (q->left->size == 0 && q->right->size == 0) return -1;
    int val = (q->left->size == q->right->size) ? dequePopBack(q->left)
                                                 : dequePopFront(q->right);
    mqBalance(q);
    return val;
}

int popBack(MiddleAccessQueue* q) {
    if (q->left->size == 0 && q->right->size == 0) return -1;
    int val = dequePopBack(q->right);
    mqBalance(q);
    return val;
}

void middleAccessQueueFree(MiddleAccessQueue* q) {
    while (q->left->size) dequePopBack(q->left);
    while (q->right->size) dequePopBack(q->right);
    free(q->left);
    free(q->right);
    free(q);
}

#### Follow-up
questions
How would you extend the structure to support O(1) access to the k-th element from the front for an arbitrary k?
Add a peekMiddle() that reads the middle without removing it, and reason about which deque end it maps to.
What changes if you must popMiddle a specific value rather than the positional middle?
Argue precisely why the two-deque balancing scheme keeps every one of the six operations O(1) worst-case rather than merely amortized.

