---
type: "practice"
difficulty: "Intermediate"
---

# Explain common cache eviction policies like LRU and LFU.

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Read-through and Write-behind Caching](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Read-through_and_Write-behind_Caching/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

A cache has finite memory, so when it fills up and a new item must be inserted, some existing item has to be thrown out. The rule that picks the victim is the eviction policy. The interviewer wants me to (1) explain why eviction exists, (2) define LRU and LFU precisely — including how each is implemented in O(1), (3) compare their behavior and know when each wins, and (4) mention the practical variants and how real systems (Redis) do it. At an intermediate level, the discriminator is whether I can describe the O(1) data structures and reason about access patterns — LRU vs LFU behave very differently depending on the workload. I'll assume an in-memory cache with a fixed capacity of N entries.

Why eviction exists

The goal of any policy is to maximize the hit ratio for a given capacity — i.e., keep the items most likely to be requested again, and discard the rest. Since we can't see the future, every policy is a heuristic that predicts future access from past access. The theoretical optimum (Bélády's algorithm — evict the item used farthest in the future) is impossible online, so LRU and LFU are two practical approximations built on different assumptions.

LRU — Least Recently Used

Policy: evict the item that was accessed longest ago. Assumption: temporal locality — something used recently is likely to be used again soon; something untouched for a while probably won't be.

How to make it O(1): combine a hash map + doubly-linked list.

The doubly-linked list keeps entries in recency order: most-recently-used (MRU) at the head, least-recently-used (LRU) at the tail.
The hash map maps key -> node for O(1) lookup.
On get: find the node via the map, unlink it, and move it to the head. O(1).
On put: insert at the head; if over capacity, remove the tail node (and its map entry). O(1).

Both operations are O(1) time; memory overhead is the list pointers. This is the classic "LRU Cache" interview problem, and it's exactly what a LinkedHashMap (Java, access-order) gives you for free. See the code block.

Strengths: cheap, predictable, great for workloads with strong recency (session data, recent pages, sliding-window access). Weakness — scan / flooding: one large sequential pass over many one-time-use keys (e.g. a full-table scan) evicts your genuinely hot items, because each cold key looks "most recently used" as it streams through. This is cache pollution.

LFU — Least Frequently Used

Policy: evict the item accessed the fewest times. Assumption: popularity — an item requested many times overall is more valuable than one requested once, regardless of when.

How to make it O(1): the naive version (a min-heap by count) is O(log n) per op. The O(1) design keeps a frequency-bucket structure:

A map key -> (value, freqCount).
A map freq -> doubly-linked list of keys at that frequency, ordered within a bucket by recency (so ties break by LRU).
A minFreq pointer to the smallest non-empty bucket.
On get/put(hit): move the key from its bucket f to bucket f+1; if bucket f was minFreq and is now empty, minFreq++. O(1).
On evict: remove the LRU key from the minFreq bucket. O(1).

Strengths: protects stable, popular items; resistant to the scan problem that hurts LRU (a one-time flood only reaches frequency 1 and is evicted first). Excellent for skewed/Zipfian workloads (a small set of very hot keys).

Weaknesses:

Cache stagnation / aging problem. An item that was hugely popular last week keeps a high count and refuses to leave, even though it's now dead. Pure LFU has no notion of when — fixed by aging (periodically decay all counts, or use a sliding-window / decayed frequency).
Cold-start bias. A brand-new item starts at count 1 and can be evicted before it proves itself, so genuinely-becoming-popular items struggle to survive.
More bookkeeping than LRU.
LRU vs LFU — the decision
Recency matters more than popularity (news, recent activity, time-windowed access) → LRU.
A stable set of hot keys dominates (popular products, celebrity profiles, Zipfian traffic) → LFU keeps them pinned; LRU might evict a hot key just because it went quiet for a moment.
Scan-heavy / mixed workloads → LFU or a scan-resistant variant, because raw LRU gets polluted.

Real systems rarely ship pure LFU. Redis, for example, offers allkeys-lru, allkeys-lfu, volatile-ttl, and allkeys-random. Crucially, Redis's LRU/LFU are approximate — it samples a handful of keys (default 5) and evicts the best among the sample rather than maintaining a global order, trading a tiny accuracy loss for much lower memory/CPU. Its LFU also uses a logarithmic, decaying counter (LFU-log-factor, LFU-decay-time) precisely to fix the aging problem. Modern caches (e.g. Caffeine) use W-TinyLFU, which combines a small LRU "admission window" with a frequency sketch, getting LFU's popularity awareness and LRU's freshness while resisting scans — often the best real-world hit ratios.

Worth naming alongside these: FIFO (evict oldest inserted — cheap, ignores usage), TTL-based (evict on expiry — orthogonal, about freshness not space), and Random (evict a random key — surprisingly decent and O(1) with zero bookkeeping).

Summary — what a strong answer emphasizes

