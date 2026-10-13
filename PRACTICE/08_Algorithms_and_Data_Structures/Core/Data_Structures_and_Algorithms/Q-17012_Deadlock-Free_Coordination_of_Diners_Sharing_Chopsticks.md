---
type: "practice"
difficulty: "Other"
---

# Deadlock-Free Coordination of Diners Sharing Chopsticks

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Concurrency](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Concurrency.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Five diners are seated around a circular table. Between every pair of adjacent diners lies a single chopstick, so there are exactly five chopsticks in total. The diners are numbered 0 through 4 in clockwise order, and the chopsticks are numbered 0 through 4 as well, with chopstick d lying immediately to the left of diner d.

#### Cons
equently, diner d shares exactly two chopsticks with their neighbours:

their left chopstick has index d, and
their right chopstick has index (d + 1) mod 5.

Each diner endlessly alternates between two activities: thinking and eating. A diner may begin eating only after they are simultaneously holding both the chopstick on their left and the chopstick on their right. A chopstick can be held by at most one diner at any instant; while a diner holds a chopstick, an adjacent diner who also needs that same chopstick must wait until it is put back down. When a diner finishes eating, they must immediately release both chopsticks so their neighbours can use them. A diner is regarded as thinking during any period in which they are not actively requesting to eat.

Five concurrent threads — one per diner — share a single instance of your class. Whenever a diner becomes hungry, the runtime invokes your method on that diner's thread. The method may be called for the same diner many times, and may even be called again before a previous call for that diner has returned. Implement:

CODE
wantsToEat(diner, pickLeft, pickRight, eat, putLeft, putRight)

where:

diner is the id (0..4) of the hungry diner making this request;
pickLeft and pickRight are callbacks that record picking up this diner's left and right chopstick, respectively;
eat is a callback that records the diner eating — call it only while holding both chopsticks;
putLeft and putRight are callbacks that record putting the left and right chopstick back down.

Your implementation must guarantee all of the following, for every possible interleaving of the five threads:

Mutual exclusion on chopsticks — no chopstick is ever held by two diners at the same time.
Eat only with both — a diner never invokes eat unless it currently holds both of its chopsticks.
No deadlock — the system can never reach a state in which every diner is blocked forever waiting on a chopstick; every hungry diner eventually eats.

The naive discipline in which every diner grabs its left chopstick first and then its right chopstick can deadlock: if all five diners pick up their left chopstick at once, each is stuck waiting on a right chopstick that a neighbour is holding — a circular wait. You must break this cycle.

#### Cons
traints:

There are exactly 5 diners, with ids 0..4 arranged clockwise.
For diner d: left chopstick index = d, right chopstick index = (d + 1) mod 5.
wantsToEat may be invoked concurrently for all diners and repeatedly; assume between 1 and 60 calls per diner.
Any correct concurrent discipline is accepted; it must satisfy the three guarantees above under arbitrary scheduling.

#### Example
s & test cases
TEXT
Because this is a concurrency problem, the "tests" are invariants that must hold under every thread interleaving rather than fixed input→output pairs.

- Input: all 5 diners call `wantsToEat` simultaneously -> Output: no deadlock; every call eventually returns and every diner eats. Why: the classic all-grab-left deadlock must be prevented.
- Input: diner `0` and diner `2` request at the same time -> Output: both may eat concurrently (they share no chopstick: diner 0 uses forks {0,1}, diner 2 uses forks {2,3}). Why: non-adjacent diners must not block each other.
- Input: diner `0` and diner `1` request at the same time -> Output: they eat one after another, never both during an overlapping window. Why: they share chopstick `1`, so mutual exclusion must serialize them on it.
- Input: any single diner called 60 times back-to-back -> Output: 60 successful eats, each holding both of its chopsticks. Why: repeated/re-entrant requests for the same diner must still be safe.
- Input: diner `4` (whose forks are 4 and 0, wrapping around) requests together with diner `0` -> Output: no deadlock even at the wrap-around seam. Why: the last diner is where a fixed left-then-right ordering would close the cycle; ordering by chopstick index must handle it.

#### Approach

Model each chopstick as a mutual-exclusion lock (mutex) and require a diner to hold both of its adjacent chopstick locks before calling eat, then release them afterward. Deadlock in the naive "left-then-right for everyone" scheme arises purely from a circular wait: diner d waits on diner d+1's chopstick around the ring. The fix is resource ordering — assign every chopstick a global index (its number 0..4) and require every diner to always lock the lower-indexed of its two chopsticks first and the higher-indexed second. With a total order on lock acquisition no cycle of waits can form, so the system is provably deadlock-free. The last diner (forks 4 and 0) naturally grabs fork 0 first under this rule, which is exactly the asymmetry that breaks the ring. (C++'s std::lock and equivalent "lock multiple mutexes at once" primitives achieve the same guarantee internally.) An alternative correct discipline is a counting semaphore that admits at most 4 diners to the table simultaneously, which likewise guarantees at least one diner can always obtain both chopsticks.

Complexity

Time: O(1) work per wantsToEat call beyond the (necessary) blocking wait for chopsticks — a fixed number of lock/unlock and callback operations. Space: O(1) extra, a fixed array of 5 chopstick locks shared by all threads. The scheme is deadlock-free because imposing a single global order on lock acquisition (always take the lower-indexed chopstick first) eliminates the circular-wait condition, one of the four necessary conditions for deadlock.

Python
PYTHON
import threading

