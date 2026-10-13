---
type: "practice"
difficulty: "Deep"
---

# What is the Global Interpreter Lock (GIL) and how does it affect multithreaded CPU-bound Python code?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ The GIL and CPU-bound Work](<../../../../TOOLS/Python/Python_Language/Cache_Invalidation_Strategies/The_GIL_and_CPU-bound_Work.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

The question asks what the GIL is and how it affects multithreaded CPU-bound Python code. At "deep understanding" level, the interviewer wants more than "the GIL means only one thread runs" — they want: why the GIL exists (reference counting / memory-model safety), the mechanism by which it's acquired/released and switched (the switch interval, not the old opcode-count model), the concrete performance consequence for CPU-bound vs I/O-bound work (including the pathological multi-core slowdown), the ways around it (multiprocessing, C extensions releasing the GIL, alternative interpreters), and the 2024/2025 developments (PEP 703 free-threaded / no-GIL builds in 3.13+, and per-interpreter GILs from PEP 684). A senior answer also states clearly that the GIL is a CPython implementation detail, not a language guarantee.

#### Assumptions
we're talking CPython (Jython/IronPython have no GIL; PyPy has one). "Threads" = OS threads via the threading module. Plan: define the GIL → why it exists → how it's scheduled → CPU-bound impact with a concrete example → I/O-bound contrast → escape hatches → modern no-GIL/sub-interpreter story → judgment on what to actually do.

What the GIL is

The Global Interpreter Lock is a single mutex in the CPython interpreter that must be held by a thread in order to execute Python bytecode. Even on a machine with 32 cores and 32 Python threads, only one thread executes Python bytecode at any instant because only one can hold the GIL. It effectively serialises Python-level execution across all threads within a single process.

Why it exists — reference counting

CPython manages memory with reference counting: every object has an ob_refcnt field, incremented/decremented as references are created/destroyed, and freed when it hits zero. These refcount updates happen constantly and would be data races if two threads touched the same object's refcount simultaneously — leading to premature frees (use-after-free) or leaked memory. Making every refcount operation atomic (via locks or atomic CPU instructions) would add overhead to every object access and slow down single-threaded code, which is the common case.

