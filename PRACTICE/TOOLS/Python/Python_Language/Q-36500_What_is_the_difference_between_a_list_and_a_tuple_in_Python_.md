---
type: "practice"
difficulty: "Basic"
---

# What is the difference between a list and a tuple in Python, and when would you pick one over the other?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Mutability](<../../../../TOOLS/Python/Python_Language/Mutability.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

What is really being asked: This is a classic "basic concept" Python fundamentals question. The interviewer wants a clear, correct explanation of how a list differs from a tuple, and — the part many freshers miss — the practical judgment of when to choose each. Because the level is "basic concept," I should define terms plainly, avoid hand-waving, use small concrete examples, and get the core facts exactly right (mutability is the headline, plus the consequences that flow from it: hashability, performance, and intent).

What they're probing for: Do I know the one fundamental difference (mutability) and can I reason about everything that follows from it — dictionary keys, safety, memory/speed, and communicating intent to other developers?

Plan: Start with the single core difference, then walk through each consequence with a tiny example, then give a concrete decision rule and a summary. I'll close with common mistakes.

The one core difference: mutability
A list is mutable — you can change it after creation: add, remove, or replace elements. Written with square brackets: [1, 2, 3].
A tuple is immutable — once created it can never be changed; you cannot add, remove, or reassign elements. Written with parentheses: (1, 2, 3).
PYTHON
nums = [1, 2, 3]
nums[0] = 99        # OK  -> [99, 2, 3]
nums.append(4)      # OK  -> [99, 2, 3, 4]

point = (10, 20)
point[0] = 99       # TypeError: 'tuple' object does not support item assignment

Everything else — the practical trade-offs — flows from this single fact. Both are ordered sequences that keep insertion order, both allow duplicate values, both support indexing and slicing (x[0], x[1:3]), both can hold mixed types, and both can be iterated with a for loop. So a beginner shouldn't overstate the differences: the container behavior is largely the same; mutability is the pivot.

#### Cons
equences of that difference

1. Tuples can be dictionary keys and set members; lists cannot. To be a dict key or a set element, an object must be hashable, and hashability requires the value not to change (so its hash stays stable). Because tuples are immutable they are hashable; lists are not.

PYTHON
locations = {(19.07, 72.87): "Mumbai"}   # tuple key — works
# {[19.07, 72.87]: "Mumbai"}             # TypeError: unhashable type: 'list'

(Caveat: a tuple is only hashable if all its elements are hashable — a tuple containing a list is not hashable.)

2. Safety / protecting data from accidental change. Using a tuple signals "this collection is fixed." It prevents a function from accidentally mutating data you pass in, which makes code easier to reason about and safer in concurrent or shared contexts.

3. Performance and memory (minor, but real). Because tuples are fixed-size, Python can allocate them more compactly and construct them slightly faster; a tuple typically uses a bit less memory than the equivalent list. For most code this is negligible, but for millions of small fixed records it can matter. I'd mention it as a secondary benefit, not the main reason.

4. Available operations. Lists have mutating methods — append, insert, remove, pop, sort, reverse, extend. Tuples have essentially only count and index. If you need to grow, shrink, or sort in place, you need a list.

5. Intent / readability. This is the underrated one. A tuple communicates a fixed record of related-but-different fields (a heterogeneous "struct": (x, y), (name, age, email)), while a list communicates a homogeneous, variable-length collection of the same kind of thing ([user1, user2, user3]). Choosing the right one tells the next reader what the data means.

When to pick which — the decision rule

Use a list when:

The collection will change — you add/remove/reorder items.
It's a homogeneous sequence of "the same kind of thing" whose length varies (e.g., a list of orders, rows read from a file, search results).
You need list-only operations like sorting in place or appending in a loop.

Use a tuple when:

The data is fixed / should not change after creation (coordinates, RGB color, a database row, a config constant).
You need a dictionary key or a set element.
You're returning multiple values from a function — return name, age creates a tuple, which the caller can unpack.
The elements are a fixed-length heterogeneous record where position has meaning.

A neat everyday example is multiple return values and unpacking:

