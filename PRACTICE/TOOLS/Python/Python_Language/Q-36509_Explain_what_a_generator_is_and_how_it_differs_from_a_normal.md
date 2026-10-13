---
type: "practice"
difficulty: "Intermediate"
---

# Explain what a generator is and how it differs from a normal function that returns a list.

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Generators and Streaming ETL](<../../../../TOOLS/Python/Python_Language/Generators_and_Streaming_ETL.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

What the question is really asking: Define a Python generator, and contrast it with an ordinary function that builds and returns a list. The interviewer wants to hear the mechanics (yield, lazy evaluation, suspended state), the practical differences (memory, streaming, one-shot iteration), and the judgment about when to prefer each. At intermediate level I should include trade-offs, real backend framing (e.g., streaming large query results), and the sharp edges (single-pass, no len()).

Key assumptions / clarifying questions: "Generator" here means a generator function (one containing yield) and, by extension, generator expressions (x for x in it). I'll treat both.

Plan: (1) Define generator + how yield suspends. (2) Contrast execution model with a list-returning function. (3) Memory/laziness trade-offs. (4) Behavioral differences (single iteration, no indexing/len). (5) When to use which, with backend examples. (6) Summary + common mistakes.

What a generator is

A generator is a function that contains at least one yield. Calling it does not run the body; it returns a generator object (an iterator). The body runs incrementally: each time you ask for the next value (via next() or a for loop), it runs until the next yield, hands back that value, and suspends — freezing all local variables and the exact point of execution. On the next request it resumes right after that yield. When the function returns (or falls off the end), iteration stops with StopIteration.

PYTHON
def count_up(n):
    i = 0
    while i < n:
        yield i          # produce a value, then pause here
        i += 1

g = count_up(3)          # nothing has run yet
next(g)  # 0   -> runs to first yield
next(g)  # 1   -> resumes, runs to next yield
next(g)  # 2
next(g)  # raises StopIteration

Contrast the list version, which computes everything up front and hands back the whole materialized list:

PYTHON
def count_up_list(n):
    result = []
    for i in range(n):
        result.append(i)
    return result        # entire list built before caller sees anything
The core difference: lazy vs eager

This is the heart of the answer.

A list function is eager. It builds all N elements in memory before returning. Memory is O(N). The caller waits for the whole computation before getting element 0.
A generator is lazy. It produces one element at a time, on demand. Peak memory is O(1) for the generator's own state (plus whatever one item costs) — it never holds all N at once. The caller gets element 0 as soon as the first yield fires; it can even stop early and never compute the rest.

That laziness enables two things a list cannot:

Huge or streaming data. You can iterate over a 50 GB file, or an unbounded stream, line by line — a list would require loading it all into RAM (or is literally infinite and impossible to materialize).
Infinite sequences. def naturals(): i=0; while True: yield i; i+=1 is fine; the list equivalent never terminates.
Behavioral differences that trip people up
Single-pass / exhaustible. A generator can be iterated once. After it's consumed, it's empty — looping again yields nothing. A list can be iterated any number of times. If you need to reuse the values, materialize with list(gen) or re-create the generator.
No len(), no indexing, no slicing. A generator has no known length and isn't subscriptable (gen[3] fails). A list supports len, indexing, slicing, membership, sorting in place.
return in a generator doesn't return a value to the caller — it just ends iteration (its value becomes the StopIteration.value, mostly used with yield from).
Side effects run lazily. Because the body only advances when pulled, any prints/DB writes inside the generator happen during iteration, not at call time — a subtle source of "why didn't my logging run?" bugs when a generator is created but never consumed.
Backend framing — when to use which

Prefer a generator when:

Streaming large result sets from a DB/file/network so you never buffer it all (e.g., yielding rows and writing them to an HTTP chunked response or a CSV export).
Building a pipeline: for line in filter_valid(parse(read(file))) — each stage is a generator, data flows one item at a time, memory stays flat.
The consumer may stop early (short-circuit search), so computing everything is wasteful.

Prefer returning a list when:

The data is small and you need random access, len, multiple passes, or to sort/index it.
You must fully materialize before releasing a resource (e.g., you'll close the DB cursor/file before the caller iterates — a generator that lazily reads a closed cursor will blow up).
The result is passed to code that expects a real sequence.

Generator expressions give you the lazy benefit inline: sum(x*x for x in nums) streams instead of building a temporary list like sum([x*x for x in nums]) would. Use [...] only when you need the list to persist.

Summary — what a strong answer emphasizes

A generator (a function with yield, or a (...) expression) is a lazy iterator: calling it returns immediately without running the body; each next()/loop step runs to the next yield, returns one value, and suspends with all state frozen. A list-returning function is eager: it computes and stores all N elements up front (O(N) memory) before the caller sees anything. Generators win on memory, streaming, infinite sequences, and early exit; lists win when you need random access, len, or multiple passes. The key gotcha: a generator is single-use and has no length. Common mistakes: iterating a generator twice and getting nothing the second time; calling len() on it; and expecting side effects to run at call time rather than during iteration.

#### Follow-up
probes: "What does calling the generator function return?" (a generator object, body not yet run); "How is peak memory different for sum([...]) vs sum(...)?"; "How would you stream a million DB rows to a CSV without OOM?" (yield rows from a server-side cursor).

**Comparison:**

```
Aspect	Generator (yield)	Function returning a list
Evaluation	Lazy — one item on demand	Eager — all items up front
Peak memory	O(1) state + one item	O(N) whole list
First result available	Immediately (after 1st yield)	Only after everything is built
Iterations	Once (exhaustible)	Any number of times
len() / index / slice	No	Yes
Infinite / streaming data	Yes	No (must fit in memory)
Early exit saves work	Yes	No (already computed)
Best for	streaming, pipelines, big/infinite data	small data, random access, reuse
```

**Code:**

```python
# Generator: lazy, one value at a time, keeps its own state
def count_up(n):
    i = 0
    while i < n:
        yield i          # pause and hand back i; resume here next time
        i += 1

# Eager equivalent: builds the whole list first
def count_up_list(n):
    result = []
    for i in range(n):
        result.append(i)
    return result

# Streaming a large file line-by-line — O(1) memory, works on a 50GB file
def read_valid_lines(path):
    with open(path) as f:
        for line in f:               # file object is itself a lazy iterator
            line = line.strip()
            if line and not line.startswith("#"):
                yield line

# Generator expression vs list comprehension
total = sum(x * x for x in range(10**7))   # lazy: no giant temp list
squares = [x * x for x in range(10)]        # materialized list (needs reuse/index)

# Infinite generator — impossible as a list
def naturals():
    i = 0
    while True:
        yield i
        i += 1
```

**Example input / output:**

```text
g = count_up(3)          # returns a generator object; body NOT run yet
next(g)   -> 0           # runs to first yield, suspends
next(g)   -> 1           # resumes after yield
next(g)   -> 2
next(g)   -> StopIteration

# Single-pass gotcha:
g = count_up(3)
list(g)   -> [0, 1, 2]
list(g)   -> []          # already exhausted!

# No length / no indexing:
len(count_up(3))         # TypeError: object of type 'generator' has no len()
count_up(3)[0]           # TypeError: 'generator' object is not subscriptable

# List version is reusable and sized:
lst = count_up_list(3)
len(lst)  -> 3
lst[0]    -> 0
sum(lst); sum(lst)       -> 3, 3   (works twice)
```

**Flow:**

1. yes
2. body returns / ends
3. Call generator func
4. Returns generator object(body NOT run)
5. next() / for-loop pulls?
6. Run body until next 'yield'
7. Emit value, SUSPEND(freeze locals + position)
8. raise StopIteration(loop ends)
9. Call list func
10. Run WHOLE body,build all N items
11. return full list (O(N) memory)