The GIL is the pragmatic answer: instead of fine-grained locking per object, take one coarse lock around the whole interpreter. This makes single-threaded code fast, keeps the C API simple (extension authors don't need to reason about concurrent refcounts), and guarantees interpreter-internal data structures stay consistent. The trade-off is that it prevents Python-level parallelism across cores.

How the GIL is scheduled — the switch interval

A common misconception is that the GIL switches every N bytecodes. That was the old (pre-3.2) model. Since Python 3.2 (Antoine Pitrou's rewrite), the GIL is time-based: a running thread is asked to give up the GIL after a switch interval (default 5 ms, tunable via sys.setswitchinterval()). A waiting thread sets a flag; the running thread checks it at a safe point and drops the GIL so another thread can run. Crucially, the interpreter releases the GIL around blocking operations — I/O, time.sleep, and inside C extensions that opt in — which is what makes threads useful for I/O.

Impact on CPU-bound multithreaded code

For CPU-bound work (tight loops, math, parsing) that stays in Python bytecode, threads give no speedup because only one runs at a time. Worse, on multi-core machines you can see a slowdown versus single-threaded, because the threads fight over the GIL: the OS wakes waiting threads, they attempt to acquire the lock, get preempted, and this "GIL battle" / convoy effect burns cycles on context switches and lock contention. So a 4-thread CPU-bound program can run slower than the same work in one thread. This is the single most important practical fact to convey.

#### Example
summing 10⁸ numbers split across 4 threads takes about the same wall-clock time as doing it in one thread (sometimes longer) — the CPU never exceeds ~1 core of useful Python work. The same split across 4 processes nearly quarters the time, because each process has its own interpreter and its own GIL on its own core.

Contrast: I/O-bound work

For I/O-bound work (network calls, disk, DB, subprocess), threads do help, because a thread releases the GIL while blocked on the OS. While thread A waits on a socket, thread B runs Python. So threading (or asyncio for cooperative concurrency) genuinely improves throughput for I/O-heavy workloads — the GIL is only a wall for CPU parallelism.

Escape hatches (the classic advice)
multiprocessing / ProcessPoolExecutor — spawn separate processes, each with its own GIL, achieving true multi-core parallelism. Cost: process startup and IPC/pickling overhead; best for coarse-grained, embarrassingly parallel work.
C extensions / native libraries that release the GIL — NumPy, pandas, SciPy, hashlib, compression, and many DB/ML libraries drop the GIL around heavy C computation, so their internal work does run in parallel across threads. This is why "NumPy-vectorised + threads" can scale even though pure-Python loops can't.
concurrent.futures / asyncio — right tool selection: threads/async for I/O, processes for CPU.
Alternative interpreters — Jython and IronPython have no GIL; PyPy has one but is much faster overall. Rewriting the hot path in Cython/Rust (PyO3)/C and releasing the GIL is a common production fix.
The modern picture — free-threading and per-interpreter GILs

This is where a deep answer earns its keep in 2025:

PEP 703 — free-threaded CPython ("no-GIL"): shipped as an experimental build in Python 3.13 (2024) and being stabilised through 3.14+. It removes the GIL using biased/deferred reference counting and per-object locking, allowing true multi-threaded parallelism of Python bytecode. It's opt-in (python3.13t), currently has a single-threaded performance cost and requires C extensions to be thread-safe, so it's not yet the default — but it's the long-term direction.
PEP 684 — per-interpreter GIL: since Python 3.12, each sub-interpreter can have its own GIL. Combined with PEP 734 (interpreters module in 3.13/3.14), you can run multiple isolated interpreters in one process, each running Python in parallel — a middle ground between threads and processes with lower overhead than multiprocessing.

Stating that the GIL is a CPython implementation detail actively being removed, and naming these PEPs, signals current, senior-level awareness.

Judgment — what I'd actually do

Diagnose the bottleneck first. If it's I/O-bound, use asyncio or threads — the GIL is a non-issue. If it's CPU-bound, don't reach for threading; use multiprocessing/ProcessPoolExecutor, push the hot loop into a GIL-releasing native library (NumPy/Cython/Rust), or evaluate the free-threaded 3.13+ build if the deployment and all C dependencies support it. Only after profiling — premature "add threads" is exactly the mistake the GIL punishes.

Summary — what a strong answer emphasises
The GIL is one mutex allowing only one thread to run Python bytecode at a time in CPython, existing primarily to make reference counting thread-safe cheaply.
It's time-sliced (default 5 ms switch interval), and released around I/O and in cooperating C extensions.
CPU-bound threads get no speedup and can even slow down (GIL contention on multi-core); I/O-bound threads benefit.
Work around it with processes, GIL-releasing native code, or async; and know the PEP 703 free-threaded build (3.13) and PEP 684 per-interpreter GIL (3.12) as the future.
It's a CPython detail, not a Python-language rule.

Common follow-ups: "Why not just remove it?" (refcounting + C-API compatibility + single-thread perf — exactly what PEP 703 is carefully solving). "Does the GIL make my code thread-safe?" (No — it protects the interpreter's internals per bytecode, but your multi-step operations can still race; you still need locks). "Why does NumPy scale with threads?" (it releases the GIL during native compute).

**Comparison:**

```
Workload	threading	asyncio	multiprocessing	Why
CPU-bound (pure Python)	No speedup / can slow down	No	Near-linear speedup	GIL serialises bytecode; processes each get a GIL/core
CPU-bound via NumPy/C ext	Speeds up	—	Speeds up	native lib releases the GIL during compute
I/O-bound (net/disk/DB)	Speeds up	Speeds up (best for many conns)	Works but heavy	GIL released while blocked on I/O
Free-threaded 3.13+ (PEP 703)	Speeds up CPU-bound too	—	—	GIL removed; true thread parallelism
Concept	Old model (<3.2)	Current model (3.2+)
GIL switching	every N bytecode instructions	time-based switch interval (default 5 ms)
Tuning knob	sys.setcheckinterval	sys.setswitchinterval
```

**Code:**

```python
import time, threading
from multiprocessing import Pool

def burn(n):                     # pure-Python CPU-bound work
    total = 0
    for i in range(n):
        total += i * i
    return total

N, WORK = 4, 20_000_000

# --- Threads: no speedup for CPU-bound work (GIL serialises bytecode) ---
def run_threads():
    t0 = time.perf_counter()
    ts = [threading.Thread(target=burn, args=(WORK,)) for _ in range(N)]
    for t in ts: t.start()
    for t in ts: t.join()
    return time.perf_counter() - t0

# --- Processes: near-linear speedup (each has its own GIL / core) ---
def run_processes():
    t0 = time.perf_counter()
    with Pool(N) as p:
        p.map(burn, [WORK] * N)
    return time.perf_counter() - t0

if __name__ == "__main__":
    print(f"4 threads   : {run_threads():.2f}s")   # ~ same as 1-thread time (or worse)
    print(f"4 processes : {run_processes():.2f}s")  # ~ 4x faster on a 4-core box

# Inspect / tune the switch interval:
import sys
print("switch interval:", sys.getswitchinterval())   # 0.005 (5 ms) by default
# sys.setswitchinterval(0.001)  # more frequent switching (rarely needed)
```

**Example input / output:**

```text
Illustrative output on a 4-core CPython 3.11 (GIL) machine:

4 threads   : 6.10s     <- no speedup; ~equal to single-thread, sometimes worse
4 processes : 1.65s     <- ~4x faster; true multi-core parallelism
switch interval: 0.005

On a free-threaded build (python3.13t, PEP 703):
4 threads   : ~1.7s     <- threads now parallelise CPU-bound work
```

**Flow:**

1. yes
2. no
3. 4 Python threads ready
4. Which holds the GIL?
5. Only ONE runs Pythonbytecode
6. Does it block on I/O orcall a GIL-releasing C ext?
7. Releases GIL early ->another thread runs -> I/Oparallelism
8. Runs until 5 ms switch interval,then hands GIL to next thread
9. CPU-bound: net work = ~1core(no multi-core speedup;contention overhead)
10. Fix: multiprocessing / nativeGIL-releasing lib /async for I/O / free-threaded3.13+

