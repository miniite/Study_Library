---
type: "practice"
difficulty: "Other"
---

# Coordinated Sequence Printing with Four Workers

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Concurrency](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Concurrency.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You must generate the number sequence 1, 2, 3, ..., n in order, but the printing of each value is split across four independent workers (threads) that all share a single controller object. Each value in the sequence is replaced according to these rules, based on its position i (1-indexed):

print the word "lohi" if i is divisible by both 3 and 5,
print the word "lo" if i is divisible by 3 but not by 5,
print the word "hi" if i is divisible by 5 but not by 3,
print the integer i itself if i is divisible by neither 3 nor 5.

So the intended output for the first fifteen positions is: 1, 2, "lo", 4, "hi", "lo", 7, 8, "lo", "hi", 11, "lo", 13, 14, "lohi".

The controller is constructed with a single integer n (the length of the sequence). The same controller instance is handed to four different threads, each of which repeatedly calls one of the controller's four methods:

Worker A calls lo(printLo) and must, whenever the current position qualifies for "lo", invoke the supplied printLo() callback (which prints "lo").
Worker B calls hi(printHi) and must invoke printHi() on every "hi" position.
Worker C calls lohi(printLoHi) and must invoke printLoHi() on every "lohi" position.
Worker D calls number(printNumber) and must invoke printNumber(i) on every position that is a plain integer.

The four threads run concurrently. Your job is to add synchronization inside the controller so that, no matter how the operating system interleaves the four threads, the tokens are emitted in the exact order of positions 1 through n, each token is emitted exactly once, and every method returns after the whole sequence has been produced (no thread deadlocks or hangs).

You are implementing the controller class/struct: its constructor and the four methods. The printLo, printHi, printLoHi, and printNumber callbacks are provided to you and each performs the actual output; you only decide when it is safe to call them.

#### Cons
traints:

1 <= n <= 50
Exactly four threads run, one per method; each method may be entered by its thread only once and loops internally until the sequence is finished.
The relative order of emitted tokens must be strictly increasing by position; no token may be skipped, duplicated, or emitted out of order.
All four methods must eventually return (the program must terminate).

#### Example
s & test cases
TEXT
Input: n = 15 -> Output: 1 2 lo 4 hi lo 7 8 lo hi 11 lo 13 14 lohi
Why: normal case exercising all four token kinds; position 15 is divisible by both 3 and 5 so it is "lohi".

Input: n = 5 -> Output: 1 2 lo 4 hi
Why: shortest run that produces a "lo" (pos 3) and a "hi" (pos 5) but no "lohi".

Input: n = 1 -> Output: 1
Why: minimum size; only Worker D ever prints, the other three threads must still start and return without hanging.

Input: n = 3 -> Output: 1 2 lo
Why: ends exactly on a "lo" position; verifies the loop stops at n without a fourth number.

Input: n = 30 -> Output: 1 2 lo 4 hi lo 7 8 lo hi 11 lo 13 14 lohi 16 17 lo 19 hi lo 22 23 lo hi 26 lo 28 29 lohi
Why: two full 15-cycles; both position 15 and position 30 are "lohi", confirming the divisible-by-15 branch fires more than once.

#### Approach

This is a thread-coordination (producer-ordering) problem, not an algorithmic one: correctness hinges on a single shared, mutex-protected counter current that names the next position to emit. Every worker loops on a condition variable, sleeping until either the sequence is finished (current > n) or current matches the predicate that worker owns (%3 only, %5 only, %15, or neither). When woken it re-checks the predicate under the lock (guarding against spurious wake-ups), performs its single emission, increments current, and broadcasts to wake whichever worker owns the new position. Because the counter is advanced under mutual exclusion and every worker exits its wait loop once current > n, the tokens come out in strict positional order with no duplication, and all four threads terminate. The reusable pattern is: shared monotonic state + condition variable with a per-worker predicate + broadcast-on-change.

Complexity

Time: O(n) total useful emissions across all threads; each of the n positions is emitted exactly once under the shared lock. With condition variables the eligible thread is woken directly, so wake-ups are O(n) overall (a busy-wait/await null variant does O(n) extra polling passes but still linear work). Space: O(1) — one shared counter, one mutex, and one condition variable regardless of n.

Python
PYTHON
import threading

class SequencePrinter:
    def __init__(self, n: int):
        self.n = n
        self.current = 1          # next position to emit, 1-indexed
        self.cond = threading.Condition()

    def _step(self, mine, emit):
        while True:
            with self.cond:
                self.cond.wait_for(
                    lambda: self.current > self.n or mine(self.current)
                )
                if self.current > self.n:
                    self.cond.notify_all()
                    return
                emit(self.current)
                self.current += 1
                self.cond.notify_all()

    def lo(self, printLo):
        self._step(lambda i: i % 3 == 0 and i % 5 != 0, lambda i: printLo())

    def hi(self, printHi):
        self._step(lambda i: i % 5 == 0 and i % 3 != 0, lambda i: printHi())

    def lohi(self, printLoHi):
        self._step(lambda i: i % 15 == 0, lambda i: printLoHi())

    def number(self, printNumber):
        self._step(lambda i: i % 3 != 0 and i % 5 != 0,
                   lambda i: printNumber(i))
JavaScript
JAVASCRIPT
// JavaScript has a single-threaded event loop, so the four "workers" are
// modeled as concurrently-running async functions. Each yields control with
// `await null` (a microtask hop) until the shared counter reaches a position
// it is responsible for, keeping the emitted order strictly increasing.
class SequencePrinter {

#### cons
tructor(n) {
        this.n = n;
        this.current = 1; // next position to emit, 1-indexed
    }

    async _step(mine, emit) {
        while (true) {
            while (this.current <= this.n && !mine(this.current)) {
                await null; // yield so the other async workers can advance it
            }
            if (this.current > this.n) return;
            emit(this.current);
            this.current++;
        }
    }

    lo(printLo) {
        return this._step(i => i % 3 === 0 && i % 5 !== 0, () => printLo());
    }
    hi(printHi) {
        return this._step(i => i % 5 === 0 && i % 3 !== 0, () => printHi());
    }
    lohi(printLoHi) {
        return this._step(i => i % 15 === 0, () => printLoHi());
    }
    number(printNumber) {
        return this._step(i => i % 3 !== 0 && i % 5 !== 0, i => printNumber(i));
    }
}
Java
JAVA
import java.util.function.IntConsumer;
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

class SequencePrinter {
    private final int n;
    private int current = 1;              // next position to emit, 1-indexed
    private final Lock lock = new ReentrantLock();
    private final Condition cond = lock.newCondition();

    public SequencePrinter(int n) { this.n = n; }

    private interface Predicate { boolean test(int i); }

    private void step(Predicate mine, Runnable emit) throws InterruptedException {
        while (true) {
            lock.lock();
            try {
                while (current <= n && !mine.test(current)) cond.await();
                if (current > n) { cond.signalAll(); return; }
                emit.run();
                current++;
                cond.signalAll();
            } finally {
                lock.unlock();
            }
        }
    }

    public void lo(Runnable printLo) throws InterruptedException {
        step(i -> i % 3 == 0 && i % 5 != 0, printLo);
    }
    public void hi(Runnable printHi) throws InterruptedException {
        step(i -> i % 5 == 0 && i % 3 != 0, printHi);
    }
    public void lohi(Runnable printLoHi) throws InterruptedException {
        step(i -> i % 15 == 0, printLoHi);
    }
    public void number(IntConsumer printNumber) throws InterruptedException {
        step(i -> i % 3 != 0 && i % 5 != 0, () -> printNumber.accept(current));
    }
}
C++
CPP
#include <mutex>
#include <condition_variable>
#include <functional>

class SequencePrinter {
    int n;
    int current = 1;                 // next position to emit, 1-indexed
    std::mutex m;
    std::condition_variable cv;

    // Wait until it is this token's turn, emit it, advance, and wake others.
    void step(const std::function<bool(int)>& mine,

#### cons
t std::function<void()>& emit) {
        for (;;) {
            std::unique_lock<std::mutex> lk(m);
            cv.wait(lk, [&] { return current > n || mine(current); });
            if (current > n) { cv.notify_all(); return; }
            emit();
            ++current;
            cv.notify_all();
        }
    }

public:
    explicit SequencePrinter(int n) : n(n) {}

    void lo(std::function<void()> printLo) {
        step([] (int i){ return i % 3 == 0 && i % 5 != 0; },
             [&]{ printLo(); });
    }
    void hi(std::function<void()> printHi) {
        step([] (int i){ return i % 5 == 0 && i % 3 != 0; },
             [&]{ printHi(); });
    }
    void lohi(std::function<void()> printLoHi) {
        step([] (int i){ return i % 15 == 0; },
             [&]{ printLoHi(); });
    }
    void number(std::function<void(int)> printNumber) {
        step([] (int i){ return i % 3 != 0 && i % 5 != 0; },
             [&]{ printNumber(current); });
    }
};
C
C
#include <pthread.h>
#include <stdio.h>

typedef struct {
    int n;
    int current;                /* next position to emit, 1-indexed */
    pthread_mutex_t lock;
    pthread_cond_t  cond;
} SequencePrinter;

void printerInit(SequencePrinter *p, int n) {
    p->n = n;
    p->current = 1;
    pthread_mutex_init(&p->lock, NULL);
    pthread_cond_init(&p->cond, NULL);
}

/* Worker A: positions divisible by 3 but not 5 -> "lo" */
void runLo(SequencePrinter *p, void (*printLo)(void)) {
    for (;;) {
        pthread_mutex_lock(&p->lock);
        while (p->current <= p->n &&
               !(p->current % 3 == 0 && p->current % 5 != 0))
            pthread_cond_wait(&p->cond, &p->lock);
        if (p->current > p->n) {
            pthread_cond_broadcast(&p->cond);
            pthread_mutex_unlock(&p->lock);
            return;
        }
        printLo();
        p->current++;
        pthread_cond_broadcast(&p->cond);
        pthread_mutex_unlock(&p->lock);
    }
}

/* Worker B: positions divisible by 5 but not 3 -> "hi" */
void runHi(SequencePrinter *p, void (*printHi)(void)) {
    for (;;) {
        pthread_mutex_lock(&p->lock);
        while (p->current <= p->n &&
               !(p->current % 5 == 0 && p->current % 3 != 0))
            pthread_cond_wait(&p->cond, &p->lock);
        if (p->current > p->n) {
            pthread_cond_broadcast(&p->cond);
            pthread_mutex_unlock(&p->lock);
            return;
        }
        printHi();
        p->current++;
        pthread_cond_broadcast(&p->cond);
        pthread_mutex_unlock(&p->lock);
    }
}

/* Worker C: positions divisible by 15 -> "lohi" */
void runLoHi(SequencePrinter *p, void (*printLoHi)(void)) {
    for (;;) {
        pthread_mutex_lock(&p->lock);
        while (p->current <= p->n && p->current % 15 != 0)
            pthread_cond_wait(&p->cond, &p->lock);
        if (p->current > p->n) {
            pthread_cond_broadcast(&p->cond);
            pthread_mutex_unlock(&p->lock);
            return;
        }
        printLoHi();
        p->current++;
        pthread_cond_broadcast(&p->cond);
        pthread_mutex_unlock(&p->lock);
    }
}

/* Worker D: positions divisible by neither 3 nor 5 -> the number */
void runNumber(SequencePrinter *p, void (*printNumber)(int)) {
    for (;;) {
        pthread_mutex_lock(&p->lock);
        while (p->current <= p->n &&
               (p->current % 3 == 0 || p->current % 5 == 0))
            pthread_cond_wait(&p->cond, &p->lock);
        if (p->current > p->n) {
            pthread_cond_broadcast(&p->cond);
            pthread_mutex_unlock(&p->lock);
            return;
        }
        printNumber(p->current);
        p->current++;
        pthread_cond_broadcast(&p->cond);
        pthread_mutex_unlock(&p->lock);
    }
}

#### Follow-up
questions
Generalize to k independent worker categories driven by arbitrary divisibility rules — how do you keep the coordinator O(1) in state?
Replace the condition variable with semaphores (one per token kind): how do you signal exactly the right semaphore as the counter advances?
What changes if a worker may crash mid-sequence — how do you guarantee the remaining threads still terminate without deadlock?
Show that your solution is free of spurious-wakeup bugs and cannot emit a token twice under any interleaving.

