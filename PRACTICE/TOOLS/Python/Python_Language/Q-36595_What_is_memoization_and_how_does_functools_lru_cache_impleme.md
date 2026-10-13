---
type: "practice"
difficulty: "Intermediate"
---

# What is memoization, and how does functools.lru_cache implement it? What is a risk of caching unbounded results?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Cache Invalidation Strategies](<../../../../TOOLS/Python/Python_Language/Cache_Invalidation_Strategies/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

The question has three parts: (1) what is memoization, (2) how does functools.lru_cache implement it, and (3) what is the risk of caching unbounded results. The interviewer (backend/fullstack SDE at Amazon, Google, Uber, Razorpay, CRED, PhonePe, etc.) is probing whether I understand a core optimization technique, know the mechanics of the stdlib tool that provides it (not just that it "caches"), and — the maturity check — whether I recognize that an unbounded cache is a memory-leak / OOM risk in a long-running service.

Key assumption to state: memoization only works for pure functions — same inputs must always produce the same output, with no side effects. If the function's result depends on time, randomness, external state, or mutates something, caching gives wrong answers.

Plan: define memoization from first principles → how lru_cache works (hashable-args key, dict + LRU eviction, maxsize, thread-safety, introspection) → the unbounded-cache risk and how to avoid it → summary and gotchas.

What is memoization

Memoization is an optimization where you cache the return value of a function keyed by its arguments, so that a repeated call with the same arguments returns the stored result instead of recomputing it. It trades memory for time: you pay to store past results in exchange for turning an expensive recomputation into a cheap dictionary lookup.

The classic example is recursive Fibonacci. Naive fib(n) re-computes the same subproblems exponentially — fib(35) makes ~29 million calls, O(2^n). Memoize it and each fib(k) is computed once; the whole thing becomes O(n) time and O(n) space. Memoization is essentially top-down dynamic programming: overlapping subproblems solved once and reused.

Memoization requires the function to be pure/deterministic and its arguments to be hashable (they become the cache key). It's a specific form of caching — the term "memoization" implies caching a function's results on its argument tuple, whereas "caching" is the general concept.

How functools.lru_cache implements it

functools.lru_cache(maxsize=128, typed=False) is a decorator that wraps your function with an automatic, bounded memoization cache. LRU = Least Recently Used eviction.

Mechanics:

