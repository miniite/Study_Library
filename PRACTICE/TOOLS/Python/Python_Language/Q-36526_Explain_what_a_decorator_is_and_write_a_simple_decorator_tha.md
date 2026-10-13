---
type: "practice"
difficulty: "Intermediate"
---

# Explain what a decorator is and write a simple decorator that logs a function's execution time.

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Mutability](<../../../../TOOLS/Python/Python_Language/Mutability.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

What the question is really asking: (1) Define what a decorator is in Python, and (2) write a working decorator that measures and logs how long a function takes to run.

What the interviewer is probing: At this level (intermediate/2-3), they want more than "it's @something above a function." They want to see that you understand a decorator is a callable that takes a function and returns a replacement function, that @deco is syntactic sugar for f = deco(f), and that a correct implementation preserves the wrapped function's metadata (functools.wraps), forwards *args, **kwargs, and returns the original return value. They also often probe: decorators with arguments, using time.perf_counter (not time.time), and where you'd actually use this (profiling, logging, retries, caching, auth).

#### Assumptions
/clarifications: I'll write a general-purpose timing decorator that works for any function signature, logs via the logging module (production-friendly) but note print is fine for quick use, and I'll show the parametrized variant as a bonus. Standard CPython.

From first principles: what a decorator is

In Python, functions are first-class objects — you can pass them as arguments, return them from other functions, and assign them to variables. A decorator exploits this: it is a callable that takes a function as input and returns a (usually enhanced) function as output. It lets you add behavior around a function — before it runs, after it runs, or both — without modifying the function's own body. This is the "open for extension, closed for modification" idea applied to functions.

The @decorator syntax is pure syntactic sugar. These two are identical:

PYTHON
@my_decorator
def greet(): ...

is exactly:

PYTHON
def greet(): ...
greet = my_decorator(greet)

So @my_decorator means: "run greet through my_decorator and rebind the name greet to whatever comes back."

Anatomy of a decorator

A decorator that adds behavior typically has three layers:

The decorator function — receives the original function func.
An inner wrapper function — this is what actually replaces func. It runs your extra logic, calls func(*args, **kwargs), and returns the result.
The decorator returns the wrapper.

The *args, **kwargs in the wrapper are essential — they let the decorated function accept any arguments, so the same decorator works on any function signature.

Why functools.wraps matters

When you replace func with wrapper, the new function loses func's identity — func.__name__ becomes "wrapper", its docstring disappears, and introspection/help() breaks. @functools.wraps(func) copies __name__, __doc__, __module__, __wrapped__, etc. from the original onto the wrapper. A correct decorator always uses it — its absence is the single most common giveaway of a junior implementation.

Building the timing decorator

The logic: record a start timestamp, call the function, record an end timestamp, log the difference, and — critically — return the function's real result so callers see no difference.

PYTHON
import functools
import logging
import time

logger = logging.getLogger(__name__)

def log_execution_time(func):
    @functools.wraps(func)            # preserve name/docstring of func
    def wrapper(*args, **kwargs):
        start = time.perf_counter()   # high-resolution monotonic clock
        try:
            return func(*args, **kwargs)   # run the real function, return its result
        finally:
            elapsed = time.perf_counter() - start
            logger.info("%s took %.6f seconds", func.__name__, elapsed)
    return wrapper

Key correctness details, and why each matters:

time.perf_counter() not time.time() — perf_counter is the highest-resolution monotonic clock; it won't jump backward if the system clock is adjusted (NTP/DST). time.time() is a wall clock and unreliable for measuring durations.
return func(...) inside try / timing in finally — this guarantees the elapsed time is logged even if the function raises, and the exception still propagates normally. A naive version that times only on success silently mis-measures the failure path.
*args, **kwargs — forwards every positional and keyword argument, so it works on any signature.
@functools.wraps(func) — keeps func.__name__, docstring, and introspection intact.
Bonus: a decorator that takes arguments

Interviewers frequently follow up with "make the log level / label configurable." A parametrized decorator adds one more layer — the outer function takes the config, and returns the real decorator:

PYTHON
def timed(label=None, level=logging.INFO):
    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            start = time.perf_counter()
            try:
                return func(*args, **kwargs)
            finally:
                name = label or func.__name__
                logger.log(level, "%s took %.6f s", name, time.perf_counter() - start)
        return wrapper
    return decorator

@timed(label="checkout")
def process_order(order): ...

Here @timed("checkout") first calls timed(...) to produce a decorator, which then wraps process_order. That's why parametrized decorators need three nested functions.

Where this is used in the real world

Decorators shine for cross-cutting concerns — behavior you want on many functions without repeating it: timing/profiling (this question), structured logging, @lru_cache for memoization, retry-with-backoff, authentication/authorization checks (@login_required), input validation, rate limiting, and framework routing (@app.route, @app.get in Flask/FastAPI). The value is that the function's core logic stays clean while the decorator handles the orthogonal concern.

Summary — what a strong answer emphasizes
A decorator is a callable that takes a function and returns a replacement function; @deco is sugar for f = deco(f).
A correct timing decorator uses a wrapper with *args, **kwargs, returns the real result, times with time.perf_counter(), logs even on exceptions (try/finally), and preserves metadata with @functools.wraps.
Mention the parametrized-decorator pattern (three nested functions) and real uses (caching, retries, auth, routing).

#### Common mistakes
forgetting functools.wraps; forgetting to return the result (breaks any function that produces a value); using time.time(); not forwarding **kwargs; timing only the success path.

#### Follow-up
probes: "How do you make it accept arguments?" (extra outer layer). "What breaks without functools.wraps?" (__name__/docstring/introspection). "How would you time an async def?" (write an async def wrapper and await func(...)). "How does @lru_cache fit this model?"

**Comparison:**

```
Design choice	Wrong / naive	Correct	Why it matters
Metadata	plain def wrapper	@functools.wraps(func)	keeps __name__, docstring, introspection
Return	calls func() but drops result	return func(...)	decorator must be transparent to callers
Args	fixed signature	*args, **kwargs	works on ANY function
Clock	time.time()	time.perf_counter()	monotonic, high-res, immune to clock changes
Errors	time only on success	try / finally	logs duration even when func raises
```

**Code:**

```

import functools import logging import time

logging.basicConfig(level=logging.INFO) logger = logging.getLogger(name)

def log_execution_time(func): """Decorator that logs how long func takes to execute.""" @functools.wraps(func) # keep func's name/docstring def wrapper(*args, **kwargs): start = time.perf_counter() # monotonic, high resolution try: return func(*args, **kwargs) # run + return the real result finally: elapsed = time.perf_counter() - start logger.info("%s took %.6f seconds", func.name, elapsed) return wrapper

@log_execution_time def slow_add(a, b): """Add two numbers, slowly.""" time.sleep(0.2) return a + b

if name == "main": print(slow_add(2, 3)) # -> 5, plus a timing log line print(slow_add.name) # -> 'slow_add' (metadata preserved) print(slow_add.doc) # -> 'Add two numbers, slowly.'

CODE
Copy

```python
# Parametrized variant (configurable label / log level)
def timed(label=None, level=logging.INFO):
    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            start = time.perf_counter()
            try:
                return func(*args, **kwargs)
            finally:
                name = label or func.__name__
                logger.log(level, "%s took %.6f s", name, time.perf_counter() - start)
        return wrapper
    return decorator

@timed(label="checkout")
def process_order(order_id):
    time.sleep(0.05)
    return f"order {order_id} done"
```

**Example input / output:**

```

Running the first block:

TEXT
Copy
INFO:__main__:slow_add took 0.200XYZ seconds
5
slow_add
Add two numbers, slowly.
The return value 5 is unchanged — the decorator is transparent to callers.
slow_add.__name__ is still 'slow_add' (not 'wrapper') thanks to functools.wraps.

Exception path (timing still logged):

PYTHON
Copy
@log_execution_time
def boom():
    time.sleep(0.1)
    raise ValueError("nope")

boom()
# INFO:__main__:boom took 0.100XYZ seconds   <- logged from finally
# Traceback ... ValueError: nope             <- exception still propagates
```

**Flow:**

1. original func
2. wrapper (replaces func)
3. Caller
4. original func
5. wrapper (replaces func)
6. Caller
7. slow_add(2, 3)
8. start = perf_counter()
9. func(2, 3)
10. return 5
11. elapsed = perf_counter() - start
12. logger.info("slow_add took ...")
13. return 5

