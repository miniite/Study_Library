---
type: "practice"
difficulty: "Other"
---

# Assign Queued Jobs to the Lightest Available Machine

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Heap (Priority Queue)](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Heap_(Priority_Queue).md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given two 0-indexed integer arrays: weights of length n and durations of length m. Value weights[i] is the weight (priority key) of machine i, and durations[j] is the number of seconds it takes to process job j.

Jobs are handed out through a single FIFO queue. Initially every machine is free and the queue is empty.

At second j, job j is inserted at the back of the queue (job 0 arrives at second 0, job 1 at second 1, and so on).
Whenever there is at least one free machine and the queue is non-empty, the job at the front of the queue is immediately assigned to a free machine with the smallest weight; ties are broken by the smallest machine index.
If the queue is non-empty but no machine is free, the system waits until a machine becomes free and assigns the next front job at that moment. If several machines become free at the same second, several front jobs are assigned in queue order, each following the smallest-weight / smallest-index rule.
A machine that starts job j at second t becomes free again at second t + durations[j].

Build and return an array answer of length m, where answer[j] is the index of the machine that processes job j.

Input

weights: array of n positive integers, the machine weights.
durations: array of m positive integers, the job processing times in seconds.

Output

An integer array answer of length m; answer[j] is the index of the machine assigned to job j.

#### Cons
traints

weights.length == n
durations.length == m
1 <= n, m <= 2 * 10^5
1 <= weights[i], durations[j] <= 2 * 10^5

#### Example
s & test cases
TEXT
Input: weights = [2, 2, 1], durations = [2, 1, 3, 1] -> Output: [2, 0, 2, 0]
Why: Machine 2 is lightest, so jobs 0 and 2 take it whenever it is free; between them machine 0 (smaller index than machine 1 at equal weight) handles the intervening jobs.

Input: weights = [2, 1], durations = [4, 4, 1] -> Output: [1, 0, 1]
Why: Job 0 takes lighter machine 1, job 1 takes machine 0; job 2 arrives at second 2 while both machines are busy until second 4, so it waits and is assigned to machine 1 the instant it frees.

Input: weights = [4, 4], durations = [2, 1] -> Output: [0, 1]
Why: Equal weights force the index tie-break — job 0 goes to machine 0, and while machine 0 is still busy job 1 goes to machine 1.

Input: weights = [5], durations = [3, 2] -> Output: [0, 0]
Why: With a single machine every job runs on machine 0 sequentially, each waiting for the previous to finish.

Input: weights = [3, 1, 2], durations = [1, 2, 1, 2, 1] -> Output: [1, 1, 2, 1, 2]
Why: The lightest machine 1 is reused as soon as it frees, and the next-lightest machine 2 picks up jobs while machine 1 is occupied; the heaviest machine 0 is never needed.

#### Approach

Event-driven simulation with two min-heaps. A "free" heap orders idle machines by (weight, index) so the front pop is always the lightest, lowest-index available machine. A "busy" heap orders working machines by their finish time so we know when the next one frees. Process jobs in order: advance the clock to at least the job's arrival second, release every busy machine whose finish time has passed, and if none is free jump the clock forward to the earliest finish time and release again. Then pop the best free machine, record it as this job's answer, and push it into the busy heap with finish time t + duration. This lazily materializes only the events that matter, giving near-linear-log behavior. The pattern is greedy assignment under two competing priorities managed by paired priority queues.

Complexity

Time: O((n + m) log n) — each of the n machines starts in the free heap, and every job assignment pops one free machine and pushes one busy entry, so each element enters and leaves a heap O(1) times, each heap operation costing O(log n). Space: O(n + m) for the two heaps plus the answer array.

Python
PYTHON
import heapq
from typing import List

def assign_jobs(weights: List[int], durations: List[int]) -> List[int]:
    free = [(w, i) for i, w in enumerate(weights)]   # (weight, index)
    heapq.heapify(free)
    busy = []                                        # (free_time, weight, index)
    ans = []
    t = 0
    for j, dur in enumerate(durations):
        t = max(t, j)
        while busy and busy[0][0] <= t:
            _, w, idx = heapq.heappop(busy)
            heapq.heappush(free, (w, idx))
        if not free:
            t = busy[0][0]
            while busy and busy[0][0] <= t:
                _, w, idx = heapq.heappop(busy)
                heapq.heappush(free, (w, idx))
        w, idx = heapq.heappop(free)
        ans.append(idx)
        heapq.heappush(busy, (t + dur, w, idx))
    return ans
JavaScript
JAVASCRIPT
class MinHeap {

#### cons
tructor(cmp) { this.a = []; this.cmp = cmp; }
    get size() { return this.a.length; }
    peek() { return this.a[0]; }
    push(v) {

#### cons
t a = this.a; a.push(v);
        let i = a.length - 1;
        while (i > 0) {

#### cons
t p = (i - 1) >> 1;
            if (this.cmp(a[i], a[p]) < 0) { [a[i], a[p]] = [a[p], a[i]]; i = p; }
            else break;
        }
    }
    pop() {

#### cons
t a = this.a, top = a[0], last = a.pop();
        if (a.length) {
            a[0] = last;
            let i = 0;
            for (;;) {

#### cons
t l = 2 * i + 1, r = 2 * i + 2; let m = i;
                if (l < a.length && this.cmp(a[l], a[m]) < 0) m = l;
                if (r < a.length && this.cmp(a[r], a[m]) < 0) m = r;
                if (m === i) break;
                [a[i], a[m]] = [a[m], a[i]]; i = m;
            }
        }
        return top;
    }
}

function assignJobs(weights, durations) {
    // free: [weight, index]

#### cons
t freeH = new MinHeap((x, y) => x[0] - y[0] || x[1] - y[1]);
    // busy: [freeTime, weight, index]

#### cons
t busyH = new MinHeap((x, y) => x[0] - y[0] || x[1] - y[1] || x[2] - y[2]);

    for (let i = 0; i < weights.length; i++) freeH.push([weights[i], i]);

#### cons
t m = durations.length;

#### cons
t ans = new Array(m);
    let t = 0;
    for (let j = 0; j < m; j++) {
        if (t < j) t = j;
        while (busyH.size && busyH.peek()[0] <= t) {

#### cons
t b = busyH.pop();
            freeH.push([b[1], b[2]]);
        }
        if (!freeH.size) {
            t = busyH.peek()[0];
            while (busyH.size && busyH.peek()[0] <= t) {

#### cons
t b = busyH.pop();
                freeH.push([b[1], b[2]]);
            }
        }

#### cons
t f = freeH.pop();
        ans[j] = f[1];
        busyH.push([t + durations[j], f[0], f[1]]);
    }
    return ans;
}
Java
JAVA
import java.util.PriorityQueue;

class Solution {
    public int[] assignJobs(int[] weights, int[] durations) {
        // free: [weight, index]
        PriorityQueue<int[]> freeH = new PriorityQueue<>(
            (a, b) -> a[0] != b[0] ? a[0] - b[0] : a[1] - b[1]);
        // busy: [freeTime, weight, index]
        PriorityQueue<long[]> busyH = new PriorityQueue<>(
            (a, b) -> a[0] != b[0] ? Long.compare(a[0], b[0])
                    : a[1] != b[1] ? Long.compare(a[1], b[1])
                    : Long.compare(a[2], b[2]));

        for (int i = 0; i < weights.length; i++) freeH.offer(new int[]{weights[i], i});

        int m = durations.length;
        int[] ans = new int[m];
        long t = 0;
        for (int j = 0; j < m; j++) {
            if (t < j) t = j;
            while (!busyH.isEmpty() && busyH.peek()[0] <= t) {
                long[] b = busyH.poll();
                freeH.offer(new int[]{(int) b[1], (int) b[2]});
            }
            if (freeH.isEmpty()) {
                t = busyH.peek()[0];
                while (!busyH.isEmpty() && busyH.peek()[0] <= t) {
                    long[] b = busyH.poll();
                    freeH.offer(new int[]{(int) b[1], (int) b[2]});
                }
            }
            int[] f = freeH.poll();
            ans[j] = f[1];
            busyH.offer(new long[]{t + durations[j], f[0], f[1]});
        }
        return ans;
    }
}
C++
CPP
#include <vector>
#include <queue>
#include <tuple>
using namespace std;

class Solution {
public:
    vector<int> assignJobs(vector<int>& weights, vector<int>& durations) {
        // free: (weight, index) min-heap
        priority_queue<pair<int,int>, vector<pair<int,int>>, greater<>> freeH;
        // busy: (freeTime, weight, index) min-heap
        priority_queue<tuple<long,int,int>, vector<tuple<long,int,int>>, greater<>> busyH;

        for (int i = 0; i < (int)weights.size(); ++i) freeH.push({weights[i], i});

        int m = (int)durations.size();
        vector<int> ans(m);
        long t = 0;
        for (int j = 0; j < m; ++j) {
            if (t < j) t = j;
            while (!busyH.empty() && get<0>(busyH.top()) <= t) {
                auto [ft, w, idx] = busyH.top(); busyH.pop();
                freeH.push({w, idx});
            }
            if (freeH.empty()) {
                t = get<0>(busyH.top());
                while (!busyH.empty() && get<0>(busyH.top()) <= t) {
                    auto [ft, w, idx] = busyH.top(); busyH.pop();
                    freeH.push({w, idx});
                }
            }
            auto [w, idx] = freeH.top(); freeH.pop();
            ans[j] = idx;
            busyH.push({t + durations[j], w, idx});
        }
        return ans;
    }
};
C
C
#include <stdlib.h>

/* Node holds three longs. Free entry: (weight, index, 0).
   Busy entry: (freeTime, weight, index). */
typedef struct { long a, b, c; } Node;

typedef struct { Node* h; int size; int mode; } Heap;  /* mode 0=free, 1=busy */

static int less(int mode, Node x, Node y) {
    if (mode == 0) {                 /* free: weight, then index */
        if (x.a != y.a) return x.a < y.a;
        return x.b < y.b;
    }
    if (x.a != y.a) return x.a < y.a;   /* busy: freeTime, weight, index */
    if (x.b != y.b) return x.b < y.b;
    return x.c < y.c;
}

static void hpush(Heap* hp, Node v) {
    int i = hp->size++;
    hp->h[i] = v;
    while (i > 0) {
        int p = (i - 1) / 2;
        if (less(hp->mode, hp->h[i], hp->h[p])) {
            Node t = hp->h[i]; hp->h[i] = hp->h[p]; hp->h[p] = t; i = p;
        } else break;
    }
}

static Node hpop(Heap* hp) {
    Node top = hp->h[0];
    hp->h[0] = hp->h[--hp->size];
    int i = 0;
    for (;;) {
        int l = 2*i+1, r = 2*i+2, m = i;
        if (l < hp->size && less(hp->mode, hp->h[l], hp->h[m])) m = l;
        if (r < hp->size && less(hp->mode, hp->h[r], hp->h[m])) m = r;
        if (m == i) break;
        Node t = hp->h[i]; hp->h[i] = hp->h[m]; hp->h[m] = t; i = m;
    }
    return top;
}

int* assignJobs(int* weights, int weightsSize, int* durations, int durationsSize, int* returnSize) {
    Heap freeH = { malloc(sizeof(Node) * weightsSize), 0, 0 };
    Heap busyH = { malloc(sizeof(Node) * weightsSize), 0, 1 };
    for (int i = 0; i < weightsSize; i++) {
        Node v = { weights[i], i, 0 };
        hpush(&freeH, v);
    }
    int* ans = malloc(sizeof(int) * durationsSize);
    *returnSize = durationsSize;
    long t = 0;
    for (int j = 0; j < durationsSize; j++) {
        if (t < j) t = j;
        while (busyH.size && busyH.h[0].a <= t) {
            Node b = hpop(&busyH);
            Node f = { b.b, b.c, 0 };     /* weight, index */
            hpush(&freeH, f);
        }
        if (freeH.size == 0) {
            t = busyH.h[0].a;
            while (busyH.size && busyH.h[0].a <= t) {
                Node b = hpop(&busyH);
                Node f = { b.b, b.c, 0 };
                hpush(&freeH, f);
            }
        }
        Node f = hpop(&freeH);
        ans[j] = (int)f.b;                /* machine index */
        Node b = { t + durations[j], f.a, f.b };  /* freeTime, weight, index */
        hpush(&busyH, b);
    }
    free(freeH.h); free(busyH.h);
    return ans;
}

#### Follow-up
questions
How would the assignment change if ties on weight were broken by the largest index instead of the smallest?
What if jobs could arrive at arbitrary timestamps rather than one per second?
How would you also report each job's waiting time or the total makespan?
What if each machine could process several jobs concurrently up to a capacity limit?

