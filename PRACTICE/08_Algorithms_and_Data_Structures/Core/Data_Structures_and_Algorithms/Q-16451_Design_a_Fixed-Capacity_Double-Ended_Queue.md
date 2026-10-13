---
type: "practice"
difficulty: "Other"
---

# Design a Fixed-Capacity Double-Ended Queue

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Design](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Design a double-ended queue (deque) that holds at most a fixed number of integer elements. When it is created you are told the maximum number of items k it may ever hold at once. Elements can be added or removed at either end.

Implement a class named CircularDeque supporting the following operations:

CircularDeque(int k) — create the deque with capacity k.
boolean insertFront(int value) — add value at the front; return true on success, or false if the deque is already full.
boolean insertLast(int value) — add value at the rear; return true on success, or false if the deque is already full.
boolean deleteFront() — remove the front element; return true on success, or false if the deque is empty.
boolean deleteLast() — remove the rear element; return true on success, or false if the deque is empty.
int getFront() — return the front element, or -1 if the deque is empty.
int getRear() — return the rear element, or -1 if the deque is empty.
boolean isEmpty() — return true if the deque holds no elements.
boolean isFull() — return true if the deque holds exactly k elements.

Every operation must run in O(1) time. Do not use a language's built-in deque/list-with-shifting so that the point of the exercise — a fixed-size ring buffer with two moving ends — is exercised.

Input: a sequence of operation calls, the first of which constructs the deque.

Output: for each call, its return value (null for the constructor, a boolean for inserts/deletes/state checks, an integer for the front/rear reads).

#### Cons
traints:

1 <= k <= 1000.
0 <= value <= 1000.
At most 2000 calls in total are made across insertFront, insertLast, deleteFront, deleteLast, getFront, getRear, isEmpty, and isFull.

#### Example
s & test cases
TEXT
Input: CircularDeque(3); insertLast(5)->true; insertFront(9)->true; insertLast(7)->true; insertLast(1)->false; getRear()->7; getFront()->9; isFull()->true; deleteLast()->true; getRear()->5
Why: mixed front/rear inserts fill the deque to capacity, the 4th insert is rejected, and front/rear reads and a rear delete work as expected.

Input: CircularDeque(2); getFront()->-1; getRear()->-1; isEmpty()->true; deleteFront()->false
Why: on an empty deque, reads return -1, isEmpty is true, and a delete safely returns false.

Input: CircularDeque(3); insertFront(1)->true; insertFront(2)->true; insertFront(3)->true; insertFront(4)->false; getFront()->3; getRear()->1
Why: repeated insertFront wraps the head index backward around the buffer; front-to-rear order is 3, 2, 1 and the full deque rejects the extra insert.

Input: CircularDeque(1); insertLast(8)->true; isFull()->true; insertFront(9)->false; getFront()->8; deleteFront()->true; isEmpty()->true
Why: a capacity-1 deque is full after a single insert and empties after one delete (minimum-capacity edge case).

Input: CircularDeque(4); insertLast(1)->true; insertLast(2)->true; deleteFront()->true; deleteFront()->true; deleteFront()->false; isEmpty()->true
Why: deleting more items than are present returns false and leaves the deque empty; the count never goes negative.

Input: CircularDeque(3); insertLast(1)->true; insertLast(2)->true; insertLast(3)->true; deleteFront()->true; insertLast(4)->true; getFront()->2; getRear()->4
Why: after a front deletion the rear insert reuses the freed slot via wraparound; front becomes 2 and rear becomes 4.

#### Approach

Back the deque with a fixed-size array used as a ring (circular) buffer and track two things: head, the index of the front element, and count, how many elements are currently stored. The rear element always lives at index (head + count - 1) mod capacity, so it need not be stored separately. Inserting at the front moves head one step backward with modular arithmetic (head - 1 + capacity) mod capacity and writes there; inserting at the rear writes to (head + count) mod capacity. Deleting from the front advances head forward one step; deleting from the rear simply decrements count, which logically drops the last slot. Fullness and emptiness are decided purely by count (count == capacity and count == 0), which is what makes the ambiguous "head == rear could mean empty or full" case disappear — a common bug when people track only two pointers. All index math is modulo the capacity so both ends wrap around the array seamlessly, giving O(1) worst-case time with no data movement. The pattern is the classic array-backed ring buffer / two-ended queue design.

Complexity

Time: O(1) per operation — every method does a constant number of arithmetic operations and one array access; no shifting or reallocation ever occurs. Space: O(k) total — a single backing array of the fixed capacity plus three integer fields, independent of the number of operations performed.

Python
PYTHON
class CircularDeque:
    def __init__(self, k: int):
        self.buf = [0] * k
        self.head = 0          # index of the front element
        self.count = 0         # number of elements currently stored
        self.capacity = k

    def insertFront(self, value: int) -> bool:
        if self.isFull():
            return False
        self.head = (self.head - 1) % self.capacity
        self.buf[self.head] = value
        self.count += 1
        return True

    def insertLast(self, value: int) -> bool:
        if self.isFull():
            return False
        self.buf[(self.head + self.count) % self.capacity] = value
        self.count += 1
        return True

    def deleteFront(self) -> bool:
        if self.isEmpty():
            return False
        self.head = (self.head + 1) % self.capacity
        self.count -= 1
        return True

    def deleteLast(self) -> bool:
        if self.isEmpty():
            return False
        self.count -= 1
        return True

    def getFront(self) -> int:
        return -1 if self.isEmpty() else self.buf[self.head]

    def getRear(self) -> int:
        if self.isEmpty():
            return -1
        return self.buf[(self.head + self.count - 1) % self.capacity]

    def isEmpty(self) -> bool:
        return self.count == 0

    def isFull(self) -> bool:
        return self.count == self.capacity
JavaScript
JAVASCRIPT
class CircularDeque {

#### cons
tructor(k) {
        this.buf = new Array(k);
        this.head = 0;         // index of the front element
        this.count = 0;        // number of elements currently stored
        this.capacity = k;
    }

    insertFront(value) {
        if (this.isFull()) return false;
        this.head = (this.head - 1 + this.capacity) % this.capacity;
        this.buf[this.head] = value;
        this.count++;
        return true;
    }

    insertLast(value) {
        if (this.isFull()) return false;
        this.buf[(this.head + this.count) % this.capacity] = value;
        this.count++;
        return true;
    }

    deleteFront() {
        if (this.isEmpty()) return false;
        this.head = (this.head + 1) % this.capacity;
        this.count--;
        return true;
    }

    deleteLast() {
        if (this.isEmpty()) return false;
        this.count--;
        return true;
    }

    getFront() { return this.isEmpty() ? -1 : this.buf[this.head]; }
    getRear()  { return this.isEmpty() ? -1 : this.buf[(this.head + this.count - 1) % this.capacity]; }
    isEmpty() { return this.count === 0; }
    isFull()  { return this.count === this.capacity; }
}
Java
JAVA
class CircularDeque {
    private final int[] buf;
    private int head;      // index of the front element
    private int count;     // number of elements currently stored
    private final int capacity;

    public CircularDeque(int k) {
        buf = new int[k];
        head = 0;
        count = 0;
        capacity = k;
    }

    public boolean insertFront(int value) {
        if (isFull()) return false;
        head = (head - 1 + capacity) % capacity;
        buf[head] = value;
        count++;
        return true;
    }

    public boolean insertLast(int value) {
        if (isFull()) return false;
        buf[(head + count) % capacity] = value;
        count++;
        return true;
    }

    public boolean deleteFront() {
        if (isEmpty()) return false;
        head = (head + 1) % capacity;
        count--;
        return true;
    }

    public boolean deleteLast() {
        if (isEmpty()) return false;
        count--;
        return true;
    }

    public int getFront() { return isEmpty() ? -1 : buf[head]; }
    public int getRear()  { return isEmpty() ? -1 : buf[(head + count - 1) % capacity]; }
    public boolean isEmpty() { return count == 0; }
    public boolean isFull()  { return count == capacity; }
}
C++
CPP
#include <vector>
using namespace std;

class CircularDeque {
public:
    CircularDeque(int k) : buf(k), head(0), count(0), capacity(k) {}

    bool insertFront(int value) {
        if (isFull()) return false;
        head = (head - 1 + capacity) % capacity;
        buf[head] = value;
        ++count;
        return true;
    }

    bool insertLast(int value) {
        if (isFull()) return false;
        buf[(head + count) % capacity] = value;
        ++count;
        return true;
    }

    bool deleteFront() {
        if (isEmpty()) return false;
        head = (head + 1) % capacity;
        --count;
        return true;
    }

    bool deleteLast() {
        if (isEmpty()) return false;
        --count;
        return true;
    }

    int getFront() { return isEmpty() ? -1 : buf[head]; }
    int getRear()  { return isEmpty() ? -1 : buf[(head + count - 1) % capacity]; }
    bool isEmpty() { return count == 0; }
    bool isFull()  { return count == capacity; }

private:
    vector<int> buf;
    int head;      // index of the front element
    int count;     // number of elements currently stored
    int capacity;
};
C
C
#include <stdlib.h>
#include <stdbool.h>

typedef struct {
    int* buf;
    int head;       /* index of the front element */
    int count;      /* number of elements currently stored */
    int capacity;
} CircularDeque;

CircularDeque* circularDequeCreate(int k) {
    CircularDeque* d = (CircularDeque*)malloc(sizeof(CircularDeque));
    d->buf = (int*)malloc(sizeof(int) * k);
    d->head = 0;
    d->count = 0;
    d->capacity = k;
    return d;
}

bool circularDequeIsEmpty(CircularDeque* d) { return d->count == 0; }
bool circularDequeIsFull(CircularDeque* d)  { return d->count == d->capacity; }

bool circularDequeInsertFront(CircularDeque* d, int value) {
    if (circularDequeIsFull(d)) return false;
    d->head = (d->head - 1 + d->capacity) % d->capacity;
    d->buf[d->head] = value;
    d->count++;
    return true;
}

bool circularDequeInsertLast(CircularDeque* d, int value) {
    if (circularDequeIsFull(d)) return false;
    d->buf[(d->head + d->count) % d->capacity] = value;
    d->count++;
    return true;
}

bool circularDequeDeleteFront(CircularDeque* d) {
    if (circularDequeIsEmpty(d)) return false;
    d->head = (d->head + 1) % d->capacity;
    d->count--;
    return true;
}

bool circularDequeDeleteLast(CircularDeque* d) {
    if (circularDequeIsEmpty(d)) return false;
    d->count--;                 /* rear = (head + count - 1); dropping count removes it */
    return true;
}

int circularDequeGetFront(CircularDeque* d) {
    return circularDequeIsEmpty(d) ? -1 : d->buf[d->head];
}

int circularDequeGetRear(CircularDeque* d) {
    return circularDequeIsEmpty(d)
        ? -1
        : d->buf[(d->head + d->count - 1) % d->capacity];
}

void circularDequeFree(CircularDeque* d) {
    free(d->buf);
    free(d);
}

#### Follow-up
questions
Make the deque grow automatically when full (dynamic ring buffer) instead of rejecting inserts.
Implement the same interface using a doubly linked list and compare the trade-offs with the array version.
Make every operation thread-safe for concurrent producers and consumers at both ends.
Add a size() and a getKth(int i) that returns the i-th element from the front in O(1).

