---
type: "practice"
difficulty: "Intermediate"
---

# What is the average-case time complexity of a hash table lookup and what makes it O(1)?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Time-space Trade-offs](<../../../../08_Algorithms_and_Data_Structures/Core/Complexity_Analysis/Time-space_Trade-offs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

What the question is really asking: State the average-case lookup cost of a hash table — it's O(1) — and, more importantly, justify it. The interviewer isn't happy with "hash tables are O(1)"; they want the mechanism: how does a key turn into a slot in constant time, and what assumptions make the average-case constant hold?

What the interviewer is probing: Whether you understand (1) the hash → index → bucket pipeline, (2) the load factor and why keeping it bounded is the real lever, (3) the uniform hashing assumption, and (4) that "O(1) average" is a statement about expected chain length, not a promise every lookup touches one slot. A strong answer connects the constant to load factor α and explicitly names the assumptions, because that's exactly where the O(1) can quietly break.

#### Assumptions
/ clarifying questions I'd state: I'll assume separate chaining as the primary model (buckets hold linked lists / small arrays) and mention open addressing as the alternative. I'd clarify: good hash function? bounded load factor with resizing? trusted keys? Those three are the pillars the O(1) rests on.

Plan: Walk the lookup pipeline step by step, define load factor, derive expected work under uniform hashing, then state the assumptions and where they fail.

From first principles: the lookup pipeline

A hash table stores an array of m slots (buckets). To look up a key k:

Compute the hash — h = hash(k). Good hash functions are O(1) in the key size for fixed-length keys (an int, a 64-bit id). For a string of length L it's O(L) to hash, but L is treated as a constant relative to the number of entries n; we're measuring cost as a function of n, and hashing doesn't grow with n.
Map to an index — idx = h mod m (or a bit-mask when m is a power of two). O(1).
Go to that bucket — array indexing table[idx] is O(1) random access.
Search within the bucket — with separate chaining, scan the chain comparing keys until you find k or exhaust the chain.

Steps 1–3 are unconditionally O(1). The entire complexity question lives in step 4: how long is the chain we have to scan?

Load factor — the one number that governs everything

Define the load factor:

CODE
α = n / m        (number of stored entries / number of buckets)

If the hash function scatters keys evenly across all m buckets (the Simple Uniform Hashing Assumption — each key equally likely to land in any bucket, independent of others), then the expected number of entries in any given bucket is exactly α. So an unsuccessful search scans on average α keys, and a successful search scans about 1 + α/2 on average. Either way, the expected work in step 4 is Θ(1 + α).

Total expected lookup cost = O(1) hash + O(1) index + O(1 + α) chain scan = O(1 + α).

Now the punchline: if we keep α bounded by a constant, the whole lookup is O(1). That's the design contract. Implementations pick a max load factor — Java's HashMap uses 0.75, C++ unordered_map defaults to 1.0, Python's dict keeps it under ~2/3 — and when α exceeds it, they resize (allocate a larger table, typically double m, and rehash all entries). Resizing keeps α roughly constant forever, so chains stay short (on average well under one comparison for α=0.75), and lookups stay O(1) on average, amortized over the resizes.

So the honest one-liner is: average-case lookup is O(1) because with a good hash function and a load factor bounded by a constant, the expected chain length is a constant, independent of the number of elements n.

Why O(1) and not O(n) — the crux

The naive worry is "as I add more keys, don't lookups get slower?" They would — if m stayed fixed. With n growing and m fixed, α = n/m grows linearly, and chains grow to O(n), and lookup degrades to O(n). The trick is that m grows with n (via resizing), so α stays constant. Constant expected chain length + constant hashing = O(1), and crucially this is independent of n: whether you have 1,000 or 10,000,000 entries, a lookup touches roughly the same tiny number of slots. That size-independence is the essence of the O(1) claim.

