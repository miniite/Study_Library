---
type: "practice"
difficulty: "Deep"
---

# What is a hash collision resolution strategy — compare chaining and open addressing.

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Hashing and Collisions](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Hashing_and_Collisions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

What the question is really asking: A hash table maps keys to buckets via bucket = hash(key) mod capacity. Because the key space is huge and the bucket count is finite, two distinct keys can land in the same bucket — a collision. "Collision resolution" is the strategy the table uses to still store and find both keys. The question then wants a deep comparison of the two families: separate chaining (each bucket holds a secondary container) versus open addressing (all entries live in the array itself, probing to another slot on collision).

What the interviewer is probing for: Not just definitions, but engineering judgment: how each behaves as the table fills (load factor), cache behavior, deletion complexity, memory overhead, resistance to clustering and to adversarial/degenerate hashing, and when you'd pick one over the other. Mentioning real implementations (Java HashMap, Python dict, Go map, open_addressing in Rust/hashbrown, C++ unordered_map) shows you've seen this in practice.

Clarifying questions I'd ask: What's the read/write mix and the expected load factor? Is memory tight or is speed paramount? Do we need frequent deletions? Is the hash function trusted or could inputs be adversarial (HashDoS)? Answers steer the choice.

Plan: Define collision and load factor α = n/m, describe chaining then open addressing (with the three probe schemes), compare across time, memory, cache, deletion, clustering, and resizing, then give a decision rule and real-world examples.

What a collision is (first principles)

index = hash(key) % capacity. By the pigeonhole principle, once you have more possible keys than buckets — always true — some keys share an index. A good hash spreads keys uniformly so collisions are rare, but they're inevitable, so every hash table needs a resolution strategy. The critical dial is the load factor α = n / m (elements / buckets). Both strategies degrade as α rises, but differently.

Strategy 1 — Separate chaining

Each bucket holds a pointer to a secondary structure — classically a linked list, sometimes a dynamic array, and in Java 8+ a list that converts to a balanced (red-black) tree once a bucket exceeds 8 entries. On insert you append to (or search-then-insert in) that bucket's list; on lookup you hash to the bucket then scan its chain; on delete you unlink the node.

Average cost: O(1 + α). With a well-distributed hash and α kept near 1, chains are short (average length α), so operations are effectively O(1).
Worst case: O(n) if every key collides into one bucket (bad hash or adversarial input) — Java mitigates this by treeifying long chains to O(log n).
Load factor: can exceed 1 gracefully; performance degrades linearly, never "fails."
Deletion: trivial — just unlink the node.
Memory: extra per-entry overhead for node pointers (and separate heap allocations), plus mostly-empty bucket array. Pointer chasing hurts CPU cache locality.
Strategy 2 — Open addressing

All entries live directly in the array; there are no external lists. On collision you probe a deterministic sequence of alternative slots until you find an empty one (insert) or the target/an empty slot (lookup). Three classic probe schemes:

Linear probing: try (h + 1), (h + 2), …. Best cache behavior (sequential memory), but suffers primary clustering — long runs of occupied slots that grow and merge, so cost spikes as α → 1.

Quadratic probing: try (h + 1²), (h + 2²), …. Breaks primary clustering; suffers milder secondary clustering (keys with the same initial hash follow the same jumps). Needs care so the probe sequence covers enough slots (works cleanly when capacity is prime or a power of two with the right formula).

Double hashing: step size comes from a second hash: (h1 + i·h2(key)). Effectively eliminates both clustering types by giving each key its own probe stride; best distribution, slightly more compute per probe.

Average cost: O(1) at low α, but degrades sharply as α → 1; you must keep α well below 1 (commonly ≤ 0.7–0.75).

Load factor: hard ceiling — cannot exceed 1 (no free slots means insertion fails); resize before that.

Deletion: the hard part. You can't just empty a slot or you'd break probe chains for other keys. You use a tombstone (a "deleted" marker) so lookups keep probing past it, or you do backward-shift deletion (Robin Hood). Tombstones accumulate and degrade performance until a rehash cleans them.

Memory: no per-node pointers and no separate allocations — compact and cache-friendly, which is why modern high-performance maps (Google's Swiss Tables, Rust's hashbrown, Go's map) use open addressing.

**Comparison:**

```

The table field lays out the head-to-head. Key takeaways:

Cache & speed at low load: open addressing wins — contiguous memory, no pointer chasing, fewer allocations. This is why it dominates modern high-performance implementations.
Robustness at high load & simplicity: chaining wins — it tolerates α > 1, deletion is trivial, and it's less sensitive to clustering.
Deletion: chaining is simple; open addressing needs tombstones/backward-shift.
Memory: chaining spends on pointers/nodes; open addressing spends on empty slots (must stay under ~70–75% full).
Adversarial inputs (HashDoS): long chains or long probe runs are the attack surface; defenses are randomized/seeded hashing plus treeifying (chaining) or resize thresholds (open addressing).
Resizing (both need it)

When α crosses a threshold, allocate a larger array (usually ~2×) and rehash all entries. Chaining resizes when α approaches its target (Java default 0.75); open addressing resizes earlier and also to purge tombstones. Rehashing is O(n) amortized to O(1) per op.

Decision rule (what I'd actually pick)
Default to open addressing for read-heavy, latency-sensitive, memory-conscious workloads where you control α (keep it ≤ 0.7) — it's faster due to cache locality. Use double hashing or Robin Hood to avoid clustering.
Choose chaining when the load factor is unpredictable or high, deletions are frequent and you want simplicity, or entries are large (pointer overhead is relatively small). Treeify long buckets to bound worst case.
Regardless of strategy, use a good, seeded hash function to defend against adversarial inputs.
Real-world anchors
Java HashMap: separate chaining, default load factor 0.75, treeifies buckets past 8 entries (→ O(log n) worst case).
Python dict, Go map, Rust HashMap (hashbrown/Swiss Tables): open addressing, tuned for cache locality; Python uses a perturbation-based probe sequence, Go/Rust use SIMD-friendly group probing.
Summary — what a strong answer emphasizes

Define collision and load factor, then contrast the two families on time (average vs worst), memory, cache locality, deletion, and behavior as α → 1. The one-liner an interviewer wants: chaining degrades gracefully past full and deletes trivially but chases pointers; open addressing is compact and cache-friendly (hence modern favorites) but demands low load factors and tombstone-based deletion. Naming double hashing / Robin Hood / treeification and a couple of real implementations seals it.

Common mistakes / follow-up probes
Saying open addressing "has no worst case" — it degrades sharply near full and clusters.
Forgetting the deletion tombstone problem in open addressing.
Follow-ups: "Why do modern languages prefer open addressing?" (cache locality, no allocations). "What is Robin Hood hashing?" (bound probe-distance variance). "How does Java bound the worst case?" (treeify to red-black tree). "Defend against HashDoS?" (seeded/randomized hash).
```

**Comparison:**

```
Dimension	Separate chaining	Open addressing
Where entries live	External list/tree per bucket	In the array itself
Avg lookup	O(1 + α)	~O(1) at low α
Worst case	O(n) chain (Java: O(log n) via treeify)	O(n) via long probe run
Load factor α	Can exceed 1; graceful degradation	Must stay < 1 (typ. ≤ 0.7–0.75)
Cache locality	Poor (pointer chasing, separate allocs)	Excellent (contiguous, no pointers)
Deletion	Trivial (unlink node)	Hard — tombstones / backward-shift
Memory overhead	Per-node pointers + allocations	Empty slots (headroom under threshold)
Clustering	None	Linear: primary; quadratic: secondary; double-hash: minimal
Probe/insert simplicity	Simple	Needs probe scheme + resize discipline
Used by	Java HashMap	Python dict, Go/Rust map, C++ absl Swiss Tables
```

**Code:**

```python
# Separate chaining — each bucket is a list of (key, value) pairs.
class ChainingHashMap:
    def __init__(self, capacity=8):
        self.buckets = [[] for _ in range(capacity)]
        self.size = 0
    def _idx(self, key): return hash(key) % len(self.buckets)
    def put(self, key, value):
        b = self.buckets[self._idx(key)]
        for i, (k, _) in enumerate(b):
            if k == key: b[i] = (key, value); return   # update in place
        b.append((key, value)); self.size += 1          # append to chain
    def get(self, key):
        for k, v in self.buckets[self._idx(key)]:
            if k == key: return v
        raise KeyError(key)
    def delete(self, key):                              # trivial: drop the node
        b = self.buckets[self._idx(key)]
        for i, (k, _) in enumerate(b):
            if k == key: b.pop(i); self.size -= 1; return

# Open addressing with LINEAR PROBING + tombstones for deletion.
_EMPTY, _DELETED = object(), object()
class OpenAddrHashMap:
    def __init__(self, capacity=8):
        self.slots = [_EMPTY] * capacity            # each slot: _EMPTY | _DELETED | (k, v)
        self.size = 0
    def put(self, key, value):
        m = len(self.slots); i = hash(key) % m; first_tomb = None
        while True:
            s = self.slots[i]
            if s is _EMPTY:
                self.slots[first_tomb if first_tomb is not None else i] = (key, value)
                self.size += 1; return
            if s is _DELETED:
                if first_tomb is None: first_tomb = i
            elif s[0] == key:
                self.slots[i] = (key, value); return   # update
            i = (i + 1) % m                            # linear probe
    def get(self, key):
        m = len(self.slots); i = hash(key) % m
        while self.slots[i] is not _EMPTY:             # stop only at a truly empty slot
            s = self.slots[i]
            if s is not _DELETED and s[0] == key: return s[1]
            i = (i + 1) % m
        raise KeyError(key)
    def delete(self, key):                             # leave a TOMBSTONE, not _EMPTY
        m = len(self.slots); i = hash(key) % m
        while self.slots[i] is not _EMPTY:
            s = self.slots[i]
            if s is not _DELETED and s[0] == key:
                self.slots[i] = _DELETED; self.size -= 1; return
            i = (i + 1) % m
```

**Example input / output:**

```

Trace: capacity m = 8; keys with these (already-computed) hash%8 values:

CODE
Copy
"a"->1, "b"->1, "c"->1   (all collide at bucket 1)

SEPARATE CHAINING after put a,b,c:

CODE
Copy
bucket[1] = [("a",..), ("b",..), ("c",..)]   # one chain, others empty
get("c") -> hash->1, scan chain 3 nodes -> found.  delete("b") -> unlink node.

OPEN ADDRESSING (linear probing) after put a,b,c:

CODE
Copy
slot[1]=("a"..)                 # h=1 empty -> place
slot[2]=("b"..)                 # h=1 taken -> probe 2, empty -> place
slot[3]=("c"..)                 # h=1,2 taken -> probe 3, empty -> place
get("c") -> h=1 (a) ->2 (b) ->3 (c) found (3 probes).
delete("b") -> slot[2]=DELETED (tombstone), NOT empty,
              so get("c") still probes 1->2(tombstone,skip)->3 and finds it.

This trace shows the deletion difference: chaining unlinks; open addressing must tombstone to preserve the probe chain.
```

**Flow:**

1. No
2. Yes: COLLISION
3. Separate chaining
4. Open addressing
5. key
6. index = hash(key) mod m
7. Slot occupied bya different key?
8. Place / read here
9. Strategy
10. Append/search in this bucket'slist or tree
11. Probe next slot per scheme
12. Linear: h+1, h+2 ...
13. Quadratic: h+1^2, h+2^2 ...
14. Double hash: h1 + i*h2(key)