PYTHON
def min_max(xs):
    return min(xs), max(xs)      # returns a tuple

lo, hi = min_max([3, 1, 7])      # tuple unpacking -> lo=1, hi=7
Summary

The headline is mutable list vs. immutable tuple. From immutability follow the practical differences: tuples are hashable (usable as dict keys / set members), a little more memory- and construction-efficient, and safer because they can't be changed by accident; lists have a rich set of mutating methods and are the right choice whenever the collection grows, shrinks, or gets reordered. Beyond mechanics, the choice communicates intent: tuple = fixed record / "these belong together and won't change"; list = variable-length collection of similar items. A strong basic-level answer names mutability first, then the hashability and intent consequences, with one tiny example each.

#### Common mistakes
to avoid: saying "tuples are faster so always use them" (the difference is minor and not the real reason); forgetting the hashability point (the most testable consequence); claiming a tuple is fully frozen when it holds a mutable object — the tuple's references are fixed, but a list inside a tuple can still be mutated, and such a tuple is not hashable. A good follow-up probe is exactly that: "Can you mutate a list stored inside a tuple?" — yes, the inner list can change even though the tuple structure cannot.

**Comparison:**

```
Property	list	tuple
Mutable?	Yes (change/add/remove)	No (immutable)
Syntax	[1, 2, 3]	(1, 2, 3); one-item (5,)
Hashable (dict key / set element)?	No	Yes (if all elements hashable)
Ordered / indexable / sliceable	Yes	Yes
Allows duplicates / mixed types	Yes	Yes
Key methods	append, insert, remove, pop, sort, reverse, extend	count, index only
Memory / build speed	Slightly more / slightly slower	Slightly less / slightly faster
Best for	Variable-length homogeneous collection	Fixed heterogeneous record, keys, multi-return
```

**Code:**

```python
# Mutability — the core difference
lst = [1, 2, 3]
lst[0] = 99          # OK
lst.append(4)        # OK -> [99, 2, 3, 4]

tup = (1, 2, 3)
# tup[0] = 99        # TypeError: 'tuple' object does not support item assignment

# Tuple as a dict key (hashable); list cannot be a key
prices = {("AAPL", "USD"): 190.5}     # works
# {["AAPL", "USD"]: 190.5}            # TypeError: unhashable type: 'list'

# Multiple return values -> tuple, then unpack
def stats(xs):
    return len(xs), sum(xs), sum(xs) / len(xs)   # returns a tuple

count, total, avg = stats([2, 4, 6])   # 3, 12, 4.0

# Gotcha: a mutable object inside a "frozen" tuple
t = (1, [2, 3])
t[1].append(4)       # allowed! inner list mutates -> (1, [2, 3, 4])
# but: hash(t)       # TypeError: unhashable type: 'list' (tuple w/ a list isn't hashable)

# Single-element tuple needs a trailing comma
one = (5,)           # tuple
not_a_tuple = (5)    # just the int 5
```

**Example input / output:**

```text
[1,2,3][0] = 99        -> [99, 2, 3]                (list is mutable)
(1,2,3)[0] = 99        -> TypeError (tuple immutable)

{("a","b"): 1}         -> {('a', 'b'): 1}           (tuple key OK)
{["a","b"]: 1}         -> TypeError: unhashable type: 'list'

stats([2,4,6])         -> (3, 12, 4.0)              (returns a tuple)
count,total,avg = ...  -> 3, 12, 4.0               (unpacking)

t=(1,[2,3]); t[1].append(4)  -> t == (1, [2, 3, 4]) (inner list mutable)
hash((1,[2,3]))        -> TypeError (contains a list -> not hashable)

(5,)  -> tuple of one    |    (5) -> int 5
```

**Flow:**

1. Yes: add/remove/reorder
2. No: fixed data
3. Yes
4. No
5. Yes
6. No, just a growingcollection
7. Need a sequence of values
8. Will it change after creation?
9. Use a list
10. Use as dict key or set member?
11. Use a tuple - hashable
12. Fixed-length record / returningmultiple values?

