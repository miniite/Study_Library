---
type: "practice"
difficulty: "Other"
---

# Enforce Sequential Execution of Three Concurrent Tasks

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Concurrency](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Concurrency.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given a single shared coordinator object whose interface exposes three methods:

taskOne(emit)
taskTwo(emit)
taskThree(emit)

Each method receives a callback emit (a zero-argument function). When a method calls emit, it appends its own token to a shared output stream. taskOne emits the token for stage 1, taskTwo for stage 2, and taskThree for stage 3.

The very same instance of the coordinator is handed to three independent threads:

one thread calls taskOne,
another thread calls taskTwo,
another thread calls taskThree.

The operating system may schedule these three threads in any order, and you have no control over which thread starts first or how they are interleaved. Your job is to add synchronization so that, no matter how the threads are scheduled, the callbacks fire in a strict order: the stage-1 token is emitted first, then the stage-2 token, then the stage-3 token.

Concretely, if stage 1 emits "one", stage 2 emits "two", and stage 3 emits "three", the concatenated output stream must always be exactly "onetwothree".

The problem provides, as a test aid, a permutation of [1, 2, 3] describing which of the three tasks each spawned thread happens to invoke. This permutation only exists to let a checker fire the three threads in different launch orders; it does NOT tell you the order in which the callbacks are allowed to run. Your synchronization must produce the same correct ordered output for every permutation.

Design the mechanism and complete the three methods.

Input / Output

Input: a permutation of [1, 2, 3] that determines which method each of the three threads calls (used only to launch the threads in varying orders).
Output: the concatenation of the emitted tokens, which must always equal the stage-ordered sequence (e.g. "onetwothree").

#### Cons
traints

The launch order is a permutation of [1, 2, 3].
Exactly one thread calls each of the three methods; every method is invoked exactly once.
No busy-wait / spin loop that burns CPU is expected; use proper blocking synchronization primitives.
Each method's emit callback must be invoked exactly once.

#### Example
s & test cases
TEXT
- Input: launch order = [1, 2, 3] -> Output: "onetwothree"
  Why: threads launched in natural order; the correct output is trivially the stage order.
- Input: launch order = [1, 3, 2] -> Output: "onetwothree"
  Why: the thread calling taskThree starts before the one calling taskTwo, but taskThree must still block until stage 2 has emitted.
- Input: launch order = [2, 1, 3] -> Output: "onetwothree"
  Why: taskTwo's thread runs first and must block until taskOne has emitted.
- Input: launch order = [3, 2, 1] -> Output: "onetwothree"
  Why: fully reversed launch order; taskThree waits on taskTwo, which waits on taskOne, so output is still ordered.
- Input: launch order = [2, 3, 1] -> Output: "onetwothree"
  Why: both later stages are launched before stage 1, yet each blocks on its predecessor gate until stage 1 releases the chain.

#### Approach

This is a thread-ordering (happens-before) problem, not an algorithmic one. The pattern is "signal-and-wait gates": maintain synchronization objects that record when each earlier stage has finished. Stage 1 runs unconditionally and then signals gate 2. Stage 2 blocks on gate 2, emits, then signals gate 3. Stage 3 blocks on gate 3 and emits. Because every later stage waits on a gate that only its predecessor can open, the emit callbacks are forced into the order 1 -> 2 -> 3 regardless of OS thread scheduling. Any equivalent primitive works: an integer stage counter guarded by a mutex + condition variable (C/C++), one-shot latches or events (Java CountDownLatch, Python Event), binary semaphores, or resolvable promises in single-threaded async JavaScript. The key ideas are: (1) block rather than busy-wait, and (2) use a monotone predicate (stage >= k) in the wait condition so spurious wakeups are handled correctly.

Complexity

Time: O(1) of real work per method — each thread does a constant amount of work plus one blocking wait, so the whole coordination is O(1). Space: O(1) — a fixed number of gates/condition variables and one small integer of shared state, independent of any input size.

Python
PYTHON
import threading

class OrderedRunner:
    def __init__(self):
        # gates start "closed"; each stage opens the next
        self.gate2 = threading.Event()
        self.gate3 = threading.Event()

    def task_one(self, emit):
        emit()                # stage 1 has no predecessor
        self.gate2.set()      # release stage 2

    def task_two(self, emit):
        self.gate2.wait()     # block until stage 1 is done
        emit()
        self.gate3.set()      # release stage 3

    def task_three(self, emit):
        self.gate3.wait()     # block until stage 2 is done
        emit()

if __name__ == "__main__":
    r = OrderedRunner()
    parts = []
    lock = threading.Lock()

    def mk(tok):
        def _emit():
            with lock:
                parts.append(tok)
        return _emit

    # launch reversed to prove ordering is enforced, not incidental
    threads = [
        threading.Thread(target=r.task_three, args=(mk("three"),)),
        threading.Thread(target=r.task_two,   args=(mk("two"),)),
        threading.Thread(target=r.task_one,   args=(mk("one"),)),
    ]
    for t in threads:
        t.start()
    for t in threads:
        t.join()
    print("".join(parts))     # -> onetwothree
JavaScript
JS
// JavaScript runs on a single thread; the callers await these async methods.
// Two resolvable promises act as the ordering gates.
class OrderedRunner {

#### cons
tructor() {
    this.gate2 = new Promise((res) => (this._open2 = res));
    this.gate3 = new Promise((res) => (this._open3 = res));
  }

  async taskOne(emit) {
    emit();          // stage 1 first
    this._open2();   // release stage 2
  }

  async taskTwo(emit) {
    await this.gate2; // wait for stage 1
    emit();
    this._open3();    // release stage 3
  }

  async taskThree(emit) {
    await this.gate3; // wait for stage 2
    emit();
  }
}

// demo
(async () => {

#### cons
t r = new OrderedRunner();
  let out = "";

#### cons
t t3 = r.taskThree(() => (out += "three"));

#### cons
t t2 = r.taskTwo(() => (out += "two"));

#### cons
t t1 = r.taskOne(() => (out += "one"));
  await Promise.all([t1, t2, t3]);

#### cons
ole.log(out); // onetwothree
})();
Java
JAVA
import java.util.concurrent.CountDownLatch;

class OrderedRunner {
    private final CountDownLatch gate2 = new CountDownLatch(1);
    private final CountDownLatch gate3 = new CountDownLatch(1);

    public void taskOne(Runnable emit) {
        emit.run();
        gate2.countDown();            // stage 1 done -> unblock stage 2
    }

    public void taskTwo(Runnable emit) throws InterruptedException {
        gate2.await();                // wait until stage 1 finished
        emit.run();
        gate3.countDown();            // stage 2 done -> unblock stage 3
    }

    public void taskThree(Runnable emit) throws InterruptedException {
        gate3.await();                // wait until stage 2 finished
        emit.run();
    }

    public static void main(String[] args) throws InterruptedException {
        OrderedRunner r = new OrderedRunner();
        StringBuffer out = new StringBuffer();
        Thread c = new Thread(() -> { try { r.taskThree(() -> out.append("three")); } catch (InterruptedException ignored) {} });
        Thread b = new Thread(() -> { try { r.taskTwo(()   -> out.append("two"));   } catch (InterruptedException ignored) {} });
        Thread a = new Thread(() -> { r.taskOne(() -> out.append("one")); });
        c.start(); b.start(); a.start();
        a.join(); b.join(); c.join();
        System.out.println(out);
    }
}
C++
CPP
#include <mutex>
#include <condition_variable>
#include <functional>
#include <thread>
#include <iostream>

class OrderedRunner {
    std::mutex m;
    std::condition_variable cv;
    int stage = 0;                       // highest completed stage
public:
    void taskOne(const std::function<void()>& emit) {
        std::unique_lock<std::mutex> lk(m);
        emit();
        stage = 1;
        cv.notify_all();
    }
    void taskTwo(const std::function<void()>& emit) {
        std::unique_lock<std::mutex> lk(m);
        cv.wait(lk, [this]{ return stage >= 1; });
        emit();
        stage = 2;
        cv.notify_all();
    }
    void taskThree(const std::function<void()>& emit) {
        std::unique_lock<std::mutex> lk(m);
        cv.wait(lk, [this]{ return stage >= 2; });
        emit();
        stage = 3;
    }
};

int main() {
    OrderedRunner r;
    auto e1 = []{ std::cout << "one";   };
    auto e2 = []{ std::cout << "two";   };
    auto e3 = []{ std::cout << "three"; };
    std::thread c([&]{ r.taskThree(e3); });
    std::thread b([&]{ r.taskTwo(e2);   });
    std::thread a([&]{ r.taskOne(e1);   });
    a.join(); b.join(); c.join();
    std::cout << "\n";
}
C
C
#include <pthread.h>
#include <stdio.h>

typedef struct {
    pthread_mutex_t lock;
    pthread_cond_t  cv;
    int stage;                 /* highest completed stage: 0,1,2,3 */
} OrderedRunner;

void runner_init(OrderedRunner *r) {
    pthread_mutex_init(&r->lock, NULL);
    pthread_cond_init(&r->cv, NULL);
    r->stage = 0;
}

void task_one(OrderedRunner *r, void (*emit)(void)) {
    pthread_mutex_lock(&r->lock);
    emit();                    /* stage 1 has no predecessor */
    r->stage = 1;
    pthread_cond_broadcast(&r->cv);
    pthread_mutex_unlock(&r->lock);
}

void task_two(OrderedRunner *r, void (*emit)(void)) {
    pthread_mutex_lock(&r->lock);
    while (r->stage < 1) pthread_cond_wait(&r->cv, &r->lock);
    emit();
    r->stage = 2;
    pthread_cond_broadcast(&r->cv);
    pthread_mutex_unlock(&r->lock);
}

void task_three(OrderedRunner *r, void (*emit)(void)) {
    pthread_mutex_lock(&r->lock);
    while (r->stage < 2) pthread_cond_wait(&r->cv, &r->lock);
    emit();
    r->stage = 3;
    pthread_mutex_unlock(&r->lock);
}

/* --- demo --- */
static void e1(void){ printf("one");   }
static void e2(void){ printf("two");   }
static void e3(void){ printf("three"); }

static OrderedRunner R;
static void *t1(void *a){ (void)a; task_one(&R, e1);   return NULL; }
static void *t2(void *a){ (void)a; task_two(&R, e2);   return NULL; }
static void *t3(void *a){ (void)a; task_three(&R, e3); return NULL; }

int main(void) {
    runner_init(&R);
    pthread_t a, b, c;
    /* launch in reversed order to prove ordering holds */
    pthread_create(&c, NULL, t3, NULL);
    pthread_create(&b, NULL, t2, NULL);
    pthread_create(&a, NULL, t1, NULL);
    pthread_join(a, NULL); pthread_join(b, NULL); pthread_join(c, NULL);
    printf("\n");
    return 0;
}

#### Follow-up
questions
Generalize to N stages that must run in order, where N is only known at runtime — how does your gate structure scale?
What changes if each method may be called multiple times or by a pool of threads instead of exactly once?
How would you avoid a lost-wakeup or deadlock if signaling happened before the waiter reached its wait call?
Reimplement using only counting semaphores (no condition variables or latches) and argue correctness.