Key construction. On each call it builds a hashable key from the positional args and keyword args (via an internal _make_key). All arguments must be hashable — that's why you can memoize f(3, 4) but not f([1,2,3]) (a list is unhashable → TypeError). With typed=True, f(3) and f(3.0) are cached separately because their types differ.
Lookup. It looks the key up in an internal dict (O(1) average). Hit → return the stored value immediately, skipping the function body. Miss → call the real function, store the result under that key, and return it.
LRU eviction. The cache is bounded to maxsize entries. Conceptually it's a dict plus a doubly-linked list ordering entries by recency of use; on a hit the entry moves to "most recently used," and when the cache is full a new entry evicts the least recently used one. (CPython's implementation uses a circular doubly-linked list of nodes plus the dict for O(1) get/insert/evict.)
maxsize values.
maxsize=128 (default): keep up to 128 entries, evict LRU beyond that — bounded, safe.
maxsize=None: cache grows without limit — no eviction ever (this is the dangerous mode; functools.cache, added in 3.9, is exactly lru_cache(maxsize=None)).
maxsize should be a power of two for best performance, though any int works.
Thread-safety. The cache operations are protected by a lock, so lru_cache is safe to call from multiple threads (though the wrapped function itself may still be invoked more than once for a key under a race — the cache bookkeeping is what's synchronized).
Introspection & control. The wrapper exposes .cache_info() → a named tuple (hits, misses, maxsize, currsize) to measure hit rate, and .cache_clear() to empty it. Great for tuning and for tests.

Because keys are argument tuples, lru_cache is ideal for pure functions with a modest set of frequently-repeated inputs: recursive DP, parsing/normalization, expensive lookups with a hot key set.

The risk of caching unbounded results

Using maxsize=None (or @cache) means entries are never evicted. In a long-running process (web server, worker) that calls the function with a large or unbounded variety of arguments, the cache grows forever → steadily increasing memory → eventually OOM / the process is killed. This is a genuine, common memory-leak pattern in production Python.

Specific dangers to name:

Unbounded key space: memoizing a function on user IDs, request bodies, timestamps, or URLs — the set of distinct arguments is effectively infinite, so the cache never stops growing.
Large cached values: even a bounded maxsize holds maxsize results; if each result is a huge DataFrame/list, memory blows up. Bound by memory footprint, not just entry count.
Cached objects are never garbage-collected: the cache holds a strong reference to every key (its arguments) and value, so those objects can't be freed while cached. Memoizing an instance method with lru_cache keeps self alive forever → the instance leaks. (Prefer cached_property or a weakref-based cache for methods.)
Stale data: if the underlying "truth" changes (config, DB row), a never-evicting cache serves outdated results indefinitely; you need cache_clear() or a TTL cache.

How to avoid the risk:

Always set a sensible finite maxsize on long-lived services; reserve maxsize=None/@cache only for provably-bounded key spaces (e.g., recursion over a fixed range, or a small enum).
Monitor cache_info() (watch currsize and hit rate) in production.
For expiring data use a TTL cache (cachetools.TTLCache) or an external cache (Redis) instead of lru_cache.
Don't memoize impure functions or functions with unhashable/huge arguments.
Summary / what a strong answer emphasizes
Memoization = cache a pure function's return keyed by its (hashable) arguments; trades memory for time; it's top-down DP for overlapping subproblems.
lru_cache builds a hashable key from args, uses a dict + LRU doubly-linked list for O(1) hit/miss/evict, bounds size to maxsize (default 128), is thread-safe, and exposes cache_info() / cache_clear().
The unbounded risk: maxsize=None with a large/unbounded argument space (or huge values, or memoized methods holding self) causes unbounded memory growth → memory leak / OOM, plus potential stale data. Fix with a finite maxsize, monitoring, or a TTL cache.

#### Follow-up
probes an interviewer might add: "Why can't you lru_cache a function taking a list?" (lists are unhashable — pass a tuple/frozenset), "How would you add expiry?" (cachetools.TTLCache), and "What's the difference between @cache and @lru_cache?" (@cache == lru_cache(maxsize=None), unbounded).

**Comparison:**

```
Setting	Behavior	When to use	Risk
lru_cache(maxsize=128) (default)	keep 128 entries, evict LRU	general use in services	bounded — safe
lru_cache(maxsize=N)	keep N entries, evict LRU	tune to hot key set / memory	bounded — safe
lru_cache(maxsize=None) / @cache	never evict	provably bounded key space (fixed-range recursion, enums)	unbounded growth → OOM
typed=True	3 vs 3.0 cached separately	type matters to result	slightly more entries
Concept	Meaning
Memoization	cache a pure function's result keyed by its (hashable) args — memory for time
Key	tuple of positional + keyword args; must be hashable
Eviction	LRU — least recently used entry dropped when full
cache_info()	(hits, misses, maxsize, currsize) — measure hit rate / size
cache_clear()	empty the cache (config/data reload)
Unbounded risk	maxsize=None + large key space → memory leak / stale data
```

**Code:**

```python
from functools import lru_cache, cache

# ---------- Memoized Fibonacci: O(2^n) -> O(n) ----------
@lru_cache(maxsize=None)          # each fib(k) computed exactly once
def fib(n):
    if n < 2:
        return n
    return fib(n - 1) + fib(n - 2)

print(fib(100))                   # instant; naive recursion would hang
print(fib.cache_info())           # CacheInfo(hits=98, misses=101, maxsize=None, currsize=101)

# ---------- Bounded cache = SAFE for long-running services ----------
@lru_cache(maxsize=1024)          # evicts least-recently-used beyond 1024 entries
def normalize_country(code: str) -> str:
    # imagine an expensive lookup here
    return code.strip().upper()

# ---------- typed=True: 3 and 3.0 cached separately ----------
@lru_cache(maxsize=128, typed=True)
def square(x):
    return x * x

# ---------- Introspection / manual control ----------
normalize_country.cache_info()    # hits, misses, maxsize, currsize -> tune maxsize
normalize_country.cache_clear()   # drop everything (e.g., on config reload)

# ---------- The RISK: unbounded cache on an unbounded key space ----------
@lru_cache(maxsize=None)          # or @cache  -> NEVER evicts
def render_user_page(user_id: str) -> str:
    return expensive_render(user_id)
# In a server, distinct user_ids grow forever -> cache grows forever -> OOM.
# Fix: use a finite maxsize (or a TTL cache for data that changes).

# ---------- Unhashable arg -> TypeError ----------
@lru_cache
def total(nums):
    return sum(nums)
# total([1, 2, 3])     -> TypeError: unhashable type: 'list'
total((1, 2, 3))       # OK: pass a tuple (hashable)
```

**Example input / output:**

```

fib(35) # with @lru_cache 9227465 # returns instantly fib.cache_info() CacheInfo(hits=33, misses=36, maxsize=None, currsize=36)

36 unique subproblems computed once; 33 repeats served from cache.
Naive fib(35) would make ~29,860,703 calls (O(2^n)); memoized is 36 (O(n)).

square(3); square(3.0) # typed=True 9 ; 9.0 square.cache_info() CacheInfo(hits=0, misses=2, maxsize=128, currsize=2) # 3 and 3.0 cached separately

total([1, 2, 3]) TypeError: unhashable type: 'list' # args must be hashable

CODE
Copy

Unbounded-cache memory growth (illustrative, `maxsize=None` in a server):

requests served : 10k 100k 1M distinct user_ids: 8k 70k 600k cache currsize : 8k 70k 600k <- never shrinks process RSS : 120 MB 600 MB 4.5 GB -> OOM kill
```

**Flow:**

1. hit
2. miss
3. yes
4. no, or maxsize=None
5. maxsize=None +unbounded keys
6. call f(args)
7. build hashable key from args
8. key in cache?
9. move key tomost-recently-used
10. return cached value
11. run f, compute result
12. store key -> result
13. cache full? maxsize reached
14. evict LEAST recently used
15. keep all
16. return result
17. cache grows forever -> OOM

