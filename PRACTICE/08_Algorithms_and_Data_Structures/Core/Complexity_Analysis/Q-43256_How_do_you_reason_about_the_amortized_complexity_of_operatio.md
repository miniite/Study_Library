---
type: "practice"
difficulty: "Expert"
---

# How do you reason about the amortized complexity of operations on a hash map when it needs to rehash?

**Difficulty:** Expert · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Amortized Complexity](<../../../../08_Algorithms_and_Data_Structures/Core/Complexity_Analysis/Amortized_Complexity.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

This is an expertise-level question: explain how to reason about the amortized complexity of hash-map operations given that the map occasionally rehashes. The interviewer is probing whether you can (a) distinguish worst-case from amortized cost, (b) apply amortized analysis (aggregate / accounting / potential method) to the specific mechanism of table growth, (c) explain why geometric (doubling) resizing yields O(1) amortized insert even though a single insert can be O(n), and (d) show senior judgment about the caveats: hashing quality, load factor, latency spikes, tombstones, and when amortized O(1) is a lie for your use case.

#### Assumptions
I'll state: an open-addressing or separate-chaining hash table that resizes when the load factor α = size/capacity crosses a threshold; a good hash function distributing keys ~uniformly; growth by a constant factor (typically ×2). "Rehash" = allocate a bigger backing array and reinsert all live entries into it.

Plan: define amortized vs worst-case → analyze a rehash → prove O(1) amortized via aggregate and accounting/potential arguments → discuss what breaks the guarantee and the operational consequences.

Amortized vs. worst-case: the mental model

Amortized cost is the average cost per operation over a worst-case sequence of operations — not a probabilistic average, but a guarantee that any sequence of m operations costs at most m × (amortized bound). The point: an individual insert that triggers a rehash is expensive (O(n) to move n entries), but such expensive inserts are rare and their cost is "paid for" by the many cheap inserts before them. The correct claim is: a single insert is O(n) worst case, but the amortized cost per insert is O(1).

Why a rehash is O(n) and why doubling makes it cheap on average

When the table crosses its load-factor threshold, we allocate a new array (usually twice the capacity) and reinsert all current n entries — each reinsertion is O(1) expected (recompute hash, place), so the rehash is O(n). The question is how often this O(n) bolt of cost strikes.

Aggregate method (the cleanest argument). Start empty, grow by doubling, and insert n items. Rehashes happen when size passes 1, 2, 4, 8, …, up to n — that's roughly log₂ n rehashes. The i-th rehash moves about 2^i elements. Total rehash work is a geometric series: 1 + 2 + 4 + … + n ≈ 2n = O(n). Add the O(n) for the n cheap placements themselves → total O(n) for all n inserts → O(1) amortized per insert. The geometric series is the whole trick: because the table doubles, the total copying across the entire history is bounded by ~2× the final size, not by (number of rehashes) × (table size).

Contrast with additive growth (the classic trap). If instead you grow by a fixed increment c (e.g., +10 slots each time), rehashes happen after every c inserts, and the k-th rehash copies ~kc elements. Total copy work is c + 2c + 3c + … ≈ O(n²/c) = O(n²), so amortized cost is O(n) per insert — a disaster. This is why real implementations always grow geometrically. Reasoning about amortized cost therefore starts with: what is the growth policy? Geometric → O(1) amortized; arithmetic → O(n) amortized.

Accounting (banker's) method — the intuition interviewers love. Charge each insert a constant "amortized fee," say 3 units: 1 unit does the actual insertion now, and 2 units are saved as credit on that element. When the table doubles, every element that has to be copied spends the credit it banked to pay for its own move (and to pre-pay the copy of an element from the previous era). Since every element arrives with enough saved credit to fund its future relocation, no rehash ever runs a deficit → constant amortized cost.

Potential method (formal version). Define a potential function Φ = 2·size − capacity (≥0 right after a resize). A cheap insert raises Φ by 2 (amortized = 1 real + 2 = 3). A resize where size = capacity does size real work but drops Φ by ~size, canceling it → amortized O(1). The potential is the "stored energy" that pays for the next expensive rehash.

Senior judgment — where the O(1) amortized guarantee bends or breaks

It's amortized, not worst-case — tail latency spikes. A single insert can stall for O(n) while it copies a huge table. For a latency-sensitive service (p99/p999 matters), this is unacceptable: a rehash of a 10M-entry map can pause you for tens of ms. Mitigations: pre-size the map (HashMap(capacity)) to avoid growth entirely if you know the final size; use incremental/concurrent rehashing (Redis rehashes gradually across operations; some maps migrate a few buckets per op) to smear the cost; or use structures with better worst-case bounds where required.

Hash quality dominates the "O(1) per op" assumption. Amortized O(1) assumes good distribution. Adversarial or poor hashes → many collisions → a bucket degrades to O(n) (chaining) regardless of rehashing; this enables hash-flooding DoS. Real maps defend with randomized seeds (SipHash) and by treeifying long chains to O(log n) (Java 8 turns a bucket into a red-black tree past ~8 entries).

Load factor is a space/time knob. A lower threshold (e.g., 0.5) means fewer collisions but more frequent rehashes and more wasted memory; a higher one (0.9) saves memory but lengthens probe sequences. "Reasoning about amortized cost" includes knowing this dial changes both the constant on lookups and the rehash frequency.

Deletions, tombstones, and shrinking. Open addressing uses tombstones on delete; they accumulate and degrade probing, sometimes forcing a rehash even without growth. Also, naive shrink-on-delete can cause thrashing (grow/shrink oscillation) if the shrink threshold isn't set well below the grow threshold (hysteresis).

The bound is per-operation-type. Insert and the average-case lookup/delete are O(1) amortized/expected; but you must be precise: worst-case lookup is O(n) (all keys collide) or O(log n) with treeified buckets, independent of rehashing.

Summary — what a strong answer emphasizes

Separate worst-case (a rehash is O(n)) from amortized (O(1) per insert). Prove it with the geometric series: doubling means total copy work over n inserts is ~2n, so amortized is constant; show you know the accounting/potential argument (banked credit / Φ = 2·size − capacity). Crucially, contrast geometric vs additive growth — additive gives O(n²) total, O(n) amortized — to demonstrate why the policy matters. Then show operational maturity: amortized ≠ worst-case, so watch tail-latency spikes (pre-size, incremental rehash), hash quality/DoS (SipHash, treeify), load-factor trade-offs, and tombstone/shrink pitfalls. Common mistakes: claiming worst-case O(1) (it's amortized), forgetting that additive growth kills the bound, and ignoring that hash quality — not rehashing — is what usually makes a real map slow.

**Comparison:**

```
Aspect	Single insert (worst)	Amortized insert	Lookup (avg)	Lookup (worst)
Geometric-growth hash map	O(n) (on rehash)	O(1)	O(1)	O(n), or O(log n) if buckets treeified
Growth policy	Total copy work over n inserts	Amortized / insert	Verdict
Geometric (×2)	~2n (geometric series)	O(1)	✅ standard
Additive (+c)	~n²/(2c)	O(n)	❌ never do this
Amortized-analysis method	Core idea
Aggregate	Sum all costs over the sequence (1+2+4+…+n≈2n), divide by n → O(1)
Accounting (banker)	Overcharge each insert by a constant; banked credit pays for the future copy
Potential	Φ = 2·size − capacity stores energy; resize spends Φ it accumulated
Real-world caveat	Consequence	Mitigation
Amortized ≠ worst-case	O(n) latency spike on rehash	pre-size; incremental/concurrent rehash
Poor/adversarial hash	O(n) bucket, hash-flood DoS	SipHash seed; treeify long buckets
Load factor too high/low	slow probes / frequent resizes	tune threshold (e.g., 0.75)
Tombstones on delete	probe degradation	rehash to purge; hysteresis on shrink
```

**Code:**

```python
# Minimal open-addressing map with geometric (x2) resize to show WHY insert is
# O(1) amortized: expensive rehash cost is spread across the cheap inserts.
class HashMap:
    def __init__(self, cap=8):
        self.cap, self.size = cap, 0
        self.keys = [None] * cap
        self.vals = [None] * cap

    def _idx(self, key):
        i = hash(key) % self.cap
        while self.keys[i] is not None and self.keys[i] != key:
            i = (i + 1) % self.cap        # linear probing
        return i

    def put(self, key, val):
        if (self.size + 1) / self.cap > 0.75:   # load-factor threshold
            self._resize(self.cap * 2)          # GEOMETRIC growth -> O(1) amortized
        i = self._idx(key)
        if self.keys[i] is None:
            self.size += 1
        self.keys[i], self.vals[i] = key, val

    def _resize(self, new_cap):                  # O(n): reinsert all live entries
        old_k, old_v = self.keys, self.vals
        self.cap, self.size = new_cap, 0
        self.keys = [None] * new_cap
        self.vals = [None] * new_cap
        for k, v in zip(old_k, old_v):
            if k is not None:
                self.put(k, v)                   # rehash into bigger table
```

**Example input / output:**

```text
Aggregate cost of inserting n = 16 keys into a table that doubles from cap 1:

 insert #   size after   rehash?   copies this op   cumulative copies
    1           1          yes(1->2)      1                 1
    2           2          yes(2->4)      2                 3
    3           3          no             0                 3
    4           4          yes(4->8)      4                 7
    5..7        5..7       no             0                 7
    8           8          yes(8->16)     8                15
    9..16       9..16      no             0                15
 -------------------------------------------------------------------
 total copies across ALL inserts = 1+2+4+8 = 15 ≈ 2n     (geometric series)
 total work (copies + placements) = O(n)  ->  O(1) AMORTIZED per insert  ✓

Additive growth (+2 each time), same 16 inserts:
 rehashes copy 2+4+6+...+16 = 72 ≈ n²/4  -> O(n) amortized per insert  ✗ (bad)
```

**Flow:**

1. no
2. yes
3. Why O(1) amortized
4. Doubling -> rehashes at1,2,4,...,n
5. copies sum = 1+2+4+...+n ≈2n
6. total O(n) over n inserts ->O(1) each
7. put(key,val)
8. (size+1)/cap > loadFactor?
9. probe + place: O(1) expected
10. resize: cap *= 2, reinsert nentries: O(n)
11. done

