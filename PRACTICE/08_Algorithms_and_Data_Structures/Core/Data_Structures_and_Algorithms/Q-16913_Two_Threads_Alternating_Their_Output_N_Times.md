---
type: "practice"
difficulty: "Other"
---

# Two Threads Alternating Their Output N Times

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Concurrency](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Concurrency.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given a single shared object that exposes two methods, each of which runs a loop n times:

emitFirst(out) — on each of its n iterations it calls the callback out to append the "first" token to a shared output stream.
emitSecond(out) — on each of its n iterations it calls out to append the "second" token.

The same instance is passed to two independent threads: one thread runs emitFirst and the other runs emitSecond. The two threads run concurrently and may be scheduled by the operating system in any interleaving.

Without synchronization, the two loops could print their tokens in any tangled order. Your task is to add coordination so that the two threads strictly alternate: the combined output must be the pair (first token, then second token) repeated exactly n times, and it must always start with the first token.

Concretely, if emitFirst appends "tick" and emitSecond appends "tock", then for a given n the combined output stream must always be "ticktock" repeated n times — for example, n = 3 must produce "ticktockticktockticktock".

Input / Output

Input: an integer n, the number of times each method loops.
Output: the concatenation of the emitted tokens, which must always equal the first token immediately followed by the second token, that whole pair repeated n times.

#### Cons
traints

1 <= n <= 1000.
Exactly one thread calls emitFirst and exactly one thread calls emitSecond.
The output must begin with the first token and never let one thread run two of its iterations before the other takes its turn.
Use proper blocking synchronization primitives — no CPU-burning spin loops.

#### Example
s & test cases
TEXT
- Input: n = 1 -> Output: "ticktock"
  Why: a single alternation; first token then second token exactly once.
- Input: n = 2 -> Output: "ticktockticktock"
  Why: two full alternations; neither thread may emit twice in a row.
- Input: n = 3 -> Output: "ticktockticktockticktock"
  Why: three alternations regardless of which thread the OS schedules first.
- Input: n = 5 -> Output: "ticktockticktockticktockticktockticktock"
  Why: longer run confirms the turn-passing stays in lockstep and never drifts.
- Input: n = 1000 -> Output: "ticktock" repeated 1000 times
  Why: upper bound; the semaphore/turn mechanism must remain correct and deadlock-free at the maximum size.

#### Approach

This is a classic two-thread alternation / turn-passing problem. Model it as a ping-pong of a single "permission to emit" token between the two threads. Two counting semaphores work cleanly: firstTurn starts with one permit (so the first token is emitted first) and secondTurn starts with zero. On every iteration a thread acquires its own semaphore, emits, then releases the other thread's semaphore, handing over the turn. Because a thread can only proceed when it holds the single available permit, the two loops are forced into strict lockstep and neither can run two iterations back-to-back. An equivalent formulation uses a mutex, a condition variable, and a boolean firstTurn flag: each thread waits on the predicate that it is its turn, emits, flips the flag, and notifies. In single-threaded async JavaScript the same idea is expressed with a small promise-backed async semaphore. The essential pattern: shared mutual-exclusion state plus a strict alternation invariant enforced by blocking, never by spinning.

Complexity

Time: O(n) — each thread performs exactly n iterations, and every iteration does O(1) work (one acquire, one emit, one release), so the total number of coordinated steps is 2n. Space: O(1) — two semaphores (or one mutex + condition variable + a boolean turn flag), independent of n.

Python
PYTHON
import threading

class Alternator:
    def __init__(self, n):
        self.n = n
        self.first_turn = threading.Semaphore(1)   # starts available
        self.second_turn = threading.Semaphore(0)  # starts blocked

    def emit_first(self, out):
        for _ in range(self.n):
            self.first_turn.acquire()
            out()
            self.second_turn.release()             # pass turn to second

    def emit_second(self, out):
        for _ in range(self.n):
            self.second_turn.acquire()
            out()
            self.first_turn.release()              # pass turn back

if __name__ == "__main__":
    a = Alternator(3)
    parts = []
    lock = threading.Lock()

    def mk(tok):
        def _out():
            with lock:
                parts.append(tok)
        return _out

    s = threading.Thread(target=a.emit_second, args=(mk("tock"),))
    f = threading.Thread(target=a.emit_first,  args=(mk("tick"),))
    s.start()
    f.start()
    f.join()
    s.join()
    print("".join(parts))                          # ticktockticktockticktock
JavaScript
JS
// Single-threaded async model: a tiny async semaphore gives the turn-passing.
class Sem {

#### cons
tructor(count) {
    this.count = count;
    this.waiters = [];
  }
  acquire() {
    if (this.count > 0) {
      this.count--;
      return Promise.resolve();
    }
    return new Promise((res) => this.waiters.push(res));
  }
  release() {

#### cons
t next = this.waiters.shift();
    if (next) next();
    else this.count++;
  }
}

class Alternator {

#### cons
tructor(n) {
    this.n = n;
    this.firstTurn = new Sem(1);   // starts available
    this.secondTurn = new Sem(0);  // starts blocked
  }

  async emitFirst(out) {
    for (let i = 0; i < this.n; i++) {
      await this.firstTurn.acquire();
      out();
      this.secondTurn.release();
    }
  }

  async emitSecond(out) {
    for (let i = 0; i < this.n; i++) {
      await this.secondTurn.acquire();
      out();
      this.firstTurn.release();
    }
  }
}

// demo
(async () => {

#### cons
t a = new Alternator(3);
  let s = "";

#### cons
t p2 = a.emitSecond(() => (s += "tock"));

#### cons
t p1 = a.emitFirst(() => (s += "tick"));
  await Promise.all([p1, p2]);

#### cons
ole.log(s); // ticktockticktockticktock
})();
Java
JAVA
import java.util.concurrent.Semaphore;

class Alternator {
    private final int n;
    private final Semaphore firstTurn  = new Semaphore(1); // starts available
    private final Semaphore secondTurn = new Semaphore(0); // starts blocked

    public Alternator(int n) { this.n = n; }

    public void emitFirst(Runnable out) throws InterruptedException {
        for (int i = 0; i < n; i++) {
            firstTurn.acquire();
            out.run();
            secondTurn.release();          // hand the turn to the second thread
        }
    }

    public void emitSecond(Runnable out) throws InterruptedException {
        for (int i = 0; i < n; i++) {
            secondTurn.acquire();
            out.run();
            firstTurn.release();           // hand the turn back
        }
    }

    public static void main(String[] args) throws InterruptedException {
        Alternator a = new Alternator(3);
        StringBuffer sb = new StringBuffer();
        Thread s = new Thread(() -> { try { a.emitSecond(() -> sb.append("tock")); } catch (InterruptedException ignored) {} });
        Thread f = new Thread(() -> { try { a.emitFirst(()  -> sb.append("tick")); } catch (InterruptedException ignored) {} });
        s.start(); f.start();
        f.join(); s.join();
        System.out.println(sb);            // ticktockticktockticktock
    }
}
C++
CPP
#include <mutex>
#include <condition_variable>
#include <functional>
#include <thread>
#include <iostream>

class Alternator {
    int n;
    std::mutex m;
    std::condition_variable cv;
    bool firstTurn = true;               // output starts with the first token
public:
    explicit Alternator(int n) : n(n) {}

    void emitFirst(const std::function<void()>& out) {
        for (int i = 0; i < n; i++) {
            std::unique_lock<std::mutex> lk(m);
            cv.wait(lk, [this]{ return firstTurn; });
            out();
            firstTurn = false;
            cv.notify_all();
        }
    }

    void emitSecond(const std::function<void()>& out) {
        for (int i = 0; i < n; i++) {
            std::unique_lock<std::mutex> lk(m);
            cv.wait(lk, [this]{ return !firstTurn; });
            out();
            firstTurn = true;
            cv.notify_all();
        }
    }
};

int main() {
    Alternator a(3);
    auto o1 = []{ std::cout << "tick"; };
    auto o2 = []{ std::cout << "tock"; };
    std::thread s([&]{ a.emitSecond(o2); });
    std::thread f([&]{ a.emitFirst(o1);  });
    f.join(); s.join();
    std::cout << "\n";
}
C
C
#include <pthread.h>
#include <stdio.h>

typedef struct {
    int n;
    pthread_mutex_t lock;
    pthread_cond_t  cv;
    int first_turn;              /* 1 => first may emit, 0 => second may emit */
} Alternator;

void alt_init(Alternator *a, int n) {
    a->n = n;
    pthread_mutex_init(&a->lock, NULL);
    pthread_cond_init(&a->cv, NULL);
    a->first_turn = 1;           /* output must start with the first token */
}

void emit_first(Alternator *a, void (*out)(void)) {
    for (int i = 0; i < a->n; i++) {
        pthread_mutex_lock(&a->lock);
        while (!a->first_turn) pthread_cond_wait(&a->cv, &a->lock);
        out();
        a->first_turn = 0;
        pthread_cond_broadcast(&a->cv);
        pthread_mutex_unlock(&a->lock);
    }
}

void emit_second(Alternator *a, void (*out)(void)) {
    for (int i = 0; i < a->n; i++) {
        pthread_mutex_lock(&a->lock);
        while (a->first_turn) pthread_cond_wait(&a->cv, &a->lock);
        out();
        a->first_turn = 1;
        pthread_cond_broadcast(&a->cv);
        pthread_mutex_unlock(&a->lock);
    }
}

/* --- demo --- */
static void o1(void){ printf("tick"); }
static void o2(void){ printf("tock"); }
static Alternator A;
static void *tf(void *x){ (void)x; emit_first(&A, o1);  return NULL; }
static void *ts(void *x){ (void)x; emit_second(&A, o2); return NULL; }

int main(void) {
    alt_init(&A, 3);
    pthread_t b, a;
    pthread_create(&b, NULL, ts, NULL);   /* second launched first on purpose */
    pthread_create(&a, NULL, tf, NULL);
    pthread_join(a, NULL); pthread_join(b, NULL);
    printf("\n");
    return 0;
}

#### Follow-up
questions
Extend to k threads that must cycle in a fixed round-robin order emitting their tokens n times each.
Guarantee fairness/ordering when many threads contend — does your semaphore solution ever starve one side?
Reimplement using only condition variables (no semaphores), and explain how you avoid lost wakeups.
What happens if n differs between the two methods, or one thread throws mid-loop — how do you avoid deadlocking the peer?

