---
type: "practice"
difficulty: "Other"
---

# Three Threads Emitting an Interleaved Marker-and-Counter Stream

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Concurrency](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Concurrency.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are building a coordinated printer that produces a single stream of digits by running three worker threads concurrently. A helper emit(x) is available that prints the integer x (for instance, emit(4) writes 4 to the output).

You are given one shared object constructed with a positive integer n. Exactly three threads will each call one of its three methods on the same object:

Marker thread calls marker(emit) — it may only print the value 0.
Odd thread calls oddValue(emit) — it may only print odd numbers.
Even thread calls evenValue(emit) — it may only print even numbers.

Your job is to add the synchronization so that, no matter how the operating system schedules the three threads, the combined output is exactly:

0, 1, 0, 2, 0, 3, 0, 4, ... up to n

That is, a 0 is printed before every counter value, and the counter runs 1, 2, 3, ..., n. Odd counter values must come from the odd thread and even counter values from the even thread. The total number of printed digits is 2n (one 0 and one counter value for each of the n steps). For n = 3 the stream is 010203.

Implement the shared class so that the three methods cooperate to produce this exact ordering.

Input

n: a positive integer passed to the constructor; the counter is printed from 1 to n.
emit: a callback each method uses to print a single integer.

Output

The concatenated console output must be the interleaved marker/counter stream described above, of length 2n.

#### Cons
traints

1 <= n <= 1000
Each method is invoked exactly once, each on its own thread, sharing the same object.
emit prints exactly one integer per call and must never be called concurrently in a way that interleaves a single value.

#### Example
s & test cases
TEXT
Input: n = 1 -> Output: "01"
Why: minimum case; only one step, so a single marker followed by the counter value 1 (from the odd thread).

Input: n = 2 -> Output: "0102"
Why: two steps; 0 then 1 (odd), 0 then 2 (even). Confirms the even thread runs after the odd thread's step.

Input: n = 3 -> Output: "010203"
Why: odd/even alternation continues; the last value 3 is odd, so the odd thread emits the final counter.

Input: n = 5 -> Output: "0102030405"
Why: longer run; verifies markers keep separating values and no thread races ahead.

Input: n = 6 -> Output: "010203040506"
Why: ends on an even value, so the even thread produces the last counter; confirms clean termination for both parities.

#### Approach

This is a signaling / turn-passing pattern built from counting semaphores. Three gates encode "whose turn it is": the marker gate starts with one permit so the marker prints the leading 0, then the marker itself decides which value-thread runs next by opening the odd gate on odd steps and the even gate on even steps. The chosen value-thread prints its number and hands control straight back to the marker. Because a thread only proceeds after acquiring its gate and only ever opens exactly one downstream gate, at most one thread is ever inside the print region, which both serializes emit and forces the precise 0, k, 0, k+1, ... ordering. The key idea is that the marker acts as a scheduler that alternates permits between the odd and even workers.

Complexity

Time: O(n) — the three threads together perform exactly 2n prints and each print is guarded by one constant-time semaphore wait/signal. Space: O(1) — only three fixed semaphores (plus, in the JS model, a bounded waiter queue of size at most one) regardless of n.

Python
PYTHON
from threading import Semaphore
from typing import Callable

class SignalSequencer:
    def __init__(self, n: int):
        self.n = n
        self.gate_marker = Semaphore(1)   # marker goes first
        self.gate_odd = Semaphore(0)
        self.gate_even = Semaphore(0)

    def marker(self, emit: Callable[[int], None]) -> None:
        for i in range(1, self.n + 1):
            self.gate_marker.acquire()
            emit(0)
            if i & 1:
                self.gate_odd.release()
            else:
                self.gate_even.release()

    def odd_value(self, emit: Callable[[int], None]) -> None:
        for i in range(1, self.n + 1, 2):
            self.gate_odd.acquire()
            emit(i)
            self.gate_marker.release()

    def even_value(self, emit: Callable[[int], None]) -> None:
        for i in range(2, self.n + 1, 2):
            self.gate_even.acquire()
            emit(i)
            self.gate_marker.release()
JavaScript
JAVASCRIPT
// Minimal async semaphore: acquire() waits for a permit, release() grants one.
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

class SignalSequencer {

#### cons
tructor(n) {
        this.n = n;
        this.gateMarker = new Sem(1); // marker goes first
        this.gateOdd = new Sem(0);
        this.gateEven = new Sem(0);
    }

    async marker(emit) {
        for (let i = 1; i <= this.n; i++) {
            await this.gateMarker.acquire();
            emit(0);
            if (i & 1) this.gateOdd.release();
            else this.gateEven.release();
        }
    }

    async oddValue(emit) {
        for (let i = 1; i <= this.n; i += 2) {
            await this.gateOdd.acquire();
            emit(i);
            this.gateMarker.release();
        }
    }

    async evenValue(emit) {
        for (let i = 2; i <= this.n; i += 2) {
            await this.gateEven.acquire();
            emit(i);
            this.gateMarker.release();
        }
    }
}
Java
JAVA
import java.util.concurrent.Semaphore;
import java.util.function.IntConsumer;

class SignalSequencer {
    private final int n;
    private final Semaphore gateMarker = new Semaphore(1); // marker goes first
    private final Semaphore gateOdd    = new Semaphore(0);
    private final Semaphore gateEven   = new Semaphore(0);

    public SignalSequencer(int n) { this.n = n; }

    public void marker(IntConsumer emit) throws InterruptedException {
        for (int i = 1; i <= n; i++) {
            gateMarker.acquire();
            emit.accept(0);
            if ((i & 1) == 1) gateOdd.release();
            else              gateEven.release();
        }
    }

    public void oddValue(IntConsumer emit) throws InterruptedException {
        for (int i = 1; i <= n; i += 2) {
            gateOdd.acquire();
            emit.accept(i);
            gateMarker.release();
        }
    }

    public void evenValue(IntConsumer emit) throws InterruptedException {
        for (int i = 2; i <= n; i += 2) {
            gateEven.acquire();
            emit.accept(i);
            gateMarker.release();
        }
    }
}
C++
CPP
#include <semaphore.h>
#include <functional>

class SignalSequencer {
    int n;
    sem_t gateMarker, gateOdd, gateEven;
public:
    SignalSequencer(int n) : n(n) {
        sem_init(&gateMarker, 0, 1);   // marker goes first
        sem_init(&gateOdd, 0, 0);
        sem_init(&gateEven, 0, 0);
    }

    void marker(std::function<void(int)> emit) {
        for (int i = 1; i <= n; ++i) {
            sem_wait(&gateMarker);
            emit(0);
            if (i & 1) sem_post(&gateOdd);
            else       sem_post(&gateEven);
        }
    }

    void oddValue(std::function<void(int)> emit) {
        for (int i = 1; i <= n; i += 2) {
            sem_wait(&gateOdd);
            emit(i);
            sem_post(&gateMarker);
        }
    }

    void evenValue(std::function<void(int)> emit) {
        for (int i = 2; i <= n; i += 2) {
            sem_wait(&gateEven);
            emit(i);
            sem_post(&gateMarker);
        }
    }
};
C
C
#include <semaphore.h>
#include <pthread.h>

/* Shared coordinator. Three gates hand control back and forth:
   only the holder of a gate may print, then it opens the next gate. */
typedef struct {
    int n;
    sem_t gateMarker;   /* starts open: marker prints first */
    sem_t gateOdd;      /* opened by marker before an odd step  */
    sem_t gateEven;     /* opened by marker before an even step */
} SignalSequencer;

void sequencerInit(SignalSequencer* s, int n) {
    s->n = n;
    sem_init(&s->gateMarker, 0, 1);
    sem_init(&s->gateOdd, 0, 0);
    sem_init(&s->gateEven, 0, 0);
}

void marker(SignalSequencer* s, void (*emit)(int)) {
    for (int i = 1; i <= s->n; ++i) {
        sem_wait(&s->gateMarker);
        emit(0);
        if (i & 1) sem_post(&s->gateOdd);
        else       sem_post(&s->gateEven);
    }
}

void oddValue(SignalSequencer* s, void (*emit)(int)) {
    for (int i = 1; i <= s->n; i += 2) {
        sem_wait(&s->gateOdd);
        emit(i);
        sem_post(&s->gateMarker);
    }
}

void evenValue(SignalSequencer* s, void (*emit)(int)) {
    for (int i = 2; i <= s->n; i += 2) {
        sem_wait(&s->gateEven);
        emit(i);
        sem_post(&s->gateMarker);
    }
}

#### Follow-up
questions
Can you produce the same stream using condition variables (a monitor with a shared state + wait/notify) instead of semaphores?
How would you generalize the coordinator to k value-producing threads that each own a residue class modulo k?
If emit were slow or could block, does your design still guarantee no two prints overlap and no deadlock?
How would you prove your solution is free of deadlock and starvation for all valid schedules?

