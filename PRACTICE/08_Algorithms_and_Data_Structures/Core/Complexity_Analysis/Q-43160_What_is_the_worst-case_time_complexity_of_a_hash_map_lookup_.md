---
type: "practice"
difficulty: "Deep"
---

# What is the worst-case time complexity of a hash map lookup and when does it happen?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Time-space Trade-offs](<../../../../08_Algorithms_and_Data_Structures/Core/Complexity_Analysis/Time-space_Trade-offs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

What the question is really asking: Not the happy-path O(1), but the worst case — O(n) — and precisely when and why a hash map lookup degrades to it. "Deep understanding" means I should go past "when there are collisions" and explain the exact mechanisms: pathological hash distribution, degenerate load factor, and — the answer that impresses — adversarial hash-flooding, plus what modern implementations do to blunt the worst case (Java 8 treeification → O(log n), randomized seeds).

What the interviewer is probing: Do you understand that O(1) is an average under assumptions, and can you enumerate the ways those assumptions break? Do you know this is a real security problem (algorithmic-complexity DoS), not just a textbook footnote? Can you reason about the insertion worst case (O(n²) to fill a table) and the mitigations? This is a favorite at cloud/fintech/e-commerce shops (per the tags) because a single degraded map on a hot path can take down a service.

#### Assumptions
/ clarifying questions: I'll assume separate chaining as the base model and cover open addressing too. I'd clarify: are keys attacker-controlled? is the hash seeded/randomized? is resizing enabled? Those determine which worst-case pathway is reachable.

Plan: Recall why average is O(1), then derive each way the worst case O(n) arises, quantify the aggregate O(n²) blow-up, cover open addressing, then the mitigations that turn O(n) into O(log n) or make it unreachable.

Baseline: why average is O(1), so we can see what breaks

Lookup = hash(k) (O(1)) → idx = h mod m (O(1)) → table[idx] (O(1)) → scan the bucket's chain. Expected chain length equals the load factor α = n/m, and implementations keep α bounded (~0.75) by resizing. So average lookup = O(1 + α) = O(1). The worst case is precisely: every element you might have to compare lands in the one bucket you probe — chain length n — so the scan is O(n). Now, when does that happen?

Worst case is O(n). The three ways to reach it.
1. All keys hash to the same bucket (pathological distribution)

If the hash function maps every key to the same index — e.g., a broken hashCode() that returns a constant, or an integer-keyed table where all keys are multiples of m so k mod m collapses to one slot — then all n entries chain in a single bucket. A lookup must scan the entire chain: O(n). This is the mechanical worst case and it doesn't require malice; a naive custom hashCode/equals or a poorly chosen table size (e.g., non-prime m with structured keys) is enough.

2. Degenerate load factor (resizing disabled or thresholds too high)

If you fix m and keep inserting, α = n/m grows without bound. Even with a perfect hash, expected chain length grows to Θ(n/m) = Θ(n) once m stops growing. So a map that never resizes (or is pre-sized wrong and capped) trends toward O(n) lookups. This is why "O(1) amortized" requires resizing — it's the mechanism that keeps chains short.

3. Adversarial hash flooding (the answer that matters at scale)

This is the deep, dangerous case. If keys come from untrusted input (HTTP headers, JSON body fields, query params, form keys) and the hash function is deterministic and public, an attacker can precompute a set of keys that all collide into the same bucket. Every insert and lookup then costs O(n). The devastating part is the aggregate: inserting n colliding keys costs 1 + 2 + ... + n = Θ(n²), because each insert scans the growing chain (to dedupe/find). A payload of a few thousand crafted keys — kilobytes of data — can consume seconds of CPU per request and pin cores. This is the classic algorithmic-complexity DoS / hash-flooding attack (CVE-2011-4815 and the 28C3 "Efficient Denial of Service" disclosure hit PHP, Java, Python, Ruby, .NET, and more simultaneously). It's the concrete reason "hash maps are O(1)" is unsafe to assume on a public surface.

Open addressing has its own worst case

With open addressing (linear/quadratic probing, double hashing) there are no chains — colliding keys probe subsequent slots. The worst case is still O(n): as α → 1 the table saturates and probe sequences lengthen catastrophically. Primary clustering (linear probing) makes runs of occupied slots coalesce, so even moderate α produces long probe chains; a full table means a search may probe every slot → O(n). Open addressing therefore demands even stricter load-factor discipline than chaining, and its worst case is triggered by high α plus clustering rather than by a single overloaded bucket.

Mitigations — how good implementations bound the worst case

A senior answer doesn't stop at "it's O(n)"; it explains how real systems fight it:

Treeify overloaded buckets (Java 8+ HashMap). When a single bucket's chain exceeds 8 nodes (and table capacity ≥ 64), the chain is converted to a red-black tree. Worst-case lookup within that bucket drops from O(n) to O(log n). This directly defangs both pathological distribution and hash flooding — an attacker forcing collisions now pays O(log n), not O(n), per operation, and O(n log n) instead of O(n²) to fill. (It untreeifies back to a list when the bucket shrinks below 6.)

Randomized / seeded hashing. Python (since 3.4), Rust's default HashMap (SipHash), and others hash with a per-process random seed, so an outsider cannot predict which keys collide — the attacker can't precompute a colliding set. This makes case 3 unreachable without breaking the map's average-case guarantees. (Note: Java historically did not seed string hashing — hence it relied on treeification instead.)

Bounded load factor + resizing (as above) keeps the expected case O(1) and prevents case 2.

Cap or reject oversized untrusted maps (limit number of JSON keys / form fields) as defense in depth, and prefer implementations with the mitigations above on internet-facing paths.

Putting the cases together

