---
type: "practice"
difficulty: "Intermediate"
---

# What are dunder (magic) methods? Give three examples and what they let you customize.

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Mutability](<../../../../TOOLS/Python/Python_Language/Mutability.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

The question is: what are dunder (magic) methods, give three examples, and explain what each lets you customize. This is an intermediate backend/fullstack probe. The interviewer wants to confirm I understand that:

Dunder methods are the protocol hooks Python's syntax and builtins call under the hood — they let user objects behave like built-in types (operator overloading, iteration, context management, etc.).
I can name three real, distinct ones and say precisely which language feature each customizes (not just "it's magic").
Bonus judgement: how Python dispatches to them (by type, on the class, not the instance), and where they fit into "protocols."

Clarifying assumption: CPython 3.x. I'll pick three that show breadth — one for representation, one for operator/comparison, and one for a protocol like iteration or context management — so I demonstrate the range rather than three variations of the same idea.

Plan: define the concept and the dispatch rule, then give three examples with concrete code and what each unlocks, then finish with a summary and common pitfalls.

What dunder methods are

"Dunder" = double underscore — methods named with leading and trailing double underscores like __init__, __len__, __eq__. They are also called magic or special methods. They aren't magic in a mystical sense: they are the named hooks that Python's syntax and builtins invoke on your behalf. When you write a + b, Python doesn't have hard-coded addition for your class — it calls type(a).__add__(a, b). When you write len(x), Python calls type(x).__len__(x). When you write for item in x:, Python calls type(x).__iter__(x).

So dunder methods are how you make your own objects participate in the language's built-in behaviors — this is Python's approach to operator overloading and protocol implementation. A "protocol" is just a set of dunder methods a type must implement to be usable in some context (the iterator protocol = __iter__ + __next__; the context-manager protocol = __enter__ + __exit__; the sequence protocol = __len__ + __getitem__; and so on).

Key dispatch rule: for implicit invocations (operators, len, iter, with, etc.), Python looks up the dunder on the type, not on the instance. So len(x) uses type(x).__len__, ignoring any per-instance attribute named __len__. This is why you define them on the class.

Three examples and what they let you customize
1. __init__(self, ...) — object initialization

Called automatically right after an instance is created (by __new__) to set up its initial state. It's the hook behind the ClassName(args) construction syntax. It lets you customize what arguments your constructor takes and how the object's attributes are initialized. (Strictly, __new__ creates the instance and __init__ initializes it; for everyday classes __init__ is the one you write.)

2. __eq__(self, other) — equality comparison (operator overloading)

Called when you write a == b. It lets you customize what "equality" means for your type — e.g. two Money objects are equal if amount and currency match, regardless of identity. Returning NotImplemented (not False) when you don't know how to compare tells Python to try the other operand's __eq__. Its sibling __hash__ matters here: if you define __eq__, Python sets __hash__ to None unless you also define it, making instances unhashable — important when you want to use them in set/dict keys. Related comparison dunders: __lt__, __le__, __gt__, __ge__ (which power <, <=, >, >= and sorting).

3. __enter__(self) / __exit__(self, exc_type, exc, tb) — the context-manager protocol

These power the with statement. __enter__ runs on entry (its return value is bound by as), and __exit__ runs on exit — guaranteed, even if an exception is raised inside the block. This lets you customize setup/teardown around a block of code: acquiring and releasing a lock, opening and closing a file or DB connection/transaction, timing a section. If __exit__ returns a truthy value, it suppresses the exception; returning falsy (the usual case) lets it propagate.

I chose these three because they span three different protocols — construction, operator overloading, and resource management — showing that dunders aren't one feature but the whole extension surface of the language.

Other high-value dunders worth a mention to show breadth: __repr__/__str__ (representation), __len__/__getitem__/__iter__/__next__ (container & iteration), __add__/__mul__ (arithmetic), __call__ (make an instance callable like a function), and __hash__.

#### Common mistakes
/ follow-up probes
Calling dunders directly everywhere (x.__len__()) instead of the idiomatic builtin (len(x)). Use the builtin/syntax; that's the point.
Defining __eq__ without __hash__ and then being surprised the object can't go in a set.
Returning False instead of NotImplemented from comparison dunders when the types are incompatible — this breaks reflected comparisons.
Thinking implicit dispatch checks the instance — it checks the type.

#### Follow-up
probes: difference between __new__ and __init__? (__new__ allocates/returns the instance, needed for immutable subclasses; __init__ configures it.) How do you make an object usable in a for loop? (implement __iter__, returning an iterator with __next__.) What makes an object truthy/falsy? (__bool__, else __len__.)
Summary — what a strong answer emphasizes

Dunder methods are the double-underscore special methods that Python's syntax and builtins call implicitly, letting your objects plug into the language's protocols — operator overloading, iteration, context management, construction, and more. Three good examples: __init__ (customize construction/initial state), __eq__ (customize equality, paired with __hash__), and __enter__/__exit__ (customize with-block setup/teardown with guaranteed cleanup). The strong-candidate detail is that these are looked up on the type, that they implement named protocols, and that some come with contracts (e.g. __eq__ ↔ __hash__, comparison dunders returning NotImplemented).

**Comparison:**

```
Dunder(s)	Invoked by	Lets you customize
__init__	ClassName(args)	Constructor args and initial attribute state
__eq__ (+ __hash__)	a == b, set/dict membership	What "equal" means; hashability for sets/dict keys
__enter__ / __exit__	with obj as x:	Guaranteed setup/teardown around a code block
__repr__ / __str__	repr(), print()	Debug and user-facing text representation
__len__	len(obj)	Object's length / emptiness
__iter__ / __next__	for x in obj:	Iteration behavior (iterator protocol)
__call__	obj(...)	Making an instance callable like a function
__add__	a + b	The + operator for your type
```

**Code:**

```python
import threading

class Money:
    """Demonstrates __init__ and __eq__ (+ __hash__ contract)."""

    def __init__(self, amount, currency):   # customize construction
        self.amount = amount
        self.currency = currency

    def __eq__(self, other):                # customize ==
        if not isinstance(other, Money):
            return NotImplemented           # let the other side try
        return (self.amount, self.currency) == (other.amount, other.currency)

    def __hash__(self):                     # keep it hashable for sets/dicts
        return hash((self.amount, self.currency))

    def __repr__(self):
        return f"Money({self.amount!r}, {self.currency!r})"

class Timer:
    """Demonstrates the context-manager protocol: __enter__ / __exit__."""

    def __init__(self, lock):
        self._lock = lock

    def __enter__(self):
        self._lock.acquire()                # setup
        return self                         # value bound by `as`

    def __exit__(self, exc_type, exc, tb):
        self._lock.release()                # teardown — runs even on error
        return False                        # do not suppress exceptions

# __init__ / __eq__ / __hash__ in action
a = Money(100, "INR")
b = Money(100, "INR")
print(a == b)                 # True  -> __eq__
print(len({a, b}))            # 1     -> __hash__ collapses equal objects

# __enter__ / __exit__ in action
lock = threading.Lock()
with Timer(lock) as t:        # __enter__ acquires the lock
    print("critical section") # ... work ...
# __exit__ released the lock here, guaranteed
```

**Example input / output:**

```

Running the code above:

CODE
Copy
True
1
critical section

Trace of the with block:

Timer(lock) -> __init__ stores the lock.
with ... as t: -> __enter__() acquires the lock, returns self, bound to t.
body prints "critical section".
on exit (normal or exception) -> __exit__(...) releases the lock; returns False so any exception would still propagate.

Trace of equality:

a == b -> Money.__eq__(a, b) compares (100,'INR') == (100,'INR') -> True.
{a, b} -> both hash identically and compare equal -> set keeps one element -> len == 1.
```

**Flow:**

1. yes
2. no
3. Python syntax / builtin (e.g.a==b, len(x), with x:)
4. look up dunder ON THE TYPE,not instance
5. dunder defined?
6. call type(obj).__dunder__(obj,...)
7. default behavior or TypeError
8. your customized behavior runs

