---
type: "practice"
difficulty: "Basic"
---

# What does the 'is' operator check versus the '==' operator? Give a case where they disagree.

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ String Immutability and Performance](<../../../../TOOLS/Python/Python_Language/String_Immutability_and_Performance.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage
— what is really being asked

The interviewer wants to confirm I understand the difference between identity (is) and equality (==) in Python, and — importantly — that I can produce a concrete case where they disagree. This is a classic "do you understand Python's object model?" question. The real probe is: do I know that is compares whether two names point to the same object in memory, while == compares whether two objects have the same value? And do I know the famous integer/string caching gotchas that trip people up? A weak answer says "they're basically the same"; a strong answer nails the definitions, gives a disagreement example, and adds the practical rule of thumb (is only for singletons like None).

#### Assumptions
/ clarifications: CPython 3.x, interactive/normal execution. I'll note that some caching behavior is implementation- and context-dependent (that nuance itself impresses interviewers).

Plan: (1) define each operator precisely, (2) tie is to id(), (3) explain how == works via __eq__, (4) give clean disagreement cases, (5) explain the misleading small-int/interned-string caching that makes them accidentally agree, and (6) give the rule of thumb.

From first principles

Every Python object has an identity — a unique, unchanging integer for its lifetime, exposed by id() (in CPython, effectively its memory address). Two names refer to the "same object" only if they share that identity.

a is b is True iff id(a) == id(b) — i.e. a and b are literally the same object. It never calls user code; it's a pointer comparison, so it's fast and cannot be overridden.
a == b asks "do these have the same value?" It invokes a.__eq__(b) (falling back to b.__eq__(a)). Types define what "equal value" means: two different list objects with the same elements are == but not is.

Key relationship: if a is b is True, then a == b is almost always True too (same object → same value), unless a type defines a pathological __eq__ (e.g. float('nan'), where x == x is False even though x is x is True). But the reverse does not hold: a == b being True says nothing about identity.

A clear case where they disagree

Two separately-built lists with identical contents:

PYTHON
a = [1, 2, 3]
b = [1, 2, 3]
a == b    # True  -> same value
a is b    # False -> different objects (different id)

They are equal in value but are two distinct objects in memory. Same story for two equal but separately constructed dicts, sets, or user objects, and for a large integer built two different ways:

PYTHON
x = 1000
y = 10 ** 3
x == y    # True
x is y    # False (typically) -> two different int objects
The confusing part: caching makes them accidentally agree

Beginners get burned because CPython caches some immutable objects, so is sometimes returns True when you'd expect False:

Small integer interning: CPython pre-creates and reuses int objects for the range -5 to 256. So 256 is 256 is True, but 257 is 257 can be False in some contexts.
PYTHON
a = 256; b = 256
a is b     # True  -> both reference the one cached int 256
a = 257; b = 257
a is b     # False -> outside the cache, two new int objects
String interning: short "identifier-like" string literals are often interned, so "hi" is "hi" may be True — but this is not guaranteed and depends on how the strings are created (a runtime-built string like "h" + "i" may not be interned).

The lesson: any code that relies on is for value comparison is a latent bug — it may "work" for small ints/interned strings and silently break for 257, dynamically built strings, or larger objects.

The correct rule of thumb
Use == to compare values — this is what you want 99% of the time.
Use is only to compare against singletons, primarily None (if x is None:), and occasionally True/False/sentinel objects created with object(). is None is idiomatic, faster, and can't be fooled by a custom __eq__.
Never use is to compare numbers or strings for equality.
Summary — what a strong answer emphasizes

is compares identity (same object, i.e. equal id()); == compares value (via __eq__). They disagree whenever two distinct objects hold the same value — e.g. [1,2,3] == [1,2,3] is True while [1,2,3] is [1,2,3] is False; likewise x = 1000; y = 10**3 gives == True, is False. The reason people confuse them is CPython's small-int cache (-5..256) and string interning, which make is accidentally True for some literals — behavior you must never depend on. Practical rule: use == for values, reserve is for None/singletons.

#### Common mistakes
/ follow-up probes: writing if x == None (should be is None); claiming small-int caching is language-guaranteed (it's a CPython implementation detail); and forgetting the nan edge case where x is x but x == x is False. A good follow-up an interviewer may ask: "Can a == b be True while a is b is False?" — yes, that's the whole point (equal value, different objects); "Can a is b be True while a == b is False?" — only with a weird __eq__ such as NaN.

**Comparison:**

```
Aspect	is	==
Compares	Identity (same object)	Value (equal contents)
Under the hood	id(a) == id(b) (pointer compare)	calls a.__eq__(b)
Overridable?	No	Yes (define __eq__)
Typical use	x is None, singletons/sentinels	comparing values (most cases)
[1,2,3] vs equal [1,2,3]	False	True
257 vs 257 (fresh)	False (usually)	True
float('nan') vs itself	True	False
```

**Code:**

```python
# is -> identity (same object / same id); == -> value equality
a = [1, 2, 3]
b = [1, 2, 3]
print(a == b)          # True  (same contents)
print(a is b)          # False (two distinct objects)
print(id(a) == id(b))  # False (matches `is`)

# Large ints: equal value, different objects
x = 1000
y = 10 ** 3
print(x == y, x is y)  # True False

# The caching trap (CPython small-int cache: -5..256)
p = 256; q = 256
print(p is q)          # True  (cached, reused)
r = 257; s = 257
print(r is s)          # False (outside cache -> new objects)

# Idiomatic identity use: singletons
val = None
print(val is None)     # True  -> the RIGHT use of `is`

# NaN edge case: is True, == False
import math
n = float('nan')
print(n is n, n == n)  # True False
```

**Example input / output:**

```text
>>> a = [1,2,3]; b = [1,2,3]
>>> a == b
True
>>> a is b
False

>>> x = 1000; y = 10**3
>>> x == y, x is y
(True, False)

>>> p = 256; q = 256; p is q
True
>>> r = 257; s = 257; r is s
False

>>> None is None
True
>>> import math; n = float('nan'); (n is n, n == n)
(True, False)
```

