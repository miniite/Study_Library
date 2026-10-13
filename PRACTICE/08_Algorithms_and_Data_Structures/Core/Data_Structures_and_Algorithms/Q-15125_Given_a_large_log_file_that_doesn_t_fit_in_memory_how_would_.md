---
type: "practice"
difficulty: "Expert"
---

# Given a large log file that doesn't fit in memory, how would you find the top-k most frequent entries?

**Difficulty:** Expert · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Top-k and Sketches](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Top-k_and_Sketches.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

What is really being asked: I have a log file far larger than RAM (say 500 GB of lines, RAM = 16 GB). I must return the k most frequently occurring entries (e.g., top 10 URLs, IPs, search queries). The naive "load everything into a hash map, then sort" fails because the number of distinct keys — not just the raw file — may exceed memory.

What the interviewer is probing for: Do I recognize this as an external-memory / streaming problem? Can I separate the two constraints (file > RAM vs. distinct-key set > RAM)? Do I know the exact-vs-approximate trade-off, the right data structures (min-heap, external merge sort, hashing/sharding, Count-Min Sketch), and how this scales to a distributed cluster (MapReduce)? At the "expertise" level they want failure modes, skew handling, and judgment about when to pick each approach.

Clarifying questions I'd ask:

Do we need an exact answer or is an approximate top-k acceptable? (Huge difference in cost.)
How many distinct entries roughly — do they fit in memory even if the file doesn't? (Log lines vs. cardinality.)
Is this one-shot batch, or a continuous stream where I need top-k over a sliding window?
Single machine only, or can I use a cluster (Spark/Hadoop)?
What is k? (10 vs. 100k changes the heap cost.)

I'll present the solution as a decision tree over these answers.

Case A — File > RAM, but distinct keys fit in RAM (most common)

If the file is 500 GB but there are only, say, 50 million distinct URLs, a hash map of {key -> count} is a few GB and fits. The file size is then irrelevant because I never hold the file — I stream it.

Algorithm:

Stream the file line by line (buffered read, O(1) memory for the file itself).
Maintain HashMap<String,Long> counts; increment per line.
Maintain a min-heap of size k keyed by count. After counting (or incrementally), keep only the k largest.

Counting is O(N) time (N = total lines). Selecting top-k from D distinct keys with a size-k min-heap is O(D log k). Total: O(N + D log k) time, O(D) space. This beats sorting all D keys (O(D log D)) when k ≪ D, which it usually is. This is the core "top-k with a min-heap" pattern: iterate keys, push if heap < k, else if count > heap-min, pop and push.

Why a min-heap of size k (not max-heap)? The heap holds the current best k; its smallest element is the "bar." Any new key must beat the bar to enter. Popping the min is O(log k).

Case B — Distinct keys also exceed RAM (true external problem)

Now the count map itself won't fit. Two standard exact techniques:

B1 — Hash sharding / partitioning (divide and conquer). Read the file once and route each entry to one of M output files by hash(key) % M. Choose M so each shard's distinct keys fit in RAM. Because hashing sends all occurrences of a given key to the same shard, I can compute each shard's local counts independently and correctly — no key is split across shards. For each shard: load, count in a hash map, extract its local top-k with a size-k min-heap. Finally, merge the M local top-k lists into a global top-k with one more size-k min-heap over M·k candidates. This is exact. Cost: two passes over the data, O(N) I/O each, plus per-shard in-memory work.

B2 — External merge sort + linear scan. Sort the file by key using external merge sort (split into RAM-sized runs, sort each, k-way merge). Then a single linear scan groups identical adjacent keys, computes each count, and feeds counts into a size-k min-heap. Cost O(N log(N/M)) I/O for the sort — more expensive than B1, so I prefer sharding unless I also need the sorted data for other reasons.

Skew handling (important at expertise level): If one key (a hot IP under attack, a viral query) dominates, its shard becomes a hotspot and may still blow memory. Mitigations: (a) a combiner — pre-aggregate within each read buffer before writing to shards; (b) salting hot keys into sub-partitions then re-summing; (c) detect and count the worst offenders separately with a heavy-hitters sketch.

Case C — Approximate top-k (streaming, bounded memory)

If approximate is acceptable — often true for "top search queries" dashboards — I use sublinear-memory sketches and never build the full map:

Count-Min Sketch (CMS): a 2-D array of counters with d hash functions; add(x) increments d cells, estimate(x) = min of those cells (overestimates only, bounded by ε·N with probability 1−δ). Pair it with a size-k min-heap of candidate heavy hitters: for each item, update CMS, query its estimate, and maintain the heap. Memory is O(1/ε · log 1/δ), independent of D.
Space-Saving / Misra–Gries (frequent algorithm): keeps a fixed table of m counters; guarantees finding all items with frequency > N/m. Very memory-cheap, deterministic error bound.

These trade a small, bounded error for constant memory and a single pass — ideal for streams where you can't revisit data.

Case D — Distributed (cluster available)

This is textbook MapReduce / Spark: map emits (entry, 1), a combiner pre-sums per partition to cut shuffle, reduce sums per key, then a top-k reduce (each reducer emits its local top-k; a final single reducer merges). In Spark: rdd.map(k->(k,1)).reduceByKey(_+_).top(k)(Ordering.by(_._2)). Sharding (Case B1) is literally the single-node version of this shuffle.

Choosing — the judgment
Situation	Choice
File > RAM, distinct keys fit	Stream-count + size-k min-heap (Case A)
Distinct keys > RAM, exact needed, one box	Hash-shard then per-shard top-k, merge (B1)
Also need sorted output	External merge sort + scan (B2)
Approximate OK / true stream / tiny memory	Count-Min Sketch + heap or Space-Saving (C)
Cluster available, huge scale	MapReduce/Spark reduceByKey + top-k (D)
Summary — what a strong answer emphasizes

Don't jump to "hash map + sort." First separate file size from cardinality: streaming solves the first, sharding/sketching solves the second. Know the size-k min-heap gives O(D log k) selection. For exact-at-scale, hash partitioning guarantees each key lands in one shard, making local top-k mergeable. For streams, Count-Min Sketch / Space-Saving give bounded-error top-k in constant memory and one pass. Mention skew/hot-key handling and combiners. Recognize sharding is the single-node twin of MapReduce.

#### Common mistakes
sorting the whole file (O(N log N) I/O, unnecessary); building the full map when it won't fit; forgetting that a single hot key can still overflow a shard; using a max-heap and re-sorting everything; claiming CMS can undercount (it only over-estimates).

#### Follow-up
probes: "top-k over a sliding time window" (add exponential decay or windowed CMS / bucketed counts), "ties," "exactly-once counting with duplicate log lines," and "how big must M be" (M ≈ D·avg_key_bytes / usable_RAM, with headroom for skew).

**Comparison:**

```
Approach	Passes	Time	Extra space	Exact?	Best when
Stream-count + size-k min-heap (A)	1	O(N + D log k)	O(D)	Yes	File>RAM, keys fit
Hash-shard + per-shard top-k (B1)	2	O(N) I/O + local	O(RAM)	Yes	Keys>RAM, one box
External merge sort + scan (B2)	O(log) merge	O(N log(N/M)) I/O	O(RAM)	Yes	Also need sorted data
Count-Min Sketch + heap (C)	1	O(N·d)	O((1/ε)·log(1/δ))	Approx (over)	Stream, tiny memory
Space-Saving / Misra–Gries (C)	1	O(N)	O(m counters)	Approx (bounded)	Guaranteed heavy hitters
MapReduce / Spark reduceByKey+top (D)	shuffle	distributed	cluster	Yes	Cluster, huge N
```

**Code:**

```

// Case A: file > RAM, distinct keys fit. Stream-count + size-k min-heap. import java.io.; import java.util.;

public class TopK { static List<Map.Entry<String,Long>> topK(String path, int k) throws IOException { Map<String,Long> counts = new HashMap<>(); try (BufferedReader br = new BufferedReader(new FileReader(path))) { String line; while ((line = br.readLine()) != null) // O(1) memory for the file counts.merge(line.trim(), 1L, Long::sum); } // size-k MIN-heap: smallest count sits on top = the "bar" to beat PriorityQueue<Map.Entry<String,Long>> heap = new PriorityQueue<>(Comparator.comparingLong(Map.Entry::getValue)); for (Map.Entry<String,Long> e : counts.entrySet()) { // O(D log k) if (heap.size() < k) heap.offer(e); else if (e.getValue() > heap.peek().getValue()) { heap.poll(); heap.offer(e); } } List<Map.Entry<String,Long>> res = new ArrayList<>(heap); res.sort((a,b) -> Long.compare(b.getValue(), a.getValue())); // desc return res; } }

CODE
Copy

```python
# Case C: approximate top-k over a STREAM with Count-Min Sketch + heap (constant memory)
import heapq, hashlib

class CountMinTopK:
    def __init__(self, k, width=1<<20, depth=5):
        self.k, self.w, self.d = k, width, depth
        self.table = [[0]*width for _ in range(depth)]
        self.heap = []              # min-heap of (est, item)
        self.in_heap = {}

    def _cols(self, x):
        for i in range(self.d):
            h = int(hashlib.md5(f"{i}:{x}".encode()).hexdigest(), 16)
            yield i, h % self.w

    def add(self, x):
        for i, c in self._cols(x):
            self.table[i][c] += 1
        est = min(self.table[i][c] for i, c in self._cols(x))  # over-estimate only
        if x in self.in_heap:
            self.in_heap[x] = est
        elif len(self.heap) < self.k:
            heapq.heappush(self.heap, (est, x)); self.in_heap[x] = est
        elif est > self.heap[0][0]:
            _, ev = heapq.heapreplace(self.heap, (est, x))
            del self.in_heap[ev]; self.in_heap[x] = est

    def topk(self):
        return sorted(((self.in_heap[x], x) for x in self.in_heap), reverse=True)
SCALA
Copy
// Case D: distributed exact top-k with Spark
val topk = rdd.map(entry => (entry, 1L))
              .reduceByKey(_ + _)        // combiner reduces shuffle
              .top(k)(Ordering.by(_._2)) // global top-k without full sort
```

**Example input / output:**

```

Input file (Case A), k = 2: /home -> 4 /login -> 2 /search -> 1

Dry run of size-k=2 min-heap over distinct keys: push (/home,4) heap = [(/home,4)] push (/login,2) heap = [(/login,2),(/home,4)] (min on top) /search count 1 < bar(2) skip result (sorted desc): [(/home,4), (/login,2)]

Expected output: ["/home" (4), "/login" (2)]

Count-Min Sketch (Case C) sanity: estimates NEVER undercount. true(/home)=4 -> cms.estimate(/home) in {4,5,6,...} (>= true, within eps*N)
```

**Data:**

```code
# Sample log lines (entry = whole line here)
GET /home 200
GET /home 200
GET /login 200
GET /home 200
GET /search?q=heap 200
GET /login 200
GET /home 200
Sharding math (Case B1): pick number of shards M
N_distinct = 5e7 keys, avg_key = 40 bytes -> ~2 GB map; usable RAM per shard = 1 GB
M >= (2 GB / 1 GB) with ~2x skew headroom => M = 4 to 8 shards
```

**Flow:**

1. yes
2. no
3. yes
4. no
5. yes
6. no
7. Large log file, top-k
8. Approximate OK?
9. Count-Min Sketch /Space-Saving + size-k heap1 pass, O(1) memory
10. Distinct keys fit in RAM?
11. Stream file + hashmap counts+ size-k min-heap
12. Cluster available?
13. MapReduce/Spark:map->combiner->reduceByKey->top-k
14. Hash-shard by key % Mper-shard count + local top-k
15. Merge M local top-k listsvia one size-k min-heap

