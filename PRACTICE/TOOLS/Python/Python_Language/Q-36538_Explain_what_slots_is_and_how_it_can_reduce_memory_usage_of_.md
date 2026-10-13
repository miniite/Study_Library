---
type: "practice"
difficulty: "Deep"
---

# Explain what '__slots__' is and how it can reduce memory usage of many small objects.

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Generators and Streaming ETL](<../../../../TOOLS/Python/Python_Language/Generators_and_Streaming_ETL.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

The question asks what __slots__ is and how it reduces memory when you have many small objects. At "deep understanding" level for a backend/fullstack SDE-2/3 (and given the tags — Databricks, DE Shaw, Google — data-heavy shops), the interviewer is probing whether you understand Python's object model well enough to know where per-instance memory actually goes (the per-instance __dict__), how __slots__ removes that overhead by storing attributes in a fixed C-level array, the concrete savings, and the trade-offs and gotchas (no dynamic attributes, inheritance rules, weakrefs, no __dict__). A shallow answer says "it saves memory"; a strong one explains the mechanism, quantifies it, and states when not to use it.

#### Assumptions
/clarifications: CPython 3.x (this is a CPython implementation detail; PyPy already handles it differently). "Many small objects" implies you're instantiating the same class millions of times — e.g., graph nodes, points, log records, ORM rows — which is exactly when it matters.

Plan: explain the default __dict__ model → what __slots__ changes → why that saves memory (with numbers) → secondary speed benefit → the rules and gotchas (inheritance, weakref, no dynamic attrs, __dict__ interplay) → when to use it and modern alternatives.

The default model: every instance carries a __dict__

By default, a Python instance stores its attributes in a per-instance dictionary, instance.__dict__. When you do self.x = 1; self.y = 2, those go into that dict. A dict is a general-purpose hash table: it over-allocates buckets, stores keys (the attribute-name strings) and values, and carries hash-table bookkeeping. So every single instance pays for a full dictionary even though every instance of the class has the same set of attribute names. For a class with two small integer attributes, the __dict__ can dominate the object's footprint. Multiply by millions of instances and you're wasting hundreds of MB purely on redundant per-instance dicts.

What __slots__ does

__slots__ is a class-level declaration listing the fixed set of attribute names instances may have:

PYTHON
class Point:
    __slots__ = ("x", "y")

This tells CPython: "don't give instances a __dict__; instead, reserve a fixed-size, contiguous array of slots (descriptor-backed member fields) in the C struct of the object, one per named attribute." Each slot is a member descriptor on the class that reads/writes a fixed offset in the instance. The attribute names are stored once on the class, not repeated per instance. The result:

No per-instance __dict__ → the biggest chunk of overhead disappears.
No per-instance __weakref__ slot either, unless you add "__weakref__" to __slots__.
Attributes live in a compact array, so instances are smaller and more cache-friendly.
Why it saves memory — with numbers

For a two-attribute class, typical CPython 3.11 figures (via sys.getsizeof plus the dict):

Regular instance: object header (~16 B) + pointer to __dict__ (~8 B) + the __dict__ itself (~104 B for a small dict, sometimes more) ≈ 150+ bytes.
__slots__ instance: object header + two slots inline ≈ 48–56 bytes.

So roughly a 60–70% reduction per instance. At 10 million instances that's the difference between ~1.5 GB and ~0.5 GB — the kind of win that decides whether a job fits in RAM. sys.getsizeof(obj) alone understates the regular case because it doesn't count the referenced __dict__; you must add sys.getsizeof(obj.__dict__) to compare fairly.

Secondary benefit: speed

Slot access is a fixed-offset array read via a member descriptor, which is often faster than a dict hash lookup for attribute get/set — a modest but real win in hot loops. It also fails fast and loud on typos: assigning an undeclared attribute raises AttributeError instead of silently creating a new key, which catches bugs.

Rules and gotchas (this is where the "deep" part is graded)
No dynamic attributes. With __slots__ you can only set the declared names. p.z = 3 on a slotted class raises AttributeError. If you need both fixed slots and occasional dynamic attributes, add "__dict__" to __slots__ — but that reintroduces the dict and defeats most of the savings.
Inheritance is the classic trap. Slots only work if every class in the hierarchy defines __slots__. If any base class (or the subclass) omits it, that class gets a __dict__, and instances end up with one anyway — you lose the savings silently. A subclass should declare only its new attributes in its own __slots__; parents' slots are inherited. Multiple inheritance with non-empty slots on more than one base raises a TypeError (layout conflict).
Weak references. Slotted instances aren't weak-referenceable unless you add "__weakref__" to __slots__. Matters if you use weakref or WeakValueDictionary.
No class-level defaults collision. You can't have a class variable with the same name as a slot — the slot descriptor and the class attribute would clash and raise ValueError at class creation.
Pickling / copy generally works, but __slots__ objects need __getstate__/__setstate__ awareness in some custom cases; the default protocol handles slots fine in modern Python.
@cached_property and other __dict__-storing helpers break on a pure-slots class because they need somewhere to cache — they store into __dict__, which doesn't exist.
When to use it — and alternatives

Use __slots__ when you have a large number of instances of a small, fixed-shape class and memory (or per-access speed) is a proven bottleneck — points, graph nodes, tokens, event records, ORM value objects. It's a targeted optimization, not a default; don't slot every class reflexively, because you lose flexibility and add maintenance cost.

Modern alternatives / complements: @dataclass(slots=True) (Python 3.10+) gives you dataclass ergonomics and slots in one line. attrs has @define(slots=True) (slots by default). namedtuple / typing.NamedTuple are even lighter and immutable but tuple-like. For massive homogeneous numeric data, don't use per-object classes at all — use NumPy arrays or Arrow/columnar structures, which beat even slotted objects by orders of magnitude.

Summary — what a strong answer emphasizes

__slots__ removes the per-instance __dict__ by storing a class's fixed attributes in a compact, offset-addressed array declared once on the class. For many small objects this cuts per-instance memory ~60–70% and speeds up attribute access, at the cost of dynamic attributes and with strict inheritance/weakref rules. Name the numbers, name the gotchas (inheritance must be slots-all-the-way, __weakref__, no dynamic attrs, cached_property breakage), and mention @dataclass(slots=True) and columnar/NumPy as the scale-up alternatives.

**Comparison:**

```
Aspect	Default (__dict__)	__slots__
Per-instance storage	Full hash-table dict	Fixed C-array of slots
~size, 2-field object	~150+ bytes	~48 bytes
Add new attribute at runtime	Allowed	AttributeError (unless __dict__ in slots)
Attribute access speed	Dict hash lookup	Fixed-offset read (often faster)
Weak references	Yes	Only if "__weakref__" in __slots__
Inheritance	Automatic	Every class must define __slots__
@cached_property	Works	Breaks (no __dict__)
Best for	General/flexible classes	Millions of small, fixed-shape objects
```

**Code:**

```python
import sys

class RegularPoint:
    def __init__(self, x, y):
        self.x = x
        self.y = y

class SlottedPoint:
    __slots__ = ("x", "y")           # fixed set of attributes; no __dict__
    def __init__(self, x, y):
        self.x = x
        self.y = y

# --- Fair memory comparison (must add the __dict__ for the regular one) ---
r = RegularPoint(1, 2)
s = SlottedPoint(1, 2)
reg_total = sys.getsizeof(r) + sys.getsizeof(r.__dict__)
slot_total = sys.getsizeof(s)
print("regular :", reg_total, "bytes")
print("slotted :", slot_total, "bytes")

# --- Dynamic attribute is rejected on the slotted class ---
try:
    s.z = 99
except AttributeError as e:
    print("blocked:", e)

# --- Inheritance done right: subclass declares only its NEW attribute ---
class SlottedPoint3D(SlottedPoint):
    __slots__ = ("z",)               # x, y inherited; still no __dict__

# --- Modern one-liner: dataclass with slots (Python 3.10+) ---
from dataclasses import dataclass

@dataclass(slots=True)
class Node:
    val: int
    next: "Node | None" = None
```

**Example input / output:**

```

Representative CPython 3.11 output:

CODE
Copy
regular : 152 bytes
slotted : 48 bytes
blocked: 'SlottedPoint' object has no attribute 'z'

Scaled impact (10,000,000 instances of a 2-field point):

CODE
Copy
regular : 152 B * 10_000_000  ~= 1.42 GiB
slotted :  48 B * 10_000_000  ~= 0.45 GiB   (~68% saved)

Inheritance gotcha demonstrated:

PYTHON
Copy
>>> class Base:            # NO __slots__
...     pass
>>> class Child(Base):
...     __slots__ = ("x",) # still gets a __dict__ from Base -> savings lost
>>> c = Child(); c.anything = 5   # succeeds! proves __dict__ is present
```

**Flow:**

1. No
2. Yes
3. No
4. Yes
5. Yes
6. No
7. Yes
8. Huge homogeneousnumeric
9. Hand-rolled
10. Many instances of a small,fixed-shape class?
11. Use a normal class - keep__dict__ flexibility
12. Memory/attr-speed a provenbottleneck?
13. Need dynamic attributes?
14. Add '__dict__' to __slots__ - butsavings mostly lost
15. Simple value object?
16. @dataclass(slots=True)
17. Use NumPy / Arrow columnarinstead
18. Define __slots__ on every classin the hierarchy

