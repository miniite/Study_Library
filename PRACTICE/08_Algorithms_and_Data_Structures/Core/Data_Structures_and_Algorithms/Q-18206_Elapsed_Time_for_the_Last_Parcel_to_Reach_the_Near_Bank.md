---
type: "practice"
difficulty: "Other"
---

# Elapsed Time for the Last Parcel to Reach the Near Bank

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Heap (Priority Queue)](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Heap_(Priority_Queue).md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

There are k couriers available to carry n identical parcels from a far depot on the far bank of a narrow river to a near depot on the near bank. The two banks are joined by a single-lane footbridge that only one courier may occupy at any moment. Every courier begins waiting on the near bank.

You are given the two integers n and k, and a 2D integer array cost of size k x 4, where cost[i] = [toFar_i, load_i, toNear_i, unload_i] describes courier i:

crossing the footbridge from the near bank to the far bank takes toFar_i minutes,
loading one parcel at the far depot takes load_i minutes,
crossing the footbridge from the far bank back to the near bank takes toNear_i minutes,
unloading the parcel at the near depot takes unload_i minutes.

Ranking couriers by efficiency. Courier i is considered less efficient than courier j when either condition holds:

toNear_i + toFar_i > toNear_j + toFar_j, or
toNear_i + toFar_i == toNear_j + toFar_j and i > j.

Rules that govern the footbridge:

At most one courier may be on the footbridge at any time.
Whenever the footbridge is free, priority is given to the least efficient courier currently waiting on the far bank who has already loaded a parcel; that courier crosses toward the near bank. If no such courier is waiting on the far bank, then the least efficient courier currently waiting on the near bank crosses toward the far bank instead.
Once enough couriers have already been dispatched from the near bank to fetch every parcel that still remains to be picked up, no further couriers are sent from the near bank.

Return the elapsed number of minutes at which the last parcel first reaches the near bank of the footbridge — that is, the moment at which its carrier finishes the crossing back to the near bank. The final unload after that crossing is not included in the answer.

#### Cons
traints:

1 <= n, k <= 10^4
cost.length == k
cost[i].length == 4
1 <= toFar_i, load_i, toNear_i, unload_i <= 1000

#### Example
s & test cases
TEXT
Input: n = 1, k = 3, cost = [[1,1,2,1],[1,1,3,1],[1,1,4,1]] -> Output: 6
Why: The single parcel is fetched by the least efficient courier (index 2, toFar+toNear = 5). It crosses out (0->1), loads (1->2), and — after waiting until it is ready — crosses back finishing at minute 6. The later unload is not counted.

Input: n = 3, k = 2, cost = [[1,5,1,8],[10,10,10,10]] -> Output: 37
Why: Standard multi-trip case; couriers are dispatched greedily by least efficiency and the bridge stays single-lane. The last parcel completes its return crossing at minute 37; the final unloads are skipped because the parcels are already on the near bank.

Input: n = 1, k = 1, cost = [[2,3,4,5]] -> Output: 9
Why: Minimal case. One courier crosses out (0->2), loads (2->5), then crosses back finishing at 5+4 = 9. Unload of 5 is not counted.

Input: n = 2, k = 1, cost = [[1,1,1,1]] -> Output: 7
Why: A single courier must make two full round trips serially. First parcel returns at minute 3 (then unloads until 4); the courier is dispatched again and the second parcel returns at minute 7.

Input: n = 2, k = 2, cost = [[2,2,2,2],[2,2,2,2]] -> Output: 8
Why: Efficiency tie (both have toFar+toNear = 4), so the larger index is the less efficient one and is dispatched first. The two parcels return at minutes 6 and 8; the answer is the later, 8.

#### Approach

Model the whole process as an event-driven simulation with four priority queues: two "waiting" queues (couriers idle on the near bank / on the far bank holding a loaded parcel) keyed so the LEAST efficient courier — larger toFar+toNear, ties broken by larger index — pops first, and two "working" queues (couriers currently loading on the far bank / unloading on the near bank) keyed by their finish time. Keep a clock cur, a counter of parcels still needing to be fetched (nPick), and parcels still undelivered (remaining). Each loop: first release every worker whose load/unload finished by cur into the matching wait queue, then apply the bridge rule — a loaded courier on the far bank returns (record cur as the delivery moment, the running answer), else if parcels remain to fetch dispatch the least efficient idle courier outward, else jump the clock forward to the next work-completion time. Because the clock is monotonic, the last recorded return time is the answer; the final unload is intentionally excluded. This is the classic multi-heap greedy simulation pattern.

Complexity

Time: O((n + k) log k). Each of the n dispatches and n return crossings performs a constant number of heap pushes/pops costing O(log k); the "advance clock" branch fires O(n) times amortized because every advance is paired with a work item that will later produce a crossing. Space: O(k) for the four priority queues holding at most k couriers in total.

Python
PYTHON
import heapq

def last_crossing_time(n, k, cost):
    # wait heaps store (-(toFar+toNear), -index) so the smallest tuple is the LEAST efficient courier
    wait_l = []
    wait_r = []
    work_l = []   # min-heap by finish time (unloading on the near bank)
    work_r = []   # min-heap by finish time (loading on the far bank)

    for i in range(k):
        eff = cost[i][0] + cost[i][2]
        heapq.heappush(wait_l, (-eff, -i))

    cur = 0
    ans = 0
    n_pick = n
    remaining = n
    while remaining > 0:
        while work_r and work_r[0][0] <= cur:
            _, i = heapq.heappop(work_r)
            eff = cost[i][0] + cost[i][2]
            heapq.heappush(wait_r, (-eff, -i))
        while work_l and work_l[0][0] <= cur:
            _, i = heapq.heappop(work_l)
            eff = cost[i][0] + cost[i][2]
            heapq.heappush(wait_l, (-eff, -i))

        if wait_r:
            i = -heapq.heappop(wait_r)[1]
            cur += cost[i][2]
            ans = cur
            remaining -= 1
            heapq.heappush(work_l, (cur + cost[i][3], i))
        elif n_pick > 0 and wait_l:
            i = -heapq.heappop(wait_l)[1]
            cur += cost[i][0]
            n_pick -= 1
            heapq.heappush(work_r, (cur + cost[i][1], i))
        else:
            nxt = None
            if work_l:
                nxt = work_l[0][0]
            if work_r:
                nxt = work_r[0][0] if nxt is None else min(nxt, work_r[0][0])
            cur = nxt

    return ans
JavaScript
JAVASCRIPT
class Heap {

#### cons
tructor(less) { this.a = []; this.less = less; }
  size() { return this.a.length; }
  peek() { return this.a[0]; }
  push(x) {

#### cons
t a = this.a; a.push(x); let i = a.length - 1;
    while (i > 0) { const p = (i - 1) >> 1; if (!this.less(a[i], a[p])) break; [a[p], a[i]] = [a[i], a[p]]; i = p; }
  }
  pop() {

#### cons
t a = this.a, top = a[0], last = a.pop();
    if (a.length) {
      a[0] = last; let i = 0; const n = a.length;
      while (true) {
        let l = 2 * i + 1, r = 2 * i + 2, s = i;
        if (l < n && this.less(a[l], a[s])) s = l;
        if (r < n && this.less(a[r], a[s])) s = r;
        if (s === i) break;
        [a[s], a[i]] = [a[i], a[s]]; i = s;
      }
    }
    return top;
  }
}

function lastCrossingTime(n, k, cost) {
  // wait heaps pop the LEAST efficient: larger (toFar+toNear), tie larger index

#### cons
t waitLess = (x, y) => (x[0] !== y[0] ? x[0] > y[0] : x[1] > y[1]);

#### cons
t workLess = (x, y) => x[0] < y[0]; // earliest finish first

#### cons
t waitL = new Heap(waitLess), waitR = new Heap(waitLess);

#### cons
t workL = new Heap(workLess), workR = new Heap(workLess);

  for (let i = 0; i < k; i++) waitL.push([cost[i][0] + cost[i][2], i]);

  let cur = 0, ans = 0, nPick = n, remaining = n;
  while (remaining > 0) {
    while (workR.size() && workR.peek()[0] <= cur) { const i = workR.pop()[1]; waitR.push([cost[i][0] + cost[i][2], i]); }
    while (workL.size() && workL.peek()[0] <= cur) { const i = workL.pop()[1]; waitL.push([cost[i][0] + cost[i][2], i]); }

    if (waitR.size()) {

#### cons
t i = waitR.pop()[1];
      cur += cost[i][2]; ans = cur; remaining--;
      workL.push([cur + cost[i][3], i]);
    } else if (nPick > 0 && waitL.size()) {

#### cons
t i = waitL.pop()[1];
      cur += cost[i][0]; nPick--;
      workR.push([cur + cost[i][1], i]);
    } else {
      let nxt = Infinity;
      if (workL.size()) nxt = Math.min(nxt, workL.peek()[0]);
      if (workR.size()) nxt = Math.min(nxt, workR.peek()[0]);
      cur = nxt;
    }
  }
  return ans;
}
Java
JAVA
import java.util.PriorityQueue;

class Solution {
    public int lastCrossingTime(int n, int k, int[][] cost) {
        // wait heaps: pop the LEAST efficient -> larger (toFar+toNear), tie larger index
        PriorityQueue<int[]> waitL = new PriorityQueue<>((a, b) -> a[0] != b[0] ? b[0] - a[0] : b[1] - a[1]);
        PriorityQueue<int[]> waitR = new PriorityQueue<>((a, b) -> a[0] != b[0] ? b[0] - a[0] : b[1] - a[1]);
        // work heaps: min-heap on finish time
        PriorityQueue<long[]> workL = new PriorityQueue<>((a, b) -> Long.compare(a[0], b[0]));
        PriorityQueue<long[]> workR = new PriorityQueue<>((a, b) -> Long.compare(a[0], b[0]));

        for (int i = 0; i < k; i++)
            waitL.offer(new int[]{cost[i][0] + cost[i][2], i});

        long cur = 0, ans = 0;
        int nPick = n, remaining = n;
        while (remaining > 0) {
            while (!workR.isEmpty() && workR.peek()[0] <= cur) {
                int i = (int) workR.poll()[1];
                waitR.offer(new int[]{cost[i][0] + cost[i][2], i});
            }
            while (!workL.isEmpty() && workL.peek()[0] <= cur) {
                int i = (int) workL.poll()[1];
                waitL.offer(new int[]{cost[i][0] + cost[i][2], i});
            }
            if (!waitR.isEmpty()) {
                int i = waitR.poll()[1];
                cur += cost[i][2];
                ans = cur;
                remaining--;
                workL.offer(new long[]{cur + cost[i][3], i});
            } else if (nPick > 0 && !waitL.isEmpty()) {
                int i = waitL.poll()[1];
                cur += cost[i][0];
                nPick--;
                workR.offer(new long[]{cur + cost[i][1], i});
            } else {
                long nxt = Long.MAX_VALUE;
                if (!workL.isEmpty()) nxt = Math.min(nxt, workL.peek()[0]);
                if (!workR.isEmpty()) nxt = Math.min(nxt, workR.peek()[0]);
                cur = nxt;
            }
        }
        return (int) ans;
    }
}
C++
CPP
#include <vector>
#include <queue>
#include <climits>
using namespace std;

class Solution {
public:
    int lastCrossingTime(int n, int k, vector<vector<int>>& cost) {
        // wait heaps: max-heap on (efficiency-key, index) -> pops the LEAST efficient courier
        priority_queue<pair<int,int>> waitL, waitR;
        // work heaps: min-heap on finish time
        priority_queue<pair<long long,int>, vector<pair<long long,int>>, greater<>> workL, workR;

        for (int i = 0; i < k; i++)
            waitL.push({cost[i][0] + cost[i][2], i});

        long long cur = 0, ans = 0;
        int nPick = n, remaining = n;
        while (remaining > 0) {
            while (!workR.empty() && workR.top().first <= cur) {
                int i = workR.top().second; workR.pop();
                waitR.push({cost[i][0] + cost[i][2], i});
            }
            while (!workL.empty() && workL.top().first <= cur) {
                int i = workL.top().second; workL.pop();
                waitL.push({cost[i][0] + cost[i][2], i});
            }
            if (!waitR.empty()) {
                int i = waitR.top().second; waitR.pop();
                cur += cost[i][2];
                ans = cur;
                remaining--;
                workL.push({cur + cost[i][3], i});
            } else if (nPick > 0 && !waitL.empty()) {
                int i = waitL.top().second; waitL.pop();
                cur += cost[i][0];
                nPick--;
                workR.push({cur + cost[i][1], i});
            } else {
                long long nxt = LLONG_MAX;
                if (!workL.empty()) nxt = min(nxt, workL.top().first);
                if (!workR.empty()) nxt = min(nxt, workR.top().first);
                cur = nxt;
            }
        }
        return (int)ans;
    }
};
C
C
#include <stdlib.h>

typedef struct { long long key; int idx; } Node;
typedef struct { Node *a; int size, cap; } Heap;

static void hinit(Heap *h, int cap) {
    h->cap = cap > 0 ? cap : 1;
    h->a = (Node *)malloc(sizeof(Node) * h->cap);
    h->size = 0;
}
static void hpush(Heap *h, long long key, int idx) {
    if (h->size == h->cap) { h->cap *= 2; h->a = (Node *)realloc(h->a, sizeof(Node) * h->cap); }
    int i = h->size++;
    h->a[i].key = key; h->a[i].idx = idx;
    while (i > 0) {
        int p = (i - 1) / 2;
        if (h->a[p].key <= h->a[i].key) break;
        Node t = h->a[p]; h->a[p] = h->a[i]; h->a[i] = t; i = p;
    }
}
static Node hpop(Heap *h) {
    Node top = h->a[0];
    h->a[0] = h->a[--h->size];
    int i = 0;
    while (1) {
        int l = 2 * i + 1, r = 2 * i + 2, s = i;
        if (l < h->size && h->a[l].key < h->a[s].key) s = l;
        if (r < h->size && h->a[r].key < h->a[s].key) s = r;
        if (s == i) break;
        Node t = h->a[s]; h->a[s] = h->a[i]; h->a[i] = t; i = s;
    }
    return top;
}

/* Two "wait" heaps pop the LEAST efficient courier: we encode (toFar+toNear, index)
   as a single increasing composite key and store its negation so the min-heap yields
   the maximum. Two "work" heaps pop the earliest finish time. */
int lastCrossingTime(int n, int k, int **cost, int costSize, int *costColSize) {
    Heap waitL, waitR, workL, workR;
    hinit(&waitL, k); hinit(&waitR, k); hinit(&workL, k); hinit(&workR, k);
    for (int i = 0; i < k; i++) {
        long long comp = (long long)(cost[i][0] + cost[i][2]) * (k + 1) + i;
        hpush(&waitL, -comp, i);
    }
    long long cur = 0, ans = 0;
    int nPick = n, remaining = n;
    while (remaining > 0) {
        while (workR.size > 0 && workR.a[0].key <= cur) {
            Node nd = hpop(&workR);
            long long comp = (long long)(cost[nd.idx][0] + cost[nd.idx][2]) * (k + 1) + nd.idx;
            hpush(&waitR, -comp, nd.idx);
        }
        while (workL.size > 0 && workL.a[0].key <= cur) {
            Node nd = hpop(&workL);
            long long comp = (long long)(cost[nd.idx][0] + cost[nd.idx][2]) * (k + 1) + nd.idx;
            hpush(&waitL, -comp, nd.idx);
        }
        if (waitR.size > 0) {
            int i = hpop(&waitR).idx;
            cur += cost[i][2];
            ans = cur;
            remaining--;
            hpush(&workL, cur + cost[i][3], i);
        } else if (nPick > 0 && waitL.size > 0) {
            int i = hpop(&waitL).idx;
            cur += cost[i][0];
            nPick--;
            hpush(&workR, cur + cost[i][1], i);
        } else {
            long long nxt = -1;
            if (workL.size > 0) nxt = workL.a[0].key;
            if (workR.size > 0) nxt = (nxt < 0) ? workR.a[0].key : (workR.a[0].key < nxt ? workR.a[0].key : nxt);
            cur = nxt;
        }
    }
    free(waitL.a); free(waitR.a); free(workL.a); free(workR.a);
    return (int)ans;
}

#### Follow-up
questions
What changes if a courier can carry more than one parcel per trip, or if the bridge allows two couriers crossing in opposite directions simultaneously?
Instead of the last arrival time, return the full schedule (which courier crossed when) so it can be audited.
How would you handle live/streaming arrivals where new parcels appear at the far depot over time rather than all being known up front?
Prove that always dispatching (and always returning) the least efficient available courier never increases the makespan compared with any other tie-breaking policy.

