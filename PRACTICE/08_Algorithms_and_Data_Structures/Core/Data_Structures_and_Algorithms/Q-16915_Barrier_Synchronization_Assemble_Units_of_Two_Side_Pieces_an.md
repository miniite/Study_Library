---
type: "practice"
difficulty: "Other"
---

# Barrier Synchronization: Assemble Units of Two Side Pieces and One Mount

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Concurrency](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Concurrency.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

An assembly line joins parts into finished units. Each finished unit needs exactly three parts: two side pieces and one mount piece. Parts arrive as concurrent threads of two kinds:

Side threads call a method side(fitSide) and use the callback fitSide() to attach one side piece.
Mount threads call a method mount(fitMount) and use the callback fitMount() to attach one mount piece.

All the threads share one coordinator object. A thread must wait at a barrier until a full unit's worth of parts (two side threads and one mount thread) can proceed together. When a complete trio is present they all pass the barrier and attach their parts, and every thread of one unit must finish attaching before any thread of the next unit begins.

You do not have to figure out which specific threads pair up — the threads are anonymous. The only requirement is that the sequence of attachments, split into consecutive groups of three, always yields exactly two side attachments and one mount attachment per group. Concretely:

If a mount thread reaches the barrier while no side threads are waiting, it must block until two side threads arrive.
If a side thread reaches the barrier alone, it must block until one more side thread and one mount thread arrive.

Write the synchronization inside side and mount that enforces these rules.

Input

A string parts describing the order in which threads are launched; 'S' starts a side thread and 'M' starts a mount thread. If there are n units then the string has length 3n, with exactly 2n characters 'S' and n characters 'M'.

Output

The concatenation of the letters actually attached (in the order the callbacks run). Any output is accepted as long as every consecutive block of three letters contains exactly two 'S' and one 'M'.

#### Cons
traints

3 * n == parts.length
1 <= n <= 20
parts[i] is either 'S' or 'M'.
There are exactly 2 * n occurrences of 'S' and exactly n occurrences of 'M'.

#### Example
s & test cases
TEXT
Input: parts = "SMS" -> Output: "SSM"
Why: one unit; "SMS" and "MSS" are equally valid — the only rule is two S and one M in the group.

Input: parts = "MSS" -> Output: "SSM"
Why: the mount thread arrives first and must block until both side threads are present before the barrier opens.

Input: parts = "SSMMSS" -> Output: "SSMSSM"
Why: two units; the first three attachments and the last three each contain two S and one M, even though input order mixes them.

Input: parts = "MMSSSS" -> Output: "SSMSSM"
Why: both mount threads arrive before any side thread; each must wait for its own pair of side threads, so no unit is completed early.

Input: parts = "SSMSSSMSM" -> Output: "SSMSSMSSM"
Why: three units (six S, three M); confirms the barrier keeps re-arming and never lets a fourth part slip into a group of three.

#### Approach

This is the classic barrier / rendezvous pattern with admission control. Two counting semaphores act as admission gates: the side semaphore starts with two permits and the mount semaphore with one, so only two side threads and one mount thread can be "in flight" for the current unit at any moment. A reusable barrier of size three then forces those exactly-right three threads to rendezvous before any of them attaches its part, which is what guarantees a full, correct unit (two sides + one mount) and prevents parts of the next unit from mixing in. After attaching, each thread releases its semaphore permit so the next unit's threads can be admitted; the barrier automatically re-arms for the following round. The key insight is separating how many of each kind may enter (semaphores) from waiting until a complete set is present (barrier).

Complexity

Time: O(1) synchronization overhead per thread (a constant number of semaphore and barrier operations), so O(total threads) = O(3n) across the whole run. Space: O(1) — two semaphores and one barrier of fixed size 3, independent of n (the JS model additionally holds at most two waiters at the barrier at a time).

Python
PYTHON
from threading import Semaphore, Barrier
from typing import Callable

class UnitAssembler:
    def __init__(self):
        self.side_slots = Semaphore(2)   # two side pieces per unit
        self.mount_slots = Semaphore(1)  # one mount piece per unit
        self.gate = Barrier(3)           # opens once a full trio arrives

    def side(self, fit_side: Callable[[], None]) -> None:
        self.side_slots.acquire()
        self.gate.wait()
        fit_side()
        self.side_slots.release()

    def mount(self, fit_mount: Callable[[], None]) -> None:
        self.mount_slots.acquire()
        self.gate.wait()
        fit_mount()
        self.mount_slots.release()
JavaScript
JAVASCRIPT
// Async semaphore: acquire() waits for a permit, release() grants one.
class Sem {

#### cons
tructor(permits) { this.permits = permits; this.queue = []; }
    async acquire() {
        if (this.permits > 0) { this.permits--; return; }
        await new Promise(res => this.queue.push(res));
    }
    release() {
        if (this.queue.length) this.queue.shift()();
        else this.permits++;
    }
}

// Reusable barrier of `n`: the n-th arrival releases all waiters, then re-arms.
class Barrier {

#### cons
tructor(n) { this.n = n; this.count = 0; this.waiters = []; }
    async wait() {
        this.count++;
        if (this.count === this.n) {

#### cons
t w = this.waiters; this.waiters = []; this.count = 0;
            w.forEach(res => res());
        } else {
            await new Promise(res => this.waiters.push(res));
        }
    }
}

class UnitAssembler {

#### cons
tructor() {
        this.sideSlots = new Sem(2);   // two side pieces per unit
        this.mountSlots = new Sem(1);  // one mount piece per unit
        this.gate = new Barrier(3);
    }

    async side(fitSide) {
        await this.sideSlots.acquire();
        await this.gate.wait();
        fitSide();
        this.sideSlots.release();
    }

    async mount(fitMount) {
        await this.mountSlots.acquire();
        await this.gate.wait();
        fitMount();
        this.mountSlots.release();
    }
}
Java
JAVA
import java.util.concurrent.Semaphore;
import java.util.concurrent.CyclicBarrier;
import java.util.concurrent.BrokenBarrierException;

class UnitAssembler {
    private final Semaphore sideSlots  = new Semaphore(2); // two side pieces per unit
    private final Semaphore mountSlots = new Semaphore(1); // one mount piece per unit
    private final CyclicBarrier gate   = new CyclicBarrier(3);

    public void side(Runnable fitSide) throws InterruptedException {
        sideSlots.acquire();
        try { gate.await(); } catch (BrokenBarrierException e) { /* ignore */ }
        fitSide.run();
        sideSlots.release();
    }

    public void mount(Runnable fitMount) throws InterruptedException {
        mountSlots.acquire();
        try { gate.await(); } catch (BrokenBarrierException e) { /* ignore */ }
        fitMount.run();
        mountSlots.release();
    }
}
C++
CPP
#include <semaphore.h>
#include <pthread.h>
#include <functional>

class UnitAssembler {
    sem_t sideSlots;          // 2 permits: two side pieces per unit
    sem_t mountSlots;         // 1 permit:  one mount piece per unit
    pthread_barrier_t gate;   // opens when 3 threads have arrived
public:
    UnitAssembler() {
        sem_init(&sideSlots, 0, 2);
        sem_init(&mountSlots, 0, 1);
        pthread_barrier_init(&gate, nullptr, 3);
    }

    void side(std::function<void()> fitSide) {
        sem_wait(&sideSlots);
        pthread_barrier_wait(&gate);
        fitSide();
        sem_post(&sideSlots);
    }

    void mount(std::function<void()> fitMount) {
        sem_wait(&mountSlots);
        pthread_barrier_wait(&gate);
        fitMount();
        sem_post(&mountSlots);
    }
};
C
C
#include <semaphore.h>
#include <pthread.h>

/* Two counting semaphores cap how many of each kind can enter a round:
   at most 2 side threads and 1 mount thread. A reusable barrier of 3
   makes the admitted trio pass together, guaranteeing complete units. */
typedef struct {
    sem_t sideSlots;          /* 2 permits: two side pieces per unit */
    sem_t mountSlots;         /* 1 permit:  one mount piece per unit */
    pthread_barrier_t gate;   /* trips once 3 threads have arrived   */
} UnitAssembler;

void assemblerInit(UnitAssembler* a) {
    sem_init(&a->sideSlots, 0, 2);
    sem_init(&a->mountSlots, 0, 1);
    pthread_barrier_init(&a->gate, NULL, 3);
}

void side(UnitAssembler* a, void (*fitSide)(void)) {
    sem_wait(&a->sideSlots);          /* claim one of the two side slots */
    pthread_barrier_wait(&a->gate);   /* wait for the full trio          */
    fitSide();
    sem_post(&a->sideSlots);          /* free the slot for the next unit */
}

void mount(UnitAssembler* a, void (*fitMount)(void)) {
    sem_wait(&a->mountSlots);
    pthread_barrier_wait(&a->gate);
    fitMount();
    sem_post(&a->mountSlots);
}

#### Follow-up
questions
Rewrite the coordinator using only a mutex and condition variables (no barrier primitive) and prove the completeness invariant still holds.
Generalize to units requiring a pieces of type A and b pieces of type B; how do the semaphore permits and barrier size change?
Without the barrier, could a subtle interleaving let a third side thread attach before the mount of the current unit? Show why the barrier prevents it.
How would you detect or avoid deadlock if the barrier could be broken (a thread cancelled while waiting)?

