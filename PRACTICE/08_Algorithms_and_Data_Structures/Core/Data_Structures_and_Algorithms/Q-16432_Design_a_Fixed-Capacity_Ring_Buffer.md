---
type: "practice"
difficulty: "Other"
---

# Design a Fixed-Capacity Ring Buffer

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Queue](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Queue.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Design a bounded, fixed-capacity queue that follows the First-In-First-Out (FIFO) discipline while reusing storage in a circular fashion. Conceptually the last slot of the underlying storage wraps back around to the first slot, forming a ring. This lets the structure reclaim slots that were vacated at the front: unlike a naive array queue that becomes permanently "full" once the write cursor reaches the end, a ring buffer can keep accepting new elements as long as there is at least one free slot anywhere in the ring.

You must implement the structure yourself using only a plain array (or equivalent raw storage) and index bookkeeping. You may NOT delegate to any built-in queue/deque container of the language.

Implement a type RingBuffer supporting the following operations:

RingBuffer(capacity) — construct an empty buffer that can hold at most capacity elements.
enqueue(x) — insert integer x at the back of the buffer. Return true if it was inserted, or false if the buffer is already full.
dequeue() — remove the element currently at the front. Return true if an element was removed, or false if the buffer is empty.
front() — return the element at the front without removing it, or -1 if the buffer is empty.
rear() — return the element at the back without removing it, or -1 if the buffer is empty.
isEmpty() — return whether the buffer currently holds no elements.
isFull() — return whether the buffer currently holds exactly capacity elements.

Every operation must run in O(1) time. The operations may be interleaved in any order (a call to dequeue on an empty buffer, or enqueue on a full buffer, simply returns false and leaves the buffer unchanged).

Input / Output The buffer is exercised by a sequence of operation calls. Each call returns the value described above (constructor returns nothing). Your job is to make every call return the correct result given the running state of the buffer.

#### Cons
traints

1 <= capacity <= 1000
0 <= x <= 1000 for every value passed to enqueue
At most 3000 total calls will be made across enqueue, dequeue, front, rear, isEmpty, and isFull.
All operations must be O(1) time; O(capacity) extra space overall.

#### Example
s & test cases
TEXT
Input: RingBuffer(3); enqueue(7) -> Output: true — first insert into an empty buffer succeeds.
Input: ... enqueue(11); enqueue(4) -> Output: true, true — buffer now holds [7, 11, 4] and is full.
Input: ... enqueue(9) -> Output: false — buffer is full so the insert is rejected.
Input: ... rear() then isFull() -> Output: 4, true — back element is 4 and the buffer reports full.
Input: ... dequeue() then enqueue(9) then rear() -> Output: true, true, 9 — dequeue frees the front (removes 7), the wrapped enqueue reuses that slot, and the new back is 9.
Input: RingBuffer(1); dequeue() -> Output: false — dequeue on an empty buffer returns false and changes nothing; front() would then return -1.

#### Approach

Store the elements in a fixed-size array of length capacity and track two pieces of state: head (the array index of the current front element) and count (how many elements are live). The back position is derived, not stored: the next free slot is (head + count) % capacity, and the current rear element sits at (head + count - 1) % capacity. Enqueue writes to the derived free slot and increments count; dequeue advances head by one modulo capacity and decrements count. Fullness is count == capacity and emptiness is count == 0. Using a count field (instead of a head/tail pair alone) removes the classic ambiguity where a full ring and an empty ring produce identical pointer positions, so no sentinel slot is wasted and every branch stays O(1). This is the canonical circular-buffer / modular-indexing technique.

Complexity

Time: O(1) per operation — every method does only constant index arithmetic and a bounded number of reads/writes. Space: O(capacity) total for the backing array; no per-operation allocation.

Python
PYTHON
class RingBuffer:
    def __init__(self, capacity: int):
        self.cap = capacity
        self.data = [0] * capacity
        self.head = 0    # index of the front element
        self.count = 0   # number of stored elements

    def enqueue(self, x: int) -> bool:
        if self.is_full():
            return False
        tail = (self.head + self.count) % self.cap
        self.data[tail] = x
        self.count += 1
        return True

    def dequeue(self) -> bool:
        if self.is_empty():
            return False
        self.head = (self.head + 1) % self.cap
        self.count -= 1
        return True

    def front(self) -> int:
        return -1 if self.is_empty() else self.data[self.head]

    def rear(self) -> int:
        if self.is_empty():
            return -1
        return self.data[(self.head + self.count - 1) % self.cap]

    def is_empty(self) -> bool:
        return self.count == 0

    def is_full(self) -> bool:
        return self.count == self.cap
JavaScript
JAVASCRIPT
class RingBuffer {

#### cons
tructor(capacity) {
        this.cap = capacity;
        this.data = new Array(capacity);
        this.head = 0;    // index of the front element
        this.count = 0;   // number of stored elements
    }

    enqueue(x) {
        if (this.isFull()) return false;

#### cons
t tail = (this.head + this.count) % this.cap;
        this.data[tail] = x;
        this.count++;
        return true;
    }

    dequeue() {
        if (this.isEmpty()) return false;
        this.head = (this.head + 1) % this.cap;
        this.count--;
        return true;
    }

    front() {
        return this.isEmpty() ? -1 : this.data[this.head];
    }

    rear() {
        if (this.isEmpty()) return -1;
        return this.data[(this.head + this.count - 1) % this.cap];
    }

    isEmpty() {
        return this.count === 0;
    }

    isFull() {
        return this.count === this.cap;
    }
}
Java
JAVA
class RingBuffer {
    private final int cap;
    private final int[] data;
    private int head;   // index of the front element
    private int count;  // number of stored elements

    public RingBuffer(int capacity) {
        this.cap = capacity;
        this.data = new int[capacity];
        this.head = 0;
        this.count = 0;
    }

    public boolean enqueue(int x) {
        if (isFull()) return false;
        int tail = (head + count) % cap;
        data[tail] = x;
        count++;
        return true;
    }

    public boolean dequeue() {
        if (isEmpty()) return false;
        head = (head + 1) % cap;
        count--;
        return true;
    }

    public int front() {
        return isEmpty() ? -1 : data[head];
    }

    public int rear() {
        if (isEmpty()) return -1;
        return data[(head + count - 1) % cap];
    }

    public boolean isEmpty() { return count == 0; }
    public boolean isFull()  { return count == cap; }
}
C++
CPP
#include <vector>
using namespace std;

class RingBuffer {
public:
    explicit RingBuffer(int capacity)
        : cap(capacity), data(capacity), head(0), count(0) {}

    bool enqueue(int x) {
        if (isFull()) return false;
        int tail = (head + count) % cap;
        data[tail] = x;
        ++count;
        return true;
    }

    bool dequeue() {
        if (isEmpty()) return false;
        head = (head + 1) % cap;
        --count;
        return true;
    }

    int front() const {
        return isEmpty() ? -1 : data[head];
    }

    int rear() const {
        if (isEmpty()) return -1;
        return data[(head + count - 1) % cap];
    }

    bool isEmpty() const { return count == 0; }
    bool isFull()  const { return count == cap; }

private:
    int cap;
    vector<int> data;
    int head;   // index of the front element
    int count;  // number of stored elements
};
C
C
#include <stdlib.h>
#include <stdbool.h>

typedef struct {
    int *data;
    int cap;
    int head;   /* index of the front element */
    int count;  /* number of elements currently stored */
} RingBuffer;

RingBuffer *ringBufferCreate(int capacity) {
    RingBuffer *rb = (RingBuffer *)malloc(sizeof(RingBuffer));
    rb->data = (int *)malloc(sizeof(int) * capacity);
    rb->cap = capacity;
    rb->head = 0;
    rb->count = 0;
    return rb;
}

bool ringBufferIsEmpty(RingBuffer *rb) { return rb->count == 0; }
bool ringBufferIsFull(RingBuffer *rb)  { return rb->count == rb->cap; }

bool ringBufferEnqueue(RingBuffer *rb, int x) {
    if (ringBufferIsFull(rb)) return false;
    int tail = (rb->head + rb->count) % rb->cap;
    rb->data[tail] = x;
    rb->count++;
    return true;
}

bool ringBufferDequeue(RingBuffer *rb) {
    if (ringBufferIsEmpty(rb)) return false;
    rb->head = (rb->head + 1) % rb->cap;
    rb->count--;
    return true;
}

int ringBufferFront(RingBuffer *rb) {
    return ringBufferIsEmpty(rb) ? -1 : rb->data[rb->head];
}

int ringBufferRear(RingBuffer *rb) {
    if (ringBufferIsEmpty(rb)) return -1;
    int tail = (rb->head + rb->count - 1) % rb->cap;
    return rb->data[tail];
}

void ringBufferFree(RingBuffer *rb) {
    free(rb->data);
    free(rb);
}

#### Follow-up
questions
Make the buffer thread-safe for a single-producer/single-consumer setup without locks (lock-free ring with atomic head/tail indices).
Generalize to a double-ended ring buffer that also supports insertFront and deleteRear in O(1).
Make it grow automatically when full (dynamic circular buffer) — what is the amortized cost and how do you copy while preserving order across the wrap?
Store arbitrary/generic payloads instead of ints and support an O(1) size() query.