Eviction picks a victim when a bounded cache is full, aiming to maximize hit ratio by predicting future use from past use. LRU bets on recency (evict the oldest-touched item), is trivially O(1) with a hashmap + doubly-linked list, but is vulnerable to scan pollution. LFU bets on popularity (evict the least-frequently-used), is O(1) with frequency buckets + a minFreq pointer, resists scans, but suffers stagnation without aging/decay. The senior tell is (a) describing the O(1) structures, (b) matching policy to access pattern, and (c) knowing real systems use approximate/sampled LRU/LFU with decay, or hybrids like W-TinyLFU, rather than the textbook forms. Common follow-ups: "code an LRU cache," "how does Redis approximate LRU," and "why does LFU need aging?"

**Comparison:**

```
Aspect	LRU (Least Recently Used)	LFU (Least Frequently Used)
Evicts	Oldest-accessed item	Least-often-accessed item
Bets on	Temporal locality (recency)	Popularity (frequency)
O(1) structure	HashMap + doubly-linked list	Freq buckets + minFreq pointer
Best workload	Recency-driven, sliding window	Skewed / Zipfian, stable hot set
Weakness	Scan/flood pollution	Stagnation (stale-but-high-count)
Fix for weakness	Scan-resistant variants (LRU-K)	Aging / decayed counts
Bookkeeping cost	Low	Higher
Real-world use	Redis allkeys-lru, LinkedHashMap	Redis allkeys-lfu (log+decay), TinyLFU
```

**Code:**

```

// LRU cache in O(1): HashMap + Doubly-Linked List (head = MRU, tail = LRU) class LRUCache { class Node { int key, val; Node prev, next; Node(int k,int v){key=k;val=v;} } private final int cap; private final Map<Integer, Node> map = new HashMap<>(); private final Node head = new Node(0,0), tail = new Node(0,0);

CODE
Copy
LRUCache(int capacity){ cap = capacity; head.next = tail; tail.prev = head; }

private void remove(Node n){ n.prev.next = n.next; n.next.prev = n.prev; }
private void addFront(Node n){ n.next = head.next; n.prev = head;
                               head.next.prev = n; head.next = n; }

public int get(int key){
    if(!map.containsKey(key)) return -1;
    Node n = map.get(key);
    remove(n); addFront(n);      // mark most-recently-used
    return n.val;
}
public void put(int key, int val){
    if(map.containsKey(key)){ Node n = map.get(key); n.val = val; remove(n); addFront(n); return; }
    if(map.size() == cap){                    // full -> evict LRU (tail.prev)
        Node lru = tail.prev; remove(lru); map.remove(lru.key);
    }
    Node n = new Node(key, val); addFront(n); map.put(key, n);
}

}

CODE
Copy

```python
# LFU cache in O(1): freq buckets (each an ordered dict = LRU within a freq) + minFreq
from collections import defaultdict, OrderedDict

class LFUCache:
    def __init__(self, capacity):
        self.cap = capacity
        self.val = {}                          # key -> value
        self.freq = {}                         # key -> access count
        self.buckets = defaultdict(OrderedDict)  # freq -> {key: None} in recency order
        self.minFreq = 0

    def _bump(self, key):                      # move key from freq f to f+1
        f = self.freq[key]
        del self.buckets[f][key]
        if not self.buckets[f]:
            del self.buckets[f]
            if self.minFreq == f: self.minFreq += 1
        self.freq[key] = f + 1
        self.buckets[f + 1][key] = None

    def get(self, key):
        if key not in self.val: return -1
        self._bump(key)
        return self.val[key]

    def put(self, key, value):
        if self.cap == 0: return
        if key in self.val:
            self.val[key] = value; self._bump(key); return
        if len(self.val) == self.cap:          # evict least-freq, LRU tie-break
            k, _ = self.buckets[self.minFreq].popitem(last=False)
            del self.val[k]; del self.freq[k]
        self.val[key] = value; self.freq[key] = 1
        self.buckets[1][key] = None; self.minFreq = 1
```

**Example input / output:**

```text
LRU trace (capacity = 2):
  put(1,A)              cache: [1]           (MRU..LRU)
  put(2,B)              cache: [2,1]
  get(1) -> A           cache: [1,2]         (1 becomes MRU)
  put(3,C)              evict 2 (LRU)  -> cache: [3,1]
  get(2) -> -1          (2 was evicted)
  put(4,D)              evict 1 (LRU)  -> cache: [4,3]

LFU trace (capacity = 2):
  put(1,A)             freq{1:1}                minFreq=1
  put(2,B)             freq{1:1, 2:1}           minFreq=1
  get(1) -> A          freq{1:2, 2:1}           minFreq=1
  put(3,C)             evict 2 (freq 1, lowest) -> {1:2, 3:1}  minFreq=1
  get(3) -> C          freq{1:2, 3:2}           minFreq=2
  put(4,D)             tie at freq 2 -> evict 1 (LRU of that bucket) -> {3:2, 4:1}

Scan-pollution contrast (capacity = 2, hot key = 1):
  Sequence: get(1)... then stream cold keys 9,8,7 once each
  LRU: cold keys push out hot key 1  -> 1 evicted (BAD)
  LFU: cold keys stay at freq 1 and are evicted first; key 1 (high freq) survives (GOOD)
```

**Flow:**

1. no
2. yes
3. LRU
4. LFU
5. put new item
6. cache full?
7. insert item
8. policy?
9. evict tail = least recently used
10. evict key in minFreq bucket,LRU tie-break
11. on future get: LRU moves tohead; LFU bumps freq +1

