---
type: "practice"
difficulty: "Deep"
---

# What is string interning and how can it affect identity comparisons of string literals?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ String Immutability and Performance](<../../../../TOOLS/Python/Python_Language/String_Immutability_and_Performance.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

This "deep understanding" question is the string cousin of the small-int cache question. The interviewer wants to see that I understand: (1) what interning means — one canonical shared object per distinct string value; (2) which strings CPython interns automatically and when; (3) how that makes s is t sometimes True and sometimes False for equal-valued strings; and (4) the correct conclusion — use == for string equality, never is — plus knowing sys.intern() for the rare legitimate use of interning. A weak answer just says "Python caches strings." A complete answer nails the rules (compile-time literals, identifier-like content, length), the non-guarantees, and the why (memory + fast == short-circuit).

#### Assumptions
/clarifications: CPython reference implementation (interning specifics are CPython details). is = identity (id()), == = value equality. I'll assume str (not bytes, though bytes behaves similarly for empties/singletons).

First principles: what interning is

Interning means keeping a single, canonical, shared copy of an immutable value in an internal table (an "intern pool"). When a string that should be interned is created, CPython checks the pool: if an equal string is already there, it returns the existing object; otherwise it inserts the new one. The consequence is that two interned strings with equal value are the same object, so is returns True and — crucially — an equality check can short-circuit: comparing two interned strings by identity first is O(1), avoiding a character-by-character scan.

Because strings are immutable, sharing one object across many references is completely safe — nobody can mutate the shared string and corrupt another user of it. Immutability is the precondition that makes interning possible (same reason ints and None can be singletons).

Which strings CPython interns (and when)

CPython interns strings automatically in these situations — but this is an implementation detail, not a language guarantee:

Compile-time string literals that look like identifiers. When a module/function is compiled, string constants in the code — especially ones consisting only of ASCII letters, digits, and underscores (i.e. valid identifier characters) — are interned and stored once in the code object's constant pool. This covers most "foo", "user_id", variable-name-like literals, and keys.
Interned strings from the language machinery — identifiers, attribute names, dict keys for __dict__, function/variable names — are interned so attribute lookup and namespace dicts compare fast.
Small/empty strings: the empty string "" and length-1 strings are effectively singletons in CPython.

Strings that are typically not interned automatically:

Strings built at runtime by concatenation, slicing, str(), .join(), formatting, or reading from input/files — e.g. "".join(["ab", "cd"]).
Literals containing characters that aren't identifier-like (spaces, punctuation), e.g. "hello world!", though this varies by version and optimizer behavior.

This is exactly why:

PYTHON
a = "hello"
b = "hello"
a is b            # True  -> both literals interned to one object

x = "hello world!"
y = "hello world!"
x is y            # may be True (same code object, folded) or False

p = "".join(["hel", "lo"])
p is "hello"      # typically False -> runtime-built string not interned
How this affects identity comparisons of string literals

Because equal literals in the same compilation unit often share one interned object, beginners see a is b return True and wrongly conclude is tests string equality. It does not — it just happens to coincide when both operands are interned. The moment one side is built at runtime, or you cross implementations/versions, is returns False for two equal strings. So the observable behavior of is on strings is unstable and must never be used to compare string values.

The correct mental model:

s == t → always a correct value comparison (calls str.__eq__, compares content).
s is t → "same object?" — True only when both happen to be the same interned/shared object; an optimization artifact, not a semantic guarantee.
Explicit interning: sys.intern()

For the rare cases where interning is a deliberate optimization — e.g. parsing a huge log/CSV with millions of repeated tokens, or building a symbol table — you can force it:

PYTHON
import sys
token = sys.intern(some_runtime_string)

sys.intern(s) returns the canonical interned object for s, inserting it if absent. Two benefits: (1) memory — all duplicates collapse to one object; (2) speed — after interning, equality checks between those strings can use identity (is), which is O(1) instead of O(length). This is genuinely useful in interpreters, tokenizers, and dedup-heavy pipelines, but it's an optimization you opt into, not something to sprinkle everywhere.

The correct rule

Always compare string values with ==. Never use is to test whether two strings have the same value. Reserve is for identity/singletons (None, sentinels). If profiling shows string equality is a hotspot with lots of duplicates, sys.intern() them yourself and then identity comparison becomes a valid, fast option — but that's a conscious decision. Like is 257, modern CPython emits a SyntaxWarning for is against a string literal, precisely because it's a common bug.

Summary — what a strong answer emphasizes
Interning = one canonical shared object per string value, kept in an intern pool; safe because strings are immutable.
CPython auto-interns compile-time literals (especially identifier-like ones), the empty/length-1 strings, and language identifiers, which is why a is b is often True for equal literals.
Runtime-built strings are usually not interned, so is returns False for equal values — proving is is unreliable for value comparison and is an implementation detail, not a guarantee.
Rule: == for value, is for identity/singletons. Use sys.intern() deliberately for memory/speed on duplicate-heavy data.

#### Common mistakes
/ follow-ups: believing is reliably compares string content; not knowing runtime concatenation breaks the coincidence; forgetting sys.intern. A good follow-up: "How would you speed up equality over millions of repeated keys?" → intern them, then compare by identity; or "How does this relate to small-int caching?" → same idea (shared immutable objects), different pool and rules.

**Comparison:**

```
String source	Interned automatically?	is vs an equal string	Reliable for ==?
Identifier-like literal "user_id"	Yes (compile time)	often True	use ==, not is
Empty / length-1 "", "a"	Yes (singleton-like)	True	use ==
Literal with spaces/punctuation "a b!"	Sometimes (optimizer/version dependent)	True or False	use ==
Runtime concat/slice/join/str()	No	usually False	use ==
sys.intern(runtime_str)	Yes (explicit)	True vs other interned equals	is becomes valid + fast
```

**Code:**

```python
import sys

# Equal literals in one compilation unit are usually interned -> same object
a = "hello"
b = "hello"
print(a is b, a == b)                 # True  True

# Identifier-like literals get interned; punctuation/spaces may not
x = "user_id"
y = "user_id"
print(x is y)                         # True

# Runtime-built string is NOT auto-interned
p = "".join(["hel", "lo"])            # value "hello", but freshly built
print(p == "hello", p is "hello")     # True  False (typically)

# Force interning for the runtime string -> now identity matches
p2 = sys.intern("".join(["hel", "lo"]))
print(p2 is sys.intern("hello"))      # True  -> canonical shared object

# The safe rule for value comparison:
name = input_value = "".join(["Al", "ice"])
print(input_value == "Alice")         # correct
# print(input_value is "Alice")       # WRONG / may warn / unreliable
```

**Example input / output:**

```text
>>> a = "hello"; b = "hello"; a is b
True
>>> p = "".join(["hel", "lo"]); p == "hello"
True
>>> p is "hello"
False
>>> import sys
>>> sys.intern(p) is sys.intern("hello")
True
>>> "" is ""            # empty string is effectively a singleton
True
```

