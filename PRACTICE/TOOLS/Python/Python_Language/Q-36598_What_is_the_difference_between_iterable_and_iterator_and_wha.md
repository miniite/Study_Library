---
type: "practice"
difficulty: "Intermediate"
---

# What is the difference between 'iterable' and 'iterator', and what do __iter__ and __next__ do?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Mutability](<../../../../TOOLS/Python/Python_Language/Mutability.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

The interviewer is probing whether you understand the iterator protocol — the machinery that makes for loops, comprehensions, in, unpacking, and generators all work. The specific confusion this question targets is the very common mix-up between an iterable (something you can iterate over) and an iterator (the stateful cursor that actually does the iterating). And they want you to explain the two dunder methods that define the contract: __iter__ and __next__. A complete answer defines both terms precisely, explains how a for loop uses them, shows a hand-written iterator, and calls out the key practical consequences (single-pass exhaustion, why iter() gives you a fresh cursor, and how generators simplify all of this). Assumption: standard CPython, Python 3.

Iterable vs iterator — the core distinction

An iterable is any object you can get an iterator from. Concretely, it's an object that implements __iter__(), which returns an iterator. Lists, tuples, strings, dicts, sets, ranges, and files are all iterables. An iterable is typically reusable — you can loop over a list many times, each time getting a fresh traversal.

