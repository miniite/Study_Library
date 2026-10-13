---
type: "practice"
difficulty: "Other"
---

# Design a Look-Ahead Iterator With Peek

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Design](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Design a wrapper iterator that adds a peek capability on top of a basic underlying integer sequence, in addition to the usual next and hasNext operations.

Implement a class (call it PreviewIterator) that is initialized from a sequence of integers and supports:

#### Cons
tructor — Initializes the object over the given sequence of integers.
int next() — Returns the current element in the sequence and advances the internal pointer to the following element.
boolean hasNext() — Returns true if there is still at least one element left to return, otherwise false.
int peek() — Returns the element that next() would return, but without advancing the internal pointer. Calling peek() repeatedly returns the same element until a next() is issued.

The essential difficulty is that a raw iterator can typically only move forward via next() and cannot look at the upcoming element without consuming it. Your design must expose a non-destructive peek() on top of such a forward-only source.

Input / interaction

A list of method calls (next, peek, hasNext) is issued after construction. Every call to next and peek is guaranteed to be valid (i.e. it is only issued when an element is actually available).

Output

Each next and peek returns the appropriate integer; each hasNext returns a boolean.

#### Example

#### Cons
truct a PreviewIterator over the sequence [6, 4, 9], then issue the calls next, peek, next, next, hasNext:

next() returns 6 and advances (cursor now at 4).
peek() returns 4 and does not advance (cursor still at 4).
next() returns 4 and advances (cursor now at 9).
next() returns 9 and advances (cursor now past the end).
hasNext() returns false.

#### Cons
traints

1 <= sequence length <= 1000
1 <= each element value <= 1000
All calls to next and peek are valid (issued only when an element is available).
At most 1000 calls total will be made across next, hasNext, and peek.

Follow up: How would you extend the design to be generic and work with any element type, not just integers?

#### Example
s & test cases
TEXT
Input: init [6, 4, 9]; ops = ["next","peek","next","next","hasNext"] -> Output: [6, 4, 4, 9, false]
Why: peek() returns the same value (4) that the following next() returns, without consuming it; hasNext is false once all three are drained.

Input: init [10]; ops = ["hasNext","peek","next","hasNext"] -> Output: [true, 10, 10, false]
Why: Single-element case; peek and next both yield 10, and hasNext flips from true to false after the one next().

Input: init [3, 3, 3]; ops = ["peek","next","peek","next"] -> Output: [3, 3, 3, 3]
Why: Duplicate values confirm peek is purely positional and does not skip or dedupe.

Input: init [1, 2, 3, 4]; ops = ["peek","peek","next","peek"] -> Output: [1, 1, 1, 2]
Why: Repeated peek() calls are idempotent and return the same head element until a next() advances the cursor.

Input: init [8, 5]; ops = ["next","hasNext","next","hasNext"] -> Output: [8, true, 5, false]
Why: hasNext correctly reports availability before and after the sequence is exhausted, using only next() (peek never called).

#### Approach

The pattern is one-element look-ahead buffering (a.k.a. "prefetch the head"). At construction, eagerly pull one element from the underlying forward-only source into a buffer slot and record whether a value was actually available. peek() simply returns the buffered value without touching the source, so it is non-destructive. next() returns the buffered value and then immediately refills the buffer by pulling the following element from the source, keeping the buffer always one step ahead. hasNext() reports whether the buffer currently holds a valid element. This decouples the caller-visible position from the source position by exactly one element, which is precisely what makes a non-consuming peek possible on top of a consume-only iterator.

Complexity

Time: O(1) per operation — peek, next, and hasNext each do a constant amount of work (read the buffer and optionally pull one element from the source). Space: O(1) extra beyond the underlying sequence — only a single buffered element and a few scalar fields are stored (the eager buffering does not scale with the number of elements).

Python
PYTHON
from typing import List

class PreviewIterator:
    """Look-ahead iterator that buffers one element so peek() does not consume."""

    def __init__(self, data: List[int]):
        self._data = data
        self._pos = 0
        self._buffered = None
        self._has = False
        self._advance()

    def peek(self) -> int:
        """Return the next element without advancing the iterator."""
        return self._buffered

    def next(self) -> int:
        """Return the next element and advance."""
        current = self._buffered
        self._advance()
        return current

    def has_next(self) -> bool:
        return self._has

    def _advance(self) -> None:
        if self._pos < len(self._data):
            self._buffered = self._data[self._pos]
            self._pos += 1
            self._has = True
        else:
            self._has = False
JavaScript
JAVASCRIPT
// Look-ahead iterator that buffers exactly one element, so peek() can return
// the head without advancing the underlying forward-only source.
class PreviewIterator {
    /** @param {number[]} data */

#### cons
tructor(data) {
        this._data = data;
        this._pos = 0;
        this._buffered = undefined;
        this._has = false;
        this._advance();
    }

    /** Returns the next element without advancing. @return {number} */
    peek() {
        return this._buffered;
    }

    /** Returns the next element and advances. @return {number} */
    next() {

#### cons
t current = this._buffered;
        this._advance();
        return current;
    }

    /** @return {boolean} */
    hasNext() {
        return this._has;
    }

    _advance() {
        if (this._pos < this._data.length) {
            this._buffered = this._data[this._pos++];
            this._has = true;
        } else {
            this._has = false;
        }
    }
}
Java
JAVA
// Look-ahead iterator: buffers exactly one element so peek() returns the head
// without advancing the underlying forward-only source.
class PreviewIterator {
    private final int[] data;
    private int pos;
    private int buffered;
    private boolean hasBuffered;

    public PreviewIterator(int[] data) {
        this.data = data;
        this.pos = 0;
        advance();
    }

    // Returns the next element without advancing.
    public int peek() {
        return buffered;
    }

    // Returns the next element and advances.
    public int next() {
        int current = buffered;
        advance();
        return current;
    }

    public boolean hasNext() {
        return hasBuffered;
    }

    private void advance() {
        if (pos < data.length) {
            buffered = data[pos++];
            hasBuffered = true;
        } else {
            hasBuffered = false;
        }
    }
}
C++
CPP
#include <vector>
using namespace std;

// Wraps a forward-only integer source and buffers one element so that
// peek() can inspect the head without consuming it.
class PreviewIterator {
public:
    explicit PreviewIterator(const vector<int>& data)
        : data_(data), pos_(0), buffered_(0), hasBuffered_(false) {
        advance();
    }

    // Returns the next element without advancing the iterator.
    int peek() const {
        return buffered_;
    }

    // Returns the next element and advances.
    int next() {
        int current = buffered_;
        advance();
        return current;
    }

    bool hasNext() const {
        return hasBuffered_;
    }

private:
    void advance() {
        if (pos_ < static_cast<int>(data_.size())) {
            buffered_ = data_[pos_++];
            hasBuffered_ = true;
        } else {
            hasBuffered_ = false;
        }
    }

    vector<int> data_;
    int pos_;
    int buffered_;
    bool hasBuffered_;
};
C
C
#include <stdlib.h>
#include <stdbool.h>

// A look-ahead iterator over an integer array. It buffers exactly one element
// so peek() can return the head without consuming it.
typedef struct {

#### cons
t int* data;
    int size;
    int pos;
    int buffered;
    bool hasBuffered;
} PreviewIterator;

static void previewAdvance(PreviewIterator* it) {
    if (it->pos < it->size) {
        it->buffered = it->data[it->pos++];
        it->hasBuffered = true;
    } else {
        it->hasBuffered = false;
    }
}

PreviewIterator* previewCreate(const int* data, int size) {
    PreviewIterator* it = (PreviewIterator*)malloc(sizeof(PreviewIterator));
    it->data = data;
    it->size = size;
    it->pos = 0;
    it->buffered = 0;
    it->hasBuffered = false;
    previewAdvance(it);
    return it;
}

// Returns the next value without advancing.
int previewPeek(PreviewIterator* it) {
    return it->buffered;
}

// Returns the next value and advances.
int previewNext(PreviewIterator* it) {
    int current = it->buffered;
    previewAdvance(it);
    return current;
}

bool previewHasNext(PreviewIterator* it) {
    return it->hasBuffered;
}

void previewFree(PreviewIterator* it) {
    free(it);
}

#### Follow-up
questions
Make the iterator generic so it works with any element type, not just integers (templates/generics), including handling of a null/absent sentinel.
Support arbitrary k-step look-ahead (peek the element k positions ahead) — what buffer structure do you need (a small queue)?
What if the underlying source is a true forward-only stream that cannot be copied or reset? Show that one-element buffering still works.
How would you make peek/next thread-safe if multiple consumers share the iterator?

