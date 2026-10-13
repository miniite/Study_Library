---
type: "practice"
difficulty: "Basic"
---

# What are Python's truthy and falsy values? List which built-in values evaluate to False.

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Membership and Collection Complexity](<../../../../TOOLS/Python/Python_Language/Membership_and_Collection_Complexity.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

The question asks what Python's truthy and falsy values are, and specifically to list which built-in values evaluate to False. It's a basic-concept question, so the interviewer wants to confirm I understand that any object has a boolean interpretation in a boolean context (like if/while/and/or/not), that only a small, well-defined set of values is falsy, and that everything else is truthy. I'll define the terms, give the exact falsy list, explain how Python decides (__bool__ then __len__), show idiomatic uses, and call out the classic traps (if x: vs if x is not None:, and 0/empty being falsy).

No clarifying questions needed; assumption is standard CPython 3.x.

Definition

Every Python object can be evaluated as a boolean when used where a truth value is expected — in if, while, and, or, not, comprehension filters, ternaries, etc. Python doesn't require the object to be an actual bool; it coerces it.

Truthy = evaluates to True in a boolean context.
Falsy (also spelled "falsey") = evaluates to False in a boolean context.

The key insight: the falsy set is small and fixed; everything not in it is truthy. So you memorize the falsy values, and everything else is truthy by default.

The complete list of built-in falsy values
False — the boolean itself.
None — the null/absence object.
0 — integer zero.
0.0 — float zero (also -0.0).
0j — complex zero.
Decimal(0) and Fraction(0, 1) — zero in those numeric types.
'' — empty string.
[] — empty list.
() — empty tuple.
{} — empty dict.
set() and frozenset() — empty set.
range(0) — an empty range.
Empty bytes (b'') and empty bytearray.

The general rule behind items 3–13: zero of any numeric type is falsy, and any empty container/sequence is falsy. Plus the two singletons False and None.

Everything else is truthy — including some surprises

Any non-empty container, any non-zero number, and essentially any other object is truthy:

"0" (a string containing the character zero) → truthy (non-empty string).
" " (a space) → truthy.
[0], [[]], {0: None} → truthy (non-empty containers, even though their contents are falsy).
-1, 0.1, float('nan') → truthy (non-zero / not empty; note nan is truthy!).
A function, class, module, or a fresh user-defined object → truthy by default.
How Python decides (the mechanism)

When Python needs a truth value for object x, it calls the built-in bool(x), which works like this:

If the object's type defines __bool__, call it; it must return True or False.
Else, if it defines __len__, call it; the object is falsy iff len(x) == 0.
Else (neither is defined), the object is truthy.

That's exactly why empty containers are falsy: list, dict, str, etc. define __len__, and an empty one has length 0. And it's why a plain custom object is truthy: it defines neither method. You can control your own class's truthiness:

PYTHON
class Cart:
    def __init__(self, items): self.items = items
    def __bool__(self): return bool(self.items)   # empty cart is falsy

bool(Cart([]))      # False
bool(Cart([1]))     # True
Idiomatic use and why it matters

Truthiness lets you write concise, Pythonic checks:

PYTHON
if items:            # instead of: if len(items) > 0
    process(items)

name = user_name or "Anonymous"   # falls back if user_name is '' or None

x or y returns x if x is truthy, otherwise y — this is short-circuit evaluation returning the operand, not a bool.

The classic trap: falsy is not the same as None

The biggest real-world bug is conflating "is this value present?" with "is this value truthy?". Consider a function parameter that can legitimately be 0 or "":

PYTHON
def set_qty(qty):
    if not qty:                 # BUG: 0 is falsy!
        qty = 1                 # accidentally overrides a valid quantity of 0

Here qty=0 is a valid input but not qty is True, so you wrongly reset it. The fix is to test for the sentinel explicitly:

PYTHON
def set_qty(qty=None):
    if qty is None:             # only replace true absence
        qty = 1

Rule of thumb: use if x: when you genuinely mean "non-empty / non-zero"; use if x is None: / is not None when you mean "was a value supplied?". This matters enormously for 0, 0.0, '', [], and False which are all valid data yet falsy.

#### Common mistakes
and follow-ups
Assuming "False", "0", or "None" (strings) are falsy — they're truthy (non-empty strings). Converting user input "0" needs int(), not truthiness.
Assuming float('nan') is falsy — it's truthy.
[] vs [[]]: the first is falsy, the second truthy (it has one element).

#### Follow-up
probe: "How does if x: work internally?" → __bool__, then __len__, then default True.
Summary — what a strong answer emphasizes

Nail the falsy set: None, False, all numeric zeros (0, 0.0, 0j, Decimal(0)), and every empty container/sequence ('', [], (), {}, set(), range(0), b''). Everything else — including "0", " ", [0], -1, nan — is truthy. Explain that Python resolves truthiness via __bool__, then __len__, else True, and warn that if x: is not a presence check: use is None when 0/''/[] are legitimate values.

**Comparison:**

```
Category	Falsy examples	Truthy examples
Booleans	False	True
None	None	—
Numbers	0, 0.0, 0j, Decimal(0), Fraction(0,1)	-1, 0.1, float('nan'), float('inf')
Strings	''	"0", " ", "False"
Sequences	[], (), range(0), b''	[0], (0,), [[]], "a"
Mappings/Sets	{}, set(), frozenset()	{0: None}, {0}
Objects	(only if __bool__/__len__ say so)	any default custom object
```

**Code:**

```python
# Falsy values — all evaluate to False
falsy = [None, False, 0, 0.0, 0j, '', [], (), {}, set(), frozenset(), range(0), b'', bytearray()]
print(all(not bool(x) for x in falsy))   # True -> every one is falsy

# Truthy surprises — all evaluate to True
truthy = ["0", " ", [0], [[]], {0: None}, -1, 0.1, float('nan')]
print(all(bool(x) for x in truthy))      # True -> every one is truthy

# How Python decides: __bool__ first, then __len__, else True
class Cart:
    def __init__(self, items): self.items = items
    def __bool__(self): return bool(self.items)
print(bool(Cart([])), bool(Cart([1])))   # False True

# The classic falsy-vs-None trap
def set_qty(qty=None):
    if qty is None:        # correct: only true absence; qty==0 is preserved
        qty = 1
    return qty
print(set_qty(0), set_qty(None))         # 0 1
```

**Example input / output:**

```text
bool(None)        -> False
bool(0), bool(0.0), bool(0j) -> False, False, False
bool(''), bool([]), bool({}), bool(set()), bool(range(0)) -> all False
bool("0")         -> True   # non-empty string
bool([0])         -> True   # non-empty list
bool(float('nan'))-> True
"" or "fallback"  -> "fallback"   # '' is falsy, so 'or' returns the second operand
0 or 5            -> 5
5 and 0           -> 0
```

**Flow:**

1. yes
2. no
3. yes
4. yes
5. no
6. no
7. bool(x) needed
8. type defines __bool__?
9. return x.__bool__()
10. type defines __len__?
11. len(x) == 0 ?
12. Falsy
13. Truthy