An iterator is the object that actually produces the values one at a time and remembers where it is. It implements two methods: __next__(), which returns the next value (and raises StopIteration when there's nothing left), and __iter__(), which returns self. That second point is the subtle one: every iterator is also an iterable (it returns itself from __iter__), but not every iterable is an iterator. A list is iterable but not an iterator — it has no __next__ and doesn't track a position.

The best mental model: an iterable is like a book; an iterator is like a bookmark in that book. The book itself doesn't have a "current page" — you can hand out many bookmarks, each tracking its own position. That's exactly why you can nest two for loops over the same list independently: each loop calls iter(the_list) and gets its own separate iterator with its own cursor.

What __iter__ and __next__ do
__iter__(self) — called by the built-in iter(obj). On an iterable, it constructs and returns a new iterator positioned at the start. On an iterator, it returns self. Its job is "give me something I can call next() on."
__next__(self) — called by the built-in next(it). It returns the next item and advances the internal state. When the sequence is exhausted, it must raise StopIteration. Only iterators have __next__.
How a for loop actually works

for x in obj: is syntactic sugar. Under the hood the interpreter does roughly:

PYTHON
it = iter(obj)          # calls obj.__iter__()  -> get a fresh iterator
while True:
    try:
        x = next(it)    # calls it.__next__()   -> next value
    except StopIteration:
        break           # loop ends cleanly when the iterator signals exhaustion
    ...loop body with x...

So the loop doesn't need to know the length or the type — it just repeatedly calls next() and stops on StopIteration. This uniform protocol is why the same for works over lists, files, generators, dict keys, and your own custom classes.

A hand-written iterator

Here's a Countdown iterable and its iterator, separated (the correct design so the source can be iterated multiple times):

PYTHON
class Countdown:                 # the ITERABLE
    def __init__(self, start):
        self.start = start
    def __iter__(self):
        return CountdownIterator(self.start)   # fresh cursor each time

class CountdownIterator:         # the ITERATOR
    def __init__(self, start):
        self.current = start
    def __iter__(self):
        return self              # iterators return themselves
    def __next__(self):
        if self.current <= 0:
            raise StopIteration  # signals "done"
        val = self.current
        self.current -= 1
        return val

Because Countdown.__iter__ builds a new CountdownIterator each call, you can loop over the same Countdown(3) twice and both work. If instead you put __next__ directly on Countdown and returned self from __iter__, the object would be a one-shot iterator: the first loop drains it, the second sees nothing.

Key practical consequences (what interviewers reward)
Iterators are single-pass / exhaustible. Once __next__ has raised StopIteration, that iterator is dead — it won't restart. Re-iterating requires a fresh iterator via iter(iterable). This is why calling list() on a generator twice gives [...] then [].
This is why you re-iter() iterables but not iterators. A for loop over a list is repeatable; a for loop over a generator object is not.
Generators are the easy way to build iterators. A function with yield returns a generator, which is an iterator (has __iter__ and __next__ auto-generated, and raises StopIteration automatically when the function returns). The Countdown above collapses to:
PYTHON
def countdown(start):
    while start > 0:
        yield start
        start -= 1
Duck typing: for accepts anything that follows the protocol; there's no required base class (though collections.abc.Iterable / Iterator exist for isinstance checks and mixins).
Summary — what a strong answer emphasizes
Iterable = has __iter__, returns an iterator, usually reusable (list, str, dict). Iterator = has __next__ (+ __iter__ returning self), holds position, single-pass.
__iter__ hands you a fresh cursor; __next__ advances it and raises StopIteration at the end.
A for loop = iter() once, then next() repeatedly until StopIteration.
Every iterator is an iterable; the reverse isn't true.
Iterators are exhaustible — that's the #1 gotcha. Generators (yield) are the concise way to implement the protocol.

#### Common mistakes
/ follow-ups: confusing the two terms; making an iterable one-shot by returning self and holding state in the same class; forgetting to raise StopIteration; asking "why can't I loop over a generator twice?" (it's an exhausted iterator); "how do lazy sequences / range differ?" (range is a reusable iterable, not an iterator).

**Comparison:**

```
Feature	Iterable	Iterator
Implements __iter__	Yes (returns a new iterator)	Yes (returns self)
Implements __next__	No	Yes (advances + raises StopIteration)
Holds current position/state	No	Yes
Reusable (multi-pass)	Usually yes	No — single-pass, exhaustible
Examples	list, str, dict, set, range, file	result of iter(x), generators, enumerate, zip, map
Analogy	Book	Bookmark in the book
isinstance(x, Iterator)	False	True
```

**Code:**

```python
from collections.abc import Iterable, Iterator

# ---- Iterable + separate Iterator (reusable) ----
class Countdown:                 # ITERABLE: only __iter__
    def __init__(self, start):
        self.start = start
    def __iter__(self):
        return CountdownIterator(self.start)   # brand-new cursor each loop

class CountdownIterator:         # ITERATOR: __next__ (+ __iter__ -> self)
    def __init__(self, start):
        self.current = start
    def __iter__(self):
        return self
    def __next__(self):
        if self.current <= 0:
            raise StopIteration
        val = self.current
        self.current -= 1
        return val

# Equivalent one-liner via a generator (auto iterator)
def countdown(start):
    while start > 0:
        yield start
        start -= 1

# Introspection
print(isinstance([1, 2, 3], Iterable))          # True
print(isinstance([1, 2, 3], Iterator))          # False (list is NOT an iterator)
print(isinstance(iter([1, 2, 3]), Iterator))    # True (iter() gives an iterator)
```

**Example input / output:**

```text
# Manual protocol use
>>> it = iter(Countdown(3))
>>> next(it)
3
>>> next(it)
2
>>> next(it)
1
>>> next(it)
Traceback (most recent call last):
  ...
StopIteration

# Reusability: Countdown is an iterable -> can loop twice
>>> c = Countdown(3)
>>> list(c)
[3, 2, 1]
>>> list(c)            # fresh iterator -> works again
[3, 2, 1]

# Iterators are single-pass -> exhaustion gotcha
>>> g = countdown(3)
>>> list(g)
[3, 2, 1]
>>> list(g)            # same iterator, already exhausted
[]
```

**Flow:**

1. Yes
2. No
3. for x in obj
4. iter(obj) -> obj.__iter__()
5. iterator with internal cursor
6. next(it) -> it.__next__()
7. More items?
8. return value, advance cursor,run loop body
9. raise StopIteration
10. for loop stops cleanly

