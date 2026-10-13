---
type: "practice"
difficulty: "Basic"
---

# What is the difference between 'del', 'remove', 'pop', and 'clear' on a list?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Mutability](<../../../../TOOLS/Python/Python_Language/Mutability.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

This is a basic-concept question testing whether I know the four common ways to delete elements from a Python list and, importantly, how they differ: del, .remove(), .pop(), and .clear(). The interviewer wants to see that I know (a) whether each removes by index or by value, (b) whether it returns the removed item, (c) whether it can clear the whole list or a slice, and (d) what errors each raises when the target isn't found. Getting these confused causes real bugs (e.g., using remove when you meant an index). I'll define each clearly with a tiny example, then give a comparison table and a decision rule.

Assumption: standard CPython lists. No clarifying questions really needed — this is definitional.

del — a statement that removes by index or slice

del is not a method; it's a Python statement. On a list it deletes the item(s) at the position(s) you name and returns nothing (you can't capture the value).

PYTHON
a = [10, 20, 30, 40]
del a[1]        # remove index 1 -> [10, 30, 40]
del a[1:3]      # remove a slice -> [10]

del a[i] raises IndexError if i is out of range. You can also del a entirely, which unbinds the name a (the list object is gone from that name). del is the only one of the four that can delete a slice (a whole range) in one shot, and it does not return the removed value.

.remove(value) — remove by value (the first match)

.remove(x) searches the list and deletes the first element equal to x (by value, using ==), not by index. It returns None.

PYTHON
a = [10, 20, 30, 20]
a.remove(20)    # removes the FIRST 20 -> [10, 30, 20]

If x isn't in the list, it raises ValueError: list.remove(x): x not in list. Only the first occurrence is removed; duplicates further along stay. It's O(n) because it must scan to find the value (and shift elements after removal).

.pop(index=-1) — remove by index AND return it

.pop() removes the element at the given index and returns it — this is the key differentiator. With no argument it removes and returns the last element (stack behavior).

PYTHON
a = [10, 20, 30]
last = a.pop()      # returns 30, a -> [10, 20]
first = a.pop(0)    # returns 10, a -> [20]

pop() on an empty list, or pop(i) with an out-of-range index, raises IndexError. Because it returns the value, pop is what you use for stacks/queues (pop() from end is O(1); pop(0) from front is O(n) due to shifting — use collections.deque if you pop from the front a lot).

.clear() — empty the entire list

.clear() removes all elements in place, leaving an empty list []. It returns None and never raises (clearing an already-empty list is fine).

PYTHON
a = [10, 20, 30]
a.clear()       # a -> []

a.clear() is equivalent to del a[:] or a[:] = []. It mutates the same list object, so any other name referencing that list also sees it emptied — different from a = [], which rebinds a to a new empty list and leaves the old one untouched.

Putting it together
Remove by index → del a[i] (no return) or a.pop(i) (returns the item).
Remove by value → a.remove(value) (first match, no return).
Remove everything → a.clear() (or del a[:]).
Remove a range/slice → del a[i:j].
Need the removed value back → only pop gives it to you.
Errors — a frequent gotcha
del a[i] / a.pop(i) → IndexError if index out of range.
a.remove(x) → ValueError if value absent (guard with if x in a: or try/except).
a.clear() → never errors.

Mixing these up is the classic mistake: calling a.remove(2) when you meant "delete index 2" removes the value 2 instead — silent wrong behavior if 2 happens to be present elsewhere.

What a strong answer emphasizes

State the three axes crisply: by-index vs by-value, returns-the-item vs returns-None, and which errors each raises. Note that del is a statement (and the only slice-capable one), pop is the only one that returns the removed element, remove is the only by-value one, and clear empties in place (vs a = [] which rebinds). Mentioning O(n) for remove/pop(0)/middle del (element shifting) is a nice bonus.

Quick self-test / follow-ups
"How do you remove all occurrences of a value?" → a = [x for x in a if x != v] (a list comprehension), since remove only kills the first.
"Difference between a.clear() and a = []?" → in-place empty (shared refs see it) vs rebinding a new list.

**Comparison:**

```
Operation	Kind	Selects by	Returns removed value?	Can clear all?	Slice?	Error if not found
del a[i] / del a[i:j]	Statement	Index / slice	No	Yes (del a[:])	Yes	IndexError
a.remove(x)	Method	Value (first ==)	No (returns None)	No	No	ValueError
a.pop([i])	Method	Index (default last)	Yes	No	No	IndexError
a.clear()	Method	All	No (returns None)	Yes	n/a	Never
```

**Code:**

```python
# del — by index or slice, returns nothing
a = [10, 20, 30, 40]
del a[1]        # -> [10, 30, 40]
del a[0:2]      # -> [40]           (slice delete: only del can do this)

# remove — by VALUE, first match, returns None
b = [10, 20, 30, 20]
b.remove(20)    # -> [10, 30, 20]   (first 20 gone; ValueError if absent)

# pop — by index, RETURNS the removed item (default: last)
c = [10, 20, 30]
x = c.pop()     # x = 30, c -> [10, 20]
y = c.pop(0)    # y = 10, c -> [20]  (IndexError if out of range)

# clear — empties the whole list in place, returns None
d = [10, 20, 30]
d.clear()       # d -> []           (same as: del d[:])

# Remove ALL occurrences of a value (remove only kills first):
e = [1, 2, 2, 3, 2]
e = [v for v in e if v != 2]   # -> [1, 3]
```

**Example input / output:**

```text
a=[10,20,30,40]; del a[1]        -> [10, 30, 40]
b=[10,20,30,20]; b.remove(20)    -> [10, 30, 20]     (first 20 removed)
c=[10,20,30];    c.pop()         -> returns 30; c=[10,20]
c.pop(0)                         -> returns 10; c=[20]
d=[10,20,30];    d.clear()       -> d=[]
[].pop()                         -> IndexError
[1,2].remove(9)                  -> ValueError
```

**Flow:**

1. Yes
2. No
3. Index
4. Yes
5. No
6. Value
7. Remove from a list
8. Remove everything?
9. a.clear or del a-colon
10. Know index or value?
11. Need the removed item back?
12. a.pop index
13. del a index or slice
14. a.remove value - first matchonly

