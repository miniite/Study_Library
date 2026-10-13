---
type: "practice"
difficulty: "Basic"
---

# Explain how a hash table works and what causes a collision.

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ O(1) Randomized Set](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/O(1)_Randomized_Set.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

What is really being asked: Explain the internal mechanics of a hash table (also called hash map / dictionary) and specifically define what a collision is and why it happens. The interviewer is checking whether you understand the machinery behind the O(1)-average lookup you use every day — the hash function, the bucket array, and the reality that collisions are unavoidable — not just that you can call dict[key].

What a strong answer emphasizes: A hash table maps a key to an index in an array using a hash function, giving average O(1) insert/lookup/delete. A collision is when two distinct keys hash to the same bucket index — which is inevitable (pigeonhole principle) and must be handled by a resolution strategy (chaining or open addressing). Bonus points for load factor, resizing, and the O(n) worst case.

Clarifying assumptions: I'll assume a general-purpose in-memory hash table (like Python dict, Java HashMap, C++ unordered_map). I'll explain both major collision-resolution families since the interviewer may probe either.

From first principles: the problem a hash table solves

Suppose you want to store key→value pairs (say, username → user record) and be able to look up any user instantly, no matter how many you store. An array gives you O(1) access — but only if you know the integer index. The whole trick of a hash table is: turn an arbitrary key into an array index.

The three moving parts

1. The backing array (buckets). Internally, a hash table is just an array of some size m (the number of "buckets" or "slots"). This is where data physically lives.

2. The hash function. A function hash(key) that takes any key (string, number, object) and produces an integer. We then compress that integer into a valid array index, typically index = hash(key) % m. A good hash function is:

Deterministic — the same key always yields the same index (or you could never find it again).
Uniform — it spreads keys evenly across all m buckets, minimizing pile-ups.
Fast — computed in O(1) for the key size.

3. The stored entries. At the computed index, we store the key and the value (we store the key too, so we can verify we found the right one during lookup — different keys can land in the same bucket).

How the core operations work

Insert put(key, value): Compute index = hash(key) % m, go to that bucket, store the (key, value) pair.

Lookup get(key): Compute the same index, go to that bucket, and find the entry whose key equals key (compare with ==/equals), return its value. Because the hash and the array access are both O(1), lookup is O(1) on average.

Delete remove(key): Same — find the bucket, remove the matching entry.

The magic is that we never scan the whole structure; we compute exactly where to look. That's why hash tables power caches, database indexes, symbol tables in compilers, deduplication, and counting problems.

What is a collision, and why it happens

A collision is when two different keys produce the same bucket index — i.e., hash(key1) % m == hash(key2) % m even though key1 != key2.

Collisions are not a bug; they are inevitable. Two reasons:

The pigeonhole principle. The universe of possible keys is essentially infinite (all possible strings), but the array has only m finite buckets. If you map infinitely many keys into finitely many slots, some must share a slot. There is no hash function that avoids this in general.

The modulo compression. Even with a great hash function producing 64-bit integers, we squash them into m slots with % m. Many distinct hash values map to the same remainder.

A weak hash function makes collisions worse by clustering keys (e.g., a hash that only looks at the first character of a string collides all words starting with the same letter). But even a perfect hash function collides once you have more keys than buckets.

Handling collisions — two main strategies

Separate chaining (most common, e.g. Java HashMap): Each bucket holds a list (linked list, or a balanced tree when the list gets long) of all entries that hashed there. On insert, append to that bucket's list; on lookup, scan the small list comparing keys. If collisions are rare, lists stay length ~1 and lookup stays O(1).

Open addressing (e.g. Python dict, C++ certain impls): Everything lives in the single array. If the target bucket is occupied, probe for the next free slot by a rule — linear probing (index+1, index+2, …), quadratic probing, or double hashing. Lookup follows the same probe sequence until it finds the key or an empty slot.

Load factor and resizing — why O(1) holds up

The load factor α = n / m (entries ÷ buckets) governs collision frequency. As α rises, buckets get crowded and lookups slow. So hash tables resize: when α exceeds a threshold (commonly 0.75 for chaining, ~0.66 for open addressing), the table allocates a bigger array (usually doubling m) and rehashes every existing entry into the new array. Resizing is O(n) for that one operation, but because it happens rarely and doubles capacity each time, inserts remain O(1) amortized.

The worst case — and its security angle

If every key collides into one bucket (terrible hash function or an attacker who crafts colliding keys), all n entries land in one list and lookup degrades to O(n) — a linear scan. This is a real denial-of-service vector called hash flooding: an attacker sends inputs engineered to collide, turning your O(1) structure into O(n) and stalling the server. Mitigations include randomized/seeded hash functions (SipHash in Python, Rust, etc.) and treeifying long buckets (Java converts a bucket to a balanced tree, capping it at O(log n)).

Summary / what a strong answer emphasizes

A hash table stores key→value pairs in an array, using a hash function to compute each key's bucket index, giving average O(1) insert/lookup/delete. A collision is two distinct keys mapping to the same bucket — unavoidable by the pigeonhole principle and modulo compression — and is resolved by separate chaining (per-bucket lists) or open addressing (probing for the next slot). Load factor drives resizing, which keeps operations O(1) amortized; a pathological all-collide scenario degrades to O(n), which modern implementations harden against with randomized hashing and tree-ified buckets.

#### Common mistakes
claiming collisions are a defect to be eliminated; saying lookups are always O(1) (worst case is O(n)); forgetting that the key itself is stored for verification; ignoring resizing.

#### Follow-up
probes: "Chaining vs open addressing trade-offs?", "What makes a good hash function?", "What's the worst-case complexity and how do you defend it?", "How does Java's HashMap treeify buckets?", "Why do we store the key, not just the value?"

**Comparison:**

```
Aspect	Separate chaining	Open addressing
Where entries live	Per-bucket lists outside the array	Inside the single array
Collision handling	Append to bucket's list	Probe next slot (linear/quadratic/double)
Memory	Extra pointers per node	Compact, cache-friendly
Load factor tolerance	Can exceed 1.0	Must stay < 1.0 (needs empty slots)
Deletion	Simple (remove from list)	Tricky (needs tombstones)
Used by	Java HashMap	Python dict, many C++ impls
Operation	Average	Worst case
Insert	O(1) amortized	O(n)
Lookup	O(1)	O(n)
Delete	O(1)	O(n)
Resize (one-time)	O(n)	O(n)
```

**Code:**

```python
# A minimal hash table using separate chaining — shows hashing, buckets, and collisions.
class HashTable:
    def __init__(self, capacity=8):
        self.m = capacity
        self.size = 0
        self.buckets = [[] for _ in range(self.m)]   # each bucket is a list (chaining)

    def _index(self, key):
        return hash(key) % self.m                    # hash -> compress to a bucket index

    def put(self, key, value):
        i = self._index(key)
        bucket = self.buckets[i]
        for pair in bucket:                          # collision handling: scan this bucket
            if pair[0] == key:                       # key already present -> update
                pair[1] = value
                return
        bucket.append([key, value])                  # different key, same bucket = collision
        self.size += 1
        if self.size / self.m > 0.75:                # load factor threshold -> resize
            self._resize()

    def get(self, key):
        i = self._index(key)
        for k, v in self.buckets[i]:                 # compare keys to find the right entry
            if k == key:
                return v
        raise KeyError(key)

    def _resize(self):
        old = self.buckets
        self.m *= 2                                  # double capacity
        self.buckets = [[] for _ in range(self.m)]
        self.size = 0
        for bucket in old:                           # rehash every entry -> O(n)
            for k, v in bucket:
                self.put(k, v)
```

**Example input / output:**

```

Dry-run with a deliberately tiny table (m=4) to force a collision:

CODE
Copy
ht = HashTable(capacity=4)     # buckets: [ [], [], [], [] ]
ht.put("apple", 1)   # hash("apple") % 4 -> say index 2  -> bucket[2] = [["apple",1]]
ht.put("grape", 2)   # hash("grape") % 4 -> also index 2  -> COLLISION
                     #   bucket[2] = [["apple",1], ["grape",2]]   (chained)
ht.get("grape")      # index 2 -> scan bucket: "apple"!=grape, "grape"==grape -> returns 2
ht.get("apple")      # index 2 -> scan bucket: "apple"==apple -> returns 1

Lookup cost = length of the bucket's list. Average bucket length ~ load factor (~1) -> O(1). Worst case (all keys in one bucket) -> scan all n entries -> O(n).
```

**Flow:**

1. empty
2. occupied by same key
3. occupied by different key
4. chaining
5. open addressing
6. yes
7. no
8. Key
9. hash(key)
10. index = hash % m
11. Bucket at index
12. Store key,value here
13. Update value
14. COLLISION
15. Append to bucket's list
16. Probe next slot until empty
17. load factor > threshold?
18. Resize: double m and rehashall - O n
19. Done