The assumptions the O(1) rests on (name these — it's what separates a good answer)
Good, well-distributed hash function. If the hash clusters (e.g., a bad hashCode returning a constant), all keys pile into one bucket, chains become O(n), and lookup degrades to O(n). Uniformity is not free — it's a property of the hash function.
Bounded load factor via resizing. Without resizing, α → ∞ and O(1) is a lie. The O(1) is really amortized once resizing is counted, because a resize is an O(n) rehash spread across many O(1) operations.
Trusted keys (no adversary). If an attacker knows the hash function, they can craft colliding keys → all land in one bucket → O(n) per lookup → hash-flooding DoS. Real libraries defend with randomized/seeded hashing (SipHash in Python/Rust) so an outsider can't predict collisions.
Separate chaining vs open addressing (worth a sentence)
Separate chaining: buckets hold lists; expected cost Θ(1 + α); tolerates α > 1.
Open addressing (linear/quadratic probing, double hashing): entries live in the array itself; expected probes for a successful search ≈ (1/α)·ln(1/(1−α)), which stays O(1) only while α is comfortably below 1 (performance collapses as α → 1). It's more cache-friendly (no pointer chasing) but demands stricter load-factor control.

Modern high-performance maps (Java 8+, Google's SwissTable/absl::flat_hash_map) add refinements — Java converts a bucket to a balanced tree (red-black) once a chain exceeds 8 entries, bounding the worst case at O(log n) instead of O(n); SwissTable uses open addressing with SIMD probing — but the average case is O(1) in all of them by the same load-factor argument.

Summary — what a strong answer emphasizes
Average-case hash lookup is O(1).
The pipeline hash → index → bucket is O(1); the only variable cost is the chain scan, whose expected length equals the load factor α = n/m.
Under simple uniform hashing with α bounded by a constant (guaranteed by resizing when α crosses ~0.75), expected work is O(1 + α) = O(1) — and, decisively, independent of n.
It's really amortized O(1) once the O(n) resize/rehash is spread across operations.
The three load-bearing assumptions: good hash function, bounded load factor, trusted (non-adversarial) keys — break any one and lookup degrades toward O(n).

#### Common mistakes
claiming O(1) with no mention of load factor or the uniform-hashing assumption; forgetting resizing is what keeps α constant; confusing average (O(1)) with worst (O(n)).

#### Follow-up
probes: "Why 0.75 and not 0.95?" (higher α → longer chains and more probe collisions → slower; trade memory for speed). "What's the worst case and when?" (O(n), all keys collide / bad hash / adversarial keys). "Why is it 'amortized'?" (the periodic O(n) resize).

**Comparison:**

```
Step in lookup	Operation	Cost	Depends on n?
1. Hash the key	hash(k)	O(1) for fixed-size key (O(L) for length-L string)	No
2. Map to index	h mod m or h & (m-1)	O(1)	No
3. Reach bucket	table[idx] array index	O(1)	No
4. Scan chain	compare keys in bucket	O(1 + α) expected	Only via α, kept constant
Total (average)		O(1 + α) = O(1)	No, if α bounded
Scenario	Chain length	Lookup cost
Good hash, α ≤ 0.75, resizing on	~α (well under 1)	O(1) average
No resizing, n grows, m fixed	~n/m → grows	O(n) eventually
All keys collide / bad hashCode	n	O(n) worst
Adversarial keys (hash flooding)	n (forced)	O(n) worst → DoS
```

**Code:**

```java
// Separate-chaining hash map — average O(1) get, showing WHERE the cost is
class HashMap<K, V> {
    static class Node<K,V> { K key; V val; Node<K,V> next; }
    Node<K,V>[] table;
    int size;
    static final double MAX_LOAD = 0.75;

    int indexFor(K key) {
        int h = key.hashCode();
        h ^= (h >>> 16);              // spread bits so lower bits aren't clustered
        return h & (table.length - 1); // table.length is a power of two -> mask = O(1)
    }

    V get(K key) {
        int idx = indexFor(key);          // O(1): hash + mask
        Node<K,V> n = table[idx];          // O(1): array index -> bucket
        while (n != null) {                // scans the CHAIN: expected length = load factor a
            if (n.key.equals(key)) return n.val;
            n = n.next;
        }
        return null;                       // total expected = O(1 + a) = O(1) when a is bounded
    }

    void put(K key, V val) {
        // ... insert ...
        if ((double) size / table.length > MAX_LOAD) resize(); // keeps a bounded -> O(1) amortized
    }
}
```

**Example input / output:**

```

Illustrating why expected chain length = load factor (m = 8 buckets):

CODE
Copy
n = 6 entries, m = 8 buckets  -> load factor a = 6/8 = 0.75
Good hash spreads them:
  bucket[0]: -> "user:12"
  bucket[1]: (empty)
  bucket[2]: -> "user:07"
  bucket[3]: -> "order:9"
  bucket[4]: (empty)
  bucket[5]: -> "cart:3" -> "sku:88"   (a rare 2-length chain)
  bucket[6]: -> "user:41"
  bucket[7]: (empty)
Lookup "order:9": hash->3, table[3], compare 1 key -> found. ~1 comparison. O(1).

BAD hash (all -> bucket[0]) with same 6 keys:
  bucket[0]: -> a -> b -> c -> d -> e -> f     (chain length = n)
Lookup: scan up to 6 keys -> O(n). This is the WORST case, not the average.

Resizing keeps it O(1): add a 7th entry -> a = 7/8 = 0.875 > 0.75
  -> resize to m = 16, rehash all -> a drops to 7/16 ~ 0.44 -> chains stay short again.
```

**Flow:**

1. yes
2. no
3. key k
4. hash(k) O(1)
5. idx = h mod m O(1)
6. table[idx] -> bucket O(1)
7. scan chain: expected length =load factor a
8. k found?
9. return value
10. return null
11. Total expected = O(1 + a) =O(1) when a is bounded byresizing

