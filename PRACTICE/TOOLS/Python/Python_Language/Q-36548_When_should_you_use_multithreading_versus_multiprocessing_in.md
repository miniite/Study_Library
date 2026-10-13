---
type: "practice"
difficulty: "Deep"
---

# When should you use multithreading versus multiprocessing in Python? Give an I/O-bound and a CPU-bound example.

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ The GIL and CPU-bound Work](<../../../../TOOLS/Python/Python_Language/Cache_Invalidation_Strategies/The_GIL_and_CPU-bound_Work.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

The interviewer is asking me to pick the right concurrency primitive for a workload and, crucially, to justify why — which means the real thing being probed is whether I understand the GIL (Global Interpreter Lock) and how it interacts with I/O-bound versus CPU-bound work. A weak answer says "use threads for I/O, processes for CPU" and stops. A strong answer explains the mechanism that makes that rule true, names the costs of each choice, and knows the exceptions.

#### Assumptions
I'd state up front: we're on CPython (the reference interpreter — Jython, PyPy-STM, or a future free-threaded 3.13+ --disable-gil build behave differently), and by "multithreading" I mean the threading module and by "multiprocessing" the multiprocessing module. Plan: (1) explain the GIL, (2) derive the I/O vs CPU rule from it, (3) give a concrete example of each, (4) cover the trade-offs and the "third option" (asyncio), (5) note the exceptions so I don't over-claim.

First principles: the GIL

In CPython, the GIL is a mutex that lets only one thread execute Python bytecode at a time, even on a multi-core machine. It exists to make CPython's reference-counting memory management thread-safe without fine-grained locks everywhere. The consequence: threads in CPython give you concurrency (interleaving) but not true parallelism of Python code.

The key nuance is when the GIL is released:

The GIL is released while a thread is blocked on I/O — reading a socket, a disk, a database, an HTTP call. During that wait the thread is not running bytecode, so it hands the GIL to another thread.
The GIL is also released inside many C extensions that do heavy number-crunching (NumPy, hashlib, compression, image libs) — they drop it around the native loop.
But pure-Python CPU work (a tight for loop doing math in Python) holds the GIL the whole time, so multiple threads doing that just take turns and give you zero speedup — sometimes a slowdown from lock contention.

Everything below falls out of those three facts.

The rule, derived

I/O-bound work → threads (or asyncio). The program spends most of its time waiting on the network/disk, not computing. While thread A waits, the GIL is free, so thread B runs. You overlap the waits, and total wall-clock time drops toward the time of the single slowest call instead of the sum of all calls. You don't need multiple cores because you were never CPU-limited — you were latency-limited.

CPU-bound work → processes. The program spends its time computing in Python. Threads can't help because only one holds the GIL at a time. multiprocessing sidesteps the GIL entirely by spawning separate OS processes, each with its own Python interpreter and its own GIL. Now N processes truly run on N cores in parallel. The cost is that processes don't share memory, so arguments and results must be pickled and sent over a pipe, and startup is heavier.

Concrete example — I/O-bound (threads win)

Fetching 20 URLs. Each request takes ~500 ms, almost all of it waiting for the server. Serially that's ~10 s. With a thread pool of 20, all requests are in flight at once and it finishes in ~0.5 s — roughly a 20× win — because while each thread waits, the GIL is released for the others. (See code_blocks and input_output.)

Concrete example — CPU-bound (processes win)

Summing primality checks / a heavy math loop over a range, in pure Python. On a 4-core box:

Threads: ~same as serial (or worse). All 4 threads fight over one GIL doing Python math; there's no I/O to release it.
Processes (4 workers): ~4× faster. Each process gets its own core and its own GIL.

That contrast — threads help the first, processes help the second — is exactly what the interviewer wants demonstrated.

#### Trade-offs
and how I actually choose
Dimension	threading	multiprocessing	asyncio
Best for	I/O-bound	CPU-bound (pure Python)	I/O-bound, very high fan-out
Parallel Python execution	No (one GIL)	Yes (GIL per process)	No (single thread)
Memory	Shared (cheap, but needs locks)	Separate (pickled IPC)	Shared
Overhead per task	Low (~KBs stack)	High (process + pickle)	Very low (~KB coroutine)
Failure isolation	A crash can take the process	A worker crash is isolated	In-process
Main risk	Race conditions, deadlocks	Serialization cost, no shared state	Blocking call stalls the loop

Practical decision process: First ask, is it CPU-bound or I/O-bound? If I/O-bound and the code is already blocking/synchronous, concurrent.futures.ThreadPoolExecutor is the least-friction choice. If I/O-bound with thousands of concurrent connections (a server), asyncio scales better because coroutines are far lighter than OS threads. If CPU-bound, ProcessPoolExecutor / multiprocessing. I lean on the unified concurrent.futures API so I can swap ThreadPoolExecutor ↔ ProcessPoolExecutor with a one-line change and measure.

Important exceptions (this is what separates senior answers)
CPU-bound but in C: NumPy, Pandas, scikit-learn, PyTorch, hashlib, zlib release the GIL around their native loops. So a "CPU-heavy" NumPy matrix multiply can parallelize with threads — the heavy lifting isn't Python bytecode. Always check whether the hot loop is Python or C before reaching for processes.
Free-threaded CPython (PEP 703, experimental in 3.13): a --disable-gil build makes threads genuinely parallel for Python code too. Rule-of-thumb "threads can't do CPU" is a CPython-with-GIL statement, not a Python-language statement.
multiprocessing gotchas: all arguments and return values must be picklable; on macOS/Windows the default start method is spawn, which re-imports your module, so process-creating code must be guarded by if __name__ == "__main__": or you get infinite spawning. Shared state needs multiprocessing.Queue, Value, Array, or shared_memory.
Summary — what a strong answer emphasizes

The decision is driven by one question — is the bottleneck waiting or computing? — and one fact — the GIL lets only one thread run Python bytecode but is released during I/O and in many C extensions. Waiting → threads/asyncio (overlap the waits, no cores needed). Computing in pure Python → processes (own GIL per core, at the cost of pickling and startup). The senior signal is naming the exceptions: NumPy-style C code that releases the GIL, the free-threaded build, and the spawn/pickling pitfalls of multiprocessing. Common mistakes to avoid: claiming threads give parallel speedup for pure-Python CPU work (they don't), forgetting the __main__ guard, and ignoring that IPC serialization can make small multiprocessing tasks slower than serial.

**Comparison:**

```
Bottleneck	Symptom	Use	Why	Speedup source
I/O-bound	high wait, low CPU%	threading / asyncio	GIL released during I/O	overlap the waits
CPU-bound (pure Python)	one core pinned at 100%	multiprocessing	separate interpreter+GIL per process	true multi-core
CPU-bound (NumPy/C)	one core at 100%, but work is in C	threading (or processes)	C extension releases the GIL	native parallel loop
Massive I/O fan-out (10k conns)	thousands of sockets	asyncio	coroutines far lighter than OS threads	cheap overlap
```

**Code:**

```python
# I/O-bound: threads overlap the network waits (GIL released while waiting)
import time, requests
from concurrent.futures import ThreadPoolExecutor

URLS = ["https://httpbin.org/delay/1"] * 20  # each ~1s, mostly waiting

def fetch(url):
    return requests.get(url).status_code

start = time.perf_counter()
with ThreadPoolExecutor(max_workers=20) as pool:
    results = list(pool.map(fetch, URLS))
print(f"threads: {time.perf_counter() - start:.2f}s")  # ~1s, not ~20s

# CPU-bound: processes give true parallelism (one GIL per process)
from concurrent.futures import ProcessPoolExecutor

def count_primes(n):
    def is_prime(k):
        if k < 2: return False
        i = 2
        while i * i <= k:
            if k % i == 0: return False
            i += 1
        return True
    return sum(is_prime(x) for x in range(n))

if __name__ == "__main__":          # REQUIRED for 'spawn' start method
    ranges = [1_000_000] * 4
    start = time.perf_counter()
    with ProcessPoolExecutor(max_workers=4) as pool:
        list(pool.map(count_primes, ranges))
    print(f"processes: {time.perf_counter() - start:.2f}s")  # ~4x faster than serial
    # Swapping ProcessPoolExecutor -> ThreadPoolExecutor here gives NO speedup (GIL).
```

**Example input / output:**

```text
I/O-bound (20 requests, ~1s each, all waiting):
  serial    : ~20.0 s   (1 + 1 + ... )
  4 threads : ~5.0 s
  20 threads: ~1.1 s     <- overlaps all the waits; GIL free during I/O

CPU-bound (4 x count_primes(1_000_000), pure Python) on a 4-core box:
  serial       : ~8.0 s
  4 threads    : ~8.2 s  <- NO speedup: all fight one GIL, sometimes slower
  4 processes  : ~2.1 s  <- ~4x: each process has its own core + own GIL

Exception (CPU-bound but NumPy):
  np.dot on large arrays with 4 threads DOES speed up
  because the C loop releases the GIL.
```

**Flow:**

1. Waiting on I/O
2. hundreds
3. thousands+
4. Computing
5. pure Python
6. NumPy/hashlib/C ext
7. Pick a concurrency model
8. Bottleneck?
9. How many concurrent tasks?
10. threading /ThreadPoolExecutor
11. asyncio
12. Work in pure Python or in C?
13. multiprocessing /ProcessPoolExecutor
14. threading - C releases the GIL