class DiningTable:
    def __init__(self):
        self._chopstick = [threading.Lock() for _ in range(5)]

    def wantsToEat(self, diner, pickLeft, pickRight, eat, putLeft, putRight):
        left, right = diner, (diner + 1) % 5
        lo, hi = min(left, right), max(left, right)

        # Acquire the lower-indexed chopstick first for every diner.
        # A total order on lock acquisition makes a circular wait impossible,
        # so the table can never deadlock.
        with self._chopstick[lo], self._chopstick[hi]:
            pickLeft()
            pickRight()
            eat()
            putLeft()
            putRight()
JavaScript
JAVASCRIPT
// A minimal FIFO async mutex: acquire() resolves to an unlock function
// once it is the caller's turn to hold the lock.
class Mutex {

#### cons
tructor() {
        this._queue = Promise.resolve();
    }
    acquire() {
        let unlock;

#### cons
t willUnlock = new Promise(res => { unlock = res; });

#### cons
t acquired = this._queue.then(() => unlock);
        this._queue = this._queue.then(() => willUnlock);
        return acquired;
    }
}

class DiningTable {

#### cons
tructor() {
        this.chopstick = Array.from({ length: 5 }, () => new Mutex());
    }

    /**
     * @param {number} diner
     * @param {function} pickLeft  @param {function} pickRight
     * @param {function} eat
     * @param {function} putLeft   @param {function} putRight
     */
    async wantsToEat(diner, pickLeft, pickRight, eat, putLeft, putRight) {

#### cons
t left = diner, right = (diner + 1) % 5;

#### cons
t lo = Math.min(left, right), hi = Math.max(left, right);

        // Acquire the lower-indexed chopstick first (global order -> no deadlock).

#### cons
t unlockLo = await this.chopstick[lo].acquire();

#### cons
t unlockHi = await this.chopstick[hi].acquire();

        await pickLeft();
        await pickRight();
        await eat();
        await putLeft();
        await putRight();

        unlockHi();
        unlockLo();
    }
}
Java
JAVA
import java.util.concurrent.locks.ReentrantLock;

class DiningTable {
    private final ReentrantLock[] chopstick = new ReentrantLock[5];

    public DiningTable() {
        for (int i = 0; i < 5; i++) chopstick[i] = new ReentrantLock();
    }

    public void wantsToEat(int diner,
                           Runnable pickLeft, Runnable pickRight, Runnable eat,
                           Runnable putLeft, Runnable putRight) throws InterruptedException {
        int left = diner, right = (diner + 1) % 5;
        int lo = Math.min(left, right), hi = Math.max(left, right);

        // Always take the lower-indexed chopstick first -> no circular wait -> no deadlock.
        chopstick[lo].lock();
        chopstick[hi].lock();
        try {
            pickLeft.run();
            pickRight.run();
            eat.run();
            putLeft.run();
            putRight.run();
        } finally {
            chopstick[hi].unlock();
            chopstick[lo].unlock();
        }
    }
}
C++
CPP
#include <mutex>
#include <functional>

class DiningTable {
    std::mutex chopstick[5];
public:
    void wantsToEat(int diner,
                    std::function<void()> pickLeft,
                    std::function<void()> pickRight,
                    std::function<void()> eat,
                    std::function<void()> putLeft,
                    std::function<void()> putRight) {
        int left = diner, right = (diner + 1) % 5;

        // std::lock acquires both mutexes atomically with an internal
        // deadlock-avoidance ordering, so no circular wait is possible.
        std::lock(chopstick[left], chopstick[right]);
        std::lock_guard<std::mutex> gl(chopstick[left], std::adopt_lock);
        std::lock_guard<std::mutex> gr(chopstick[right], std::adopt_lock);

        pickLeft();
        pickRight();
        eat();
        putLeft();
        putRight();
        // gl / gr release both chopsticks on scope exit.
    }
};
C
C
#include <pthread.h>

typedef void (*ForkOp)(void);

typedef struct {
    pthread_mutex_t chopstick[5];
} DiningTable;

/* Call once before starting the diner threads. */
void diningTableInit(DiningTable* t) {
    for (int i = 0; i < 5; i++)
        pthread_mutex_init(&t->chopstick[i], NULL);
}

void wantsToEat(DiningTable* t, int diner,
                ForkOp pickLeft, ForkOp pickRight, ForkOp eat,
                ForkOp putLeft, ForkOp putRight) {
    int left  = diner;
    int right = (diner + 1) % 5;
    int lo = left < right ? left : right;   /* lower-indexed chopstick */
    int hi = left < right ? right : left;   /* higher-indexed chopstick */

    /* Global acquisition order (lower index first) breaks any circular wait. */
    pthread_mutex_lock(&t->chopstick[lo]);
    pthread_mutex_lock(&t->chopstick[hi]);

    pickLeft();
    pickRight();
    eat();
    putLeft();
    putRight();

    pthread_mutex_unlock(&t->chopstick[hi]);
    pthread_mutex_unlock(&t->chopstick[lo]);
}

#### Follow-up
questions
Prove your discipline is not just deadlock-free but also starvation-free — can a particular diner be perpetually skipped while its neighbours keep eating? If so, how would you add fairness (e.g., fair locks or ticketing)?
Re-solve it using a single counting semaphore that lets at most 4 diners sit down at once, and argue why that alone prevents deadlock.
Generalize to k diners and k chopsticks around the ring — does the lower-index-first ordering argument still hold, and what changes?
Suppose one buggy diner ignores the ordering rule and grabs right-then-left. How would you detect the resulting deadlock at runtime (e.g., wait-for graph cycle detection or lock-acquisition timeouts)?

