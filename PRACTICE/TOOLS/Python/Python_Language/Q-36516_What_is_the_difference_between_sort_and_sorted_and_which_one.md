---
type: "practice"
difficulty: "Basic"
---

# What is the difference between 'sort' and 'sorted', and which one mutates its input?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Mutability](<../../../../TOOLS/Python/Python_Language/Mutability.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

The question asks me to distinguish two Python sorting facilities that look almost identical by name: the list.sort() method and the built-in sorted() function. The critical part — "which one mutates its input?" — tells me the interviewer is really probing whether I understand in-place vs. return-a-new-object semantics, which is a recurring source of bugs (accidentally getting None, or unexpectedly clobbering a shared list). This is a basic-concept question, so I'll define both clearly, state the mutation behavior explicitly, show a couple of quick examples, and mention the shared key/reverse parameters plus one or two common mistakes.

No clarifying questions are truly needed, but I'll note my assumptions: we're on CPython 3.x, and by "sort" the interviewer means list.sort() (the method), not some other library.

First principles: two ways to order data

Python gives you two ways to produce ordered data, and the difference is entirely about where the result goes.

1. list.sort() — a method on list, sorts in place. It rearranges the elements of the existing list object and returns None. It exists only on lists (it's a method of the list type). Because it mutates the object, any other name bound to that same list also sees the new order.

PYTHON
nums = [3, 1, 2]
result = nums.sort()      # sorts nums in place
print(nums)               # [1, 2, 3]  -> the original list changed
print(result)             # None       -> sort() returns nothing useful

The classic bug is nums = nums.sort() — you overwrite your list with None because sort() returns None, not the sorted list.

2. sorted() — a built-in function, returns a new list. It takes any iterable (list, tuple, string, set, dict keys, generator, etc.), builds and returns a brand-new sorted list, and leaves the original untouched.

PYTHON
nums = (3, 1, 2)              # a tuple
result = sorted(nums)        # new list
print(result)                # [1, 2, 3]
print(nums)                  # (3, 1, 2)  -> original unchanged

Note sorted() on a tuple still returns a list, not a tuple — it always produces a list.

The direct answer to "which one mutates its input?"
list.sort() mutates its input (in place) and returns None.
sorted() does not mutate its input; it returns a new sorted list and leaves the original as-is.

That single sentence is the heart of the question. Everything else is supporting detail.

What they have in common

Both accept the same two keyword arguments and use the same underlying algorithm:

key: a one-argument function used to extract a comparison key from each element (e.g. key=str.lower, key=lambda x: x[1]).
reverse: a boolean; True sorts descending.

Both use Timsort, a stable, adaptive merge/insertion hybrid. "Stable" means elements that compare equal keep their original relative order — important when sorting by a secondary key.

PYTHON
words = ["banana", "Apple", "cherry"]
print(sorted(words, key=str.lower))          # ['Apple', 'banana', 'cherry']
words.sort(key=len, reverse=True)            # sort in place by length, longest first
When to use which
Use sorted() when you need to keep the original (e.g. it's shared, or you need both orders), when the source isn't a list (a tuple, set, dict, or generator), or in a functional/expression context: for x in sorted(data): ....
Use list.sort() when you own the list, don't need the original order back, and want to avoid the memory cost of allocating a second list — it's marginally more memory-efficient because it doesn't create a copy.
Complexity and stability

Both are O(n log n) time in the general case and stable. list.sort() sorts in place so it uses O(n) auxiliary space in the worst case (Timsort's merge buffers), while sorted() additionally allocates the new result list, so it needs at least O(n) extra for that copy.

#### Common mistakes
and follow-ups
x = x.sort() bug — returns None; you lose your list. Very common in interviews.
Assuming sorted() returns the same type it received — it always returns a list. To get a sorted tuple: tuple(sorted(t)).
Calling .sort() on something that isn't a list (e.g. a tuple or set) → AttributeError; tuples/sets have no sort method, which is exactly why sorted() exists.
On mixed, non-comparable types (e.g. [1, "a"]) both raise TypeError in Python 3.
Summary — what a strong answer emphasizes

A strong answer nails one line: list.sort() mutates the list in place and returns None; sorted() leaves the input untouched and returns a new list, accepting any iterable. Then it adds that both share key/reverse, both are stable Timsort at O(n log n), and it calls out the x = x.sort() pitfall. That shows you understand not just the API but the mutation semantics that cause real bugs.

**Comparison:**

```
Aspect	list.sort()	sorted()
Kind	Method on list	Built-in function
Input accepted	A list only	Any iterable (list, tuple, set, str, dict, generator)
Mutates input?	Yes — sorts in place	No — original untouched
Return value	None	A new list
Return type	(none)	Always list
Params	key, reverse	key, reverse
Algorithm	Timsort (stable, O(n log n))	Timsort (stable, O(n log n))
Extra memory	O(n) merge buffers	O(n) for new list + buffers
Use when	You own the list, don't need original	Need a new list / source isn't a list
```

**Code:**

```python
# 1) list.sort() — in place, returns None (mutates input)
nums = [3, 1, 2]
ret = nums.sort()
print(nums)   # [1, 2, 3]  (original changed)
print(ret)    # None

# The classic bug:
data = [5, 2, 9]
data = data.sort()   # BUG: data is now None
print(data)          # None

# 2) sorted() — new list, original untouched (does not mutate)
src = (3, 1, 2)          # works on any iterable
out = sorted(src)
print(out)               # [1, 2, 3]  (a list, even though src was a tuple)
print(src)               # (3, 1, 2)  (unchanged)

# Shared parameters: key and reverse, stable Timsort
words = ["banana", "Apple", "cherry"]
print(sorted(words, key=str.lower))        # ['Apple', 'banana', 'cherry']
words.sort(key=len, reverse=True)          # in place, longest first
print(words)                               # ['banana', 'cherry', 'Apple']
```

**Example input / output:**

```text
Input : nums = [3, 1, 2]; ret = nums.sort()
Output: nums = [1, 2, 3], ret = None      # sort() mutates, returns None

Input : src = (3, 1, 2); out = sorted(src)
Output: out = [1, 2, 3], src = (3, 1, 2)  # sorted() returns new list, src unchanged
```