| The map's worst case is O(n) for a lookup, reached when the bucket you probe holds Θ(n) entries. That happens with a degenerate hash (all collide), an unbounded load factor (no resizing), or an adversary crafting collisions (hash flooding, aggregate O(n²)). Open addressing degrades identically as the table saturates. Mitigations turn O(n) into O(log n) (Java treeification) or make the adversarial path unreachable (seeded hashing).

Summary — what a strong answer emphasizes
Worst-case lookup = O(n), occurring when all (or Θ(n)) keys land in the single bucket/probe sequence you traverse.
Three triggers: (a) pathological hash → all keys one bucket; (b) unbounded load factor / no resizing; (c) adversarial hash flooding on untrusted keys — deterministic public hash lets an attacker force collisions; aggregate cost to fill is O(n²), a real DoS.
Open addressing hits O(n) via high α + primary clustering (table saturation), not a single fat bucket.
Mitigations: Java 8 treeifies buckets > 8 nodes → O(log n) worst case; randomized/seeded hashing (SipHash) makes the adversarial path unreachable; bounded load factor + resizing prevents the degenerate case.

#### Common mistakes
saying "worst case is O(n) because of collisions" without specifying when Θ(n) collisions occur; not knowing hash flooding is a security vulnerability; being unaware of Java's O(log n) treeification; confusing worst (O(n)) with average (O(1)) or amortized-insert (O(n²) aggregate under attack).

#### Follow-up
probes: "How does Java bound it to O(log n)?" (red-black tree bins). "How would you defend a public API accepting a JSON object as a map?" (seeded hashing + cap key count). "What's the insertion worst case under attack and why?" (O(n²): each insert scans a chain growing to n).

**Comparison:**

```
Trigger	Mechanism	Lookup cost	Aggregate insert cost
Pathological hash (all keys → 1 bucket)	Chain length = n	O(n)	O(n²)
No resizing / unbounded load factor	Expected chain = Θ(n/m) → Θ(n)	O(n)	O(n²)
Adversarial hash flooding (untrusted keys, public hash)	Attacker forces collisions	O(n)	O(n²) — the DoS
Open addressing, α → 1 + clustering	Long probe sequences / table saturation	O(n)	O(n²)
Java 8+ treeified bucket (> 8 nodes)	Bucket = red-black tree	O(log n)	O(n log n)
Seeded/randomized hash (SipHash)	Collisions unpredictable to attacker	O(1) avg restored	O(n)
	Average	Worst (unmitigated)	Worst (mitigated)
Chaining lookup	O(1)	O(n)	O(log n) (treeify)
Open-addressing lookup	O(1)	O(n)	keep α low; seed hash
Insert n keys (under attack)	O(n)	O(n²)	O(n log n) / prevented
```

**Code:**

```

// Reproducing the worst case: force every key into one bucket. // A collider whose hashCode is constant makes chain length = n -> O(n) lookup, // and O(n^2) to insert n of them (each insert scans the growing chain). class Collider { final String id; Collider(String id) { this.id = id; } @Override public int hashCode() { return 0; } // <-- all collide into bucket 0 @Override public boolean equals(Object o) { return o instanceof Collider && ((Collider) o).id.equals(id); } }

Map<Collider, Integer> m = new HashMap<>(); for (int i = 0; i < 100_000; i++) m.put(new Collider("k" + i), i); // ~O(n^2) work m.get(new Collider("k99999")); // scans the whole bucket -> O(n) // In Java 8+, once the bucket exceeds 8 nodes it becomes a red-black tree, // so this get() is actually O(log n), not O(n) -- treeification in action.

CODE
Copy
```python
# Python defends via a per-process RANDOM hash seed (PYTHONHASHSEED),
# so an attacker cannot precompute colliding string keys -> hash flooding blocked.
#   $ PYTHONHASHSEED=random python app.py   # default since 3.4
d = {}                      # dict lookup average O(1); worst O(n) only if collisions forced,
d["untrusted_key"] = 1      # which seeded hashing makes practically impossible from outside.
```

**Example input / output:**

```text
--- Average case (good hash, alpha bounded) ---
n = 1,000,000 entries, resizing on
lookup "user:42": hash -> one bucket -> ~1 comparison  =>  O(1)

--- Worst case: all keys collide (bad hashCode / hash flooding) ---
n = 1,000,000 entries all in bucket[0]
lookup any key: scan up to 1,000,000 nodes             =>  O(n)
inserting all n colliding keys: 1 + 2 + ... + n        =>  O(n^2)  (~5*10^11 ops)

--- Same attack against Java 8+ HashMap (treeified bucket) ---
bucket[0] becomes a red-black tree once > 8 nodes
lookup any key: ~log2(1,000,000) ~ 20 comparisons      =>  O(log n)
inserting all n colliding keys                          =>  O(n log n)

--- Open addressing, load factor near 1 ---
m = 1,048,576 slots, n = 1,040,000 (alpha ~ 0.99)
lookup a missing key: probe a long run of occupied slots (primary clustering) -> up to O(n)
```

**Flow:**

1. No: good hash + boundedalpha
2. Yes
3. Bad hashCode / all keys oneslot
4. No resizing, alphaunbounded
5. Attacker crafted collidingkeys
6. Java 8 treeify bucket
7. Seeded hash SipHash
8. None
9. Hash map lookup
10. Do Theta(n) keys sharethe bucket / probe path I hit?
11. O(1) average
12. Why did they collide?
13. O(n) chain scan
14. Hash flooding DoSlookup O(n), fill O(n^2)
15. Mitigation present?
16. O(log n)
17. Adversarial path unreachable
18. Stays O(n) -> servicedegradation

